
# 🚀 Redis Infra

<p align="center">
  <b>Production-Ready Redis with Docker Compose</b>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Redis-Production--Ready-red?logo=redis" />
  <img src="https://img.shields.io/badge/Docker-Compose-blue?logo=docker" />
  <img src="https://img.shields.io/badge/Ubuntu-22.04%20LTS-orange?logo=ubuntu" />
</p>

<br/>

<details>
<summary><b>✨ Features</b></summary>

- 🔒 Password authentication (`requirepass`)
- 💾 Persistent storage
- 🛡️ Safe Redis config (AOF + RDB)
- ☁️ Easy migration to any VM (Azure / AWS / DigitalOcean)

</details>

---

## 🖥️ 1) Server Requirements

<ul>
	<li><b>Ubuntu 22.04 LTS</b></li>
	<li>1 vCPU minimum <i>(2 vCPU better)</i></li>
	<li>2GB RAM minimum</li>
	<li>20GB disk minimum</li>
</ul>

---

## ⚙️ 2) VM Setup (Ubuntu 22.04)

### 2.1 Update server
```bash
sudo apt update && sudo apt upgrade -y
```

### 2.2 Install Docker (supports <code>sudo docker compose up -d</code>)
```bash
sudo apt install -y docker.io docker-compose-v2
sudo systemctl enable --now docker
```

### 2.3 Install Redis CLI tools (for testing)
```bash
sudo apt install -y redis-tools
```

---

## 🚀 3) Deploy Redis

### 3.1 Clone repo
```bash
git clone <YOUR_REPO_URL> Redis-Infra
cd Redis-Infra
```

### 3.2 Create <code>.env</code>
```bash
nano .env
```
<details>
<summary>Example <code>.env</code> file</summary>

```
REDIS_PORT=6380
REDIS_PASSWORD=change_this_to_a_secure_password
```
</details>

Generate secure password:
```bash
openssl rand -hex 32
```

### 3.3 Start Redis
```bash
sudo docker compose up -d
```
Check running container:
```bash
sudo docker ps
```

---

## 🧪 4) Test Redis (inside VM)
```bash
redis-cli -h 127.0.0.1 -p 6380 -a "<REDIS_PASSWORD>" ping
```
<details>
<summary>Expected output</summary>

```
PONG
```
</details>

---

## 🌐 5) Get VM Public IP (for Backend Connection)

Run this on the Redis VM:
```bash
curl -4 ifconfig.me
```
<details>
<summary>Example output</summary>

```
4.240.88.165
```
</details>

---

## 🔗 6) Backend Connection (Destyl)

Use this in your backend <code>.env</code>:
```
REDIS_URL=redis://:<REDIS_PASSWORD>@<VM_PUBLIC_IP>:6380
```
<details>
<summary>Example</summary>

```
REDIS_URL=redis://:YOUR_PASSWORD@4.240.88.165:6380
```
</details>

---

## 🛠️ 7) Useful Commands

<ul>
	<li><b>Stop Redis:</b>
		<pre>sudo docker compose down</pre>
	</li>
	<li><b>Restart Redis:</b>
		<pre>sudo docker compose restart</pre>
	</li>
	<li><b>View logs:</b>
		<pre>sudo docker logs -f destyl-redis</pre>
	</li>
</ul>

---

## ⚡ Quick Deploy Summary

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y docker.io docker-compose-v2 redis-tools
sudo systemctl enable --now docker

git clone <YOUR_REPO_URL> Redis-Infra
cd Redis-Infra
nano .env

sudo apt install -y redis-tools
sudo docker compose up -d
redis-cli -h 127.0.0.1 -p 6380 -a "<REDIS_PASSWORD>" ping
curl -4 ifconfig.me
```