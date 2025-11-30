## Cloudflare Tunnel (cloudflared)

Stack obsahuje konfiguraci pro nasazení **Cloudflare Tunnel (cloudflared)** konektoru. Ten umožňuje bezpečný a šifrovaný přístup k lokálním službám v homelabu bez nutnosti otevírat porty na routeru.

### 1\. Proč Cloudflare Tunnel? (Zero Trust Access)

Cloudflare Tunnel je **Zero Trust** řešení pro vystavení služeb:

  * **Šifrované Spojení:** Konektor `cloudflared` navazuje **šifrované spojení směrem ven** (outbound) k Cloudflare síti.
  * **Bezpečnost:** Veřejná IP adresa zůstává skrytá. Na routeru **není potřeba otevírat porty** (žádné NAT/Port Forwarding).
  * **HTTPS Zabezpečení:** Cloudflare automaticky zajišťuje HTTPS zabezpečení pro vystavené služby.

### 2\. Konfigurace Cloudflare Zero Trust

Před spuštěním kontejneru je nutné získat **Tunnel Token** a definovat cílové služby (Public Hostnames) v Cloudflare konzoli.

#### 2.1. Získání Tunnel Tokenu

1.  Přihlas se do Cloudflare a přejdi do **Zero Trust** $\rightarrow$ **Networks** $\rightarrow$ **Tunnels**.
2.  Klikni na **Create a tunnel**, zvol typ **Cloudflared** a pojmenuj jej (např. `homelab-gateway`).
3.  Cloudflare ti vygeneruje příkaz pro spuštění. Zvol možnost **Docker** a **zkopíruj unikátní hodnotu tokenu** (hodnota proměnné `TUNNEL_TOKEN`).

#### 2.2. Definice Public Hostnames

Jakmile je kontejner spuštěn, tunel potřebuje vědět, kam má provoz směrovat.

1.  V nastavení tvého nově vytvořeného tunelu přejdi na záložku **Public Hostname**.
2.  Klikni na **Add a public hostname**.

| Pole | Příklad Hodnoty | Popis |
| :--- | :--- | :--- |
| **Subdomain** | `photo` | Subdoména pro přístup (např. `photo.mojedomena.cz`). |
| **Domain** | `mojedomena.cz` | Tvá doména spravovaná Cloudflare. |
| **Service Type** | `HTTP` | Protokol lokální služby. |
| **URL** | `192.168.10.50:9000` | **Lokální IP adresa** (např. IP Proxmox VM s Dockerem) a **port** cílové služby. |

> **Doporučení:** Pro přístup ke službám běžícím přímo na Docker hostiteli (např. Proxmox WebUI, pokud **cloudflared** běží na LXC/VM) použij v URL `host.docker.internal:<port>`.

### 3\. 📄 Docker Compose

Tato konfigurace používá `.env` soubor pro bezpečné uložení citlivého tokenu.

#### 3.1. Soubor `.env`

Vytvoř soubor `.env` a nahraď zástupné hodnoty tvými daty:

```env
# .env
# Casova zona, napr. Europe/Prague
TZ=Europe/Prague

# Unikatni token z Cloudflare Zero Trust konzole
TOKEN="UNIKATNI_CLOUDPLARE_TOKEN_ZDE"
```

#### 3.2. Soubor `docker-compose.yml`

```yaml
# docker-compose.yml
version: '3.8'

services:
  cloudflared:
    image: cloudflare/cloudflared:latest
    container_name: cloudflared
    
    # Natazeni promennych z .env souboru
    environment:
      - TZ=${TZ}
      - TUNNEL_TOKEN=${TOKEN}
    
    # Restart, pokud neni explicitne zastaven
    restart: unless-stopped
    
    # Prikaz pro spusteni tunelu bez autoupdate
    command: tunnel --no-autoupdate run
    
    # Umozni kontejneru pristupovat ke sluzbam na hostiteli
    extra_hosts:
      - "host.docker.internal:host-gateway"
```

#### 3.3. Spuštění Služby

Spusť kontejner pomocí Docker Compose:

```bash
docker-compose up -d
```
