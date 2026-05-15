# ⚙️ ChessMind — Backend

> Spring Boot 3 backend for ChessMind. Provides REST APIs, WebSocket game sessions, JWT authentication, AI move generation via Ollama, and Redis-based caching.

---

## 📦 Tech Stack

| Technology       | Purpose                                     |
|------------------|---------------------------------------------|
| Java 17          | Core language                               |
| Spring Boot 3    | REST API, WebSocket, Security framework     |
| MySQL 8          | Persistent data storage (users, games)      |
| Redis 7          | Session caching, game state management      |
| Ollama (llama3)  | LLM-based AI move generation                |
| Docker           | Containerized deployment                    |
| Maven            | Build tool                                  |

---

## 📁 Directory Structure

```
backend/
├── src/
│   └── main/
│       ├── java/
│       │   └── com/chessmind/
│       │       ├── auth/         # JWT auth, login, registration
│       │       ├── game/         # Game logic, move validation
│       │       ├── ai/           # Ollama integration
│       │       ├── websocket/    # Real-time game sessions
│       │       └── config/       # Security, CORS, Redis config
│       └── resources/
│           └── application.yml  # Spring Boot configuration
├── Dockerfile                   # Production Docker image
├── pom.xml                      # Maven dependencies
└── .env                         # Environment variables
```

---

## 🔧 Environment Variables

Create `backend/.env` with the following values:

| Variable         | Description                              | Example                        |
|------------------|------------------------------------------|--------------------------------|
| `DB_HOST`        | MySQL hostname (Docker service name)     | `mysql`                        |
| `DB_PORT`        | MySQL port                               | `3306`                         |
| `DB_NAME`        | Database name                            | `chessdb`                      |
| `DB_USER`        | Database username                        | `chessuser`                    |
| `DB_PASSWORD`    | Database password                        | `chess_secure_pass_123`        |
| `REDIS_HOST`     | Redis hostname (Docker service name)     | `redis`                        |
| `REDIS_PORT`     | Redis port                               | `6379`                         |
| `REDIS_PASSWORD` | Redis password (leave empty if none)     | *(empty)*                      |
| `OLLAMA_URL`     | Ollama API base URL                      | `http://ollama:11434`          |
| `OLLAMA_MODEL`   | Ollama model to use                      | `llama3`                       |
| `JWT_SECRET`     | Base64-encoded secret for signing JWTs   | *(generated via openssl)*      |
| `FRONTEND_URL`   | Allowed CORS origin for frontend         | `http://YOUR_SERVER_IP:3000`   |

### Generate a Secure JWT Secret
```bash
JWT_SECRET=$(openssl rand -base64 64 | tr -d '\n')
echo $JWT_SECRET
```

### Full `.env` Example
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
JWT_SECRET=YOUR_GENERATED_SECRET
FRONTEND_URL=http://YOUR_SERVER_IP:3000
```

---

## 🐳 Running with Docker (Recommended)

The backend is managed as part of the full stack via Docker Compose.

```bash
# From the project root
cd ~/Chess

# Start only the backend (and its dependencies)
docker compose up -d backend

# View backend logs
docker compose logs -f backend

# Restart backend
docker compose restart backend

# Rebuild after code changes
docker compose up -d --build backend
```

### Health Check
```bash
curl http://localhost:8080/api/auth/me
# Expected: HTTP 401 {"success":false,"message":"..."} ← correct, auth is working
```

---

## 💻 Local Development (Without Docker)

### Prerequisites
- Java 17 (JDK)
- Maven 3.8+
- MySQL 8 running locally
- Redis 7 running locally
- Ollama running locally

### Setup
```bash
cd backend

# Copy and configure environment
cp .env.example .env
# Edit .env — set DB_HOST=localhost, REDIS_HOST=localhost, OLLAMA_URL=http://localhost:11434

# Build the project
mvn clean install -DskipTests

# Run the application
mvn spring-boot:run
```

API will be available at: `http://localhost:8080`

---

## 🔌 API Endpoints

### Authentication

| Method | Endpoint              | Description             | Auth Required |
|--------|-----------------------|-------------------------|---------------|
| POST   | `/api/auth/register`  | Register a new user     | ❌            |
| POST   | `/api/auth/login`     | Login, receive JWT      | ❌            |
| GET    | `/api/auth/me`        | Get current user info   | ✅            |

### Game

| Method | Endpoint              | Description                  | Auth Required |
|--------|-----------------------|------------------------------|---------------|
| POST   | `/api/game/new`       | Create a new game            | ✅            |
| GET    | `/api/game/{id}`      | Get game state by ID         | ✅            |
| POST   | `/api/game/{id}/move` | Submit a move                | ✅            |

### AI

| Method | Endpoint              | Description                          | Auth Required |
|--------|-----------------------|--------------------------------------|---------------|
| POST   | `/api/ai/move`        | Get AI-generated move from Ollama    | ✅            |

---

## 🤖 Ollama AI Integration

The backend communicates with Ollama via its REST API at `OLLAMA_URL`.

**Difficulty levels:**
| Level  | Strategy                                        |
|--------|-------------------------------------------------|
| Easy   | Random legal moves (no Ollama call)            |
| Medium | Ollama generates a move suggestion             |
| Hard   | Ollama with more aggressive prompting          |

> ⚠️ LLMs are not chess engines — Ollama may make imperfect moves. This is intentional for Medium/Hard difficulty to keep games winnable.

**Switching AI models (if disk space is limited):**
```bash
# Pull the smaller 1B model
docker exec chess-ollama ollama pull llama3.2:1b

# Update backend config
sed -i 's/OLLAMA_MODEL=llama3/OLLAMA_MODEL=llama3.2:1b/' ~/Chess/backend/.env

# Restart backend
docker compose restart backend
```

---

## 🔐 Authentication

- JWT-based stateless authentication
- Tokens signed with `JWT_SECRET` (HS256)
- Token passed as `Authorization: Bearer <token>` in request headers
- Token validation handled by Spring Security filter chain

---

## 🗄️ Database

- **Engine:** MySQL 8
- **Database name:** `chessdb`
- Schema is auto-managed by Spring Boot (Hibernate DDL)
- Data persists in a named Docker volume (`mysql_data`)

**Connect to MySQL for inspection:**
```bash
docker exec -it chess-mysql mysql -u chessuser -p chessdb
# Enter: chess_secure_pass_123
```

---

## ⚡ Redis

Redis is used for:
- Caching active game states
- Managing WebSocket session data
- Reducing repeated DB reads during live games

**Connect to Redis for inspection:**
```bash
docker exec -it chess-redis redis-cli
> KEYS *
```

---

## 🐛 Troubleshooting

### Backend Won't Start
```bash
docker compose logs backend | tail -50
```
Common causes:
- MySQL not ready yet → run `docker compose up -d` again (MySQL needs a few seconds on first boot)
- Wrong `DB_PASSWORD` in `backend/.env`
- `JWT_SECRET` missing, empty, or too short

### MySQL Keeps Restarting
```bash
docker compose logs mysql | tail -30
```
Fix corrupted volume:
```bash
docker compose down -v
docker compose up -d --build
```

### Ollama Not Responding
```bash
curl http://localhost:11434/api/tags
# Should list llama3

docker compose logs ollama | tail -20
```

### CORS Errors in Browser
- Make sure `FRONTEND_URL` in `backend/.env` matches the exact origin of the frontend (including port)
- Rebuild after updating env: `docker compose up -d --build backend`

---

## 📌 Key Notes

- Backend compiles **44 Java source files** via Maven during Docker build — first build takes longer.
- MySQL health check ensures the backend waits for the DB to be ready before starting.
- The backend exposes both REST (`/api/**`) and WebSocket (`/ws`) endpoints on port `8080`.
- All services communicate via Docker's internal network — external access is only on port `8080`.