# 🚀 Deploy N8N with Docker Compose 🚀

> 🚀 AI Automation Tool | How To Host n8n For Free FOREVER 🚀

## 🤖 What is n8n?

**n8n** (pronounced "n-eight-n") is a powerful, **free and open-source** workflow automation tool that helps you connect different apps and services together. Think of it as a self-hosted alternative to tools like [Zapier](https://zapier.com/) or [Make](https://www.make.com/en) (formerly Integgy), but with:

- ✅ **No monthly fees** - Host it yourself for free forever
- ✅ **Unlimited workflows** - Create as many automations as you need
- ✅ **300+ integrations** - Connect to almost any service
- ✅ **Full control** - Your data stays on your server
- ✅ **Visual workflow builder** - Easy drag-and-drop interface

---

## 📋 Prerequisites & Infrastructure Checklist

Before you begin, make sure you have the following installed on your system:

| Requirement | Version | Check Command |
|------------|---------|---------------|
| 🐳 Docker | 20.10+ | `docker --version` |
| 🔧 Docker Compose | 2.0+ | `docker-compose --version` |
| 💻 Operating System | Linux/macOS/Windows | - |

---

## 🚀 Quick Start Guide
Deploy n8n in simple steps:
### 📂 Clone & Configure Repository
```bash
git clone https://github.com/meibraransari/N8N-Deploy-Using-Docker-Compose.git
cd N8N-Deploy-Using-Docker-Compose
cp -a env.sample .env && nano .env # Update the environment variables
docker compose up -d
docker compose logs -f
# Fix permission
Error: EACCES: permission denied, open '/home/node/.n8n/config'
docker compose down
# set permissions to path in env file like "DOCKER_VOLUME_STORAGE=/mnt/docker-volumes"
chmod -R 777 /mnt/docker-volumes/n8n/
docker compose up -d
docker compose logs -f
```

## 🌐 Network: DNS & SSL Configuration
### 🔗 Local Network Access
```
http://192.168.1.100:5678
```
### 📡 Technitium DNS Configuration
```
https://dns.devopsinaction.lab/
Point domain to IP address
192.168.1.222 n8n.devopsinaction.lab
```

### 🛡️ Nginx Proxy Manager Setup
```
https://npm.devopsinaction.lab/
Domain Names n8n.devopsinaction.lab
Forward Hostname/IP: 192.168.1.100
Forward Port: 5678
SSL Certificate: 
Force SSL: True
```

### ⚙️ Manual Nginx Configuration (Optional) if you have your own nginx server.
```bash
server {
    listen 80;
    server_name n8n.devopsinaction.lab;
    return 301 https://$host$request_uri;
    if ($host = n8n.devopsinaction.lab) {
        return 301 https://$host$request_uri;
    } 
}
# HTTPS reverse proxy (Dont forget to add ssl file | letsencrypt | certbot)
server {
    listen 443 ssl;
    server_name n8n.devopsinaction.lab;
    ssl_certificate /etc/letsencrypt/live/n8n.devopsinaction.lab/fullchain.pem; 
    ssl_certificate_key /etc/letsencrypt/live/n8n.devopsinaction.lab/privkey.pem; 
    # Optional: secure SSL headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    # Global proxy timeouts (10 minutes)
    proxy_connect_timeout 600s;
    proxy_send_timeout    600s;
    proxy_read_timeout    600s;
    send_timeout          600s;
    location / {
        proxy_pass http://127.0.0.1:5678;
        proxy_http_version 1.1;
        proxy_hide_header X-Powered-By;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    location = /robots.txt {
        allow all;
        log_not_found off;
        access_log off;
    }
}
```

## 🔓 Accessing your n8n Instance
https://n8n.devopsinaction.lab/

> Fill form to create account
```
Email *
First Name *
Last Name *
Password *
```
### 💡 Implementation Blueprints
- Very quick quickstart
- First AI agent

## 📚 Resources & Documentation
### 📖 Official n8n Documentation
- https://docs.n8n.io/hosting/
- https://docs.n8n.io/hosting/configuration/environment-variables/
- https://github.com/n8n-io
- https://github.com/n8n-io/n8n
- https://n8n.io/workflows/
- https://n8n.io/sitemap-workflows.xml

### 🗂️ Community Workflow Repositories
- https://github.com/Zie619/n8n-workflows
- https://github.com/topics/n8n-workflows



---
# ⚙️ Configuration file used in this project

### 📝 docker-compose.yml
```yaml
services:
  n8n-db:
    image: postgres:${POSTGRES_IMAGE_TAG:-16-alpine}
    container_name: n8n-db
    hostname: n8n-db
    restart: unless-stopped
    ports:
      - "${DB_POSTGRESDB_PORT:-5432}:${DB_POSTGRESDB_PORT:-5432}"
    environment:
      - POSTGRES_USER=${DB_POSTGRESDB_USER}
      - POSTGRES_PASSWORD=${DB_POSTGRESDB_PASSWORD}
      - POSTGRES_DB=${DB_POSTGRESDB_DATABASE}
      - POSTGRES_NON_ROOT_USER=${POSTGRES_NON_ROOT_USER}
      - POSTGRES_NON_ROOT_PASSWORD=${POSTGRES_NON_ROOT_PASSWORD}
    volumes:
      - ${DOCKER_VOLUME_STORAGE:-/mnt/docker-volumes}/n8n/database:/var/lib/postgresql/data
      #- ${DOCKER_VOLUME_STORAGE:-/mnt/docker-volumes}/n8n/init-database.sh:/docker-entrypoint-initdb.d/init-data.sh:ro
    healthcheck:
      test: ['CMD-SHELL', 'pg_isready -h localhost -U ${DB_POSTGRESDB_USER} -d ${DB_POSTGRESDB_DATABASE}']
      interval: ${HEALTHCHECK_INTERVAL:-5s}
      timeout: ${HEALTHCHECK_TIMEOUT:-5s}
      retries: ${HEALTHCHECK_RETRIES:-10}

  n8n:
    image: n8nio/n8n:${N8N_IMAGE_TAG:-latest}
    container_name: n8n
    hostname: n8n
    restart: unless-stopped
    ports:
      - "${N8N_PORT:-5678}:${N8N_PORT:-5678}"
    links:
      - n8n-db
    depends_on:
      n8n-db:
        condition: service_healthy
    environment:
      # N8N Settings
      - N8N_HOST=${N8N_HOST}
      - N8N_PROTOCOL=${N8N_PROTOCOL}
      - N8N_URL=${N8N_URL}
      - N8N_PORT=${N8N_PORT}
      - N8N_EDITOR_BASE_URL=${N8N_EDITOR_BASE_URL}
      - N8N_WEBHOOK_URL=${N8N_WEBHOOK_URL}
      - WEBHOOK_URL=${N8N_WEBHOOK_URL}
      - N8N_TRUSTED_PROXIES=${N8N_TRUSTED_PROXIES}
      - N8N_PROXY_HOPS=${N8N_PROXY_HOPS}
      - N8N_BASIC_AUTH_ACTIVE=${N8N_BASIC_AUTH_ACTIVE}
      - N8N_BASIC_AUTH_USER=${N8N_BASIC_AUTH_USER}
      - N8N_BASIC_AUTH_PASSWORD=${N8N_BASIC_AUTH_PASSWORD}
      - N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=${N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS}
      - N8N_SECURE_COOKIE=${N8N_SECURE_COOKIE}
      # Application settings
      - NODE_ENV=${N8N_NODE_ENV}
      - GENERIC_TIMEZONE=${N8N_TZ}
      # Database settings
      - DB_TYPE=${DB_TYPE}
      - DB_POSTGRESDB_HOST=${DB_POSTGRESDB_HOST}
      - DB_POSTGRESDB_PORT=${DB_POSTGRESDB_PORT}
      - DB_POSTGRESDB_DATABASE=${DB_POSTGRESDB_DATABASE}
      - DB_POSTGRESDB_USER=${DB_POSTGRESDB_USER}
      - DB_POSTGRESDB_PASSWORD=${DB_POSTGRESDB_PASSWORD}
      # npm extra options
      - EXTRA_NODE_MODULES=${N8N_EXTRA_NODE_MODULES}
      - NODE_FUNCTION_ALLOW_EXTERNAL=${N8N_NODE_FUNCTION_ALLOW_EXTERNAL}
      # Runner configuration
      - N8N_RUNNERS_ENABLED=${N8N_RUNNERS_ENABLED:-true}
      - N8N_RUNNERS_MODE=${N8N_RUNNERS_MODE:-external}
      - N8N_RUNNERS_BROKER_LISTEN_ADDRESS=${N8N_RUNNERS_BROKER_LISTEN_ADDRESS:-0.0.0.0}
      - N8N_RUNNERS_AUTH_TOKEN=${N8N_RUNNERS_AUTH_TOKEN}
      - N8N_NATIVE_PYTHON_RUNNER=${N8N_NATIVE_PYTHON_RUNNER:-true}
    volumes:
      - ${DOCKER_VOLUME_STORAGE:-/mnt/docker-volumes}/n8n/storage:/home/node/.n8n
      - ${DOCKER_VOLUME_STORAGE:-/mnt/docker-volumes}/n8n/files:/files
    #labels:
    #  - traefik.enable=true
    #  - traefik.docker.network=proxy    
    #  - traefik.http.routers.n8n.rule=Host(`${N8N_HOST}`)
    #  - traefik.http.services.n8n.loadbalancer.server.port=${N8N_PORT}
    #  # Part for optional traefik middlewares
    #  - traefik.http.routers.n8n.middlewares=local-ipwhitelist@file,basic-auth@file

  # Task Runners: https://docs.n8n.io/hosting/configuration/task-runners/#3-run-the-image
  task-runners:
    image: n8nio/runners:${N8N_RUNNERS_IMAGE_TAG:-latest}
    container_name: n8n-runners
    environment:
      - N8N_RUNNERS_TASK_BROKER_URI=${N8N_RUNNERS_TASK_BROKER_URI}
      - N8N_RUNNERS_AUTH_TOKEN=${N8N_RUNNERS_AUTH_TOKEN}
    depends_on:
      - n8n
```

### 📝 .env
```bash
# ========================================
# Docker Volume Storage
# ========================================
DOCKER_VOLUME_STORAGE=/mnt/docker-volumes

# ========================================
# N8N Host & URL Configuration
# ========================================
N8N_IMAGE_TAG=latest
N8N_HOST=n8n.devopsinaction.lab
N8N_PROTOCOL=https
N8N_URL=https://n8n.devopsinaction.lab
N8N_EDITOR_BASE_URL=https://n8n.devopsinaction.lab/
N8N_WEBHOOK_URL=https://n8n.devopsinaction.lab

# ========================================
# N8N Network & Proxy Settings
# ========================================
N8N_PORT=5678
N8N_TRUSTED_PROXIES=0.0.0.0/0
N8N_PROXY_HOPS=1

# ========================================
# N8N Security Settings
# ========================================
N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=false
N8N_SECURE_COOKIE=false

# ========================================
# N8N Application Settings
# ========================================
N8N_NODE_ENV=production
N8N_TZ=Europe/Berlin

# ========================================
# Database Configuration
# ========================================
POSTGRES_IMAGE_TAG=16-alpine
DB_TYPE=postgresdb
DB_POSTGRESDB_HOST=n8n-db
# Alternative external DB host (if needed)
#DB_POSTGRESDB_HOST=91.99.230.76
DB_POSTGRESDB_PORT=5432
DB_POSTGRESDB_DATABASE=n8n
DB_POSTGRESDB_USER=postgres
DB_POSTGRESDB_PASSWORD=Wo032FnQloAp7K

# ========================================
# PostgreSQL Non-Root User Configuration
# ========================================
POSTGRES_NON_ROOT_USER=n8n
POSTGRES_NON_ROOT_PASSWORD=Wo032FnQloAp7K

# ========================================
# Health Check Settings
# ========================================
HEALTHCHECK_INTERVAL=5s
HEALTHCHECK_TIMEOUT=5s
HEALTHCHECK_RETRIES=10

# ========================================
# NPM Extra Options
# ========================================
# This will install additional npm packages during container start
N8N_EXTRA_NODE_MODULES=lodash
# This will whitelist additional npm packages
N8N_NODE_FUNCTION_ALLOW_EXTERNAL=lodash,axios,qs

# ========================================
# Task Runner Configuration
# ========================================
N8N_RUNNERS_ENABLED=true
N8N_RUNNERS_IMAGE_TAG=1.111.0
N8N_RUNNERS_MODE=external
N8N_RUNNERS_BROKER_LISTEN_ADDRESS=0.0.0.0
# Change this to a secure random string
N8N_RUNNERS_AUTH_TOKEN=your-secret-here
N8N_RUNNERS_TASK_BROKER_URI=http://n8n:5679
N8N_NATIVE_PYTHON_RUNNER=true

# ========================================
# N8N Authentication Settings
# ========================================
N8N_BASIC_AUTH_ACTIVE=true
N8N_BASIC_AUTH_USER=user
N8N_BASIC_AUTH_PASSWORD='N8N@2026'

# ========================================
# SSL Certificate Settings (Optional)
# ========================================
#N8N_SSL_KEY=/home/node/certs/key.pem
#N8N_SSL_CERT=/home/node/certs/cert.pem
```

---
## 📝 License

This guide is provided as-is for educational and professional use.

## 🤝 Contributing
Feel free to suggest improvements or report issues via pull requests or the issues tab.

## 💼 Connect with Me 👇😊

*   🔥 [**YouTube**](https://www.youtube.com/@DevOpsinAction?sub_confirmation=1)
*   ✍️ [**Blog**](https://ibraransari.blogspot.com/)
*   💼 [**LinkedIn**](https://www.linkedin.com/in/ansariibrar/)
*   👨‍💻 [**GitHub**](https://github.com/meibraransari?tab=repositories)
*   💬 [**Telegram**](https://t.me/DevOpsinActionTelegram)
*   🐳 [**Docker Hub**](https://hub.docker.com/u/ibraransaridocker)

### ⭐ If You Found This Helpful...

***Please star the repo and share it! Thanks a lot!*** 🌟
