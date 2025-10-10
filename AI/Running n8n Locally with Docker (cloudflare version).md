# 🚀 Tutorial: Running n8n Locally with Docker and Exposing It to the Outside World

This guide shows you how to:
1. Run **n8n locally using Docker**.  
2. Create and test workflows on your own machine.  
3. Expose your workflows to external services using **Cloudflare Tunnel** (super reliable, free).  

---

## 📋 Prerequisites

- An Ubuntu host (with Docker & Docker Compose installed).
- Installed:
  - **Docker**: https://docs.docker.com/get-docker/
  - **Docker Compose**: https://docs.docker.com/compose/install/
- A Cloudflare account (free).
- Optional but recommended: Your own domain set up with Cloudflare DNS.

---

## 🐳 Step 1: Directory & Compose File

Create a folder called `n8n-docker` and inside create a file **docker-compose.yml**:

    version: '3.1'
    
    services:
      n8n:
        image: n8nio/n8n
        restart: always
        ports:
          - "5678:5678"
        environment:
          - N8N_BASIC_AUTH_ACTIVE=true
          - N8N_BASIC_AUTH_USER=admin
          - N8N_BASIC_AUTH_PASSWORD=supersecurepassword
          - N8N_HOST=localhost
          - N8N_PORT=5678
          - N8N_PROTOCOL=http
        volumes:
          - ./n8n_data:/home/node/.n8n

---

## ▶️ Step 2: Start Your n8n Instance

From the `n8n-docker` folder:

    docker-compose up -d

Check logs:

    docker-compose logs -f

Visit your instance at: **http://localhost:5678**  
Login with the username/password you set.

---

## 📝 Step 3: Create & Test a Workflow

1. Open **http://localhost:5678**.  
2. Create a new workflow.  
3. Add a **Manual Trigger** node.  
4. Add a **Set** node with dummy JSON data.  
5. Execute workflow → It runs locally 🎉  

---

## 🌍 Step 4: Expose n8n Using Cloudflare Tunnel

n8n workflows often need to receive **webhooks** (e.g. Slack, GitHub). External services can’t reach your local machine without a tunnel. Here’s how to set up **Cloudflare Tunnel** on Ubuntu.

---

### Step 4.1: Install Cloudflared on Ubuntu

Run the following commands:

    # Add Cloudflare GPG key
    curl -fsSL https://pkg.cloudflare.com/gpg | sudo tee /usr/share/keyrings/cloudflare-main.gpg > /dev/null

    # Add the package repository
    echo "deb [signed-by=/usr/share/keyrings/cloudflare-main.gpg] https://pkg.cloudflare.com/ bionic main" | sudo tee /etc/apt/sources.list.d/cloudflare-client.list

    # Update apt and install cloudflared
    sudo apt-get update && sudo apt-get install cloudflared -y

Verify installation:

    cloudflared --version

---

### Step 4.2: Quick Tunnel (for Testing)

To quickly expose n8n running on port 5678:

    cloudflared tunnel --url http://localhost:5678

You’ll see a public URL like:

    https://randomname.trycloudflare.com

Use this URL in your n8n webhook nodes.  
(Keep the terminal window running — the tunnel closes if you exit.)

---

### Step 4.3: Persistent Tunnel with Your Own Domain (Recommended)

For stability and cleaner URLs:

1. **Login Cloudflared:**

        cloudflared tunnel login

   A browser window opens → log in to Cloudflare → choose your domain.

2. **Create a Tunnel:**

        cloudflared tunnel create myn8n

   This creates credentials in `~/.cloudflared/`.

3. **Configure Routing:**  
   Create `~/.cloudflared/config.yml`:

        tunnel: myn8n
        credentials-file: /home/youruser/.cloudflared/<generated-tunnel-id>.json

        ingress:
          - hostname: n8n.yourdomain.com
            service: http://localhost:5678
          - service: http_status:404

   Replace `n8n.yourdomain.com` with the subdomain you want (managed under Cloudflare DNS).

4. **Update Docker Compose (n8n environment):**

   Add in `docker-compose.yml`:

        - WEBHOOK_URL=https://n8n.yourdomain.com/

5. **Run the Tunnel:**

        cloudflared tunnel run myn8n

   Or install it as a service to always run on boot:

        sudo cloudflared service install

---

## 🔐 Step 5: Security Practices

- Keep **Basic Auth** active in n8n (`N8N_BASIC_AUTH...` settings).  
- Cloudflare gives you **free HTTPS** and optional rules (mTLS, JWT, Google/Microsoft SSO if you use Access).  
- For learning, a quick `trycloudflare.com` URL is fine. For anything semi-permanent, use your own domain.

---

## 🎯 Wrap-Up

You now have:
- ✅ Local **n8n instance** on Docker (with persistent data).  
- ✅ Ability to create and trigger workflows.  
- ✅ External webhook exposure via **Cloudflare Tunnel**.  

With Cloudflare, you get **zero-cost, rock-solid reliability** for your learning lab, unlike LocalTunnel hiccups or ngrok limitations.  

✨ Bonus: Once you master this, try **self-hosting LocalTunnel** for the fun of running infra yourself and comparing reliability. Two birds, one nerd 🦉💻.