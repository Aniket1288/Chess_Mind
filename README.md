# ♟️ ChessMind

> A full-stack AI-powered chess platform with real-time multiplayer, AI opponent, and intelligent move assistance — all containerized with Docker.

---

## 🧠 Overview

**ChessMind** is a modern chess application that combines a Spring Boot backend, a Next.js frontend, and an on-device LLM (via Ollama) to offer a rich chess experience — from casual local play to AI-assisted competitive games.

---

## 🏗️ Tech Stack

| Layer       | Technology                          |
|-------------|-------------------------------------|
| Frontend    | Next.js 14 (React)                  |
| Backend     | Java 17 · Spring Boot 3             |
| Database    | MySQL 8                             |
| Cache       | Redis 7                             |
| AI Engine   | Ollama (llama3 / llama3.2:1b)       |
| Container   | Docker · Docker Compose             |

---

## 📁 Project Structure

```
Chess/
├── backend/          # Spring Boot REST API + WebSocket server
│   ├── src/
│   └── .env
├── frontend/         # Next.js 14 application
│   ├── app/
│   └── public/
├── docker-compose.yml
├── .env.example
├── README.md
├── frontend.md       # Frontend setup & dev guide
└── backend.md        # Backend setup & dev guide
```

---

## 🚀 Quick Start (Docker)

### 1. Clone the Repository
```bash
git clone https://github.com/Rohit-1920/Chess.git
cd Chess
```

### 2. Configure Environment Variables
```bash
# Get your server's public IP
curl -s http://checkip.amazonaws.com

# Generate a JWT secret
JWT_SECRET=$(openssl rand -base64 64 | tr -d '\n')
echo $JWT_SECRET
```

Create the root `.env` file:
```env
EC2_IP=YOUR_SERVER_IP
DB_USER=chessuser
DB_PASSWORD=chess_secure_pass_123
MYSQL_ROOT_PASSWORD=root_secure_pass_123
JWT_SECRET=YOUR_JWT_SECRET
NEXT_PUBLIC_API_URL=http://YOUR_SERVER_IP:8080
NEXT_PUBLIC_WS_URL=http://YOUR_SERVER_IP:8080/ws
```

Create `backend/.env`:
```env
DB_HOST=mysql
DB_PORT=3306
DB_NAME=chessdb
DB_USER=chessuser
DB_PASSWORD=chess_secure_pass_123
REDIS_HOST=redis
REDIS_PORT=6379
REDIS_PASSWORD=
OLLAMA_URL=http://ollama:11434
OLLAMA_MODEL=llama3
JWT_SECRET=YOUR_JWT_SECRET
FRONTEND_URL=http://YOUR_SERVER_IP:3000
```

### 3. Pull the AI Model
```bash
docker compose up -d ollama
sleep 20
docker exec chess-ollama ollama pull llama3
```

### 4. Build & Launch
```bash
docker compose up -d --build
```

> ⏱️ First build takes **8–15 minutes**. Subsequent builds use cache and take under 1 minute.

---

## 🌐 Application URLs

| Service       | URL                              |
|---------------|----------------------------------|
| Frontend      | `http://YOUR_SERVER_IP:3000`     |
| Backend API   | `http://YOUR_SERVER_IP:8080/api` |
| Ollama API    | `http://YOUR_SERVER_IP:11434`    |

---

## ✅ Health Check

```bash
# Backend — expects HTTP 401 (auth is working)
curl http://localhost:8080/api/auth/me

# Frontend — expects HTTP 200
curl -I http://localhost:3000

# Ollama — lists available models
curl http://localhost:11434/api/tags
```

Expected container status:
```
NAME               STATUS
chess-mysql        Up (healthy)
chess-redis        Up (healthy)
chess-ollama       Up
chess-backend      Up (healthy)
chess-frontend     Up
```

---

## 🎮 Game Modes

- **vs AI** — Play against the Ollama LLM at Easy / Medium / Hard difficulty
- **Local 2-Player** — Pass-and-play on the same device
- **Online Multiplayer** — Real-time play via WebSocket

> **Note:** Easy mode uses random legal moves. Medium/Hard use Ollama (llama3), which may make imperfect moves — this is intentional game design.

---

## 🛠️ Management Commands

```bash
# View running containers
docker compose ps

# View all logs
docker compose logs -f

# Restart a single service
docker compose restart backend

# Stop all (data preserved)
docker compose down

# Full reset (deletes all data)
docker compose down -v

# Rebuild after code changes
docker compose up -d --build
```

---

## ⚙️ System Requirements

| Requirement | Minimum                         |
|-------------|----------------------------------|
| OS          | Ubuntu 22.04 LTS or 24.04 LTS   |
| RAM         | 8 GB                             |
| CPU         | 2 vCPU                           |
| Disk        | 20 GB free (llama3 = 4.7 GB)    |
| Ports       | 22, 3000, 8080                   |

---

## 📖 Detailed Docs

- 📄 [Frontend Guide](.frontend/frontend.md)
- 📄 [Backend Guide](.backend/backend.md)

---

## 👤 Author

**Aniket** — [GitHub Profile](https://github.com/Aniket1288)

---

## 📄 License

This project is open-source. See `LICENSE` for details.
