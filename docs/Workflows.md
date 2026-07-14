# Plaisoram Business Workflows

This document outlines the core business processes of the Plaisoram ecosystem (Web App, Server, and Player). The diagrams use standard BPMN-style flowcharts via Mermaid syntax to illustrate the step-by-step interactions between the various components.

---

## 1. Account Creation (Individual or Enterprise)

The registration process allows users to create an account and a dedicated Workspace. Depending on the account type, different details are required, but they funnel into the same manager.

```mermaid
flowchart TD
    A([User visits Register Page]) --> B{Select Account Type}
    
    B -->|Individual| C["Fill Personal Details<br>First/Last Name, Email"]
    B -->|Enterprise| D["Fill Enterprise Details<br>Org Name, Timezone, etc."]
    
    C --> E["Submit to RegistrationController::register"]
    D --> E
    
    E --> F["Validate Payload via RegistrationDTO"]
    F -->|Invalid| G([Return 400 Bad Request])
    
    F -->|Valid| H["UserManager::registerUser"]
    H --> I[Check for Duplicate Email]
    I -->|Exists| J([Return 409 Conflict])
    
    I -->|Unique| K["Hash Password & Create User Entity"]
    K --> L["Create Workspace Entity linked to User"]
    L --> M[(Save to Database)]
    
    M --> N([Return 201 Created])
```

---

## 2. Authentication and Redirection

Standard JWT-based authentication flow used by the Web Dashboard.

```mermaid
flowchart TD
    A([User enters Email/Password]) --> B["POST /api/login_check"]
    B --> C[LexikJWTAuthenticationBundle intercepts]
    
    C --> D{Verify Credentials in DB}
    D -->|Invalid| E([Return 401 Unauthorized])
    
    D -->|Valid| F[Generate JWT Token]
    F --> G([Return JWT Token to Client])
    
    G --> H["Client stores Token in LocalStorage/Cookies"]
    H --> I([Client redirects to Web Dashboard])
```

---

## 3. Adding a New Device (with Mercure Hub)

This process connects a physical Android Player to a user's Workspace without requiring the user to type long URLs or credentials into the TV.

```mermaid
flowchart TD
    A([Android Player Starts]) --> B["Player calls DeviceController::initDevice"]
    B --> C["DeviceManager generates 6-digit Pairing Code"]
    C --> D[(Save Device as Pending in DB)]
    D --> E([Player displays Pairing Code on TV])
    
    F([User logs into Web App]) --> G["User clicks 'Add Screen' & enters Code"]
    G --> H["POST /api/devices/pair via DeviceController"]
    
    H --> I["DeviceManager::pairDevice"]
    I --> J{Code Valid?}
    J -->|No| K([Return 400 Error])
    
    J -->|Yes| L["Assign Device to User's Workspace"]
    L --> M[(Update Device in DB)]
    
    M --> N["DeviceNotifier dispatches event to Mercure Hub"]
    N --> O((Mercure Hub))
    O -.->|Server-Sent Event| P["Android Player receives 'paired' event"]
    
    P --> Q["Player fetches JWT via /api/devices/{code}/pair"]
    Q --> R([Player connects to Workspace & awaits Playlists])
```

---

## 4. Heartbeat Process for Devices Health Check

To ensure screens are online and functioning, the Android Player sends periodic heartbeats.

```mermaid
flowchart TD
    A([Android Player Timer]) -->|Every X seconds| B["POST /api/devices/{id}/heartbeat"]
    
    B --> C["DeviceController::heartbeat"]
    C --> D["DeviceManager::heartbeat"]
    D --> E["(Update isOnline and lastSeen in DB)"]
    
    E --> F["DeviceNotifier pushes status to Mercure Hub"]
    F --> G((Mercure Hub))
    G -.->|Server-Sent Event| H[Web Dashboard receives update]
    H --> I([UI indicates Screen is Online/Offline])
```

---

## 5. Process for Newly Added Media Upload

Uploading a new video or image from the user's computer, through the Web App, and eventually playing on the Android TV. This utilizes direct-to-cloud uploads to save server bandwidth.

```mermaid
flowchart TD
    A([User selects File in Web App]) --> B["Web App calls MediaController::getPresignedUrl"]
    B --> C["MediaManager requests upload URL from B2/S3 Storage"]
    C --> D([Return Presigned URL to Web App])
    
    D --> E[Web App uploads physical file directly to Cloud Storage]
    
    E -->|Upload Success| F["Web App calls MediaController::confirmUpload"]
    F --> G["MediaManager creates Media entity"]
    G --> H[(Save Media to DB)]
    
    H --> I[User assigns Media to a Playlist Zone]
    I --> J[User clicks 'Publish to Screen']
    
    J --> K["DeviceManager updates Device's currentPlaylist"]
    K --> L["DeviceNotifier triggers Mercure Event"]
    L --> M((Mercure Hub))
    M -.-> N["Android Player receives 'playlist_updated' event"]
    
    N --> O["Player calls /api/media/serve/{filename} to fetch stream"]
    O --> P([Player renders new media on TV])
```

---

## 6. Process for Handling Existing Media

When media is already uploaded, the server simply manages the database records, while the player handles caching to save bandwidth.

```mermaid
flowchart TD
    A([User opens Media Library]) --> B["MediaController::list"]
    B --> C["MediaManager fetches Workspace Media"]
    C --> D([Web App displays Media Gallery])
    
    D --> E[User adds existing Media to Playlist & Publishes]
    E --> F[Mercure notifies Android Player of new Playlist JSON]
    
    F --> G[Player parses Playlist JSON]
    G --> H{Does Player have Media cached locally?}
    
    H -->|Yes| I([Player renders media from Local Storage])
    H -->|No| J["Player fetches stream from /api/media/serve/{filename}"]
    J --> K[(Player saves file to Local Cache)]
    K --> I
```

---

## 7. Playlist with Multiple Zones and Sections

How the Android Player safely interprets complex playlists (Sections acting as slides/scenes, Zones acting as widgets on the screen) without mutation or visual tearing.

```mermaid
flowchart TD
    A([Player receives Playlist JSON]) --> B[Engine parses JSON into Memory Models]
    B --> C[Pre-load Media/Assets for Section 1]
    
    C --> D[Render Frame for Section 1]
    D --> E["Zone 1: Mount Video Player"]
    D --> F["Zone 2: Mount Image View"]
    D --> G["Zone 3: Mount Ticker Tape"]
    
    E & F & G --> H["Start Section Timer (e.g., 10 seconds)"]
    
    H -->|Timer Expires| I[Fade Out / Unmount current Zones gracefully]
    I --> J{Are there more Sections?}
    
    J -->|Yes| K[Pre-load assets for Section 2]
    K --> L[Render Frame for Section 2]
    L --> H
    
    J -->|No| M[Reset to Section 1]
    M --> C
```

---

## 8. Schedule Creation & Consumption

### Why Symfony Messenger over a basic Cron Job?
Instead of a basic daemon process that polls the database every minute (which causes DB strain, latency, and race conditions), Plaisoram uses **Symfony Messenger with Delay Stamps**. 

When a schedule is created for a future date, a message is immediately dispatched to a Message Queue (like RabbitMQ or Redis) with a specific timestamp. The worker daemon sits completely idle until that exact millisecond. 
- **Precision:** Execution happens exactly when requested, down to the millisecond.
- **Reliability:** If the server crashes during execution, the message queue automatically retries the job.
- **Scalability:** Multiple workers can consume the queue simultaneously without locking database rows.

```mermaid
flowchart TD
    A([User creates Schedule in Web App]) --> B["ScheduleController::create"]
    
    B --> C["ScheduleManager::createSchedule"]
    C --> D{Validate: Is Date in the past?}
    D -->|Yes| E([Return 400 Bad Request])
    
    D -->|No| F{Validate: Conflict with existing schedule?}
    F -->|Yes| G([Return 409 Conflict])
    
    F -->|No| H["(Save PublishSchedule as 'pending' in DB)"]
    
    H --> I["Create PublishPlaylistMessage"]
    I --> J["Attach DelayStamp calculated from Scheduled Date"]
    J --> K[Dispatch to Symfony Messenger Bus]
    
    K --> L[(Message Queue / Redis)]
    
    L -.->|Waits until exact scheduled time| M([Worker Daemon Process])
    M --> N["PublishPlaylistMessageHandler consumes message"]
    
    N --> O["DeviceManager::publishPlaylist"]
    O --> P["(Update Device currentPlaylist in DB)"]
    
    P --> Q["DeviceNotifier pushes to Mercure Hub"]
    Q --> R((Mercure Hub))
    R -.-> S([Android Player immediately updates Screen])
    
    Q --> T["(Update Schedule status to 'published' in DB)"]
```

---

## Future Processes to Consider (V2 Architecture)

These processes are critical for scaling Plaisoram into a robust, enterprise-grade digital signage platform.

### A. Offline Resilience & Auto-Recovery
Ensuring the screen never goes black even if the internet drops for days, and recovering automatically after a power loss.

```mermaid
flowchart TD
    A([Player loses Internet / Power Cycle]) --> B{Is internet available?}
    
    B -->|No| C["Load cached Schedule & Playlist from local DB"]
    C --> D["Play media from Local Storage (Offline Mode)"]
    D --> E["Queue Heartbeats & Analytics locally"]
    
    B -->|Yes| F["Reconnect to Mercure Hub"]
    F --> G["Sync missed messages & schedules from Server"]
    G --> H["Flush local queued Analytics/Heartbeats to Server"]
    H --> I([Resume normal Online Mode])
```

### B. Proof of Play & Analytics Batching
Tracking exactly how many times an ad or media file was played, batched to prevent server overload.

```mermaid
flowchart TD
    A([Media finishes playing on TV]) --> B["Log event (MediaID, Timestamp) in Player's local DB"]
    
    B --> C{Has 1 hour passed?}
    C -->|No| D([Continue playing next media])
    
    C -->|Yes| E["Player compiles batch of logs into JSON"]
    E --> F["POST /api/analytics/batch"]
    
    F --> G["AnalyticsController validates payload"]
    G --> H["Dispatch AnalyticsBatchMessage to Messenger"]
    H --> I[(Queue)]
    
    I -.-> J([Worker Daemon])
    J --> K["Bulk Insert logs into Database/ClickHouse"]
    K --> L([Return 200 OK to Player])
    L --> M["Player clears local logs"]
```

### C. Remote Device Management & OTA Updates
Remotely commanding screens without physical intervention.

```mermaid
flowchart TD
    A([Admin clicks 'Reboot Device' in Web App]) --> B["POST /api/devices/{id}/command"]
    
    B --> C["DeviceManager validates Permissions"]
    C --> D["DeviceNotifier sends specific command payload"]
    
    D --> E((Mercure Hub))
    E -.->|Server-Sent Event| F["Player receives 'COMMAND_REBOOT'"]
    
    F --> G["Player executes Android System Reboot"]
    G --> H([Device Restarts & Auto-Launches App])
```
