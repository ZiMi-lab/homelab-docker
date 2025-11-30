# Cloudflare Tunnel (cloudflared)

The stack includes configuration for deploying the **Cloudflare Tunnel (cloudflared)** connector. This allows for secure and encrypted access to local services in your homelab without needing to open ports on your router.

## 1\. Why Cloudflare Tunnel? (Zero Trust Access)

Cloudflare Tunnel is a **Zero Trust** solution for exposing services:

  * **Encrypted Connection:** The `cloudflared` connector establishes an **outbound encrypted connection** to the Cloudflare network.
  * **Security:** Your public IP address remains hidden. There is **no need to open ports** on the router (no NAT/Port Forwarding).
  * **HTTPS Security:** Cloudflare automatically provides HTTPS security for the exposed services.

-----

## 2\. Cloudflare Zero Trust Configuration

Before starting the container, you must obtain a **Tunnel Token** and define the target services (Public Hostnames) in the Cloudflare console.

### 2.1. Obtaining the Tunnel Token

1.  Log in to Cloudflare and navigate to **Zero Trust** $\rightarrow$ **Networks** $\rightarrow$ **Tunnels**.
2.  Click on **Create a tunnel**, select the type **Cloudflared**, and name it (e.g., `homelab-gateway`).
3.  Cloudflare will generate a startup command for you. Select the **Docker** option and **copy the unique token value** (the value of the `TUNNEL_TOKEN` variable).

### 2.2. Defining Public Hostnames

Once the container is running, the tunnel needs to know where to route traffic.

1.  In the settings for your newly created tunnel, go to the **Public Hostname** tab.
2.  Click on **Add a public hostname**.

| Field | Example Value | Description |
| :--- | :--- | :--- |
| **Subdomain** | `photo` | Subdomain for access (e.g., `photo.mydomain.com`). |
| **Domain** | `mydomain.com` | Your domain managed by Cloudflare. |
| **Service Type** | `HTTP` | The protocol of the local service. |
| **URL** | `192.168.10.50:9000` | The **Local IP address** (e.g., IP of the Proxmox VM with Docker) and **port** of the target service. |

> **Recommendation:** For accessing services running directly on the **Docker host** (e.g., Proxmox WebUI, if **cloudflared** is running on an LXC/VM), use `host.docker.internal:<port>` in the URL.

-----

## 3\. 📄 Docker Compose

This configuration uses an `.env` file to securely store the sensitive token.

### 3.1. The `.env` File

Create an `.env` file and replace the placeholder values with your data:

```env
# .env
# Time zone, e.g., Europe/Prague
TZ=Europe/Prague

# Unique token from Cloudflare Zero Trust console
TOKEN="UNIQUE_CLOUDPLARE_TOKEN_HERE"
```

### 3.2. The `docker-compose.yml` File

```yaml
# docker-compose.yml
version: '3.8'

services:
  cloudflared:
    image: cloudflare/cloudflared:latest
    container_name: cloudflared
    
    # Pull variables from the .env file
    environment:
      - TZ=${TZ}
      - TUNNEL_TOKEN=${TOKEN}
    
    # Restart unless explicitly stopped
    restart: unless-stopped
    
    # Command to run the tunnel without autoupdate
    command: tunnel --no-autoupdate run
    
    # Allows the container to access services on the host
    extra_hosts:
      - "host.docker.internal:host-gateway"
```

### 3.3. Starting the Service

Start the container using Docker Compose:

```bash
docker-compose up -d
```