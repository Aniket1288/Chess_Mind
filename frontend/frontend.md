# 🖥️ ChessMind — Frontend

> Next.js 14 frontend for the ChessMind chess platform. Handles UI, real-time WebSocket communication, and game rendering.

---

## 📸 Screenshots

### 🏠 Landing Page — Hero Section
![ChessMind Hero](Screenshot/frontend/scr.png)
> *"Chess, Elevated by Intelligence" — The main landing page with CTA buttons and a live chessboard preview.*

### 🎮 Features Section — Game Modes & Board Themes
![ChessMind Features](Screenshot/frontend/scr1.png)
> *Three game modes (AI Opponent, Local Multiplayer, Online Multiplayer) and 7 selectable board themes.*

---

## 📦 Tech Stack

| Technology     | Purpose                          |
|----------------|----------------------------------|
| Next.js 14     | React framework (App Router)     |
| React          | UI component library             |
| WebSocket      | Real-time game communication     |
| Docker         | Containerized deployment         |

---

## 📁 Directory Structure

```
frontend/
├── app/              # Next.js App Router pages & layouts
├── components/       # Reusable React components
│   ├── Board/        # Chessboard rendering
│   ├── Game/         # Game logic UI
│   └── Auth/         # Login / Register components
├── public/           # Static assets (icons, images)
├── styles/           # Global CSS / Tailwind config
├── Dockerfile        # Production Docker image
└── package.json      # npm dependencies
```

---

## 🔧 Environment Variables

These variables must be set before building or running the frontend.

| Variable                | Description                                      | Example                          |
|-------------------------|--------------------------------------------------|----------------------------------|
| `NEXT_PUBLIC_API_URL`   | Base URL of the Spring Boot backend API          | `http://YOUR_SERVER_IP:8080`     |
| `NEXT_PUBLIC_WS_URL`    | WebSocket endpoint for real-time game events     | `http://YOUR_SERVER_IP:8080/ws`  |

> ⚠️ Both variables are prefixed with `NEXT_PUBLIC_` so they are exposed to the browser bundle. Do **not** put secrets here.

---

## 🐳 Running with Docker (Recommended)

The frontend is managed as part of the full stack via Docker Compose.

```bash
# From the project root
cd ~/Chess

# Start only the frontend container
docker compose up -d frontend

# View frontend logs
docker compose logs -f frontend

# Restart after changes
docker compose restart frontend

# Rebuild after code changes
docker compose up -d --build frontend
```

### Health Check
```bash
curl -I http://localhost:3000
# Expected: HTTP/1.1 200 OK
```

---

## 💻 Local Development (Without Docker)

### Prerequisites
- Node.js 18+
- npm or yarn

### Setup
```bash
cd frontend

# Install dependencies
npm install

# Create a local .env.local file
cat > .env.local << EOF
NEXT_PUBLIC_API_URL=http://localhost:8080
NEXT_PUBLIC_WS_URL=http://localhost:8080/ws
EOF

# Start development server
npm run dev
```

App will be available at: `http://localhost:3000`

---

## 🏗️ Production Build

```bash
cd frontend

# Build for production
npm run build

# Start production server
npm start
```

> In production deployments, the Docker image handles the build automatically via `docker compose up --build`.

---

## 🎮 Pages & Routes

| Route            | Description                          |
|------------------|--------------------------------------|
| `/`              | Landing / Home page                  |
| `/register`      | User registration                    |
| `/login`         | User login                           |
| `/dashboard`     | Main dashboard after login           |
| `/game`          | Active chess game view               |
| `/game/new`      | Create a new game (mode & difficulty)|

---

## 🔌 WebSocket Integration

The frontend connects to the backend via WebSocket for real-time game events:

- **Connection URL:** `NEXT_PUBLIC_WS_URL` (e.g., `http://SERVER_IP:8080/ws`)
- **Events handled:** move updates, opponent connections, game state sync, checkmate/draw detection

---

## 🐛 Troubleshooting

### Blank Page or 502 Error
```bash
docker compose logs frontend | tail -50
```
Common causes:
- `NEXT_PUBLIC_API_URL` has the wrong IP in `.env`
- Backend is not running → verify: `curl http://localhost:8080/api/auth/me`

### Port 3000 Not Accessible
- Check AWS Security Group — port `3000` must be open to `0.0.0.0/0`
- Verify container is running: `docker compose ps`

### Frontend Not Reflecting Code Changes
```bash
docker compose up -d --build frontend
```

---

## 🌐 Nginx Proxy (Optional)

To serve the frontend on port 80 instead of 3000, configure Nginx to proxy to `localhost:3000`.

See the full Nginx setup in the main [README.md](./README.md#optional-nginx-on-port-80).

---

## 📌 Key Notes

- The frontend is a **Next.js production build** inside Docker — hot reload is not available in the Docker container. Use local dev mode for active development.
- All API calls go through `NEXT_PUBLIC_API_URL` — make sure this matches your actual backend IP.
- WebSocket connections require the backend to be healthy before starting a game session.
