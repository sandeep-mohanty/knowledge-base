# A Visual Guide to SSH Tunnels: Local and Remote Port Forwarding
---

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Local Port Forwarding](#local-port-forwarding)
3. [Remote Port Forwarding](#remote-port-forwarding)
4. [Summarizing](#summarizing)

---
SSH is yet another example of an ancient technology that is still in wide use today. It may very well be that learning a couple of SSH tricks is more profitable in the long run than mastering a dozen Cloud Native tools destined to become deprecated next quarter.

With nothing but standard tools and often using just a single command, you can achieve the following:

Access internal VPC endpoints through a public-facing EC2 instance.
Open a port from the localhost of a development VM in the host's browser.
Expose any local server from a home/private network to the outside world.
And more 😍

It always takes a while to figure out the right command. Should it be a Local or a Remote tunnel? What are the flags? Is it a local_port:remote_port or the other way around?

![SSH visual cheat sheet](https://iximiuz.com/ssh-tunnels/ssh-tunnels-2000-opt.png)
---
## Prerequisites

SSH Tunnels are about connecting hosts over the network, so every lab involves multiple "machines." To simplify setup, **containers** are used to simulate network hosts. A single Linux server with Docker Engine (or similar) can run all the labs.

**Note:** Running these examples with Docker Desktop isn't possible due to IP access limitations.

**Pro Tip:** Use ephemeral servers for running examples. Get one quickly at [labs.iximiuz.com/playgrounds/docker](#).

Every example requires a passphrase-less key pair mounted into containers for simplified access management. If you don't have one, the labs provide instructions for generating it.

> **Important:** SSH daemons in containers here are solely for educational purposes. Avoid using SSH in real-world containers!

---
## Local Port Forwarding

Starting with the most commonly used feature: **Local Port Forwarding**. This is useful when you need to access services on a private interface or `localhost` of a machine via its public IP.

### Typical Use Cases:
- Accessing databases (MySQL, Postgres, Redis, etc.) from your laptop.
- Using your browser to access web applications exposed only to private networks.
- Accessing container ports without publishing them on the server's public interface.

The command for local port forwarding is:

```bash
ssh -L [local_addr:]local_port:remote_addr:remote_port [user@]sshd_addr
```
- -L: Indicates local port forwarding.
- Behavior:
    - The SSH client listens on local_port (usually localhost, but depends on GatewayPorts).
    - Traffic to this port is forwarded to remote_addr:remote_port on the SSH server.

![How this looks in a diagram](https://iximiuz.com/ssh-tunnels/local-port-forwarding-2000-opt.png)

**Pro Tip:** Use ssh -f -N -L to run the session in the background.
---

## Local Port Forwarding with a Bastion Host

The `ssh -L` command allows forwarding a local port to any remote machine, not just the SSH server itself. This is often referred to as using a bastion host .

```bash
ssh -L [local_addr:]local_port:remote_addr:remote_port [user@]sshd_addr
```
![How this looks in a diagram](https://iximiuz.com/ssh-tunnels/local-port-forwarding-bastion-2000-opt.png)

## Remote Port Forwarding

In contrast to local forwarding, Remote Port Forwarding is used to expose a local service to the outside world. You'll need a public-facing ingress gateway server.

**The command is:**

```bash
ssh -R [remote_addr:]remote_port:local_addr:local_port [user@]gateway_addr
```
- `-R:` Indicates remote port forwarding.
- Pitfall:
    - By default, the tunnel allows access only via the gateway's `localhost`.
    - To expose the service publicly, configure the SSH server with GatewayPorts `yes`.

![how this looks like in a diagram](https://iximiuz.com/ssh-tunnels/remote-port-forwarding-2000-opt.png)

### Use Cases:
- Exposing a development service from your laptop to the public Internet for demos.

**Pro Tip:** Use ssh -f -N -R to run the port-forwarding session in the background.

## Remote Port Forwarding from a Home/Private Network

Remote port forwarding also supports exposing devices within a home/private network through an ingress gateway.
```bash
ssh -R [remote_addr:]remote_port:local_addr:local_port [user@]gateway_addr
```

![how this looks like in a diagram](https://iximiuz.com/ssh-tunnels/remote-port-forwarding-home-network-2000-opt.png)

### Summarizing
From the drawings, it is observed that:

- The word `local` can mean either the SSH client machine or an upstream host accessible from this machine.
- The word `remote` can mean either the SSH server machine (sshd) of an upstream host accessible from it.
- Local port forwarding (`ssh -L`) implies it's the ssh client that starts listening on a new port.
- Remote port forwarding (`ssh -R`) implies it's the sshd server that starts listening on an extra port.
- The mnemonics are `ssh -L local:remote` and `ssh -R remote:local` and it's always the left-hand side that opens a new port.