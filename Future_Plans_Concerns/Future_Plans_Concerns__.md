# Future Plans & Concerns

## The "Silent Crash" Byzantine Failure
By eliminating background polling on the Android Player to save database writes, the TV becomes a "silent listener" via SSE (Mercure). If a TV is forcefully killed (e.g., unplugged, or force-stopped via remote control), it cannot fire `onCleared()` and thus cannot send a `/status` POST to declare itself offline.

This creates a scenario where the backend database thinks a device is `is_online = true`, but it is physically dead.

## Future Solution (Option C: Mercure Hub Subscriptions Polling)
Currently, we are using a **Frontend-Driven Ping-Pong** (Option A) mixed with a backend Cron Job to detect dead devices. 
However, for true zero-CPU detection at scale, the ideal future architecture is to use the **Mercure Hub Subscriptions API**.

### How it works:
1. The PHP Backend completely stops relying on the `/status` API for offline detection.
2. The Backend runs a fast cron job (or listens to a Redis stream) every 1 minute.
3. The Backend queries the Mercure Hub directly: `GET <MERCURE_URL>/.well-known/mercure/subscriptions`
4. The Mercure Hub replies with exactly which topics (`device/{id}/updates`) currently have an active TCP connection open.
5. The Backend instantly updates `is_online = true` for active connections, and `is_online = false` for inactive ones.

### Why we are not doing it yet:
This requires a specific Mercure Hub deployment (enabling the subscriptions API or Webhooks), which is complex to secure and configure. Standard deployments do not have this enabled by default. As the project scales to thousands of screens, this should be investigated to eliminate all "ping-pong" HTTP overhead.
