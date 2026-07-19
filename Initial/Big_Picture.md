# Plaisoram System Architecture

This document maps the complete system architecture of the **Plaisoram** digital signage platform, covering global cloud infrastructure, real-time networking, and the internal offline-first database architecture of the Android signage players.

---

## 1. Global Cloud & Network Architecture

The global architecture connects workspace managers on the Next.js web dashboard with distributed Android player devices through a Symfony REST API and real-time Mercure SSE WebSockets.

```mermaid
flowchart LR
    %% Custom Styling Definitions
    classDef dev fill:#2b2b2b,stroke:#007acc,stroke-width:2px,color:#fff
    classDef ci fill:#1e3a8a,stroke:#3b82f6,stroke-width:2px,color:#fff
    classDef server fill:#0f172a,stroke:#38bdf8,stroke-width:2px,color:#fff
    classDef db fill:#064e3b,stroke:#10b981,stroke-width:2px,color:#fff
    classDef storage fill:#0f172a,stroke:#8b5cf6,stroke-width:2px,color:#fff
    classDef user fill:#1e293b,stroke:#f59e0b,stroke-width:2px,color:#fff
    classDef device fill:#1e293b,stroke:#ef4444,stroke-width:2px,color:#fff

    %% 1. Developer Environment & CI/CD
    subgraph Development_and_CI ["Development & CI/CD Pipeline"]
        direction TB
        Dev["💻 Solo Developer<br>(VS Code)"]:::dev
        GitHook["🛡️ Pre-Push Git Hook<br>(PHPUnit & SQLite Test DB)"]:::ci
        DO_Pipeline["⚙️ DigitalOcean App Platform<br>(Auto-Build & Deploy Pipeline)"]:::ci
    end

    %% 2. Core Server Infrastructure
    subgraph Cloud_Infrastructure ["DigitalOcean Cloud (Core Infrastructure)"]
        direction TB
        LB["🌐 Load Balancer / Ingress"]:::server
        Web["🖥️ plaisoram_web<br>(Next.js Dashboard App)"]:::server
        API["⚙️ Plaisoram_Server<br>(Symfony 7 REST API)"]:::server
        Mercure["⚡ Mercure Hub<br>(Real-time SSE WebSockets)"]:::server
    end

    %% 3. External Managed Data Services
    subgraph Data_Storage ["Managed Data & Media Storage"]
        direction TB
        NeonDB[("🗄️ Neon Serverless<br>(PostgreSQL Database)")]:::db
        B2[("☁️ Backblaze B2<br>(S3 Object Media Storage)")]:::storage
    end

    %% 4. End Users and Client Devices
    subgraph Clients ["End Users & Player Devices"]
        direction TB
        Admins["👥 Workspace Admins / Managers"]:::user
        Android1["📺 Plaisoram_Player 1<br>(Android Digital Signage)"]:::device
        Android2["📺 Plaisoram_Player N<br>(Android Digital Signage)"]:::device
    end

    %% ==== DEFINING THE CONNECTIONS ====

    %% Dev Flow
    Dev -. "1. git push" .-> GitHook
    GitHook -- "2. Tests Pass" --> DO_Pipeline
    GitHook -. "❌ Fails (Push Blocked)" .-> Dev
    DO_Pipeline -- "3. Deploy Updates" --> Web
    DO_Pipeline -- "3. Deploy Updates" --> API

    %% Admins to Web
    Admins == "HTTPS Traffic" ==> LB
    LB --> Web
    Web -. "Internal Proxy API Calls" .-> API
    
    %% Android Players to API
    Android1 == "REST (Init, Status, Heartbeats)" ==> LB
    Android2 == "REST (Init, Status, Heartbeats)" ==> LB
    LB --> API
    
    %% Server to Data
    API == "Doctrine ORM (Queries/Writes)" ==> NeonDB
    API == "Flysystem (Upload Media)" ==> B2
    
    %% Android Downloading Media
    Android1 -. "Direct Media Download" .-> B2
    Android2 -. "Direct Media Download" .-> B2

    %% Real-time Updates (Mercure)
    API -. "Publish Server Events" .-> Mercure
    Mercure -. "SSE Push (Refresh/Power/Playlists)" .-> Android1
    Mercure -. "SSE Push (Refresh/Power/Playlists)" .-> Android2
    Mercure -. "SSE Push (Dashboard Sync)" .-> Web
```

## 2. Android Player Internal Offline-First Architecture

A critical component missing from standard high-level network diagrams is the **internal persistent database and caching layer** of the digital signage player (`Plaisoram_Player`). Because digital signage screens operate in commercial environments where internet connectivity can be intermittent or unstable, the Android player is engineered as an **Offline-First App**.

Instead of streaming video directly from the cloud on every loop, the player uses a local **Room SQLite Database** and an internal **Media File Cache** to ensure continuous, zero-latency playback even during total network outages.

```mermaid
flowchart TB
    classDef device fill:#1e293b,stroke:#ef4444,stroke-width:2px,color:#fff
    classDef db fill:#064e3b,stroke:#10b981,stroke-width:2px,color:#fff
    classDef storage fill:#0f172a,stroke:#8b5cf6,stroke-width:2px,color:#fff
    classDef net fill:#0f172a,stroke:#38bdf8,stroke-width:2px,color:#fff

    subgraph Cloud ["Remote Cloud Services"]
        API["⚙️ Plaisoram REST API<br>(Playlists & Auth)"]:::net
        Mercure["⚡ Mercure SSE Hub<br>(Real-Time Trigger)"]:::net
        B2["☁️ Backblaze B2 S3<br>(Remote Media Assets)"]:::net
    end

    subgraph Android_Player ["📱 Plaisoram_Player (Android Offline-First Device)"]
        direction TB
        SSE_Client["📡 SSE Event Listener<br>(Mercure Subscriber)"]:::device
        SyncManager["🔄 Sync & Download Engine<br>(Background Coroutine Worker)"]:::device
        PlayerUI["📺 Presentation Engine<br>(ExoPlayer & Jetpack Compose)"]:::device
        
        subgraph Internal_Storage ["Internal Persistent Storage Layer"]
            direction LR
            RoomDB[("🗄️ Room SQLite Database<br>(AppDatabase v2)")]:::db
            FileCache[("💾 Local Media File Cache<br>(/filesDir/media/)")]:::storage
        end
    end

    %% Connections
    Mercure -. "1. SSE Event (PlaylistUpdated)" .-> SSE_Client
    SSE_Client --> SyncManager
    SyncManager == "2. Fetch Playlist JSON" ==> API
    SyncManager == "3. Save Items & SyncStatus" ==> RoomDB
    SyncManager == "4. Download Asset Bytes" ==> B2
    SyncManager == "5. Save & Validate Checksum (SHA-256)" ==> FileCache
    SyncManager == "6. Update Status to COMPLETED" ==> RoomDB
    
    PlayerUI == "Query Active Playlists & Orders" ==> RoomDB
    PlayerUI == "Read Local Video & Image Bytes" ==> FileCache
```

### A. Internal Room SQLite Database (`AppDatabase`)
The player utilizes Android's **Room ORM** (`AppDatabase` v2) to manage persistent local state without relying on server availability. The database consists of two core tables:

1. **`device_config` (`DeviceConfigEntity`)**:
   - Stores the unique device identifier (`deviceId`), screen pairing secret (`screenKey`), pairing status (`isPaired`), configured server URL (`serverUrl`), and cryptographic authentication tokens (`accessToken`, `refreshToken`).
   - Ensures the device survives reboots and power cycles without requiring manual re-authentication by screen technicians.

2. **`playlist_items` (`PlaylistItemEntity`)**:
   - Functions as the local queue and playback schedule. Stores individual media item records including `id`, `playlistId`, `mediaId`, `type` (image/video), `remoteUrl`, `localPath`, `durationSeconds`, `displayOrder`, and expected SHA-256 `checksum`.
   - Tracks real-time sync progression using the `downloadStatus` enum (`PENDING`, `DOWNLOADING`, `COMPLETED`, `FAILED`).

### B. Local Media File Cache (`/filesDir/media/`)
- Managed by `MediaRepositoryImpl`, all remote video and image assets fetched from Backblaze B2 are downloaded directly into internal device storage (`context.filesDir/media/`).
- **Integrity Verification**: As each file completes downloading, the repository calculates its SHA-256 hash and verifies it against the `expectedChecksum` provided by the server. If valid, `localPath` is updated in Room and `downloadStatus` transitions to `COMPLETED`.
- **Zero-Latency Playback**: The UI presentation layer (`ExoPlayer` / Compose canvas) reads video and image streams exclusively from `localPath` on disk. This completely eliminates buffering, bandwidth costs, and playback stuttering.

### C. Real-Time Synchronization Lifecycle
1. **Trigger**: When a workspace manager publishes a new playlist or schedule from `plaisoram_web`, `Plaisoram_Server` pushes a lightweight `PlaylistUpdated` SSE event to the Mercure Hub.
2. **Fetch**: The player's background listener catches the event and invokes `PlaylistRepositoryImpl` to fetch the latest playlist structure from the REST API.
3. **Diff & Download**: The local Room database is updated with new `playlist_items`. Any missing or updated media files are downloaded in the background into `/filesDir/media/`.
4. **Seamless Switch**: Once all items reach `SyncStatus.COMPLETED`, the presentation engine seamlessly transitions to broadcasting the new schedule from local storage.
