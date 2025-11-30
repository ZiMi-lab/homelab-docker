## Přehled projektu

Repozitář slouží jako centrální úložiště pro správu služeb běžících v mém **Homelabu**. Cílem je implementace principů **Infrastructure as Code (IaC)** pro snadnou replikaci, správu a verzování služeb.

## 🔒 Správa tajných dat (Secrets)

**Repozitář je veřejný a neobsahuje skutečné přihlašovací údaje, API klíče ani hesla.**

* **Šablony:** Pro každou službu je k dispozici soubor **`.env.example`**, který slouží jako šablona. Obsahuje názvy všech potřebných proměnných, které je nutné ručně doplnit a uložit do lokálního souboru `.env`.

> **POZOR:** Při nasazení jakékoliv služby je nutné vytvořit lokální soubor `.env` a doplnit do něj silná, unikátní hesla.

## 📂 Struktura repozitáře

Každá služba je umístěna ve vlastním adresáři.

```text
homelab-docker/
├── .gitignore               # Pravidla pro ignorování .env souborů.
├── README.md                # Tento dokument.
├── traefik/                 # Reverse Proxy & Load Balancer
│   ├── docker-compose.yml
│   └── .env.example
├── cloudflared/             # Cloudflare Tunnel
│   ├── docker-compose.yml
│   └── .env.example
└── immmich/                 # Galerie fotografií
    ├── docker-compose.yml
    └── .env.example
````

-----

## 🚀 Průvodce nasazením

Pro nasazení nového Docker stacku na cílovém serveru (typicky Proxmox LXC nebo VM):

### 1\. Klonování repozitáře

```bash
# Na cílovém serveru (VM/LXC s Dockerem)
git clone [https://github.com/ZiMi-lab/homelab-docker.git](https://github.com/ZiMi-lab/homelab-docker.git)
cd homelab-docker
```

### 2\. Konfigurace služby

Přejdi do adresáře konkrétní služby (např. `immich`).

```bash
cd immich
# Zkopírování šablony
cp .env.example .env

# Úprava a vložení skutečných hesel a domén
nano .env 
```

### 3\. Spuštění stacku

Spuštění služby na pozadí:

```bash
docker compose up -d
```

## 📝 Licence

Tento projekt je zveřejněn pod licencí **MIT** (TBD) pro volné použití, modifikace a sdílení v rámci komunity.

```