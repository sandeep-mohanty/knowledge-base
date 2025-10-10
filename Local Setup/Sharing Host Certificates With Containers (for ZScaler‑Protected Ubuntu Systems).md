# 🐳 Complete Tutorial — Sharing Host Certificates With Containers (for ZScaler‑Protected Ubuntu Systems)

This tutorial explains, in practical detail, how to make sure your **Docker containers** trust the same SSL/TLS certificates as your **Ubuntu host** after ZScaler is installed.  
It covers *why*, *how*, and *exactly what to type*—both for **Docker Compose** setups and **manual `docker run`** commands.

---

## 🧭 1. Why Share the Host’s CA Bundle?

After ZScaler is installed, your Ubuntu system trusts traffic intercepted and re‑signed by **ZScalerRootCA.crt**, which was merged into the unified Linux CA bundle:

```
/etc/ssl/certs/ca-certificates.crt
```

Containers, by default, use their own certificate bundles inside their image layers, so they don’t automatically recognize your corporate certificate.  
Mounting the host CA file into each container ensures all HTTPS tools inside the container—`curl`, `pip`, `npm`, `git`, etc.—also trust it.

---

## 🧱 2. Docker Compose Configuration

Let’s walk through a real example with **pgAdmin** needing to reach a **PostgreSQL** server on the host, and both of them making outbound HTTPS calls through ZScaler.

### 📄 `docker-compose.yml`

```yaml
version: "3.9"

services:
  pgadmin:
    image: dpage/pgadmin4
    container_name: pgadmin
    ports:
      - "5050:80"
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@example.com
      PGADMIN_DEFAULT_PASSWORD: secret123
      # Optional proxy variables if ZScaler requires explicit proxy access
      # http_proxy: http://gateway.zscaler.net:9480
      # https_proxy: http://gateway.zscaler.net:9480
      # no_proxy: localhost,127.0.0.1,.internal.company.com
    volumes:
      # 🧩 The key line: mount system CA bundle as read‑only
      - /etc/ssl/certs/ca-certificates.crt:/etc/ssl/certs/ca-certificates.crt:ro
    extra_hosts:
      # Allow containers to reach host services via this DNS name
      - "host.docker.internal:host-gateway"

  app:
    build: .
    environment:
      REQUESTS_CA_BUNDLE: /etc/ssl/certs/ca-certificates.crt
    volumes:
      - /etc/ssl/certs/ca-certificates.crt:/etc/ssl/certs/ca-certificates.crt:ro
```

### ▶️ Running the Compose stack

```bash
docker-compose up -d
```

### ✅ Verification inside the container

```bash
docker exec -it pgadmin ls -l /etc/ssl/certs/ca-certificates.crt
```

You should see something like:

```
-r--r--r-- 1 root root 200K Jul 10 15:30 /etc/ssl/certs/ca-certificates.crt
```

`r--r--r--` confirms it’s **read‑only** (`:ro` mount option keeps the file immutable).

---

## 🧩 3. If Not Using Docker Compose (Direct `docker run`)

You can do the same bind‑mount via the `-v` flag.

### 🖥️ Example 1 — Running pgAdmin Manually

```bash
docker run -d \
  --name pgadmin \
  -p 5050:80 \
  -e PGADMIN_DEFAULT_EMAIL=admin@example.com \
  -e PGADMIN_DEFAULT_PASSWORD=secret123 \
  -v /etc/ssl/certs/ca-certificates.crt:/etc/ssl/certs/ca-certificates.crt:ro \
  dpage/pgadmin4
```

### 🧰 Example 2 — Interactive Container for Testing

```bash
docker run --rm -it \
  -v /etc/ssl/certs/ca-certificates.crt:/etc/ssl/certs/ca-certificates.crt:ro \
  ubuntu bash
```

Inside that shell, check:

```bash
ls -l /etc/ssl/certs/ca-certificates.crt
```

Then verify HTTPS trust:

```bash
curl https://www.google.com -I
```

If it shows `HTTP/1.1 200 OK` rather than any SSL error, the system CA mount works.

---

## ⚙️ 4. Mounting the Entire CA Directory (Optional)

Sometimes you’d rather expose the entire `/etc/ssl/certs` directory instead of a single file.

### With Docker Compose
```yaml
volumes:
  - /etc/ssl/certs:/etc/ssl/certs:ro
```

### With docker run
```bash
docker run -v /etc/ssl/certs:/etc/ssl/certs:ro myimage
```

Advantages:
- Includes intermediate certificates.
- Simplifies CA updates—when `update-ca-certificates` refreshes the directory, containers instantly see the new chains.

---

## 🧠 5. Understanding the `:ro` Suffix

`/path1:/path2:ro`  
→ Mounts path 1 from the host into the container at path 2, **read‑only**.

| Option | Meaning |
|---------|----------|
| `:ro` | Read‑only (recommended for CA bundles) |
| `:rw` or no option | Read‑write |

Using `:ro` prevents any process inside the container from corrupting host CA data.

---

## 🧾 6. When to Add These Mounts

| Container Type | Needs CA Mount? | Rationale |
|-----------------|----------------|------------|
| Internal‑only (no HTTPS) | ❌ | Communicates locally, unaffected by ZScaler |
| Public API consumers (`curl`, `npm`, `pip`) | ✅ | Must trust ZScalerRootCA |
| Build / CI containers fetching dependencies | ✅ | Global trust consistency |
| DB GUIs or internal tools | ✅ (optional) | For updates / plugin downloads |

---

## 🧩 7. Troubleshooting Checklist

1. **Verify the file exists** on host:  
   ```bash
   ls -l /etc/ssl/certs/ca-certificates.crt
   ```

2. **Inspect inside container:**  
   ```bash
   docker exec <container> cat /etc/ssl/certs/ca-certificates.crt | head
   ```

3. **Check network via curl:**  
   ```bash
   docker exec <container> curl -I https://www.google.com
   ```

4. **If SSL errors persist:**  
   - Confirm ZScalerRootCA was installed correctly on host (`sudo update-ca-certificates`).  
   - Ensure container actually mounted the read‑only file.  
   - Confirm no conflicting proxy environment variables are mis‑set.

---

## 🌍 8. Putting It All Together

With these mounts and (if required) proxy variables, your Docker containers will:
- Use the **same CA store** as your host Ubuntu system.  
- Correctly trust ZScaler’s re‑signed SSL connections.  
- Retain secure, read‑only protection of host certificate data.  
- Continue to communicate locally via their bridge network (pgAdmin ↔ Postgres) unaffected.

---

## 🏁 9. Final Recap

| Environment | Key Command / Setting |
|--------------|----------------------|
| **Compose File Mount** | `- /etc/ssl/certs/ca-certificates.crt:/etc/ssl/certs/ca-certificates.crt:ro` |
| **Manual docker run** | `-v /etc/ssl/certs/ca-certificates.crt:/etc/ssl/certs/ca-certificates.crt:ro` |
| **Optional Directory Mount** | `-v /etc/ssl/certs:/etc/ssl/certs:ro` |
| **Run Compose Stack** | `docker-compose up -d` |
| **Interactive Test** | `docker run --rm -it … bash` |

Once this is done, every container and the host share one consistent trusted CA root set.  
Your development‑to‑cloud workflow stays smooth, even behind ZScaler’s very watchful (but friendly) eyes. 😄