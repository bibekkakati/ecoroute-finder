# 🌱 EcoRoute Finder

> **Air Quality-Aware Intelligent Logistics & Delivery Routing Engine**  
> Dynamic graph-based routing platform combining road network topology, real-time Air Quality Index (AQI) monitoring, and sub-millisecond autocomplete for sustainable urban mobility in Delhi NCR.

[![Node.js](https://img.shields.io/badge/Node.js-18+-339933?style=flat-square&logo=node.js&logoColor=white)](https://nodejs.org/)
[![Express](https://img.shields.io/badge/Express-5.x-000000?style=flat-square&logo=express&logoColor=white)](https://expressjs.com/)
[![React](https://img.shields.io/badge/React-19-61DAFB?style=flat-square&logo=react&logoColor=black)](https://react.dev/)
[![Vite](https://img.shields.io/badge/Vite-8.x-646CFF?style=flat-square&logo=vite&logoColor=white)](https://vitejs.dev/)
[![Neo4j](https://img.shields.io/badge/Neo4j-Graph_DB-008CC1?style=flat-square&logo=neo4j&logoColor=white)](https://neo4j.com/)
[![Redis](https://img.shields.io/badge/Redis-Cache_%26_Search-DC382D?style=flat-square&logo=redis&logoColor=white)](https://redis.io/)
[![SQLite](https://img.shields.io/badge/SQLite-Drizzle_ORM-003B57?style=flat-square&logo=sqlite&logoColor=white)](https://www.sqlite.org/)
[![License: ISC](https://img.shields.io/badge/License-ISC-blue.svg?style=flat-square)](LICENSE)

---

## 📌 Executive Summary

Urban delivery fleets and logistics operators frequently navigate heavily congested and severely polluted metropolitan corridors. In cities like Delhi NCR, spikes in the Air Quality Index (AQI > 400, categorized as *Severe/Hazardous*) expose delivery couriers to acute health risks while prolonging travel times.

**EcoRoute Finder** solves this challenge by introducing an environmental-aware routing engine:
- Computes **Top-K optimal delivery corridors** balancing path distance and cumulative air pollution exposure.
- Enforces an **AQI safety threshold** (default: 400) to strictly detour around hazardous pollution hotspots.
- Features a **live background synchronization worker** simulating real-time atmospheric shifts across 97 urban anchor sectors every 5 seconds.
- Automatically initiates **instant rerouting alerts** whenever an active transit waypoint breaches safety limits.

---

## 📸 Interface Previews

| Interactive Origin / Destination & Route Exploration | Real-time Corridors & Pollution Hotspot Detection |
| :---: | :---: |
| ![Route selection preview](previews/ss-1.png) | ![Route recommendations preview](previews/ss-2.png) |

---

## 🚀 Key Features

- **Multi-Criteria Graph Routing**: Evaluates road network distances alongside live AQI telemetry using a composite scoring heuristic ($Score = Distance + \overline{AQI} \times \omega$).
- **Strict & Soft AQI Cutoffs**: Strict exclusion filters candidate paths traversing areas with AQI > 400. If no clean paths exist, fallback mechanisms evaluate the safest possible routes with transparent warnings.
- **Sub-Millisecond Redis Autocomplete**: Backed by a Redis Sorted Set (`ZSET`) lexicographical index (`ZRANGEBYLEX`), offering prefix lookups in $O(\log N + M)$ time across thousands of neighborhoods.
- **Spatial Resolution & Geohashing**: Seamlessly maps raw GPS coordinates `(lat, lng)` to the nearest graph node utilizing precision-6 geohashes within a 1 km radius.
- **Dynamic Background Synchronization**: Background worker simulates realistic pollution fluctuations across 97 urban anchors (65% in 300–400 range, 35% in hazardous 401–500 range).
- **Interactive GIS Map Visualization**: Leaflet-powered interface featuring route polylines, interactive waypoint popups, color-coded AQI severity badges, and distance summaries.
- **Production-Grade Security**: API key validation with SHA-256 hashed storage, session activity auditing, Helmet HTTP security headers, and configurable rate limiting.

---

## 🏗️ System Architecture

```text
┌─────────────────────────────────────────────────────────────┐
│                 React 19 + Leaflet Dashboard                │
│             (Vite, Interactive Map, Autocomplete)           │
└──────────────────────────────┬──────────────────────────────┘
                               │ HTTP / JSON (API Key Authenticated)
                               ▼
┌─────────────────────────────────────────────────────────────┐
│                   Express 5 REST API Gateway                │
│    (Rate Limiting, Helmet, Zod Validation, Error Handler)    │
└──────┬───────────────────────┼───────────────────────┬──────┘
       │                       │                       │
       ▼                       ▼                       ▼
┌──────────────┐       ┌──────────────┐       ┌──────────────┐
│    Neo4j     │       │    Redis     │       │    SQLite    │
│   Graph DB   │       │   (ioredis)  │       │(Drizzle ORM) │
│              │       │              │       │              │
│  Area Nodes  │       │ Autocomplete │       │ API Keys &   │
│  ROAD Edges  │       │ Sorted Sets  │       │ Session Log  │
│ Top-K Paths  │       │ (O(log N))   │       │ Audits       │
└──────▲───────┘       └──────────────┘       └──────────────┘
       │ Updates (Every 5s)
┌──────┴───────────────────────┐
│     AQI Background Worker    │
│  (97 NCR Anchors, Live Sync) │
└──────────────────────────────┘
```

---

## 🛠️ Technology Stack

### Frontend Client (`/web`)
| Technology | Version | Purpose |
| :--- | :--- | :--- |
| **React** | `^19.2.8` | Declarative UI components and state management |
| **Vite** | `^8.3.0` | Next-generation frontend tooling and bundler |
| **Leaflet** | `^1.9.4` | Interactive mobile-friendly spatial mapping |
| **Axios** | `^1.20.0` | Promise-based HTTP client with request/response interceptors |
| **Lucide React** | `^1.47.0` | Comprehensive iconography library |
| **Vanilla CSS** | Modern CSS3 | Custom high-performance design system with responsive grid |

### Backend API (`/api`)
| Technology | Version | Purpose |
| :--- | :--- | :--- |
| **Node.js** | `>= 18.0.0` | Asynchronous JavaScript runtime engine |
| **Express** | `^5.2.1` | REST API routing and middleware pipeline |
| **Neo4j Driver** | `^6.2.0` | Bolt protocol connection driver for graph database operations |
| **ioredis** | `^6.0.0` | High-performance Redis client for sorted-set search indexing |
| **better-sqlite3** | `^13.0.3` | Ultra-fast synchronous SQLite3 binding |
| **Drizzle ORM** | `^0.45.2` | TypeScript/JavaScript ORM for relational SQLite data mapping |
| **Zod** | `^4.6.5` | Strict schema declaration and request validation |
| **ngeohash** | `^0.6.4` | Spatial geohash encoding, decoding, and proximity bounding |
| **Helmet** | `^8.3.0` | HTTP response headers security middleware |
| **Express Rate Limit** | `^8.7.0` | DoS protection and API rate throttling |

---

## 🔑 Environment Variables

### Backend Configuration (`api/.env`)

Configure these values in [`api/.env`](file:///Users/bibek/Documents/ignite-hackathon/api/.env) (or copy from [`api/.env.example`](file:///Users/bibek/Documents/ignite-hackathon/api/.env.example)):

| Variable | Type | Required | Default | Description |
| :--- | :---: | :---: | :--- | :--- |
| `PORT` | `number` | No | `3000` | Port on which the Express server listens. |
| `NODE_ENV` | `string` | No | `development` | Runtime environment (`development`, `production`, `test`). |
| `CORS_ORIGIN` | `string` | No | `*` | Allowed CORS origin (set to frontend URL in production). |
| `RATE_LIMIT_MAX` | `number` | No | `200` | Max requests permitted per 15-minute window per IP address. |
| `SQLITE_DB_PATH` | `string` | No | `./data/eco_route.db` | Filesystem path for the SQLite database. |
| `REDIS_URL` | `string` | **Yes** | `redis://localhost:6379` | Redis connection URI (supports standalone and cloud Redis). |
| `GRAPH_DATABASE_PROVIDER` | `string` | No | `neo4j` | Graph database provider identifier. |
| `GRAPH_DATABASE_URL` | `string` | **Yes** | `bolt://localhost:7687` | Neo4j Bolt connection URI (`bolt://` or `neo4j+s://`). |
| `GRAPH_DATABASE_USERNAME` | `string` | **Yes** | `neo4j` | Username for Neo4j database authentication. |
| `GRAPH_DATABASE_PASSWORD` | `string` | **Yes** | — | Password for Neo4j database authentication. |
| `ENABLE_AQI_WORKER` | `boolean` | No | `true` | Enables or disables the background AQI sync worker. |
| `AQI_WORKER_INTERVAL_MS`| `number` | No | `5000` | Interval in milliseconds between AQI sync cycles (5s). |

### Frontend Configuration (`web/.env`)

Configure these values in [`web/.env`](file:///Users/bibek/Documents/ignite-hackathon/web/.env) (or copy from [`web/.env.example`](file:///Users/bibek/Documents/ignite-hackathon/web/.env.example)):

| Variable | Type | Required | Default | Description |
| :--- | :---: | :---: | :--- | :--- |
| `VITE_API_URL` | `string` | No | `http://localhost:3000/api/v1` | Base URL of the backend REST API. |

---

## ⚡ Getting Started

### Prerequisites

Ensure the following runtimes and services are installed and active:
- **Node.js**: v18.0.0 or higher
- **npm**: v9.0.0 or higher
- **Redis Server**: Local instance (`localhost:6379`) or a managed cloud Redis instance (e.g. Redis Cloud)
- **Neo4j Database**: Local instance (Desktop / Community Edition) or managed Neo4j AuraDB

---

### Step-by-Step Installation

#### 1. Clone Repository
```bash
git clone https://github.com/bibekkakati/eco-route-neo4j.git
cd ignite-hackathon
```

#### 2. Backend Setup (`/api`)
```bash
cd api

# Install dependencies
npm install

# Setup environment configuration
cp .env.example .env
# Edit .env to set your Neo4j and Redis credentials

# Run database migrations for SQLite (API keys & sessions schema)
npm run migrate

# Seed graph topology into Neo4j and populate Redis autocomplete index
npm run seed

# Start the API server
npm start
```
> The API server will start on `http://localhost:3000` with the background AQI worker active.

#### 3. Frontend Setup (`/web`)
Open a new terminal window:
```bash
cd web

# Install dependencies
npm install

# Setup environment configuration (optional if using default localhost:3000)
cp .env.example .env

# Start the Vite development server
npm run dev
```
> Open your browser at `http://localhost:5173` to access the dashboard.

---

## 📋 Available Scripts

### API (`/api`)
- `npm start`: Runs the production Express server (`server.js`).
- `npm run dev`: Starts the server with Node's native file watcher (`node --watch server.js`).
- `npm run migrate`: Creates SQLite tables and indexes for API authentication via Drizzle.
- `npm run seed`: Seeds NCR grid areas, road distances, and indexes into Neo4j and Redis.
- `npm run worker:aqi`: Runs the AQI synchronization worker independently as a standalone process.
- `npm test`: Executes the end-to-end automated test suite using `supertest`.

### Web Dashboard (`/web`)
- `npm run dev`: Starts the local Vite development server with HMR.
- `npm run build`: Compiles optimized static assets for production deployment into `/dist`.
- `npm run preview`: Locally previews the production build.
- `npm run lint`: Analyzes codebase with `oxlint`.

---

## 📡 REST API Reference

All protected endpoints require the `X-API-Key` header. A pre-configured demonstration key is provided in the web client.

| Method | Endpoint | Description |
| :--- | :--- | :--- |
| `GET` | `/health` | Unauthenticated service health and timestamp check. |
| `POST` | `/api/v1/routes/find` | Calculate top-K routes between origin and destination nodes or coordinates. |
| `GET` | `/api/v1/areas/search` | Prefix-based autocomplete search for locations (Redis-backed). |
| `GET` | `/api/v1/areas` | Retrieve list of all registered area nodes with current AQI. |
| `GET` | `/api/v1/areas/:areaId` | Fetch details and metadata for a specific area node. |
| `POST` | `/api/v1/areas/batch` | Batch fetch multiple area nodes in a single request (for polling). |
| `GET` | `/api/v1/areas/lookup/coordinates` | Match coordinates to nearest area node using 1 km geohash lookup. |
| `PATCH`| `/api/v1/areas/aqi` | Update AQI for a specific coordinate pair. |
| `POST` | `/api/v1/areas/aqi/bulk` | Bulk update multiple coordinate AQI readings simultaneously. |
| `GET` | `/api/v1/roads` | Retrieve all road relationships across the network. |

### Sample Route Request: `POST /api/v1/routes/find`

```json
{
  "originId": "Area 1",
  "destinationId": "Area 45",
  "k": 3,
  "maxVariation": 0.30,
  "aqiThreshold": 400,
  "softCheck": false
}
```

---

## 🧪 Testing

### Automated Test Suite
Run the test suite covering authentication, rate limiting, area operations, Redis autocomplete, and routing algorithms:
```bash
cd api
npm test
```

### Manual Verification Flow
1. Navigate to the web application at `http://localhost:5173`.
2. In the **Origin** field, type `Connaught` and select `Connaught Place, Central Delhi` from the autocomplete dropdown.
3. In the **Destination** field, type `DLF Phase` and select `Cyber City, DLF Phase 2, Gurgaon`.
4. Click **Find Optimal Path**.
5. Inspect the recommended corridors:
   - Primary route displays total distance, average AQI, and path status.
   - Map polyline overlays trace the selected path across Delhi NCR.
6. Observe live polling every 5 seconds: when a node on the active route exceeds AQI 400, the system triggers a reroute notification and computes a safer alternate path.

---

## 📄 License

This project is licensed under the [ISC License](LICENSE).
