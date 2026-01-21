<!-- Destyl Redis Infra (Production) -->

Production-ready Redis deployment using **Docker Compose** with:
- Password authentication (`requirepass`)
- Persistent storage
- Safe Redis config (AOF + RDB)
- Easy migration to any VM (Azure / AWS / DigitalOcean)

---

## 1) Server Requirements

Recommended VM:
- Ubuntu 22.04 LTS
- 1 vCPU minimum (2 vCPU better)
- 2GB RAM minimum
- 20GB disk minimum

---

## 2) VM Setup (Ubuntu 22.04)

### 2.1 Update server
```bash
sudo apt update && sudo apt upgrade -y
```

### 2.2 Install Docker
```bash
sudo apt install -y docker.io
sudo systemctl enable --now docker
```

### 2.3 Install Docker Compose (Plugin)
```bash
sudo apt install -y docker-compose-plugin
```

**Verify:**
```bash
docker --version
docker compose version
```
If docker-compose-plugin is not available on your VM, use:
```bash
sudo apt install -y docker-compose
```

---

## 3) Clone Repo on VM
```bash
cd ~
git clone <YOUR_REPO_URL> Redis-Infra
cd Redis-Infra
```

---

## 4) Create .env (DO NOT COMMIT)

Create `.env`:
```bash
nano .env
```

Example `.env`:
```
REDIS_PORT=6380
REDIS_PASSWORD=CHANGE_ME_STRONG_PASSWORD
```

Generate a strong password:
```bash
openssl rand -hex 32
```

---

## 5) Deploy Redis

### 5.1 Start Redis container
```bash
sudo docker compose up -d
```

Check container:
```bash
sudo docker ps
```

Check logs:
```bash
sudo docker logs -n 100 destyl-redis
```

---

## 6) Test Redis on VM (Local Test)

Run:
```bash
redis-cli -h 127.0.0.1 -p 6380 -a "<YOUR_PASSWORD>" ping
```
Expected output:
```
PONG
```

---

## 7) Make Redis Accessible From Outside (Production)

### 7.1 Azure NSG (Required)

In Azure Portal:
- VM → Networking → Add inbound port rule
- Allow: TCP 6380
- Source: Your Backend VM Public IP (recommended)
- Destination: Any
- Action: Allow

⚠️ **Do NOT keep Redis open to the world in production.**
Always restrict to only your backend server IP.

---

## 8) Verify Redis is Listening on Public Port

On Redis VM:
```bash
sudo ss -lntp | grep 6380
```
Expected:
```
LISTEN ... 0.0.0.0:6380 ...
```

---

## 9) Test Redis From Backend Server (Important)

Login to backend VM and test connectivity:

### 9.1 Check port open
```bash
nc -vz <REDIS_VM_PUBLIC_IP> 6380
```
Expected:
```
succeeded
```

### 9.2 Test Redis ping
```bash
redis-cli -h <REDIS_VM_PUBLIC_IP> -p 6380 -a "<YOUR_PASSWORD>" ping
```
Expected:
```
PONG
```

---

## 10) Redis Connection URL (Use in Backend)

Your Redis URL format:
```
redis://:<PASSWORD>@<REDIS_VM_PUBLIC_IP>:6380
```
Example:
```
REDIS_URL=redis://:yourStrongPassword@4.230.84.164:6380
```

---

## 11) Restart / Stop / Update Commands

**Stop Redis:**
```bash
sudo docker compose down
```

**Start Redis:**
```bash
sudo docker compose up -d
```

**Restart Redis:**
```bash
sudo docker compose restart
```

**View logs:**
```bash
sudo docker logs -f destyl-redis
```

---

## 12) Backup & Migration (Move Redis to New Server)

### 12.1 On old server (Redis VM)

Stop Redis cleanly:
```bash
sudo docker compose down
```

Backup persistent data folder:
```bash
tar -czvf redis_backup.tar.gz redis_data
```

Copy backup to new server:
```bash
scp redis_backup.tar.gz <USER>@<NEW_SERVER_IP>:~
```

### 12.2 On new server

Extract backup:
```bash
tar -xzvf redis_backup.tar.gz
```

Place it inside repo:
```bash
mv redis_data ~/Redis-Infra/
```

Deploy again:
```bash
cd ~/Redis-Infra
sudo docker compose up -d
```

---

## 13) Security Notes (Production Best Practice)

- Always use a strong password (`REDIS_PASSWORD`)
- Restrict port 6380 to only your backend server IP (NSG rule)
- Do NOT expose Redis to the public internet
- Keep Docker + OS updated regularly

---

## 14) Troubleshooting

**Problem: Port already in use**

Error:
```
address already in use
```

Fix: change host port in `.env`:
```
REDIS_PORT=6380
```
Then:
```bash
sudo docker compose down
sudo docker compose up -d
```

**Problem: Connection timeout from backend**

Cause:
- Azure NSG not allowing 6380
- Wrong IP used
- Redis not running

Fix:
```bash
sudo docker ps
sudo ss -lntp | grep 6380
```
Test from backend:
```bash
nc -vz <REDIS_VM_PUBLIC_IP> 6380
```

---

## 15) Quick Deploy Summary (Copy-Paste)

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y docker.io docker-compose-plugin
sudo systemctl enable --now docker

cd ~
git clone <YOUR_REPO_URL> Redis-Infra
cd Redis-Infra

openssl rand -hex 32
nano .env

sudo docker compose up -d
sudo docker ps

redis-cli -h 127.0.0.1 -p 6380 -a "<YOUR_PASSWORD>" ping
```
