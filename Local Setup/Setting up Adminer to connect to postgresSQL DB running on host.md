# Connect Adminer (Docker) to PostgreSQL running on the host

Goal: Run Adminer in Docker Compose and connect it to a PostgreSQL server that runs directly on the Docker host (not in a container). Containers can’t use `localhost` to reach services on the host, so we’ll point Adminer at the host using `host.docker.internal`.

## Why not localhost?
Inside a container, `localhost` means “this container.” You need to target the host machine’s IP or a special alias that resolves to it.

## Recommended approach: use host.docker.internal
- Works out-of-the-box on Docker Desktop (macOS/Windows).
- On Linux, add an `extra_hosts` entry that maps `host.docker.internal` to Docker’s host-gateway.

### 1) Ensure PostgreSQL accepts connections from Docker
On the host, make sure PostgreSQL:
- Listens on a non-loopback interface
- Allows connections from the Docker bridge subnet (e.g., `172.17.0.0/16`)

Edit `postgresql.conf`:
```ini
# postgresql.conf
listen_addresses = '*'     # or a specific IP; '*' is simplest for localhost dev
```

Edit `pg_hba.conf` to allow the Docker subnet:
```conf
# pg_hba.conf
# Allow Docker containers (adjust subnet if needed)
host    all     all     172.17.0.0/16     md5
# If your cluster uses SCRAM:
# host  all     all     172.17.0.0/16     scram-sha-256
```

Find your Docker bridge subnet if unsure:
```bash
docker network inspect bridge --format '{{(index .IPAM.Config 0).Subnet}}'
```

Restart PostgreSQL on the host (examples):
```bash
# Debian/Ubuntu
sudo systemctl restart postgresql

# Fedora/RHEL/CentOS
sudo systemctl restart postgresql

# Homebrew on macOS
brew services restart postgresql
```

### 2) Docker Compose for Adminer
This Compose file runs Adminer and points it at `host.docker.internal`. On Linux, it also maps that alias to the host-gateway.

```yaml
# docker-compose.yml
version: "3.9"
services:
  adminer:
    image: adminer:latest
    environment:
      # Pre-fill the "Server" field
      ADMINER_DEFAULT_SERVER: host.docker.internal
    ports:
      - "127.0.0.1:8080:8080"    # expose only on localhost
    extra_hosts:
      # Required on Linux; harmless elsewhere.
      - "host.docker.internal:host-gateway"
```

Bring it up:
```bash
docker compose up -d
```

Open Adminer:
- URL: http://localhost:8080
- In the login form:
  - System: PostgreSQL
  - Server: host.docker.internal
  - Username: your_db_user
  - Password: your_db_password
  - Database: your_db_name
  - Port: 5432 (or whatever your host Postgres uses)

## Alternative: host networking (Linux only)
If you prefer, put Adminer on the host network so `127.0.0.1` in Adminer refers to the host. Note: this removes network isolation and binds Adminer directly to your host ports (ensure 8080 is free).

```yaml
version: "3.9"
services:
  adminer:
    image: adminer:latest
    network_mode: "host"
    environment:
      ADMINER_DEFAULT_SERVER: 127.0.0.1
    # No ports: section with host networking
```

Open: http://127.0.0.1:8080 and log in with:
- Server: 127.0.0.1
- Port: 5432
- (Same credentials)

Not supported on Docker Desktop for macOS/Windows in the same way; use the first approach instead.

## Troubleshooting
- Connection refused
  - Postgres may be listening only on `127.0.0.1`. Set `listen_addresses='*'` and restart.
  - Firewall/ufw may block port 5432. Allow it on localhost/bridge.
  - Verify connectivity from another container:
    ```bash
    docker run --rm -it postgres:16-alpine \
      psql -h host.docker.internal -U your_user -d your_db -p 5432
    ```
- Authentication failed
  - Ensure the username/password match and `pg_hba.conf` authentication method (md5 vs scram-sha-256) aligns with your cluster.
- Linux only: `host-gateway` not recognized
  - Your Docker Engine may be too old. Either upgrade, use `network_mode: "host"`, or hardcode the gateway IP:
    ```bash
    # Find host IP from inside a one-off container:
    docker run --rm alpine sh -c "ip route | awk '/default/ {print \$3}'"
    ```
    Then map it:
    ```yaml
    extra_hosts:
      - "host.docker.internal:172.17.0.1"   # example IP; replace with your gateway
    ```

## Security tips
- Keep Adminer bound to localhost: `127.0.0.1:8080:8080`
- Use a non-superuser DB account for day-to-day queries
- Limit `pg_hba.conf` to your Docker subnet, not `0.0.0.0/0`
- Tear down when done: `docker compose down`
