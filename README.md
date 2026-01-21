# Destyl Redis Infra (Production-Ready)

This repository provides a **production-grade Redis deployment** using **Docker Compose**, featuring:

- 🔒 Password protection (`requirepass`)
- 💾 Persistent storage (AOF + RDB)
- 🛡️ Safe Redis config for queues & caching
- 🚀 Easy migration to any VM (Azure / AWS / DigitalOcean)

---

## 1️⃣ Repository Structure

```
destyl-redis-infra/
├── config/
│   └── redis.conf
├── docker-compose.yml
├── .env.example
└── README.md
```

---

## 2️⃣ Prerequisites

### Local Machine
- Docker installed
- Docker Compose installed

### VM Server (Ubuntu 22.04 Recommended)
- Public IP available
- Port allowed in Firewall/NSG (see below)

---

## 3️⃣ Environment Variables

Create a `.env` file in the root directory:

```bash
nano .env
```

Example `.env`:

```
REDIS_PORT=6380
REDIS_PASSWORD=YOUR_SECURE_PASSWORD
```

Generate a secure password:

```bash
openssl rand -hex 32
```

---

## 4️⃣ Docker Compose (How it Works)

- Redis runs inside the container on port **6379**
- Host exposes it on port **6380** (or your chosen port)

**Port Mapping:**

```
HOST:6380  --->  CONTAINER:6379
```

---

## 5️⃣ Run Redis Locally (Testing)

Start Redis:

```bash
docker compose up -d
```

Check container:

```bash
docker ps
```

Test Redis locally:

```bash
redis-cli -h 127.0.0.1 -p 6380 -a "YOUR_SECURE_PASSWORD" ping
```

Expected output:

```
PONG
```

Stop Redis:

```bash
docker compose down
```

---

## 🚀 Deploy Redis on a VM (Production)

Follow these steps to deploy Redis on a fresh Ubuntu VM.

### Step 1: SSH into VM

```bash
ssh ubuntu@YOUR_VM_PUBLIC_IP
# Or using Azure Cloud Shell:
az ssh vm --resource-group YOUR_RG --vm-name YOUR_VM_NAME
```

### Step 2: Install Docker (Ubuntu 22.04)

```bash
sudo apt update -y
sudo apt install -y docker.io
sudo systemctl enable --now docker
```

Verify installation:

```bash
docker --version
sudo systemctl status docker --no-pager
```

### Step 3: Install Docker Compose

**Option A (Recommended): Compose Plugin**

```bash
sudo apt install -y docker-compose-plugin
docker compose version
```

**Option B (Legacy):**

```bash
sudo apt install -y docker-compose
docker-compose --version
```

> ⚠️ If your server installs `podman-docker` by mistake, remove it and use Docker only.

### Step 4: Clone the Repo

```bash
git clone YOUR_GITHUB_REPO_URL
cd destyl-redis-infra
```

### Step 5: Create .env

```bash
nano .env
```

Paste and update:

```
REDIS_PORT=6380
REDIS_PASSWORD=CHANGE_THIS_PASSWORD
```

Generate a strong password:

```bash
openssl rand -hex 32
```

### Step 6: Start Redis on VM

```bash
sudo docker compose up -d
sudo docker ps
```

Expected:

```
Container running
Port mapped like: 0.0.0.0:6380->6379/tcp
```

### Step 7: Verify Redis on VM

```bash
redis-cli -h 127.0.0.1 -p 6380 -a "YOUR_PASSWORD" ping
```

Expected:

```
PONG
```

---

## 🌍 Allow External Access (Important)

If your backend is on another server, it must connect using the VM Public IP.

### 1) Azure NSG Rule (Required)

In Azure Portal:

- VM → Networking → Inbound port rules → Add inbound rule
	- Source: IP Addresses (recommended)
	- Source IP: your backend server public IP
	- Source port ranges: *
	- Destination: Any
	- Service: Custom
	- Destination port ranges: 6380
	- Protocol: TCP
	- Action: Allow
	- Priority: 310 (or any safe priority)
	- Name: redis_port_development

### 2) Ubuntu Firewall (UFW)

Check status:

```bash
sudo ufw status
```

If UFW is active, allow Redis port:

```bash
sudo ufw allow 6380/tcp
```

### 3) Confirm Redis is Listening

```bash
sudo ss -lntp | grep 6380
```

Expected:

```
LISTEN 0 4096 0.0.0.0:6380 ...
```

---

## 🔗 Redis Connection URL (Use in Backend)

Format:

```
redis://:<PASSWORD>@<HOST>:<PORT>
```

Example:

```
REDIS_URL=redis://:YOUR_PASSWORD@4.230.84.164:6380
```

---

## 🧪 Test From Another Server (Backend VM / Local)

From your backend machine:

```bash
redis-cli -h 4.230.84.164 -p 6380 -a "YOUR_PASSWORD" ping
```

Expected:

```
PONG
```

> If it hangs, check NSG inbound rule or firewall settings.

---

## 🔒 Production Security Best Practices

**Recommended:**

- Keep Redis on a separate VM
- Only allow port 6380 from:
	- Backend server IP
	- Worker server IP
- Never expose Redis to the public internet (0.0.0.0/0)
- Always use a strong password (`requirepass`)
- Keep AOF enabled for queue safety

---

## 📌 Useful Commands

**Logs:**

```bash
sudo docker logs -f destyl-redis
```

**Restart:**

```bash
sudo docker restart destyl-redis
```

**Stop:**

```bash
sudo docker compose down
```

**Update (pull new config):**

```bash
git pull
sudo docker compose up -d --build
```

---

## ✅ Monitoring (Basic)

**Live Redis Stats:**

```bash
redis-cli -h 127.0.0.1 -p 6380 -a "YOUR_PASSWORD" info
```

**Connected Clients:**

```bash
redis-cli -h 127.0.0.1 -p 6380 -a "YOUR_PASSWORD" client list
```

**Memory Usage:**

```bash
redis-cli -h 127.0.0.1 -p 6380 -a "YOUR_PASSWORD" info memory
```

---

## 📝 License

MIT
