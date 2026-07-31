# Plaisoram Real-Time Engine & Migration Roadmap

This document outlines the architectural roadmap for optimizing and evolving Plaisoram's real-time communication infrastructure. It analyzes the current stack, explains resource allocation trade-offs, and provides step-by-step migration paths ranging from zero-code Node.js V8 optimizations to an enterprise Go-based Centrifugo engine.

---

## 1. Current Architecture & Bottleneck Analysis

```
┌─────────────────────────────────┐
│        Symfony 7 PHP            │  <-- CORE BUSINESS ENGINE (DeviceManager,
│       (Plaisoram_Server)        │      PlaylistManager, ScheduleManager, DB)
└────────────────┬────────────────┘
                 │ HTTP Broadcasts (/events/emit)
                 ▼
┌─────────────────────────────────┐
│     Node.js / Socket.io         │  <-- TRANSPORT PIPE (Tracks active socket map,
│     (Plaisoram_Realtime)        │      relays pings/commands to dashboards & player)
└────────────────┬────────────────┘
                 │ WebSockets (0ms)
        ┌────────┴────────┐
        ▼                 ▼
┌───────────────┐ ┌───────────────┐
│ Android TV    │ │ Next.js      │
│ Player App    │ │ Dashboard     │
└───────────────┘ └───────────────┘
```

### Key Bottleneck Findings:
* **Node.js RAM Usage (63% on 512MB DO Droplet)**: Caused by V8 JavaScript engine baseline heap memory (~120MB), not business data.
* **Separation of Concerns**: Symfony is the business domain engine. `Plaisoram_Realtime` is a transport pipe.
* **Mercure SSE Hub (`plaisoram-hub`)**: Redundant ($5/mo). Socket.io handles two-way WebSockets while Mercure handles only one-way SSE.

---

## 2. Option C: Immediate Node.js RAM Optimizations (Low Effort)

Before migrating engines, you can optimize your existing `Plaisoram_Realtime` container to reduce memory usage from **63% down to ~25-30%**:

### Steps:
1. **Cap V8 Heap Memory**:
   Add `--max-old-space-size=128` to Node's startup command in Docker/DO:
   ```bash
   node --max-old-space-size=128 dist/server.js
   ```
2. **Ensure Pure JavaScript Production Build**:
   Do not run `ts-node` in production. Always build TypeScript to JS (`tsc`) and execute `dist/server.js`.
3. **Upgrade to `uWebSockets.js` C++ Engine Driver**:
   Replace the default Node.js HTTP server driver with compiled C++ bindings:
   ```bash
   pnpm add @socket.io/uws
   ```
   * **Result**: Drops Node.js baseline RAM from ~120MB to ~35MB with zero architectural changes.

---

## 3. Option A: Centrifugo Go Migration (Recommended Long-Term)

Centrifugo is an open-source, battle-tested real-time messaging server written in **Go**. It replaces **BOTH** Mercure ($5/mo) and Node.js (`Plaisoram_Realtime`) with **1 single Go process (~15MB RAM)**.

### Why Centrifugo Fits Plaisoram:
* **RAM Reduction**: Drops real-time memory from ~120MB to **~15MB** (an 85% drop).
* **Cost Savings**: Eliminates `plaisoram-hub` ($5/month).
* **Dual Transport**: Supports 2-way WebSockets (Android TV Player) AND SSE (Next.js Dashboard).
* **Built-in Channel Presence**: Native `presence:leave` events trigger instant offline screen alerts without custom polling code.

### Step-by-Step Migration Plan:

#### Phase 1: Deploy Centrifugo Docker Container
Add Centrifugo to DigitalOcean App Platform or Docker Compose:
```yaml
version: "3"
services:
  centrifugo:
    image: centrifugo/centrifugo:v5
    ports:
      - "8000:8000"
    volumes:
      - ./config.json:/centrifugo/config.json
    command: centrifugo -c config.json
```

#### Phase 2: Update Symfony Backend (`Plaisoram_Server`)
Install the official PHP SDK:
```bash
composer require centrifugal/php-centrifugo
```
Create `CentrifugoDeviceNotifier.php` to publish updates to Centrifugo channels (`device:{id}`, `workspace:{id}`).

#### Phase 3: Update Android TV Player (`Plaisoram_Player`)
Use Centrifugo’s official Kotlin client (`centrifuge-java` / `centrifuge-kt`). Replace socket listeners in `PlayerViewModel.kt`. One WebSocket connection handles heartbeats, commands, and layout sync.

#### Phase 4: Update Next.js Web Dashboard (`plaisoram_web`)
Use `centrifuge-js` client in `useSocket.ts` to subscribe to workspace channel presence and broadcast events.

---

## 4. Option B: Custom Go Microservice (`centrifuge` Library)

If you ever require custom edge computing or proprietary binary protocol transformations inside the WebSocket gateway itself:
* Use Centrifugo's core Go library (`github.com/centrifugal/centrifuge`).
* Write a custom `server.go` microservice.
* **Benefits**: Ultra-low memory (~10MB RAM), full Go control, zero V8 overhead.

---

## 5. Horizontal Scale-Out Strategy (100,000 to 1,000,000+ Screens)

When Plaisoram grows to tens of thousands of active screens:
1. Run multiple Centrifugo nodes behind a DigitalOcean Load Balancer.
2. Attach a **Redis Pub/Sub** or **NATS** engine cluster to Centrifugo (`"engine": "redis"` in `config.json`).
3. Centrifugo handles inter-node message routing automatically with **zero code changes** to Symfony or Android TV!
