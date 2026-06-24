# Plaisoram Broadcasting Architecture: Mercure

## Overview

This document explains the broadcasting mechanism used in the Plaisoram Digital Signage Platform to communicate between the **Server** (Symfony), the **Web Dashboard** (Next.js), and the **Digital Media Players** (Android TV).

The platform uses **Mercure** (based on Server-Sent Events - SSE) to broadcast commands, statuses, and real-time updates across the system.

---

## 1. Why Mercure? (Long-Term Choice Evaluation)

Mercure is an open protocol designed specifically for pushing data to web browsers and connected devices in real-time. For a digital signage platform like Plaisoram, it is an **excellent long-term choice**, though it comes with minor trade-offs.

### Pros (Why it's a good choice)
*   **Unidirectional Flow (Server to Client):** Digital signage players predominantly *receive* instructions (e.g., "play this playlist", "restart", "turn off"). They rarely need to push heavy real-time data back to the server (status checks are handled via simple HTTP polling/heartbeats). Server-Sent Events (SSE) perfectly matches this asymmetrical flow.
*   **Native Reconnection & Resiliency:** Unlike WebSockets which require complex client-side reconnection logic, SSE automatically handles connection drops and reconnects natively. This is critical for remote Android TV players that might experience flaky network connections.
*   **Firewall & Proxy Friendly:** WebSockets sometimes struggle with strict corporate firewalls or aggressive proxies because they use a different protocol upgrade. Mercure uses standard HTTP/HTTPS (`text/event-stream`), meaning it works seamlessly anywhere standard web traffic works.
*   **Built-in Symfony Integration:** Symfony provides a native, highly integrated `symfony/mercure-bundle`. This avoids the overhead of managing third-party WebSocket libraries like Ratchet or Socket.io.
*   **Stateless Server Scaling:** The Mercure Hub acts as a standalone binary (usually Caddy-based). Your Symfony app remains completely stateless, pushing a message to the Hub via a simple HTTP POST request, and the Hub distributes it to millions of connected clients.

### Cons (What to watch out for)
*   **Not Bidirectional:** If your players ever need to stream heavy real-time data *back* to the server (e.g., live remote screen-viewing or real-time diagnostic streaming), Mercure won't work for the upstream. You would have to rely on standard HTTP POST requests from the player (which is exactly how the current `Heartbeat` mechanism works).
*   **Mobile Battery Impact:** SSE keeps an HTTP connection open. While fine for a plugged-in Android TV box, it can drain batteries on mobile phones (though this isn't an issue for dedicated signage hardware).

**Verdict:** For Plaisoram, Mercure is the optimal, lightweight, and resilient choice for broadcasting orders.

---

## 2. How the Broadcasting Mechanism Works

The broadcasting flow relies on a Publisher/Subscriber (PubSub) model:

1.  **The Hub:** A standalone Mercure Hub server runs alongside your backend (likely configured in `Plaisoram_Server/compose.yaml` and `Plaisoram_Server/mercure_app/`).
2.  **The Publisher (Symfony Server):** When an action occurs (e.g., a user clicks "Publish" on the Web Dashboard, or a device heartbeats), the Symfony backend uses the `HubInterface` to push a JSON payload to a specific **Topic**.
3.  **The Subscribers (Android Player / Web Dashboard):** The client devices keep an open HTTP stream to the Mercure Hub's `/.well-known/mercure` endpoint, subscribing to specific topics. When the Hub receives a message from the Server, it instantly pushes it down the open stream to the listening clients.

### Topics Used in Plaisoram:
*   `device/{id}/updates`: Subscribed to by individual Android Players to receive direct commands.
*   `dashboard/updates`: Subscribed to by the Web Dashboard to receive live status updates (e.g., when a screen goes online/offline).

---

## 3. How the Project is Using It (Configs & Implementation)

### A. The Server (Publisher)

The Symfony server acts as the central command brain. It pushes events to the Mercure hub.

**Relevant Configurations:**
*   `Plaisoram_Server/config/packages/mercure.yaml`: Defines the Hub URL and the JWT secret required to authorize the server to publish messages.
*   `Plaisoram_Server/.env`: Contains the `MERCURE_URL`, `MERCURE_PUBLIC_URL`, and `MERCURE_JWT_SECRET` environment variables.

**Responsible Files & Actions:**
*   **`src/Controller/DeviceController.php`**
    *   **Publishing Playlists (`/{id}/publish` & `/{id}/publish-media`):** Sends a `PlaylistUpdated` event to `device/{id}/updates`.
    *   **Power Management (`/{id}/toggle-power`):** Sends a `PowerCommand` to the device and a `DeviceStatusUpdated` to the dashboard.
    *   **Remote Refresh (`/{id}/refresh`):** Sends a `RefreshCommand` to force the Android player to restart its cycle.
    *   **Pairing (`/pair`):** When the web app successfully pairs a screen, the server broadcasts a `DevicePaired` event to tell the waiting Android TV to proceed.
*   **`src/EventListener/DeviceStatusListener.php`**
    *   **Real-time Dashboard:** Intercepts incoming requests. If a device that was offline makes a request (comes back online), it instantly publishes a `DeviceStatusUpdated` to `dashboard/updates` so the web interface updates its status indicators from red to green without needing a page refresh.

### B. The Android Player (Subscriber)

The Android TV app acts as the listener. It uses Kotlin to maintain an open HTTP stream to the Mercure Hub.

**Responsible Files:**
*   **`app/src/main/java/com/sobrus/plaisoramplayer/presentation/pairing/PairingViewModel.kt`**
    *   Opens an SSE connection to `/.well-known/mercure?topic=device/{deviceId}/updates`.
    *   Listens for the raw Mercure data. If it receives a `DevicePaired` event, it automatically transitions the TV from the pairing screen to the actual player interface.
*   **`app/src/main/java/com/sobrus/plaisoramplayer/presentation/player/PlayerViewModel.kt`**
    *   Maintains the active SSE connection while the signage is running.
    *   Parses incoming JSON payloads:
        *   If `PlaylistUpdated`, it triggers `SyncEngine.startImmediateSync()` to download and synchronize the new media.
        *   If `PowerCommand` or `RefreshCommand`, it executes the respective local Android functions to sleep the screen or reload the UI.

### C. The Web Dashboard (Subscriber)

While the dashboard acts as the origin of commands (by calling Symfony APIs), it is also a *subscriber* to monitor fleet health.
*   **`src/app/(dashboard)/devices/page.tsx`**: Uses the browser's native `EventSource` listening to `${mercureUrl}?topic=dashboard/updates` to dynamically update the online/offline indicators for all devices in real-time without needing a manual page refresh.


## Summary

Mercure acts as the nervous system of Plaisoram. It removes the need for devices to constantly "ask" the server if there are updates (which would DDOS the server with thousands of devices). Instead, the devices sit quietly and listen, and the server "taps them on the shoulder" via Mercure the exact millisecond an admin publishes a new layout or hits the restart button.
