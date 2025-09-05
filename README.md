# dolibarr-docker-compose-windows 🚀

A **Docker setup dedicated to development and maintenance of Dolibarr core and modules**, optimized for **Windows with Docker Desktop (WSL2, x86_64 / amd64)**, using **Nginx + PHP-FPM + MariaDB + Traefik**.  
Lightweight, performant, and ready for core and module development.

> ⚠️ **IMAP is disabled by default** to prevent crashes in Dolibarr.

---

## Why this stack? 💡

### 1️⃣ No phpMyAdmin
We decided **not to include phpMyAdmin** to keep the stack lightweight and development-focused.  
- Connect directly to the database using any MySQL client.  
- For Windows, we recommend **[HeidiSQL](https://www.heidisql.com/)** or **DBeaver** – free, lightweight, and efficient.  
- Reduces unnecessary exposure of database management interfaces.

### 2️⃣ Nginx vs Apache
Nginx is chosen over Apache for several reasons:  
- **Performance & memory efficiency**  
- **Docker-friendly** (small containers)  
- **Seamless PHP-FPM integration**  
- **Simplified SSL management via Traefik**

### 3️⃣ Why Traefik
- Automatic SSL via Let’s Encrypt  
- Dynamic routing to containers without manual Nginx tweaks  
- Dashboard to monitor and manage services  

### 4️⃣ Windows Compatibility
- Optimized for **x86_64 / amd64 architecture**  
- Runs inside **Docker Desktop with WSL2**  
- Relative paths (`../`) work fine with WSL2; absolute `C:/...` paths can be used if needed  

### 5️⃣ IMAP Warning
- **IMAP not installed by default**  
- Enabling IMAP may cause instability → avoid unless required  

---

## PHP-FPM – Engine Highlights ⚡

- **Base image:** `php:8.4-fpm`  
- **Extensions installed:** `mysqli`, `pdo_mysql`, `zip`, `intl`, `calendar`, `gd`, `opcache`  
- **OPcache config:**  
  - 128 MB memory  
  - 4000 max files  
  - 2s revalidate  
  - CLI enabled  
- **Volumes:**  
  - Dolibarr core (read/write)  
  - Custom plugins  
  - Configuration files  
- **Role:** Executes Dolibarr PHP code for Nginx  

---

## Dockerfile Summary 🐳

- **Base:** `php:8.4-fpm` (Debian Linux inside Docker)  
- **PHP extensions:** MySQL, Zip, Intl, Calendar, GD, OPCache  
- **System libraries:** `zip`, `unzip`, `curl`, `bash`, `libzip-dev`, `libonig-dev`, `libicu-dev`, `libpng-dev`, `libjpeg-dev`, `libfreetype6-dev`  
- **GD compiled with JPEG & FreeType**  
- **Workdir:** `/var/www/html`  
- **Expose:** `9000` (PHP-FPM)  

---

## Nginx – Web Server

- **Image:** `nginx:stable`  
- **Serves Dolibarr at `/`**  
- **Static files cached 30 days**  
- **Blocks `.ht*` files**  
- **FastCGI → PHP-FPM**  
- **Traefik labels for HTTPS**  

---

## Traefik – Reverse Proxy

- **Image:** `traefik:latest`  
- **Ports:**  
  - `443` → HTTPS  
  - `8080` → Dashboard  
- **Features:**  
  - Let’s Encrypt SSL (`myresolver`)  
  - Dynamic container routing  
  - Dashboard: [http://localhost:8080](http://localhost:8080)  

---

## MariaDB – Database

- **Image:** `mariadb:latest`  
- **Port:** 3306  
- **Env:**  
  - Root: `rootPASS`  
  - DB: `dolibarr`  
  - User: `dbuser`  
  - Password: `dbpass`  
- **Storage:** `./dbdata` (mounted on Windows folder)  

---

## Docker Network 🌐

All services share `traefikNetwork`, enabling:  
- Nginx ↔ PHP-FPM  
- PHP-FPM ↔ MariaDB  
- Traefik ↔ Nginx (HTTPS routing)  

---

## Accessing Services 🔑

| Service        | URL / Port                  | Notes |
|----------------|----------------------------|-------|
| Dolibarr       | `https://localhost/`        | via Traefik |
| Traefik dash   | `http://localhost:8080`     | Dashboard |
| MariaDB        | `localhost:3306`            | Connect with HeidiSQL / DBeaver |
| PHP-FPM        | internal only              | Not exposed |

---

## Quick Start ⚡

1. Launch the stack:

```powershell
docker compose up -d
