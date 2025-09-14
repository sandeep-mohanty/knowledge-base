# n8n Self-Hosted Tutorial with Docker 🐳

Welcome to this detailed guide on how to self-host **n8n** using Docker. You will learn how to install, configure, and run workflows with plenty of examples and step-by-step instructions.

---

## Table of Contents

1. Introduction  
2. Prerequisites  
3. Directory Structure  
4. Docker Compose Setup  
5. Starting n8n  
6. Basic Usage & UI Overview  
7. Creating Your First Workflow  
8. Example Workflows  
   - Hello World HTTP Request  
   - Slack Notification  
   - MySQL Data Sync  
9. Managing Credentials & Environment Variables  
10. Upgrading n8n  
11. Tips & Best Practices  
12. Troubleshooting  

---

## 1. Introduction

n8n is a free and open-source workflow automation tool. It allows you to connect various services (APIs, databases, messaging platforms) and automate complex tasks without writing code.

Key features:
- 200+ pre-built integrations (nodes)  
- Local development & self-hosting  
- Visual workflow editor  
- Extensible with JavaScript functions  

---

## 2. Prerequisites

Before starting, ensure you have:

- A Linux or macOS server (or WSL on Windows)  
- Docker installed (>= 20.10)  
- Docker Compose installed (>= 1.29)  
- Basic familiarity with CLI and YAML  

---

## 3. Directory Structure

Choose or create a directory for your n8n files. Example:
```
    ~/n8n-docker/
      ├── docker-compose.yml
      └── .env
```
---

## 4. Docker Compose Setup

Create a file named `docker-compose.yml` in your project directory:

```yaml
version: '3.8'
services:
  n8n:
    image: n8nio/n8n:latest
    restart: always
    ports:
      - "5678:5678"
    environment:
      - GENERIC_TIMEZONE=UTC
      - DB_TYPE=sqlite
      - DB_SQLITE_VACUUM_ON_STARTUP=true
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=user
      - N8N_BASIC_AUTH_PASSWORD=strongpassword
      - N8N_HOST=your.domain.com
      - N8N_PROTOCOL=https
    volumes:
      - n8n_data:/home/node/.n8n
volumes:
  n8n_data:
    driver: local
```

Create a `.env` file (optional) to store secrets:

```dotenv
N8N_BASIC_AUTH_USER=user
N8N_BASIC_AUTH_PASSWORD=strongpassword
GENERIC_TIMEZONE=UTC
```

> 🔐 **Security tip:** Never commit your `.env` with real passwords to public repos.

---

## 5. Starting n8n

From the project directory:
```bash
    cd ~/n8n-docker
    docker-compose up -
```

Check that the container is running:
```bash
    docker-compose ps
```

Or:
```bash
    docker ps | grep n8n
```
Access logs:
```bash
    docker-compose logs -f 
```
---

## 6. Basic Usage & UI Overview

1. Open your browser at `http://localhost:5678` (or your domain).  
2. Log in with basic auth credentials (`user` / `strongpassword`).  
3. You will see the workflow canvas:  
   - Left sidebar: Nodes library  
   - Center: Canvas  
   - Right: Node settings panel  

---

## 7. Creating Your First Workflow

### Workflow: Fetch public API data

1. Click **+** → HTTP Request node.  
2. Set **Method** to `GET`.  
3. Set **URL** to `https://api.coindesk.com/v1/bpi/currentprice.json`.  
4. Connect this node to a **Set** node:  
   - Adds or transforms data fields.  
   - Example: extract `bpi.USD.rate`.  
5. Add a **Debug** node (console output).  
6. Execute workflow manually.  

Result: You see Bitcoin price in the execution log.

---

## 8. Example Workflows

### 8.1 Hello World HTTP Request

Workflow steps:
1. Trigger: **Webhook** (GET) at `/webhook/hello-world`.  
2. Node: **Set** with JSON  
```json
{
  "message": "Hello, n8n!"
}
```  
3. Node: **Respond to Webhook** returns the JSON.  

Test with `curl`:
```bash
    curl http://localhost:5678/webhook/hello-world
```

### 8.2 Slack Notification

Prerequisite: Slack incoming webhook URL.

Workflow steps:
1. **Cron** node (e.g., every day at 9am).  
2. **HTTP Request** node:  
   - Method: `POST`  
   - URL: your Slack webhook  
   - Body:  
```json
{
  "text": "Good morning from n8n!"
}
```  
   - Content Type: `application/json`  

### 8.3 Postgres Data Sync

Prerequisites: PostgreSQL server credentials (host, port, database, user, password).

Workflow steps:

1. Postgres node (Read)  
   - Node Type: Postgres  
   - Operation: Execute Query  
   - Credentials: (select or create your Postgres credential in the UI)  
   - Query:
```sql
SELECT id, name
FROM users
WHERE updated_at > NOW() - INTERVAL '1 day'
```

2. Function node (Transform Rows)  
   - Node Type: Function  
   - Purpose: Map the rows into HTTP payload items  
   - Code:
```javascript
// items is an array of rows from the Postgres node
return items.map(item => {
  return {
    json: {
      userId: item.json.id,
      userName: item.json.name
    }
  };
});
```

3. HTTP Request node (Send Data)  
   - Node Type: HTTP Request  
   - Method: POST  
   - URL: https://example.com/api/users-sync  
   - Authentication: (as needed)  
   - Body Content Type: application/json  
   - Body Parameters:
```json
{
  "users": {{ $json }}
}
```

4. Connect the nodes in this order:  
   Postgres → Function → HTTP Request  

5. Execute the workflow manually or add a Trigger node (e.g., Cron) to run on a schedule.

Result: Each day, the workflow will fetch users updated in the last 24 hours from your Postgres database, transform them into the expected JSON format, and post them to your external API endpoint.

## 9. Managing Credentials & Environment Variables

Use n8n UI **Credentials** tab to securely store:
- API keys  
- OAuth2 credentials  
- Database logins  

Environment variables you can set:

| Variable                   | Description                                   | Default                   |
|----------------------------|-----------------------------------------------|---------------------------|
| N8N_PORT                   | Port to listen on                             | 5678                      |
| N8N_HOST                   | Hostname for webhook URLs                     | localhost                 |
| N8N_PROTOCOL               | http or https                                 | http                      |
| N8N_BASIC_AUTH_ACTIVE      | Enable basic auth (true/false)                | false                     |
| N8N_BASIC_AUTH_USER        | Username for basic auth                       |                           |
| N8N_BASIC_AUTH_PASSWORD    | Password for basic auth                       |                           |
| DB_TYPE                    | Type of database (sqlite, postgres, mysql)    | sqlite                    |
| DB_MYSQLDB_HOST / PORT     | MySQL host and port                           |                           |

---

## 10. Upgrading n8n

To upgrade:
1. Stop containers:  
```bash
       docker-compose down
```  
2. Pull latest image:
```bash  
       docker-compose pull
```
3. Start containers:  
```bash
       docker-compose up -d  
```

Data remains safe in the Docker volume `n8n_data`.

---

## 11. Tips & Best Practices

- Use a reverse proxy (NGINX, Traefik) for SSL termination.  
- Enable basic auth or OAuth on your reverse proxy.  
- Back up the `n8n_data` volume regularly.  
- Use PostgreSQL for production workloads.  
- Monitor using `docker stats` or external tools.  
- Leverage Functions and Function Items for custom JS logic.

---

## 12. Troubleshooting

Issue                           | Possible Fix  
------------------------------- | ------------------------------------------------  
n8n container won’t start      | Check `docker-compose logs` for syntax errors  
Workflow never triggers        | Ensure Trigger node is activated and URL is correct  
Credential missing             | Create or re-enter credential in UI  
Permission denied on volume    | Verify folder permissions on host  
Cannot reach webhook endpoint  | Check firewall rules and DNS settings  

---

🎉 You’re all set to explore the power of n8n! Build, automate, and connect all your services. Happy automating!