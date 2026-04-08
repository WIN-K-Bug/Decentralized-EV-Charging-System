# ⚡ Decentralized EV Charging System

A real-time distributed systems simulation demonstrating decentralized coordination for Electric Vehicle (EV) charging slot bookings using the **Ricart–Agrawala mutual exclusion algorithm**.

> No central server. No single point of failure. Just nodes talking to each other.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Algorithm](#algorithm---ricart-agrawala-mutual-exclusion)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Getting Started](#getting-started)
  - [Docker (Recommended)](#option-1-docker-recommended)
  - [Local Development](#option-2-local-development)
- [Usage](#usage)
- [Project Structure](#project-structure)
- [API Reference](#api-reference)
- [License](#license)

---

## Overview

This project simulates a **5-node distributed cluster** where each node represents an EV charging station. When a vehicle wants to book a charging slot, the node must enter a **critical section** (mutual exclusion) to ensure only one station books at a time — without any central coordinator.

The system features:
- A **Python/FastAPI backend** running 5 independent nodes that communicate via HTTP
- A **React/TypeScript frontend** dashboard with real-time WebSocket updates
- An animated **network topology graph** showing message passing between nodes
- Pre-built **simulation scenarios** for testing fault tolerance

---

## Architecture

```
┌─────────────────────────────────────────────────┐
│                 React Dashboard                  │
│    (WebSocket connections to all 5 nodes)        │
└────────┬────────┬────────┬────────┬────────┬────┘
         │        │        │        │        │
    ws://:8001  :8002    :8003    :8004    :8005
         │        │        │        │        │
┌────────▼──┐ ┌───▼───┐ ┌─▼─────┐ ┌▼──────┐ ┌▼──────┐
│  Node A   │ │Node B │ │Node C │ │Node D │ │Node E │
│ (FastAPI) │ │       │ │       │ │       │ │       │
└───────────┘ └───────┘ └───────┘ └───────┘ └───────┘
      ▲ ▲ ▲ ▲     HTTP POST (inter-node)    ▲ ▲ ▲ ▲
      └─┴─┴─┴─────── REQUEST / REPLY ───────┴─┴─┴─┘
```

Each node is a fully independent FastAPI server. Nodes communicate with each other via HTTP POST requests and push state updates to the frontend via WebSockets.

---

## Algorithm — Ricart-Agrawala Mutual Exclusion

The [Ricart–Agrawala algorithm](https://en.wikipedia.org/wiki/Ricart%E2%80%93Agrawala_algorithm) provides mutual exclusion in a fully distributed system using **Lamport timestamps**:

| Step | Action |
|------|--------|
| **1. Request** | Node increments its Lamport clock and sends `REQUEST(timestamp, node_id)` to all active peers |
| **2. Receive** | Upon receiving a REQUEST, a node either sends an immediate `REPLY` (if idle or lower priority) or **defers** it (if holding or has higher priority) |
| **3. Enter CS** | Once a node receives REPLYs from **all active peers**, it enters the critical section |
| **4. Exit CS** | After finishing, the node sends all deferred REPLYs and returns to IDLE |

### Tie-Breaking Rule
When two nodes request simultaneously with the same timestamp, the node with the **lower ID** wins (e.g., A < B < C < D < E).

### Fault Handling
- **Node Failure**: Failed nodes are removed from active peer lists; remaining nodes can still reach consensus with fewer replies needed
- **Recovery**: Recovered nodes broadcast a `RECOVER` message; peers re-add them to their active list
- **Network Partition**: Subsets of nodes can operate independently

---

## Features

- 🟢 **Real-time node state visualization** — IDLE, REQUESTING, HELD, FAILED states with color-coded cards
- 📊 **Animated network topology** — Pentagon SVG graph with animated message packets (yellow = REQUEST, blue = REPLY)
- 📝 **Live log stream** — Timestamped logs from all nodes in a scrollable sidebar
- 🎮 **Interactive controls** — Book slots, fail nodes, and recover nodes individually
- 🧪 **Pre-built simulation scenarios**:
  - **High Load** — All nodes request simultaneously
  - **Sequential** — Nodes request one by one with delays
  - **Contention Failure** — Node fails mid-request
  - **Recovery & Rejoin** — Fail → Recover → Request cycle
  - **Network Partition** — Drops 2 nodes to simulate cluster split
- 🔄 **Auto-reconnecting WebSockets** — Frontend reconnects automatically if a node goes down
- 🐳 **Docker support** — One command to spin up the entire cluster

---

## Tech Stack

| Component | Technology |
|-----------|-----------|
| Backend | Python 3.11, FastAPI, Uvicorn |
| Inter-node Communication | httpx (async HTTP) |
| Frontend | React 18, TypeScript |
| Build Tool | Vite 5 |
| Styling | TailwindCSS 3 |
| Animations | Framer Motion |
| Icons | Lucide React |
| Containerization | Docker, Docker Compose |

---

## Getting Started

### Prerequisites

- **Docker** & **Docker Compose** (recommended), OR
- **Python 3.11+** and **Node.js 18+** for local development

### Option 1: Docker (Recommended)

```bash
# Clone the repository
git clone https://github.com/<your-username>/Decentralized-EV-Charging-System.git
cd Decentralized-EV-Charging-System

# Start everything
docker-compose up --build
```

This spins up:
- 5 backend nodes on ports `8001`–`8005`
- Frontend dashboard on port `5173`

Open **http://localhost:5173** in your browser.

### Option 2: Local Development

#### Windows (PowerShell)
```powershell
# Run the included script
.\run-local.ps1
```

#### Manual Setup
```bash
# 1. Backend
cd backend
python -m pip install -r requirements.txt

# Start each node in a separate terminal:
# Terminal 1
set NODE_ID=A && set PEERS=http://localhost:8002,http://localhost:8003,http://localhost:8004,http://localhost:8005 && uvicorn app.main:app --port 8001

# Terminal 2
set NODE_ID=B && set PEERS=http://localhost:8001,http://localhost:8003,http://localhost:8004,http://localhost:8005 && uvicorn app.main:app --port 8002

# ... repeat for C (8003), D (8004), E (8005)

# 2. Frontend
cd frontend
npm install
npm run dev
```

---

## Usage

1. **Open the dashboard** at `http://localhost:5173`
2. **Book a slot** — Click any node button (A–E) under "Book Slot" to initiate a critical section request
3. **Fail a node** — Click the red button to simulate a node crash
4. **Recover a node** — Click the recover button to bring a failed node back online
5. **Run scenarios** — Use the pre-built simulation buttons to test various distributed scenarios
6. **Watch the graph** — Observe animated message packets moving between nodes in the topology view
7. **Read the logs** — Monitor real-time Lamport-timestamped events in the sidebar
8. **Reset** — Click "Reset Network" to clear all state and start fresh

---

## Project Structure

```
.
├── docker-compose.yml          # 5 backend nodes + frontend container
├── run-local.ps1               # Windows local dev script
├── backend/
│   ├── Dockerfile
│   ├── requirements.txt
│   └── app/
│       ├── __init__.py
│       ├── main.py             # FastAPI endpoints (REST + WebSocket)
│       └── node.py             # Ricart-Agrawala algorithm implementation
└── frontend/
    ├── Dockerfile
    ├── package.json
    ├── vite.config.ts
    ├── tailwind.config.js
    ├── tsconfig.json
    ├── index.html
    └── src/
        ├── main.tsx            # React entry point
        ├── App.tsx             # Dashboard layout + simulation scenarios
        ├── index.css           # Tailwind + custom neon glow utilities
        ├── components/
        │   └── NetworkGraph.tsx # SVG network topology with animations
        └── hooks/
            └── useNodes.ts     # WebSocket management & node state hook
```

---

## API Reference

Each node exposes the following endpoints:

### Inter-Node Communication
| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/request` | Receive a mutual exclusion REQUEST from a peer |
| `POST` | `/api/reply` | Receive a REPLY granting access |
| `POST` | `/api/recover` | Receive a recovery broadcast from a peer |

### UI Triggers
| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/trigger_booking` | Initiate a critical section request |
| `POST` | `/api/trigger_fail` | Simulate node failure |
| `POST` | `/api/trigger_recover` | Recover a failed node |
| `POST` | `/api/reset` | Reset node to initial state |

### WebSocket
| Endpoint | Messages |
|----------|----------|
| `ws://<host>:<port>/ws` | `STATE_UPDATE`, `LOG`, `MESSAGE_EVENT` |

---

## License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.
