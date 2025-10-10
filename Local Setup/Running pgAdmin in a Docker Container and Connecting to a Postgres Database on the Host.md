# Tutorial: Running pgAdmin in Docker (Standalone and with Docker Compose) to Connect to a Host Postgres

This tutorial walks you through two approaches to run **pgAdmin** in Docker, and connect it to a **Postgres database running directly on your host machine**:

1. **Standalone run using docker run**
2. **Managed run using docker-compose**

---

## Option 1: Run pgAdmin with `docker run`

### 1. Pull the Image

    docker pull dpage/pgadmin4

### 2. Run the Container

    docker run -d \
      --name pgadmin \
      -e PGADMIN_DEFAULT_EMAIL=admin@example.com \
      -e PGADMIN_DEFAULT_PASSWORD=secret \
      -p 8080:80 \
      dpage/pgadmin4

- `PGADMIN_DEFAULT_EMAIL`: login email for web UI  
- `PGADMIN_DEFAULT_PASSWORD`: pgAdmin admin password  
- `-p 8080:80`: exposes pgAdmin on http://localhost:8080  

### 3. Access pgAdmin

Go to:

    http://localhost:8080

Log in with the email and password you specified.

---

## Option 2: Run pgAdmin with Docker Compose

Using Docker Compose makes it easier to manage configuration and persist data.

### 1. Create `docker-compose.yml`

Create a folder (e.g., `pgadmin-docker/`) and inside it, add the file `docker-compose.yml`:

    version: "3.9"

    services:
      pgadmin:
        image: dpage/pgadmin4
        container_name: pgadmin
        restart: unless-stopped
        ports:
          - "8080:80"                # pgAdmin UI: http://localhost:8080
        environment:
          PGADMIN_DEFAULT_EMAIL: admin@example.com
          PGADMIN_DEFAULT_PASSWORD: secret
        volumes:
          - pgadmin_data:/var/lib/pgadmin
        networks:
          - pgadmin-net

    volumes:
      pgadmin_data:

    networks:
      pgadmin-net:
        driver: bridge

### 2. Start pgAdmin

From the same folder as your `docker-compose.yml`:

    docker-compose up -d

Access pgAdmin at:

    http://localhost:8080

---

## Configure Connection to Host Postgres

Whether you use `docker run` or `docker-compose`, the method to link pgAdmin to your host Postgres is the same.

### 1. Allow Postgres to Accept Connections

Edit the Postgres config files on the host:

- In `postgresql.conf`:

        listen_addresses = '*'

- In `pg_hba.conf` add (assuming Docker bridge):

        host all all 172.17.0.0/16 md5

Restart Postgres afterward.

### 2. Host Address to Use inside Container

- **Windows/macOS**:  
  Use `host.docker.internal` as the host address in pgAdmin.

- **Linux**:  
  - Try `host.docker.internal` (works in recent Docker versions).  
  - Otherwise, use the bridge gateway (often `172.17.0.1`). Find with:  

        ip addr show docker0

  - Alternatively, run pgAdmin in **host network mode**, so `localhost:5432` is directly reachable:

        docker run -d \
          --name pgadmin \
          --network host \
          -e PGADMIN_DEFAULT_EMAIL=admin@example.com \
          -e PGADMIN_DEFAULT_PASSWORD=secret \
          dpage/pgadmin4

    Or if using Compose, add:

        network_mode: host

    under the pgAdmin service.

### 3. Register the Host Postgres Server in pgAdmin

1. In pgAdmin: Right‑click **Servers** → **Register → Server…**  
2. General Tab → Name: e.g., `Host Postgres`  
3. Connection Tab:  
   - **Host name/address**: `host.docker.internal` (or bridge IP / localhost when host network is used)  
   - **Port**: 5432  
   - **Username**: Postgres username  
   - **Password**: Postgres password  

Click **Save**, and pgAdmin should connect.

---

## Summary

You now have two easy ways to run pgAdmin against a Postgres database on your host:

- **Standalone `docker run`** — fast one-liner for quick setups.  
- **Docker Compose** — structured config, persistent storage, easy lifecycle management with `docker-compose up -d` / `docker-compose down`.

Both approaches let pgAdmin connect to host Postgres by pointing it at `host.docker.internal` (or the bridge IP on Linux), provided Postgres is configured to listen beyond localhost.