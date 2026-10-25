# 🦙 Complete Tutorial: Install & Configure Ollama on Ubuntu Server VM
### Build Your Own Private, Local AI Infrastructure for Coding Assistance

> **Who this is for:** Developers who want a dedicated, always-on, private AI backend running local LLMs — no cloud costs, no data leaving your machine, with 2–3 AI coding agents connected simultaneously.

---

## 📋 Table of Contents

1. [Introduction & What You'll Build](#1-introduction--what-youll-build)
2. [Understanding the Architecture](#2-understanding-the-architecture)
3. [Prerequisites & Planning](#3-prerequisites--planning)
4. [Choosing Your Ubuntu Server Version](#4-choosing-your-ubuntu-server-version)
5. [Creating the Ubuntu Server VM](#5-creating-the-ubuntu-server-vm)
6. [Preparing the Server Environment](#6-preparing-the-server-environment)
7. [Installing Ollama](#7-installing-ollama)
8. [Configuring Ollama for Network Access](#8-configuring-ollama-for-network-access)
9. [Firewall & Port Setup](#9-firewall--port-setup)
10. [Verifying the Installation](#10-verifying-the-installation)
11. [Working with AI Models](#11-working-with-ai-models)
12. [Integrating with AI Coding Agents](#12-integrating-with-ai-coding-agents)
13. [Securing with Nginx Reverse Proxy](#13-securing-with-nginx-reverse-proxy)
14. [Monitoring & Maintenance](#14-monitoring--maintenance)
15. [Troubleshooting Guide](#15-troubleshooting-guide)
16. [Real-World Use Cases](#16-real-world-use-cases)

---

## 1. 🚀 Introduction & What You'll Build

**Ollama** is an open-source runtime that lets you run large language models (LLMs) entirely on your own hardware — no subscriptions, no cloud APIs, no data privacy risks. It wraps model downloads, loading, and inference behind a clean HTTP API that is fully compatible with the OpenAI API format, which means every tool built for ChatGPT can point to your Ollama instance instead.

By placing Ollama inside a **dedicated Ubuntu Server VM**, you get:

- **Isolation** — heavy GPU/CPU inference doesn't compete with your development tools
- **Always-on access** — start the VM once, leave it running, connect from any machine on your network
- **Multi-agent support** — Cline, Continue.dev, Cursor, and Aider all connect simultaneously
- **Zero cost** — after initial hardware, running local models costs nothing per token

### Why a VM Instead of Running Ollama Directly?

| Approach | Pros | Cons |
|---|---|---|
| **Run on dev machine directly** | Simple, no VM overhead | Slows IDE & competes for RAM |
| **Cloud API (OpenAI, Anthropic)** | Easy, scalable, fast | Per-token cost, data leaves machine |
| **Docker container** | Portable, isolated | Harder GPU passthrough |
| **✅ Dedicated Ubuntu Server VM** | Always-on, isolated, private, free | Needs enough RAM on host |
| **Kubernetes cluster** | Highly scalable, fault-tolerant | Overkill for personal use |

---

## 2. 🏗️ Understanding the Architecture

Here is the complete picture of what you're building, from host machine down to individual agent connections:

```mermaid
graph TB
    subgraph HOST["🖥️ Ubuntu Desktop Host Machine"]
        subgraph VM["⚙️  Ubuntu Server VM — 192.168.1.100"]
            UFW["🛡️ UFW Firewall\nPort 11434 TCP Open"]
            NGINX["🔒 Nginx Reverse Proxy\nTLS + Basic Auth :443\n(Optional)"]
            OLLAMA["🦙 Ollama Service\n0.0.0.0:11434"]
            MODELS["📦 Model Storage\n~/.ollama/models\nllama3 · codellama · mistral\nphi3 · deepseek-coder"]

            UFW -->|Allows inbound traffic| OLLAMA
            NGINX -->|Proxies to internal port| OLLAMA
            OLLAMA <-->|Load & Inference| MODELS
        end
        VNET["🌐 Virtual Network\nBridge / NAT"]
        VM <--> VNET
    end

    CLINE["🤖 Cline\nVS Code Extension"] -->|"POST /v1/chat/completions"| UFW
    CONTINUE["🔄 Continue.dev\nVS Code & JetBrains"] -->|"POST /v1/chat/completions"| UFW
    CURSOR["⌨️ Cursor IDE\nFork of VS Code"] -->|"POST /v1/chat/completions"| UFW
    AIDER["🛠️ Aider\nCLI Pair Programmer"] -->|"POST /api/generate"| UFW
```

### Key Components Explained

| Component | Role | Default Location |
|---|---|---|
| **Ubuntu Server VM** | Isolated AI inference host | Hypervisor on your desktop |
| **Ollama Service** | LLM runtime + OpenAI-compatible API server | VM, port `11434` |
| **AI Models** | The actual GGUF-format weight files | `/usr/share/ollama/.ollama/models/` |
| **UFW Firewall** | Kernel-level traffic control | VM |
| **Nginx (optional)** | TLS termination + basic auth gate | VM, port `443` |
| **AI Agents** | Clients that send prompts and display responses | Your dev machine(s) |

---

## 3. 📋 Prerequisites & Planning

### Hardware Requirements for the VM

| Resource | Minimum | Recommended | Notes |
|---|---|---|---|
| **CPU (vCPUs)** | 4 cores | 8+ cores | More cores = faster CPU-only inference |
| **RAM** | 8 GB | **16 GB** | Needed for 7B+ parameter models |
| **Storage** | 50 GB | **100 GB** | Each model is 4–20 GB on disk |
| **GPU (optional)** | — | NVIDIA 8 GB+ VRAM | 10–50× faster than CPU inference |
| **Network** | Any | 1 Gbps LAN | Keeps latency low between VM and agents |

### Model Size Reference — Choose Wisely

| Model | Parameters | RAM Needed | Best For |
|---|---|---|---|
| `phi3:mini` | 3.8 B | ~4 GB | Fast autocomplete, light tasks |
| `llama3:8b` | 8 B | ~8 GB | ✅ Great all-purpose balance |
| `mistral:7b` | 7 B | ~7 GB | Fast, reasoning tasks |
| `deepseek-coder:6.7b` | 6.7 B | ~7 GB | Code generation & review |
| `codellama:13b` | 13 B | ~13 GB | ✅ Best for serious coding |
| `llama3:70b` | 70 B | ~40 GB | Excellent quality, needs GPU |

> 💡 **Rule of thumb:** With 16 GB RAM in your VM, you can comfortably run one 7B–13B model. Running two simultaneously requires ~14–16 GB total, so keep swap space ready.

### Software Prerequisites

- Ubuntu Desktop (host) with VirtualBox, VMware Workstation, or KVM installed
- Ubuntu Server 22.04 LTS ISO downloaded from `ubuntu.com/download/server`
- Terminal familiarity (basic `sudo`, `apt`, `systemctl` commands)

---

## 4. 🐧 Choosing Your Ubuntu Server Version

```mermaid
flowchart TD
    A([🤔 Which Ubuntu Server Version?]) --> B{Primary Goal?}

    B -->|"Stability for long-running\nAI inference server"| C["✅ Ubuntu Server 22.04 LTS\nJammy Jellyfish\nRECOMMENDED"]
    B -->|"Need latest NVIDIA drivers\nor Python 3.12+"| D["⚡ Ubuntu Server 24.04 LTS\nNoble Numbat"]
    B -->|"Already have existing\n20.04 infrastructure"| E["⚠️ Ubuntu Server 20.04 LTS\nFocal Fossa\nApproaching EOL"]

    C --> CF["🗓️ Support: April 2032\n🐧 Kernel: 5.15 LTS\n🐍 Python: 3.10\n📚 Community: Vast documentation"]
    D --> DF["🗓️ Support: April 2036\n🐧 Kernel: 6.8\n🐍 Python: 3.12\n🎮 NVIDIA: Latest drivers out-of-box"]
    E --> EF["🗓️ EOL: April 2025\n⚠️ Do NOT use for new setups"]

    CF --> REC["Best for personal AI server\n— use this unless you\nhave a specific reason not to"]
    DF --> REC2["Use if you have a newer\nNVIDIA GPU (RTX 40xx series)"]
```

### Version Comparison Table

| Feature | 22.04 LTS ✅ | 24.04 LTS |
|---|---|---|
| **Support Until** | April 2032 | April 2036 |
| **Kernel** | 5.15 | 6.8 |
| **Python** | 3.10 | 3.12 |
| **NVIDIA Driver Support** | Good (proprietary) | Excellent (open kernel module) |
| **Stability** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Community Docs / StackOverflow** | Extremely extensive | Growing |
| **Ollama Compatibility** | Fully tested | Fully tested |

**Verdict:** Use **Ubuntu Server 22.04 LTS** for everything in this tutorial. Switch to 24.04 only if you need cutting-edge NVIDIA open kernel modules for RTX 40xx+ GPUs.

---

## 5. 💻 Creating the Ubuntu Server VM

### VM Resource Configuration

```mermaid
graph LR
    subgraph VM_SPEC["Ubuntu Server VM — Recommended Spec"]
        CPU["⚙️ CPU\n4 vCPUs minimum\n8 vCPUs recommended\nEnable VT-x / AMD-V"]
        RAM["💾 RAM\n16 GB\nMinimum: 8 GB\nNeeded for 7B–13B models"]
        DISK["💽 Storage\n100 GB\nSSD-backed VHD preferred\nDynamic allocation OK"]
        NET["🌐 Network\nBridged Adapter\n(gives VM its own LAN IP)\nor NAT + Port Forward 11434"]

        CPU --- RAM --- DISK --- NET
    end
```

### Option A: KVM/QEMU — Best Performance on Ubuntu Host

KVM is the native Linux hypervisor. It has the lowest overhead and best performance.

```bash
# --- On your Ubuntu Desktop host ---

# Install KVM, management tools, and bridge utilities
sudo apt update
sudo apt install -y qemu-kvm libvirt-daemon-system \
    virt-manager virt-viewer bridge-utils

# Add yourself to the required groups (logout/login after)
sudo usermod -aG libvirt $USER
sudo usermod -aG kvm $USER

# Verify KVM is available
kvm-ok
# Expected: "KVM acceleration can be used"

# Launch the GUI VM manager
virt-manager
```

**In virt-manager:**
1. File → New Virtual Machine → Local install media (ISO)
2. Browse to your Ubuntu Server 22.04 ISO
3. RAM: **16384 MB** | CPUs: **4**
4. Disk: **100 GB** (sparse)
5. Network: **Default NAT** (or `virbr0` bridge for LAN visibility)
6. Click **Finish** and complete the Ubuntu installer

---

### Option B: VirtualBox

```bash
# Install on Ubuntu Desktop host
sudo apt install -y virtualbox virtualbox-ext-pack
```

**In the VirtualBox GUI:**
1. Machine → New → Name: `ollama-server`, Type: Linux, Version: Ubuntu (64-bit)
2. RAM: **16384 MB**
3. Create virtual hard disk → **VDI, Dynamically Allocated, 100 GB**
4. Settings → System → Processor → **4 CPUs**, enable PAE/NX
5. Settings → Network → Adapter 1 → **Bridged Adapter** (select your physical NIC)

> 💡 **Bridged Adapter** gives the VM its own IP address on your LAN — this is critical so that your dev machine agents can connect directly without port forwarding.

---

### Option C: VMware Workstation

1. File → New Virtual Machine → **Custom (Advanced)**
2. Guest OS: Linux → Ubuntu 64-bit
3. RAM: **16 GB** | Processors: **4** (2 cores × 2)
4. Disk: **100 GB, SCSI, Store as single file**
5. Network: **Bridged (Autodetect)**
6. Finish, then attach your Ubuntu Server ISO and boot

---

### Ubuntu Server Installation Checklist

During the Ubuntu installer:

- [x] Choose **"Ubuntu Server (minimized)"** — no GUI, smallest footprint
- [x] Configure network (note the assigned IP — you'll use it throughout this tutorial)
- [x] Skip optional snaps
- [x] Enable **OpenSSH server** — lets you SSH in from host without VM window
- [x] Set a strong username and password

```bash
# After first boot, SSH in from your Desktop host:
ssh youruser@<VM_IP>
# Example: ssh dev@192.168.1.100
```

---

## 6. 🔧 Preparing the Server Environment

### Step 1: Update All Packages

```bash
sudo apt update && sudo apt upgrade -y
```

**Why this matters:**
- `apt update` refreshes the package index from Ubuntu repositories
- `apt upgrade` closes known security vulnerabilities in the base OS
- Always run this before installing new software

---

### Step 2: Install Essential Tools

```bash
sudo apt install -y curl wget git htop net-tools unzip jq
```

| Tool | Why You Need It |
|---|---|
| `curl` | Download files, test Ollama API endpoints |
| `wget` | Alternative downloader for scripts |
| `git` | Pull config files and utilities |
| `htop` | Monitor CPU/RAM usage during model inference |
| `net-tools` | `netstat`, `ifconfig` for network diagnostics |
| `jq` | Pretty-print JSON responses from Ollama API |
| `unzip` | Extract archives when needed |

---

### Step 3: Find and Lock Down the VM's IP Address

```bash
# Find current IP
ip addr show
# or
hostname -I
```

**Assign a Static IP (Strongly Recommended)**

Prevents the VM's IP from changing on reboots, which would break all your agent configs.

```bash
# Find your network interface name
ip link show
# Common names: enp1s0, ens3, eth0

# Edit Netplan configuration
sudo nano /etc/netplan/00-installer-config.yaml
```

```yaml
network:
  version: 2
  ethernets:
    enp1s0:                        # ← Replace with YOUR interface name
      dhcp4: false
      addresses:
        - 192.168.1.100/24         # ← Your chosen static IP
      routes:
        - to: default
          via: 192.168.1.1         # ← Your router's IP
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
```

```bash
# Apply the configuration
sudo netplan apply

# Confirm the IP is set
ip addr show enp1s0
```

> ⚠️ **Note this IP address.** It appears in every agent config file for the rest of this tutorial. In the examples below, we use `192.168.1.100` — replace with your actual VM IP.

---

## 7. 🦙 Installing Ollama

### The One-Line Installer

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

### What the Installer Does (Behind the Scenes)

```mermaid
flowchart TD
    A["curl fetches install.sh from ollama.com"] --> B["Detects OS & CPU architecture\nLinux x86_64 or arm64"]
    B --> C["Downloads Ollama binary\nfrom GitHub Releases\n(latest version)"]
    C --> D["Installs binary to\n/usr/local/bin/ollama"]
    D --> E["Creates system user 'ollama'\nwith home: /usr/share/ollama"]
    E --> F["Writes systemd unit file\n/etc/systemd/system/ollama.service"]
    F --> G["Runs: systemctl enable ollama\nRuns: systemctl start ollama"]
    G --> H{NVIDIA GPU\nDetected?}
    H -->|Yes| I["Also installs CUDA libraries\nEnables GPU inference"]
    H -->|No| J["CPU-only mode\n(still works, just slower)"]
    I --> K["✅ Ollama running on\nlocalhost:11434"]
    J --> K
```

### Verify the Installation

```bash
# Confirm the binary is on your PATH
which ollama
# Expected: /usr/local/bin/ollama

# Check the version
ollama --version
# Expected: ollama version 0.x.x

# Check the systemd service
systemctl status ollama
```

Expected service output:
```
● ollama.service - Ollama Service
     Loaded: loaded (/etc/systemd/system/ollama.service; enabled; preset: enabled)
     Active: active (running) since Mon 2025-01-01 10:00:00 UTC; 5s ago
   Main PID: 1234 (ollama)
```

### Inspect the Default Service File

```bash
cat /etc/systemd/system/ollama.service
```

```ini
[Unit]
Description=Ollama Service
After=network-online.target

[Service]
ExecStart=/usr/local/bin/ollama serve
User=ollama
Group=ollama
Restart=always
RestartSec=3
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"

[Install]
WantedBy=multi-user.target
```

> 🔍 **Notice:** No `OLLAMA_HOST` variable — meaning it defaults to `127.0.0.1:11434` (localhost only). The next step fixes this so network agents can connect.

---

## 8. 🌐 Configuring Ollama for Network Access

By default, Ollama only accepts connections from within the VM itself (`127.0.0.1`). To allow your dev machines and agents to connect, we need to bind to `0.0.0.0` — all network interfaces.

### Network Configuration Flow

```mermaid
flowchart LR
    subgraph BEFORE["❌ Default — Localhost Only"]
        OL1["Ollama Service\n127.0.0.1:11434"]
        VM_ONLY["Only the VM itself\ncan send requests"]
        VM_ONLY --> OL1
    end

    subgraph AFTER["✅ After Configuration — Network Access"]
        OL2["Ollama Service\n0.0.0.0:11434"]
        DEV1["Dev Machine\n192.168.1.50"] --> OL2
        DEV2["Dev Machine\n192.168.1.51"] --> OL2
        DEV3["VM Itself\n127.0.0.1"] --> OL2
    end

    BEFORE -->|"Add OLLAMA_HOST env var"| AFTER
```

### Method 1: systemd Override (Recommended)

Using a systemd **override** is better than editing the service file directly — it survives Ollama updates without being overwritten.

```bash
# Open the override editor
sudo systemctl edit ollama
```

The editor opens a blank file at `/etc/systemd/system/ollama.service.d/override.conf`. Add:

```ini
[Service]
Environment="OLLAMA_HOST=0.0.0.0:11434"
Environment="OLLAMA_KEEP_ALIVE=24h"
Environment="OLLAMA_NUM_PARALLEL=2"
Environment="OLLAMA_MAX_QUEUE=10"
```

Save and exit (`Ctrl+X`, `Y`, `Enter` in nano), then:

```bash
# Reload systemd daemon to pick up the override
sudo systemctl daemon-reexec

# Restart Ollama to apply
sudo systemctl restart ollama

# Confirm the environment variable is active
sudo systemctl show ollama | grep Environment
# Should contain: OLLAMA_HOST=0.0.0.0:11434
```

### Method 2: Direct Edit of the Service File

Use this if you prefer a simpler approach (note: may be overwritten on updates):

```bash
sudo nano /etc/systemd/system/ollama.service
```

Add the environment lines under `[Service]`:

```ini
[Service]
ExecStart=/usr/local/bin/ollama serve
User=ollama
Group=ollama
Restart=always
RestartSec=3
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
Environment="OLLAMA_HOST=0.0.0.0:11434"
Environment="OLLAMA_KEEP_ALIVE=24h"
Environment="OLLAMA_NUM_PARALLEL=2"
```

```bash
sudo systemctl daemon-reload
sudo systemctl restart ollama
```

### All Available Environment Variables

| Variable | Example Value | Effect |
|---|---|---|
| `OLLAMA_HOST` | `0.0.0.0:11434` | Bind address and port |
| `OLLAMA_KEEP_ALIVE` | `24h` | How long to keep model in RAM after last use |
| `OLLAMA_NUM_PARALLEL` | `2` | Simultaneous inference requests |
| `OLLAMA_MAX_QUEUE` | `10` | Request queue depth before rejecting |
| `OLLAMA_MAX_VRAM` | `6000000000` | VRAM cap in bytes (GPU setups) |
| `OLLAMA_DEBUG` | `1` | Enable verbose debug logging |
| `OLLAMA_MODELS` | `/data/models` | Custom model storage path |
| `OLLAMA_FLASH_ATTENTION` | `1` | Enable Flash Attention (faster) |

### Verify Ollama Is Listening on All Interfaces

```bash
# Check which address the process is bound to
ss -tlnp | grep 11434
# Expected: LISTEN  0  128  0.0.0.0:11434  0.0.0.0:*

# Test from the VM itself
curl http://localhost:11434
# Expected: Ollama is running

# Test from your dev machine (replace with your VM IP)
curl http://192.168.1.100:11434
# Expected: Ollama is running

# Test the models API endpoint
curl http://192.168.1.100:11434/api/tags | jq
```

---

## 9. 🛡️ Firewall & Port Setup

### Basic Firewall Configuration

```bash
# ALWAYS allow SSH first — or you'll lock yourself out!
sudo ufw allow ssh
sudo ufw allow 22/tcp

# Enable the firewall
sudo ufw enable

# Allow Ollama API port
sudo ufw allow 11434/tcp

# Reload rules
sudo ufw reload

# Verify
sudo ufw status verbose
```

Expected output:
```
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere
11434/tcp                  ALLOW IN    Anywhere
22/tcp (v6)                ALLOW IN    Anywhere (v6)
11434/tcp (v6)             ALLOW IN    Anywhere (v6)
```

### Firewall Strategy Decision Tree

```mermaid
flowchart TD
    A([🛡️ Configure Firewall]) --> B{What is your\nenvironment?}

    B -->|"Home LAN\nOnly you use it"| C["Open to All\nufw allow 11434/tcp\n✅ Simple & Fast"]
    B -->|"Shared office or\nhome network\nwith others"| D["Restrict to Specific IPs\nufw allow from 192.168.1.x\nto any port 11434"]
    B -->|"Cloud VM or\nInternet-accessible"| E["Block Direct Port\nUse Nginx + TLS\nOnly open 443/tcp"]

    C --> F["Fastest setup\nAny LAN device can connect"]
    D --> G["Balanced security\nOnly known machines"]
    E --> H["Maximum security\nSee Section 13: Nginx"]
```

### Restrict to Specific Machines (More Secure)

```bash
# Allow only specific dev machines
sudo ufw allow from 192.168.1.50 to any port 11434 comment "Dev Machine - Alice"
sudo ufw allow from 192.168.1.51 to any port 11434 comment "Dev Machine - Bob"

# Or allow your entire subnet
sudo ufw allow from 192.168.1.0/24 to any port 11434

# Remove the overly broad rule
sudo ufw delete allow 11434/tcp
```

### If Using a Cloud VM (AWS / GCP / Azure)

For cloud VMs, UFW alone isn't enough — you also need to open the port in the cloud provider's security group / firewall:

```bash
# AWS CLI example — allow port 11434 from your home IP
aws ec2 authorize-security-group-ingress \
    --group-id sg-xxxxxxxx \
    --protocol tcp \
    --port 11434 \
    --cidr YOUR.HOME.IP.ADDRESS/32
```

> ⚠️ **For cloud VMs:** Strongly consider using Nginx with TLS instead of opening 11434 directly. See Section 13.

---

## 10. ✅ Verifying the Installation

Run through these tests in order. Each one confirms a deeper layer of the stack is working.

### Full Verification Flow

```mermaid
flowchart TD
    T1["🔍 Test 1\nsystemctl status ollama\nCheck: active running"] --> T2
    T2["🔍 Test 2\ncurl localhost:11434\nCheck: Ollama is running"] --> T3
    T3["🔍 Test 3\nss -tlnp | grep 11434\nCheck: 0.0.0.0:11434"] --> T4
    T4["🔍 Test 4\ncurl from DEV machine\nCheck: reachable over network"] --> T5
    T5["🔍 Test 5\ncurl /api/tags\nCheck: JSON model list"] --> T6
    T6["🔍 Test 6\ncurl /v1/models\nCheck: OpenAI-compat endpoint"] --> DONE

    T1 -->|Fails| FIX1["journalctl -u ollama\nCheck install logs"]
    T2 -->|Fails| FIX2["systemctl restart ollama\nCheck OLLAMA_HOST"]
    T3 -->|Shows 127.0.0.1| FIX3["systemctl edit ollama\nAdd OLLAMA_HOST=0.0.0.0:11434"]
    T4 -->|Fails| FIX4["ufw allow 11434/tcp\nCheck VM network mode"]
    T5 -->|Empty list| FIX5["ollama pull llama3\nPull at least one model"]

    DONE(["✅ All Tests Pass\nProceed to Section 11"])
```

### Test Commands

```bash
# Test 1: Service running?
systemctl status ollama

# Test 2: Local API responding?
curl http://localhost:11434
# Expected: Ollama is running

# Test 3: Bound to all interfaces?
ss -tlnp | grep 11434
# Expected: 0.0.0.0:11434

# Test 4: Reachable from dev machine?
# (Run this on your dev machine, not the VM)
curl http://192.168.1.100:11434
# Expected: Ollama is running

# Test 5: Models API?
curl http://192.168.1.100:11434/api/tags | jq
# Expected: {"models": [...]}

# Test 6: OpenAI-compatible endpoint?
curl http://192.168.1.100:11434/v1/models | jq
# Expected: {"object":"list","data":[...]}
```

### Check Logs for Any Issues

```bash
# View the last 50 lines of Ollama logs
journalctl -u ollama -n 50

# Follow live logs (useful when running a model)
journalctl -u ollama -f

# Filter for errors only
journalctl -u ollama -p err
```

---

## 11. 🧠 Working with AI Models

### Pulling Your First Models

```bash
# General purpose — good starting point
ollama pull llama3

# Best for code generation and review
ollama pull codellama:13b

# Fast, lightweight — great for autocomplete
ollama pull phi3:mini

# Excellent reasoning + code
ollama pull deepseek-coder:6.7b

# Mistral — fast and smart
ollama pull mistral
```

### Model Pull Flow

```mermaid
sequenceDiagram
    participant U as 👤 You (CLI)
    participant OC as 🦙 Ollama CLI
    participant REG as 📡 Ollama Registry
    participant DISK as 💽 Local Disk

    U->>OC: ollama pull llama3
    OC->>REG: Fetch model manifest (layers list)
    REG-->>OC: Manifest: sha256 hashes + sizes per layer

    loop For each GGUF layer
        OC->>REG: GET /v2/library/llama3/blobs/{sha256}
        REG-->>DISK: Stream binary data
        OC->>U: ▓▓▓▓░░░░ 45% (3.2 GB / 7.1 GB)
    end

    OC->>DISK: Verify all checksums
    DISK-->>OC: ✅ All layers valid
    OC-->>U: success — model: llama3
```

### Running a Quick Test

```bash
# Interactive chat mode (on the VM terminal or via SSH)
ollama run llama3

# >>> Write a Python quicksort implementation
# (model responds here)
# >>> /bye   ← exit

# One-shot generation (non-interactive)
ollama run codellama "Write a Python class for a binary search tree"

# Pipe code for review
cat myfile.py | ollama run codellama "Review this code for bugs and suggest improvements"
```

### Model Management Commands

```bash
# List all downloaded models (with sizes)
ollama list

# Show model metadata and parameters
ollama show llama3

# Remove a model to free disk space
ollama rm llama3

# Copy/alias a model
ollama cp llama3 my-custom-llama

# Check currently loaded models (in RAM)
curl http://localhost:11434/api/ps | jq
```

### API Examples — Test Before Connecting Agents

```bash
# Native Ollama API — streaming generation
curl http://192.168.1.100:11434/api/generate \
  -H "Content-Type: application/json" \
  -d '{
    "model": "llama3",
    "prompt": "Write a Python decorator that logs function call time",
    "stream": false
  }' | jq '.response'

# OpenAI-compatible chat completions (what agents use)
curl http://192.168.1.100:11434/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "llama3",
    "messages": [
      {"role": "system", "content": "You are a helpful coding assistant."},
      {"role": "user", "content": "Explain Python context managers with examples"}
    ]
  }' | jq '.choices[0].message.content'
```

---

## 12. 🤖 Integrating with AI Coding Agents

Ollama's `/v1/*` endpoints are OpenAI API-compatible. Any tool that supports OpenAI can point to Ollama instead — just change the base URL.

### Request Flow from Agent to Model

```mermaid
sequenceDiagram
    participant DEV as 👩‍💻 Developer
    participant IDE as 🛠️ IDE / VS Code
    participant AGENT as 🤖 AI Agent
    participant OLLAMA as 🦙 Ollama API
    participant LLM as 🧠 Local LLM

    DEV->>IDE: Write code or type a question
    IDE->>AGENT: Trigger completion event
    AGENT->>AGENT: Read config\napiBaseUrl, model, apiKey
    AGENT->>OLLAMA: POST /v1/chat/completions\n{"model":"llama3","messages":[...]}
    OLLAMA->>LLM: Tokenize + begin inference
    loop Streaming tokens
        LLM-->>OLLAMA: Next token
        OLLAMA-->>AGENT: SSE: data: {"choices":[{"delta":{"content":"..."}}]}
    end
    AGENT-->>IDE: Render streamed suggestion
    IDE-->>DEV: Display AI response inline
```

---

### Agent 1: Cline (VS Code Extension)

Cline is a fully autonomous AI coding agent — it reads files, writes code, runs commands, and iterates.

**Install:** VS Code Extensions panel → search **"Cline"** → Install

**Configuration:** Create `.cline/config.json` in your workspace root:

```json
{
  "provider": "openai",
  "apiBaseUrl": "http://192.168.1.100:11434/v1",
  "apiKey": "ollama",
  "defaultModel": "llama3",
  "models": {
    "llama3": {
      "provider": "openai",
      "apiBaseUrl": "http://192.168.1.100:11434/v1",
      "apiKey": "ollama",
      "contextWindow": 8192,
      "maxTokens": 4096
    },
    "codellama": {
      "provider": "openai",
      "apiBaseUrl": "http://192.168.1.100:11434/v1",
      "apiKey": "ollama",
      "contextWindow": 16384,
      "maxTokens": 8192
    },
    "deepseek-coder:6.7b": {
      "provider": "openai",
      "apiBaseUrl": "http://192.168.1.100:11434/v1",
      "apiKey": "ollama",
      "contextWindow": 8192,
      "maxTokens": 4096
    }
  }
}
```

> 💡 The `apiKey` value can be any non-empty string — Ollama requires it in the schema but ignores its value.

**Global config** (applies to all workspaces): `~/.cline/config.json`

---

### Agent 2: Continue.dev

Continue is an open-source coding assistant for VS Code and JetBrains with inline chat, slash commands, and tab autocomplete.

**Install:** VS Code Extensions → search **"Continue"** → Install

**Config file:** `~/.continue/config.json` on your **dev machine**:

```json
{
  "models": [
    {
      "title": "LLaMA 3 — General Chat",
      "provider": "ollama",
      "model": "llama3",
      "apiBase": "http://192.168.1.100:11434"
    },
    {
      "title": "CodeLlama 13B — Code Review",
      "provider": "ollama",
      "model": "codellama:13b",
      "apiBase": "http://192.168.1.100:11434"
    },
    {
      "title": "DeepSeek Coder — Code Generation",
      "provider": "ollama",
      "model": "deepseek-coder:6.7b",
      "apiBase": "http://192.168.1.100:11434"
    }
  ],
  "tabAutocompleteModel": {
    "title": "Phi3 Mini — Fast Autocomplete",
    "provider": "ollama",
    "model": "phi3:mini",
    "apiBase": "http://192.168.1.100:11434"
  },
  "embeddingsProvider": {
    "provider": "ollama",
    "model": "nomic-embed-text",
    "apiBase": "http://192.168.1.100:11434"
  },
  "contextProviders": [
    {"name": "code"},
    {"name": "docs"},
    {"name": "git"},
    {"name": "terminal"}
  ]
}
```

---

### Agent 3: Cursor IDE

Cursor is an AI-first IDE (VS Code fork with deeply integrated AI completions).

**Via UI:** Cursor → Settings → Models → Add Model:
- Base URL: `http://192.168.1.100:11434/v1`
- API Key: `ollama`
- Model: `llama3`

**Via config file** (`.cursor/config.json` in workspace):

```json
{
  "ai": {
    "provider": "openai-compatible",
    "baseUrl": "http://192.168.1.100:11434/v1",
    "apiKey": "ollama",
    "model": "codellama:13b",
    "temperature": 0.1,
    "maxTokens": 4096
  }
}
```

---

### Agent 4: Aider (CLI Pair Programmer)

Aider is a terminal-based AI coding tool that makes commits to your git repo.

```bash
# Install on your dev machine
pip install aider-chat

# Set the Ollama base URL
export OPENAI_API_BASE=http://192.168.1.100:11434/v1
export OPENAI_API_KEY=ollama

# Run aider with a specific model
aider --model openai/codellama:13b

# Or use Aider's native Ollama support
aider --model ollama/codellama:13b yourfile.py

# For a single task without entering chat
aider --model ollama/llama3 \
      --message "Add type hints to all functions" \
      src/utils.py
```

---

### Agent 5: Open WebUI (Browser Chat Interface)

For a ChatGPT-like web UI pointing to your Ollama backend:

```bash
# On your dev machine (requires Docker)
docker run -d \
  --name open-webui \
  -p 3000:8080 \
  -e OLLAMA_BASE_URL=http://192.168.1.100:11434 \
  -v open-webui:/app/backend/data \
  ghcr.io/open-webui/open-webui:main

# Access at: http://localhost:3000
```

---

### Multi-Agent Connection Overview

```mermaid
graph TB
    subgraph OLLAMA_SERVER["🖥️ Ubuntu Server VM — 192.168.1.100:11434"]
        SCHED["🦙 Ollama Scheduler\nOLLAMA_NUM_PARALLEL=3"]
        subgraph MODELS_POOL["Loaded Models"]
            M1["llama3:8b\n(Chat + Docs)"]
            M2["codellama:13b\n(Code Review)"]
            M3["phi3:mini\n(Fast Autocomplete)"]
        end
        SCHED --> MODELS_POOL
    end

    subgraph WS1["💻 Workstation 1"]
        CLINE_A["Cline\ncodellama:13b"]
        CONTINUE_A["Continue.dev\nllama3"]
    end

    subgraph WS2["💻 Workstation 2"]
        CURSOR_B["Cursor IDE\ndeepseek-coder:6.7b"]
    end

    subgraph WS3["💻 Workstation 3"]
        AIDER_C["Aider CLI\ncodellama:13b"]
    end

    CLINE_A    -->|"POST /v1/chat/completions"| SCHED
    CONTINUE_A -->|"POST /v1/chat/completions"| SCHED
    CURSOR_B   -->|"POST /v1/chat/completions"| SCHED
    AIDER_C    -->|"POST /api/generate"| SCHED
```

> 💡 **Concurrency tip:** With `OLLAMA_NUM_PARALLEL=2` and 16 GB RAM, Ollama can handle 2 simultaneous requests to the same model, or switch between different models sequentially.

---

## 13. 🔒 Securing with Nginx Reverse Proxy

For shared networks or internet-accessible VMs, add TLS encryption and password authentication in front of Ollama.

### Security Architecture

```mermaid
flowchart LR
    subgraph CLIENTS["🖥️ Dev Machines"]
        A1["Cline"]
        A2["Continue.dev"]
        A3["Cursor"]
    end

    subgraph PROXY["🔒 Nginx Reverse Proxy  :443"]
        TLS["TLS Termination\nssl_certificate .pem"]
        AUTH["Basic Auth\n.htpasswd file"]
        PASS["proxy_pass\n→ 127.0.0.1:11434"]
        TLS --> AUTH --> PASS
    end

    subgraph BACKEND["🦙 Ollama  127.0.0.1:11434\n(Internal Only)"]
        SVC["Ollama Service\nNot exposed externally"]
        LLM["LLM Engine\nCPU / GPU Inference"]
        SVC --> LLM
    end

    A1 & A2 & A3 -->|"HTTPS :443\nEncrypted + Password"| TLS
    PASS --> SVC

    FIREWALL["🛡️ UFW\n443/tcp: OPEN\n11434/tcp: BLOCKED externally"]
    CLIENTS -.->|"Port 11434 BLOCKED"| FIREWALL
```

### Step 1: Install Nginx

```bash
sudo apt install -y nginx
sudo systemctl enable nginx
sudo systemctl start nginx
```

---

### Step 2: Create SSL Certificate

**Option A: Self-signed (for LAN use)**

```bash
sudo openssl req -x509 -nodes -days 3650 -newkey rsa:2048 \
  -keyout /etc/ssl/private/ollama.key \
  -out /etc/ssl/certs/ollama.crt \
  -subj "/CN=ollama-server/O=LocalDev/C=US"

# Lock down the private key
sudo chmod 600 /etc/ssl/private/ollama.key
```

**Option B: Let's Encrypt (for internet-facing VM with a domain)**

```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d yourdomain.example.com
# Certbot auto-renews via cron — no manual renewal needed
```

---

### Step 3: Create User Credentials

```bash
# Install htpasswd utility
sudo apt install -y apache2-utils

# Create the password file and first user
sudo htpasswd -c /etc/nginx/.htpasswd developer
# Enter and confirm a strong password when prompted

# Add additional users (omit -c to append, not overwrite)
sudo htpasswd /etc/nginx/.htpasswd alice
sudo htpasswd /etc/nginx/.htpasswd bob

# View the file (passwords are bcrypt-hashed)
cat /etc/nginx/.htpasswd
```

---

### Step 4: Configure Nginx

```bash
sudo nano /etc/nginx/sites-available/ollama
```

```nginx
# HTTP → HTTPS redirect
server {
    listen 80;
    server_name ollama-server.local 192.168.1.100;
    return 301 https://$host$request_uri;
}

# Main HTTPS server block
server {
    listen 443 ssl;
    server_name ollama-server.local 192.168.1.100;

    # --- SSL Configuration ---
    ssl_certificate     /etc/ssl/certs/ollama.crt;
    ssl_certificate_key /etc/ssl/private/ollama.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers off;
    ssl_session_cache shared:SSL:10m;

    # --- Authentication ---
    auth_basic "Ollama AI Server — Authorized Access Only";
    auth_basic_user_file /etc/nginx/.htpasswd;

    # --- Proxy to Ollama ---
    location / {
        proxy_pass http://127.0.0.1:11434;
        proxy_http_version 1.1;

        # Forward client info to Ollama
        proxy_set_header Host              $host;
        proxy_set_header X-Real-IP         $remote_addr;
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Connection        '';

        # CRITICAL: Disable buffering for LLM streaming
        proxy_buffering            off;
        proxy_cache                off;
        proxy_read_timeout         300s;
        proxy_send_timeout         300s;
        chunked_transfer_encoding  on;
    }
}
```

---

### Step 5: Enable the Site and Harden Firewall

```bash
# Enable the Ollama site
sudo ln -s /etc/nginx/sites-available/ollama /etc/nginx/sites-enabled/

# Remove the default placeholder site
sudo rm -f /etc/nginx/sites-enabled/default

# Test configuration syntax
sudo nginx -t
# Expected: syntax is ok + test is successful

# Restart Nginx
sudo systemctl restart nginx

# Update firewall: open 443, block direct 11434 from outside
sudo ufw allow 443/tcp
sudo ufw deny in on eth0 to any port 11434  # Block external access to 11434
sudo ufw allow in on lo to any port 11434    # Keep localhost access
sudo ufw reload

sudo ufw status verbose
```

---

### Step 6: Update Agent Configs for HTTPS + Auth

Generate your Basic Auth header value:

```bash
echo -n "developer:yourpassword" | base64
# Output: ZGV2ZWxvcGVyOnlvdXJwYXNzd29yZA==
```

Update agent configs to use HTTPS:

**Cline `.cline/config.json`:**
```json
{
  "provider": "openai",
  "apiBaseUrl": "https://192.168.1.100/v1",
  "apiKey": "ollama",
  "defaultModel": "llama3",
  "apiHeaders": {
    "Authorization": "Basic ZGV2ZWxvcGVyOnlvdXJwYXNzd29yZA=="
  }
}
```

**Continue.dev `~/.continue/config.json`:**
```json
{
  "models": [
    {
      "title": "LLaMA 3 (Secured)",
      "provider": "openai",
      "model": "llama3",
      "apiBase": "https://192.168.1.100/v1",
      "apiKey": "Basic ZGV2ZWxvcGVyOnlvdXJwYXNzd29yZA=="
    }
  ]
}
```

**curl test with Basic Auth:**
```bash
curl -u developer:yourpassword https://192.168.1.100/api/tags \
  --cacert /etc/ssl/certs/ollama.crt  # For self-signed cert
```

---

## 14. 📊 Monitoring & Maintenance

### Real-Time Resource Monitoring

```bash
# Full system overview — CPU, RAM, processes
htop

# Watch RAM usage while model is running
watch -n 2 'free -h'

# Watch Ollama process specifically
watch -n 2 'ps aux | grep "[o]llama"'

# If using NVIDIA GPU
watch -n 1 nvidia-smi

# Disk usage for models
du -sh /usr/share/ollama/.ollama/models/
```

### Ollama API Health Checks

```bash
# Check what models are currently in memory (loaded/running)
curl http://localhost:11434/api/ps | jq

# Detailed model info
curl http://localhost:11434/api/show \
  -H "Content-Type: application/json" \
  -d '{"name": "llama3"}' | jq

# List all pulled models with sizes
ollama list
```

### Keeping Ollama Updated

```bash
# Re-run the installer — it updates in place
curl -fsSL https://ollama.com/install.sh | sh

# Restart after updating
sudo systemctl restart ollama

# Verify new version
ollama --version
```

### Cron Job for Automatic Model Updates

```bash
# Edit the ollama user's crontab
sudo crontab -u ollama -e

# Add: update models every Sunday at 3 AM
0 3 * * 0 /usr/local/bin/ollama pull llama3 && \
           /usr/local/bin/ollama pull codellama && \
           /usr/local/bin/ollama pull phi3:mini
```

---

## 15. 🔍 Troubleshooting Guide

### Diagnostic Decision Tree

```mermaid
flowchart TD
    START([🐛 Something is not working]) --> Q1{Does\nsystemctl status ollama\nshow 'active running'?}

    Q1 -->|No — failed or inactive| FIX1["Check install:\njournalctl -u ollama -n 50\nRe-run: curl -fsSL ollama.com/install.sh | sh"]
    Q1 -->|Yes| Q2

    Q2{Does\ncurl localhost:11434\nreturn 'Ollama is running'?}
    Q2 -->|No| FIX2["Service started but not binding\nsystemctl restart ollama\nCheck OLLAMA_HOST in override"]
    Q2 -->|Yes| Q3

    Q3{"Does\nss -tlnp | grep 11434\nshow 0.0.0.0:11434?"}
    Q3 -->|Shows 127.0.0.1 only| FIX3["Override missing or not applied\nsystemctl edit ollama\nAdd OLLAMA_HOST=0.0.0.0:11434\nsystemctl restart ollama"]
    Q3 -->|Yes| Q4

    Q4{"Can you ping\nthe VM from\nyour dev machine?"}
    Q4 -->|No| FIX4["VM network problem\nCheck hypervisor network mode\nSet Bridged adapter\nCheck host firewall"]
    Q4 -->|Yes| Q5

    Q5{"Does\ncurl VM_IP:11434\nwork from dev machine?"}
    Q5 -->|No| FIX5["UFW blocking traffic\nsudo ufw allow 11434/tcp\nufw status\nufw reload"]
    Q5 -->|Yes| Q6

    Q6{"Is the agent\ngetting errors\nor no response?"}
    Q6 -->|Model not found| FIX6["ollama pull modelname\nConfirm with: ollama list"]
    Q6 -->|Timeout| FIX7["free -h — check RAM\nOLLAMA_KEEP_ALIVE=1h\nTry smaller model: phi3:mini"]
    Q6 -->|Connection refused| FIX8["Verify apiBaseUrl in config\nInclude /v1 suffix for OpenAI compat"]
    Q6 -->|Working fine| DONE(["✅ All good!"])
```

### Common Issues Quick Reference

| Symptom | Likely Cause | Fix |
|---|---|---|
| `Connection refused` on 11434 | Service not running / wrong host | `systemctl restart ollama` + check `OLLAMA_HOST` |
| Works on VM, not from dev machine | Firewall blocking | `ufw allow 11434/tcp` |
| Works sometimes, times out | Model too large for RAM | Use smaller model or add swap |
| `model not found` in agent | Model not pulled | `ollama pull <modelname>` |
| Nginx 502 Bad Gateway | Ollama not running locally | `curl localhost:11434` to verify |
| Streaming not working | Proxy buffering enabled | Add `proxy_buffering off` to Nginx config |
| Slow first response | Model not in memory (cold start) | Set `OLLAMA_KEEP_ALIVE=24h` |

### Add Swap Space for Memory Relief

If you're running large models and hitting OOM errors:

```bash
# Create an 8 GB swap file
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make permanent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# Verify
free -h
swapon --show
```

---

## 16. 🎯 Real-World Use Cases

### Use Case 1: Private Code Review for Proprietary Projects

**Problem:** Your company prohibits sending source code to cloud AI providers (OpenAI, Claude, etc.) for IP reasons.

**Solution:** Use Cline + CodeLlama on your Ollama VM to review PRs and suggest refactors — all traffic stays on your LAN.

```mermaid
flowchart LR
    DEV["👩‍💻 Developer\npushed a branch"] --> IDE["VS Code\n+ Cline"]
    IDE -->|"Code diff sent locally"| OLLAMA["🦙 Ollama VM\n192.168.1.100"]
    OLLAMA -->|"Review response"| IDE
    IDE -->|"Inline suggestions"| DEV

    NOTE["✅ Zero data\nleaves your network\n\n❌ No OpenAI\nAPI calls"]
```

**Workflow:**
1. Pull the feature branch locally
2. Open file in VS Code with Cline active
3. Ask: *"Review this function for edge cases and performance issues"*
4. CodeLlama responds with specific, local feedback

---

### Use Case 2: Multi-Developer Team Sharing One AI Backend

**Problem:** 3 developers on a team each want AI assistance, but paying for 3 × cloud API subscriptions is expensive.

**Solution:** One shared Ollama VM on the office LAN, everyone connects their IDE.

```mermaid
graph TB
    subgraph SERVER["🖥️ Shared Ollama Server\n192.168.1.100:11434"]
        SCHED["Request Scheduler\nOLLAMA_NUM_PARALLEL=3"]
        subgraph POOL["Available Models"]
            LLAMA["llama3:8b — Chat"]
            CODE["codellama:13b — Code"]
            PHI["phi3:mini — Autocomplete"]
        end
        SCHED --> POOL
    end

    ALICE["👩 Alice\nContinue.dev"]   -->|llama3| SCHED
    BOB["👨 Bob\nCursor IDE"]         -->|codellama| SCHED
    CAROL["👩 Carol\nAider CLI"]      -->|phi3:mini| SCHED
```

**systemd config for multi-user:**
```ini
[Service]
Environment="OLLAMA_HOST=0.0.0.0:11434"
Environment="OLLAMA_NUM_PARALLEL=3"
Environment="OLLAMA_MAX_QUEUE=20"
Environment="OLLAMA_KEEP_ALIVE=8h"
```

**Monthly cost: $0** (vs. $60+/month for 3 cloud API subscriptions)

---

### Use Case 3: Fully Offline Development

**Problem:** You work in environments without internet access (airplane, secure facility, remote site).

**Solution:** Models are pre-downloaded to the VM. Start the VM on your laptop, set `OLLAMA_HOST` to `localhost`, and work with full AI assistance — completely offline.

```bash
# On your laptop (no internet needed):
# Start the VM (KVM example)
virsh start ollama-server

# All agent configs already point to localhost or VM IP
# — everything works exactly as in the office
```

---

### Use Case 4: Automated Code Quality in Local CI

Use Ollama's API in your local CI pipeline to automatically review changed files:

```python
#!/usr/bin/env python3
# ci_review.py — call from your pre-commit hook or local CI script

import requests
import sys
from pathlib import Path

OLLAMA_URL = "http://192.168.1.100:11434/v1/chat/completions"
MODEL = "codellama:13b"

def review_file(file_path: str) -> str:
    code = Path(file_path).read_text()

    response = requests.post(
        OLLAMA_URL,
        json={
            "model": MODEL,
            "messages": [
                {
                    "role": "system",
                    "content": (
                        "You are a senior code reviewer. "
                        "Find bugs, security issues, and style problems. "
                        "Be concise and specific. List issues with line numbers."
                    )
                },
                {
                    "role": "user",
                    "content": f"Review this code:\n\n```\n{code}\n```"
                }
            ],
            "stream": False
        },
        timeout=120
    )

    return response.json()["choices"][0]["message"]["content"]

if __name__ == "__main__":
    for file_arg in sys.argv[1:]:
        print(f"\n{'='*60}")
        print(f"📋 Review: {file_arg}")
        print('='*60)
        print(review_file(file_arg))
```

```bash
# Usage in a pre-commit hook or Makefile
python ci_review.py src/auth.py src/database.py

# Or review all changed files in a git diff
git diff --name-only HEAD | xargs python ci_review.py
```

---

### Use Case 5: Documentation Generation Pipeline

Automatically generate docstrings, README sections, and API docs for internal tools:

```bash
# Generate docstrings for a Python file
ollama run llama3 "$(cat <<'EOF'
Add Google-style docstrings to all functions and classes in this file.
Return only the updated code, no explanation.

$(cat src/utils.py)
EOF
)" > src/utils_documented.py

# Generate a README for a project
find . -name "*.py" -not -path "*/.*" | \
  xargs cat | \
  ollama run llama3 "Generate a comprehensive README.md for this Python project." \
  > README.md
```

---

## 🎓 Summary & Recommended Next Steps

### What You've Built

```mermaid
flowchart LR
    VM["✅ Ubuntu Server VM\nConfigured & Static IP"]
    OL["✅ Ollama Service\nInstalled & Network-exposed"]
    FW["✅ UFW Firewall\nPort 11434 open"]
    MODELS["✅ AI Models\nPulled & Ready"]
    AGENTS["✅ AI Agents Connected\nCline · Continue · Cursor · Aider"]
    SEC["✅ Optional: Nginx\nTLS + Basic Auth"]

    VM --> OL --> FW --> MODELS --> AGENTS --> SEC
```

| Milestone | Status |
|---|---|
| Ubuntu Server VM created and configured | ✅ |
| Ollama installed and listening on `0.0.0.0:11434` | ✅ |
| UFW firewall configured | ✅ |
| AI models pulled (llama3, codellama, phi3) | ✅ |
| AI coding agents connected | ✅ |
| Nginx TLS + auth (optional) | ✅ |

### Recommended Next Steps

1. **Enable GPU Passthrough** — Pass your NVIDIA GPU into the VM with VKVM for 10–50× faster inference
2. **Set up Open WebUI** — Browser-based ChatGPT-like interface for your Ollama backend
3. **Explore Embedding Models** — Pull `nomic-embed-text` and build RAG pipelines over your codebase
4. **Try Larger Models** — With more RAM or a GPU, upgrade to `llama3:70b` or `mixtral:8x7b`
5. **Add Model Monitoring** — Use Prometheus + Grafana to track request latency and throughput
6. **Explore Model Fine-tuning** — Customize a base model on your own code patterns with Ollama's Modelfile system

---

*Tutorial Version 1.0 · Built for Ubuntu Server 22.04 LTS · Ollama personal/team setup*
*Tested with: Cline, Continue.dev, Cursor, Aider · 2–3 concurrent agents · 16 GB RAM VM*