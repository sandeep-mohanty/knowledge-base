# Tutorial: Grafana Persistence with Docker Compose and Provisioning

This guide adds a **Persistent Volume** to your setup. Without this, any dashboards, folders, or alerts you create manually in the Grafana UI would be lost every time the container is stopped or removed. By adding a named volume, your work is stored on your Ubuntu host's disk.

---

## 1. Updated Architecture Overview
Notice the addition of the **Persistent Volume (Grafana Storage)** which keeps your `grafana.db` safe.

```text
      +-----------------------------------------------------------------------+
      |                      LOCAL MACHINE (UBUNTU)                           |
      |                                                                       |
      |   +-----------------------+           +---------------------------+   |
      |   |   Grafana Container   |           |    Docker Desktop VM      |   |
      |   | (Running on Docker)   |           |    Network Gateway        |   |
      |   +-----------|-----------+           +-------------|-------------+   |
      |      |        |                                     |                 |
      |      |   (HTTP Request to)                          |                 |
      |      |   host.docker.internal:9090  ----------------+                 |
      |      |        |                                     |                 |
      |      |        |                       +-------------v-------------+   |
      |      |        |                       |   kubectl port-forward    |   |
      |      |        +---------------------->|   (Local 9090:9090)       |   |
      |      |                                +-------------|-------------+   |
      |      |                                              |                 |
      |   +--v--------------------+                (Secure Tunnel/SSH)        |
      |   |  Persistent Volume    |                         |                 |
      |   |  (grafana-storage)    |           +-------------v-------------+   |
      |   +-----------------------+           |    REMOTE AKS CLUSTER     |   |
      |                                       |  (Prometheus Svc: 9090)   |   |
      +---------------------------------------+---------------------------+   |
```



---

## 2. Directory Structure
Ensure your project looks like this:

```text
grafana-practice/
├── docker-compose.yml
└── provisioning/
    └── datasources/
        └── prometheus.yaml
```

---

## 3. Step 1: Data Source Provisioning
Keep your `provisioning/datasources/prometheus.yaml` as is. This ensures your Prometheus connection is always there on a fresh start.

```yaml
apiVersion: 1

datasources:
  - name: Prometheus-AKS
    type: prometheus
    access: proxy
    url: http://host.docker.internal:9090
    isDefault: true
    editable: true
```

---

## 4. Step 2: The Persistent Docker Compose File
Update your `docker-compose.yml` with the `volumes` definitions. This maps the internal Grafana data directory to a managed volume on your machine.

```yaml
services:
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    restart: unless-stopped
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      # Link the provisioning configs
      - ./provisioning:/etc/grafana/provisioning
      # Persist all dashboards and settings
      - grafana-storage:/var/lib/grafana
    extra_hosts:
      - "host.docker.internal:host-gateway"

volumes:
  grafana-storage:
    driver: local
```

---

## 5. Step 3: Execution Steps

### A. Start the AKS Tunnel
Ensure your tunnel is running in its own terminal.
```bash
kubectl port-forward svc/<service-name> 9090:9090 -n <namespace> --address 0.0.0.0
```

### B. Launch the Container
Run the stack from your `grafana-practice` folder:
```bash
docker compose up -d
```

---

## 6. How to Verify Persistence
1.  Open **`http://localhost:3000`** and log in.
2.  Create a new dashboard and click **Save**. Give it a name like "AKS Identity Health".
3.  Stop and remove your container:
    ```bash
    docker compose down
    ```
4.  Start it back up:
    ```bash
    docker compose up -d
    ```
5.  Refresh your browser. Your "AKS Identity Health" dashboard will still be there!

---

## Pro-Tips
* **Backup:** Since you are using a named volume, Docker Desktop manages the files. If you want to see where they are on Ubuntu, they are typically located in `/var/lib/docker/volumes/` (though Docker Desktop uses a virtual disk).
* **Resetting:** If you ever want to wipe everything and start a clean practice session, run `docker compose down -v`. The `-v` flag deletes the volumes along with the containers.

---
