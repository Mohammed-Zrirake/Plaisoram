# Managing Multiple Devices (Scalability Guide)

This document outlines the transition from a single-device hardcoded setup to a professional, multi-device signage network.

## 1. Eliminate Hardcoded IPs
The biggest bottleneck is the hardcoded `10.131.215.247` in your Kotlin files. To support multiple boxes, the app must "discover" or "be told" where the server is.

### Strategy A: Manual Configuration UI (The Reliable Way)
Instead of starting directly on the pairing screen, add a "Setup" screen where the technician can enter the Server IP once.
*   **Implementation**: Store the `server_url` in Android `SharedPreferences` or `DataStore`.
*   **Benefit**: Works across different networks and VPNs.

### Strategy B: Network Service Discovery (NSD / mDNS)
Android devices can "search" the local network for a specific service (like a printer or a server).
1.  **Server Side**: Use a library to broadcast a service named `_plaisoram._tcp` on your computer.
2.  **App Side**: Use Android's `NsdManager` to listen for that service. When found, the app automatically extracts the IP address.
*   **Benefit**: Zero-touch configuration. The app "just finds" the server.

---

## 2. Unique Device Identification
Your system is already designed for this! 
*   **Initialization**: Every time a new TV box is installed, it calls `/api/devices/init`.
*   **ID Persistence**: The server returns a **unique Integer ID**. The TV box stores this ID in its local Room database.
*   **Heartbeats**: Each box sends its own ID (`/api/devices/{id}/heartbeat`), so the dashboard can distinguish between Box A (Green) and Box B (Red).

---

## 3. QR Code Pairing (The Modern Way)
To make linking a new TV box extremely fast:
1.  **TV Box**: Generates a Pairing Code (e.g., `ABCDEF`) and shows it on screen.
2.  **Mobile App/Dashboard**: The user scans a QR code or enters the code in the dashboard.
3.  **Dynamic Base URL**: If the QR code contains `http://10.131.215.247:8000|ABCDEF`, the app can configure its own `BASE_URL` and `PairingCode` in one step.

---

## 4. Scalable Media Delivery
When you have 50 TV boxes downloading 4K videos at the same time, your computer might struggle.
*   **Content Delivery Network (CDN)**: In a production environment, you would host your `uploads/media` folder on a dedicated file server or S3 bucket.
*   **Local Caching**: The current `MediaRepositoryImpl` already caches files locally. This is crucial as it prevents the TV box from re-downloading the same file every time the playlist loops.

---

## 5. Proposed Code Changes for "Multi-Device" Support

### Update `DeviceConfig.kt`
Ensure the `serverUrl` is used in all API calls instead of a companion object constant.

### Update `PlaisoramApi.kt` (Dynamic Base URL)
Use a Retrofit Interceptor or a dynamic `@Url` parameter to switch servers on the fly.

```kotlin
// Example of a dynamic request in Retrofit
@GET
suspend fun getPlaylist(@Url url: String): List<PlaylistItemDto>
```

---

## ⚙️ Immediate Recommendation for your 2nd Box:
1.  **Assign a Static IP** to your computer in your router settings (so it stays `10.131.215.247`).
2.  **Use a Build Config Variable**: In `build.gradle`, set the `BASE_URL` so you can change it for different "environments" (Home, Office, Client) without touching Kotlin code.
3.  **Install the APK**: The 2nd box will receive its own ID (e.g., ID: 13) and will appear as a separate row in your dashboard automatically.
