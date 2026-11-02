# HomeLab Server: Ubuntu + VirtualBox + Docker + Coolify + Tailscale

A complete guide to building a self-hosted home lab using Ubuntu Server, VirtualBox, Docker, Coolify, and Tailscale Funnel.

This project documents the setup of a home server that can host websites and applications and make them accessible over the internet without directly exposing the home network.

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Requirements](#2-requirements)
3. [VirtualBox Installation](#3-virtualbox-installation)
4. [Ubuntu Server Installation](#4-ubuntu-server-installation)
5. [VirtualBox Configuration](#5-virtualbox-configuration)
6. [Linux Basics and Initial Setup](#6-linux-basics-and-initial-setup)
7. [Docker Installation](#7-docker-installation)
8. [Coolify Installation](#8-coolify-installation)
9. [Deploying a Website](#9-deploying-a-website)
10. [Tailscale Installation](#10-tailscale-installation)
11. [Tailscale Funnel Setup](#11-tailscale-funnel-setup)
12. [Custom Domain Configuration](#12-custom-domain-configuration)
13. [Starting the Server After a Reboot](#13-starting-the-server-after-a-reboot)
14. [Troubleshooting](#14-troubleshooting)
15. [Useful Linux and Docker Commands](#15-useful-linux-and-docker-commands)
16. [Backup and Security](#16-backup-and-security)
17. [Official Documentation](#17-official-documentation)

---

# 1. Project Overview

### Architecture

Windows Host
↓
Oracle VirtualBox
↓
Ubuntu Server 26.04.1 LTS
↓
Docker Engine
↓
Coolify
↓
Website / Web Application
↓
Tailscale Funnel
↓
Public Internet

### Technologies Used

| Technology | Purpose |
|---|---|
| Windows | Host operating system |
| VirtualBox | Virtual machine |
| Ubuntu Server | Linux server OS |
| Docker | Container management |
| Coolify | Self-hosted deployment platform |
| Traefik | Reverse proxy managed through Coolify |
| Tailscale | Private networking and secure connectivity |
| Tailscale Funnel | Public HTTPS access to services |

---

# 2. Requirements

### Hardware

Recommended starting configuration:

- 64-bit Windows computer
- 8 GB RAM or more
- 20 GB or more free disk space
- CPU with hardware virtualization support
- Stable internet connection

### Software

- Oracle VirtualBox
- Ubuntu Server 26.04.1 LTS ISO
- Docker Engine
- Coolify
- Tailscale account

---

# 3. VirtualBox Installation

Download VirtualBox from:

https://www.virtualbox.org/wiki/Downloads

Install VirtualBox on Windows using the default installation options.

Restart Windows if the installer requests it.

---

# 4. Ubuntu Server Installation

Download Ubuntu Server ISO:

https://ubuntu.com/download/server

Use the 64-bit AMD64 version.

### Create a Virtual Machine

1. Open VirtualBox.
2. Click **New**.
3. Name: `Ubuntu server`
4. Type: Linux
5. Version: Ubuntu (64-bit)
6. Select the downloaded Ubuntu Server ISO.
7. Create a virtual hard disk.

### Recommended VM resources

| Setting | Value |
|---|---|
| RAM | 4096 MB (4 GB) |
| CPUs | 2 |
| Disk | 40 GB or more |
| Network | Bridged Adapter |

If your host has 8 GB RAM, avoid allocating all of it to Ubuntu. Windows also needs memory.

### Ubuntu installation steps

1. Start the VM.
2. Select the Ubuntu Server installation option.
3. Choose language and keyboard layout.
4. Configure network.
5. Select storage and use the default guided installation.
6. Set hostname:

   `homelab-server`

7. Create your user account.
8. Enable OpenSSH Server if you want SSH access.
9. Complete the installation.
10. Reboot and remove the installation ISO if prompted.

After reboot, log in using your username and password.

---

# 5. VirtualBox Configuration

These settings are important for avoiding VM performance and CPU stability problems.

Power off the VM before changing settings.

### System → Motherboard

- Base Memory: 4096 MB
- Boot Order: Hard Disk first

### System → Processor

- Number of CPUs: 2
- Execution Cap: 100%

### System → Acceleration

If the options are available:

- Paravirtualization Interface: KVM
- Nested Paging: Enabled

Some options are automatically managed by newer VirtualBox versions.

### Display

- Video Memory: 64 MB
- 3D Acceleration: Disabled

### Network

- Enable Network Adapter: Checked
- Attached to: Bridged Adapter
- Name: Your physical Wi-Fi or Ethernet adapter

Bridged networking allows Ubuntu to receive an IP address from your home router.

---

# 6. Linux Basics and Initial Setup

After logging in, you will see a terminal prompt similar to:

`username@homelab-server:~$`

### Update Ubuntu

```bash
sudo apt update
sudo apt upgrade -y
```

### Install useful utilities

```bash
sudo apt install -y curl wget git nano htop unzip ca-certificates
```

### Check Ubuntu version

```bash
lsb_release -a
```

### Check IP address

```bash
ip addr
```

Look for the IPv4 address of the network interface, usually `enp0s3`.

Example:

`10.150.241.103`

Your IP may change after reboot if your router uses DHCP.

### Check system resources

```bash
free -h
```

```bash
df -h
```

```bash
nproc
```

### Check internet connectivity

```bash
ping -c 4 google.com
```

### Check network routes

```bash
ip route
```

---

# 7. Docker Installation

Official documentation:

https://docs.docker.com/engine/install/ubuntu/

### Step 1: Install dependencies

```bash
sudo apt update
sudo apt install -y ca-certificates curl
```

### Step 2: Add Docker's official GPG key

```bash
sudo install -m 0755 -d /etc/apt/keyrings
```

```bash
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
-o /etc/apt/keyrings/docker.asc
```

```bash
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

### Step 3: Add Docker repository

```bash
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF
```

### Step 4: Install Docker Engine

```bash
sudo apt update
```

```bash
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Step 5: Start Docker

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

### Step 6: Verify installation

```bash
sudo systemctl status docker --no-pager
```

```bash
sudo docker run hello-world
```

Expected result: Docker prints a successful test message.

### Important

On this server, use `sudo docker ...` if your user is not configured for Docker access.

Docker access is effectively root-level access. Do not add users to the Docker group unless you understand this security implication.

---

# 8. Coolify Installation

Official documentation:

https://coolify.io/docs/start-with-self-hosted

### Step 1: Install Coolify

Run:

```bash
curl -fsSL https://cdn.coollabs.io/coolify/install.sh | sudo bash
```

Wait until the installation completes.

### Step 2: Access Coolify

Open your Windows browser:

`http://YOUR_UBUNTU_IP:8000`

Example:

`http://10.150.241.103:8000`

Create your Coolify administrator account immediately.

### Step 3: Verify containers

```bash
sudo docker ps
```

You should see Coolify-related containers such as:

- coolify
- coolify-realtime
- coolify-db
- coolify-redis

The proxy container may appear after the first deployment.

### Step 4: Check Coolify status

```bash
sudo docker ps -a
```

### Step 5: Verify the proxy

Inside Coolify:

1. Open Servers.
2. Select your server.
3. Check the Proxy section.
4. Start the proxy if it is not running.

Coolify generally manages Traefik as its reverse proxy.

---

# 9. Deploying a Website

Official guide:

https://coolify.io/docs/deploy-your-first-app

### Step 1: Connect GitHub

1. Open Coolify.
2. Navigate to Sources.
3. Connect your GitHub account or repository.

### Step 2: Create a project

1. Open Projects.
2. Create a project.
3. Add a resource.
4. Choose the appropriate application deployment type.

### Step 3: Configure application

For a Git-based application:

- Select your repository.
- Select the branch.
- Configure the build method.
- Set the required environment variables.
- Configure the application's internal port.

For example:

| Application | Typical port |
|---|---|
| Nginx | 80 |
| Next.js | 3000 |
| Node.js | 3000 |
| Flask | 5000 |
| FastAPI | 8000 |

The port must match the actual port on which your application listens.

### Step 4: Configure domain

Enter the application domain in the Coolify domain field.

Example:

`https://your-app.example.com`

### Step 5: Deploy

Click Deploy and wait for the build to finish.

### Step 6: Verify

Check the resource status and logs in Coolify.

Once the application is healthy, test its domain.

---

# 10. Tailscale Installation

Official documentation:

https://tailscale.com/docs/install/linux

### Step 1: Install Tailscale

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

### Step 2: Start Tailscale

```bash
sudo systemctl enable --now tailscaled
```

### Step 3: Authenticate

```bash
sudo tailscale up
```

The terminal will provide an authentication URL.

Open it in your Windows browser and sign in to your Tailscale account.

### Step 4: Verify

```bash
tailscale status
```

```bash
tailscale ip -4
```

Your server should appear in your Tailscale network.

---

# 11. Tailscale Funnel Setup

Official documentation:

https://tailscale.com/kb/1223/funnel/

Tailscale Funnel makes a service accessible from the public internet through a Tailscale HTTPS domain.

### Step 1: Enable Funnel

If Funnel is not enabled for your tailnet, follow the authorization link shown in the terminal.

### Step 2: Expose the website

For an application routed through Coolify's HTTP proxy on port 80:

```bash
sudo tailscale funnel --bg 80
```

### Step 3: Check Funnel status

```bash
tailscale funnel status
```

### Step 4: Test locally

```bash
curl -I http://127.0.0.1:80
```

A response such as `HTTP/1.1 200`, `301`, or `302` indicates the local HTTP service is responding.

### Step 5: Open the public URL

Tailscale will provide a URL similar to:

`https://your-machine.your-tailnet.ts.net`

Open that HTTPS URL from your Windows browser or another device.

### Important

Funnel serves the service configured on the local port. If your application requires host-based routing, the correct domain must also be configured in Coolify.

---

# 12. Custom Domain Configuration

There are two separate approaches.

## Option A: Coolify Custom Domain

If you have a public IP address:

1. Open your domain registrar's DNS settings.
2. Create an A record pointing to your public server IP.
3. Configure the domain in Coolify.
4. Ensure ports 80 and 443 are reachable.
5. Let Coolify/Traefik obtain an HTTPS certificate.

Do not point an A record to `10.x.x.x` for public internet access. That is a private LAN IP.

## Option B: Tailscale Funnel Domain

Funnel provides a public HTTPS hostname ending in:

`.ts.net`

Use the URL displayed by:

```bash
tailscale funnel status
```

Your custom domain and your Tailscale Funnel domain are different routing methods unless you explicitly configure DNS and routing between them.

---

# 13. Starting the Server After a Reboot

This is the normal startup procedure after shutting down Windows or Ubuntu.

### Step 1: Start VirtualBox

Open VirtualBox on Windows and start the Ubuntu Server VM.

### Step 2: Log in

Enter your Ubuntu username and password.

### Step 3: Check Docker

```bash
sudo systemctl status docker --no-pager
```

If Docker is inactive:

```bash
sudo systemctl start docker
```

### Step 4: Check containers

```bash
sudo docker ps
```

### Step 5: Check Tailscale

```bash
sudo systemctl status tailscaled --no-pager
```

If required:

```bash
sudo systemctl start tailscaled
```

### Step 6: Check Funnel

```bash
tailscale funnel status
```

### Step 7: Test website

Open your Tailscale Funnel URL in the browser.

Docker, Coolify and Tailscale services should start automatically if configured correctly. If Funnel is not active after reboot, re-run:

```bash
sudo tailscale funnel --bg 80
```

---

# 14. Troubleshooting

## A. Ubuntu CPU Soft Lockup

Symptoms:

```text
watchdog: BUG: soft lockup - CPU#0 stuck for 30s
```

This means the virtual CPU is not responding to scheduled kernel work in time.

### VirtualBox settings to check

- CPU: 2
- RAM: 4096 MB
- Execution Cap: 100%
- Paravirtualization: KVM, if available
- Nested Paging: Enabled, if available
- 3D Acceleration: Disabled

Also verify that hardware virtualization (Intel VT-x / AMD-V) is enabled in the host computer's BIOS/UEFI.

### Collect logs

```bash
sudo journalctl -k -b | grep -iE 'soft lockup|watchdog|clocksource|stall' | tail -50
```

If the issue persists, update VirtualBox on Windows and check host CPU virtualization settings before reinstalling Ubuntu or Docker.

---

## B. Docker Is Inactive

Check:

```bash
sudo systemctl status docker --no-pager
```

Start:

```bash
sudo systemctl start docker
```

Enable at boot:

```bash
sudo systemctl enable docker
```

Verify:

```bash
docker ps
```

---

## C. Coolify Dashboard Does Not Open

Check:

```bash
sudo docker ps -a
```

Check port 8000:

```bash
sudo ss -tulpn | grep :8000
```

Check Coolify logs:

```bash
sudo docker logs coolify --tail 100
```

Check Docker service:

```bash
sudo systemctl status docker --no-pager
```

Coolify dashboard:

`http://YOUR_UBUNTU_IP:8000`

---

## D. Website Does Not Open

### Check all containers

```bash
sudo docker ps -a
```

### Check application logs

Replace `YOUR_CONTAINER_NAME` with the actual application container name.

```bash
sudo docker logs YOUR_CONTAINER_NAME --tail 100
```

### Check local HTTP service

```bash
curl -I http://127.0.0.1:80
```

### Check Funnel

```bash
tailscale funnel status
```

### Check proxy

```bash
sudo docker ps --filter "name=coolify-proxy"
```

Confirm the proxy is running and the application's domain and internal port are correctly configured in Coolify.

---

## E. Tailscale Funnel Is Not Running

Check:

```bash
tailscale status
```

```bash
tailscale funnel status
```

Re-enable the HTTP service:

```bash
sudo tailscale funnel --bg 80
```

If the terminal says Funnel is not enabled for your tailnet, open the authorization URL it provides and enable Funnel.

---

# 15. Useful Linux and Docker Commands

### Linux

```bash
pwd
```

Show current directory.

```bash
ls -la
```

List files.

```bash
cd /path/to/directory
```

Change directory.

```bash
nano filename.txt
```

Edit a file.

```bash
sudo reboot
```

Restart Ubuntu.

```bash
sudo poweroff
```

Shut down Ubuntu safely.

### Docker

```bash
docker ps
```

Running containers.

```bash
docker ps -a
```

All containers.

```bash
docker images
```

List images.

```bash
docker logs CONTAINER_NAME
```

View logs.

```bash
docker logs -f CONTAINER_NAME
```

Follow logs live.

```bash
docker stats
```

Monitor resource usage.

```bash
docker inspect CONTAINER_NAME
```

Inspect container configuration.

### Services

```bash
sudo systemctl status docker
```

```bash
sudo systemctl restart docker
```

Use restart only when necessary.

---

# 16. Backup and Security

## Coolify Backup

Important file:

`/data/coolify/source/.env`

This file contains installation secrets. Back it up securely.

Never commit this file to GitHub.

## Database Backup

Use Coolify's database backup functionality for PostgreSQL, Redis or other deployed databases.

## Security Checklist

- Use strong Ubuntu and Coolify passwords.
- Do not publish credentials in GitHub.
- Never publish Tailscale authentication keys or tokens.
- Keep Ubuntu and Docker updated.
- Avoid exposing PostgreSQL and Redis directly to the public internet.
- Use HTTPS for public websites.
- Back up databases and application configuration.
- Restrict access to the Coolify dashboard.
- Keep `.env` files out of Git.

---

# 17. Official Documentation

| Resource | Link |
|---|---|
| Ubuntu Server Download | https://ubuntu.com/download/server |
| Ubuntu Server Docs | https://ubuntu.com/server/docs |
| VirtualBox Downloads | https://www.virtualbox.org/wiki/Downloads |
| Docker Installation | https://docs.docker.com/engine/install/ubuntu/ |
| Docker Documentation | https://docs.docker.com/ |
| Coolify Self-hosted Installation | https://coolify.io/docs/start-with-self-hosted |
| Coolify First App Deployment | https://coolify.io/docs/deploy-your-first-app |
| Coolify Troubleshooting | https://coolify.io/docs/troubleshoot |
| Tailscale Linux Installation | https://tailscale.com/docs/install/linux |
| Tailscale Funnel | https://tailscale.com/kb/1223/funnel/ |
| Git Documentation | https://git-scm.com/doc |

---

# Final Result

The final home lab setup provides:

- Ubuntu Server running inside VirtualBox
- Docker Engine for container management
- Coolify for deploying and managing applications
- Traefik reverse proxy for web routing
- Tailscale for private connectivity
- Tailscale Funnel for public HTTPS access
- GitHub integration for application deployments

This setup can be extended later with PostgreSQL, Redis, monitoring, automated backups, CI/CD, and additional self-hosted services.

---

**Author:** Nashit  
**Project:** HomeLab Server  
**Environment:** Windows + Oracle VirtualBox + Ubuntu Server
