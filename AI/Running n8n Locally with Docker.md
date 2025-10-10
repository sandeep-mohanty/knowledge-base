# 🚀 Tutorial: Running n8n Locally with Docker and Exposing It to the Outside World

This tutorial will guide you through setting up **n8n** (the open-source workflow automation tool) on your **local machine using Docker**, so you can build and test workflows without needing the cloud-hosted version.  

We’ll also cover ways to **expose your local n8n instance to external services**, so webhooks and APIs can talk to your workflows using **LocalTunnel**.

---

## 📋 Prerequisites

- A working local machine (Linux/Mac/Windows with Docker & Docker Compose).
- Installed:
  - **Docker**: [Get Docker](https://docs.docker.com/get-docker/)
  - **Docker Compose** (if using `docker-compose.yml`).
  - **Node.js & npm** (needed to install `localtunnel`).
- Basic command-line familiarity.
- Optional but helpful: A public IP if you want to configure your own domain instead of tunnels.

---

## 🐳 Step 1: Create a Docker Compose File

We’ll keep things clean by using Docker Compose to configure n8n.

Create a folder (say `n8n-docker`) and inside make a file called: **`docker-compose.yml`**

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

### 🔍 Breakdown
- **`ports`**: Binds container’s port `5678` to host’s `5678`. Visit the UI at `http://localhost:5678`.
- **Authentication**: A simple username/password is enabled.
- **Volume**: Our workflows/Data are saved locally (`./n8n_data`) so they survive container restarts.

---

## ▶️ Step 2: Start Your Local n8n Instance

From the `n8n-docker` folder, run:

    docker-compose up -d

Check logs to ensure it’s running properly:

    docker-compose logs -f

Your n8n instance is now live at 👉 **http://localhost:5678**  
Login with the username/password you set in the compose file.

---

## 📝 Step 3: Create and Test a Simple Workflow

1. Open n8n designer via browser (`http://localhost:5678`).
2. Click **"New Workflow"**.
3. Drag in a **Manual Trigger** node.
4. Add a **Set** node to add some dummy JSON data.
5. Click **Execute Workflow** → you should see the results right in the UI.

Boom 🎉 your n8n playground is active and isolated on your machine.

---

## 🌍 Step 4: Expose Workflows (Accept External Traffic via LocalTunnel)

Many workflows depend on webhooks from external services (GitHub, Slack, Stripe). Normally, these can’t reach your local machine. **LocalTunnel** helps by creating a secure temporary public URL.

### Step 4.1: Install LocalTunnel
Install globally via npm:

    npm install -g localtunnel

### Step 4.2: Open a Tunnel to Your Localhost
Run this in your terminal (while n8n is running):

    lt --port 5678

This will return something like:

    your url is: https://fancy-frog-23.loca.lt

### Step 4.3: Configure n8n for Correct Webhook URL

In your `docker-compose.yml`, add or update the environment variable:

    - WEBHOOK_URL=https://fancy-frog-23.loca.lt/

Then restart n8n:

    docker-compose down && docker-compose up -d

Now all webhook nodes in n8n will advertise this LocalTunnel URL so external services can communicate with it.

---

## 🔐 Step 5: Security

- **Temporary tunnels**: LocalTunnel URLs change every time unless you request a subdomain with `--subdomain`.
- **Authentication**: Keep `N8N_BASIC_AUTH_ACTIVE=true` for preventing unauthorized access.
- **Reverse proxy + HTTPS**: For production setups, use a real domain and TLS instead.

---

## 🎯 Wrap-Up

You now have:
- ✅ A fully working **local Dockerized n8n** instance.  
- ✅ The ability to **build/test workflows** offline.  
- ✅ A way to **expose your workflows** to external services using LocalTunnel.  

This setup gives you a safe lab to experiment and connect APIs, without needing the hosted SaaS version.

---

✨ Pro Tip: Try LocalTunnel with the `--subdomain` flag if you want a predictable URL (like `https://myapp.loca.lt`).  

Now go forth and use tunnels like a pro tunnel-digger 🦫!