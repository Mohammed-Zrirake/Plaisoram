# Plaisoram Critical Situations & Robustness Enhancements

This document outlines the critical vulnerabilities, bottlenecks, and edge cases in the current V1 architecture, along with strategies to mitigate them to ensure the system is robust against high traffic, concurrent actions, and malicious abuse.

## 1. Security & Abuse Prevention

### Registration (Process 1)
- **Email-existence leak:** Returning `409 Conflict` on duplicate emails tells an attacker which emails are registered. In a high-security context, mask this by accepting the request, returning `201 Created`, but sending a "you already have an account" email. At minimum, aggressive rate-limiting (per IP / fingerprint) and a CAPTCHA should be added.
- **No email verification:** Registrations should require a confirmation link before the account is usable. Otherwise, bots can pollute the database with spam Workspaces and devices.

### Authentication (Process 2)
- **Token storage in LocalStorage:** This is an XSS vector. Use HttpOnly, Secure cookies with `SameSite=Strict` for the JWT.
- **Missing Refresh Mechanism:** Maintain a short-lived access token + refresh token pattern. If the token expires mid-session, the dashboard currently breaks.
- **Brute-force protection:** The `login_check` endpoint needs exponential backoff or account locking after N failed attempts.

### Device Pairing (Process 3)
- **TTL for 6-digit codes:** The code is a critical resource. The `initDevice` call should store an `expiresAt` (e.g., 5 minutes) and the code must be strictly single-use.
- **Rate-limiting:** Rate-limit `initDevice` per IP or device fingerprint to prevent attackers from exhausting the pairing-code space and creating thousands of pending devices.
- **Replay Attacks:** The confirmation step `POST /api/devices/{code}/pair` must invalidate the code immediately after successful pairing to prevent replays if the player crashes and retries.

### Heartbeat (Process 4)
- **Database Write Storms:** Every heartbeat writes `isOnline` and `lastSeen` directly to the relational database. For thousands of devices pinging every 30s, this will crush the database.
- **Robust Alternative:** Write heartbeats to a fast key-value store (like Redis) with a TTL. A separate, lower-frequency background worker can bulk-update the SQL database and publish aggregate status changes to Mercure.
- **Dead-man switch:** Offline detection needs a scheduler that periodically scans for devices whose Redis TTL has expired, otherwise a crashed player will never be marked offline.

---

## 2. Idempotency & Distributed Consistency

### Media Upload (Process 5)
- **Orphaned Cloud Objects:** If the `confirmUpload` call fails after the file is already uploaded to B2/S3, the cloud object becomes orphaned. Use a multi-part upload with an `upload-id`, or implement a background job to periodically clean unconfirmed blobs older than N hours.
- **Idempotency:** `confirmUpload` must be idempotent. It should check if the `Media` entity already exists for that file key, and only create it if it doesn't.

### Playlist Publishing (Processes 5, 6)
- **Missed Events:** The pattern "update Device's currentPlaylist → push Mercure event" assumes the player is online to receive the SSE. If offline, the event may be missed.
- **Robustness:** The player should always re-fetch the latest playlist from the server upon reconnecting (via `/api/devices/{id}/playlist`). The Mercure notification should be treated as a "hint to update", not the sole source of truth.

### Schedule Publishing (Process 8)
- **Cancel/Update Problems:** Using Symfony Messenger with `DelayStamp` introduces an issue if a schedule is deleted or modified *after* the message is queued. The delayed job will still fire at the scheduled time.
- **Mitigation:** The message handler must check the schedule's status in the database (e.g., verifying it is still 'pending') and compare its version/timestamp before applying the playlist.
- **Concurrency Conflicts:** Conflict detection (time-range overlap) is prone to race conditions if two requests arrive simultaneously. Use a unique constraint, an explicit pessimistic lock on the workspace's schedule range when inserting, or optimistic locking with a retry on `409`.

---

## 3. Player-Side State & Resilience

### Playlist Section Transitions (Process 7)
- **Mid-Section Interruptions:** A new `playlist_updated` event may arrive mid-section. The player engine needs to abort the current timer, clean up Zone resources (video decoders, textures), and switch immediately without visual tearing.
- **State Machine Architecture:** Model the player as a state machine with explicit `STOPPING -> IDLE -> LOADING -> PLAYING` states, allowing incoming commands to safely transition out of any state.
- **Resource Leaks:** Unmounting a Zone that contains a video player must explicitly release the media player instance to avoid memory leaks that will degrade performance over time.
- **Timer Drift:** System clock timers can drift when the device sleeps. For precise durations (especially for ads), the engine should use a media clock tied to a monotonic source, not wall-clock `setTimeout`.

### Media Caching (Process 6)
- **Cache Invalidation:** The "if cached locally play, else fetch" logic fails if a user replaces a media file while keeping the same filename. The player will show stale content indefinitely.
- **Robustness:** Include a version hash or ETag in the playlist JSON for each media item. The player must compare this with the cached version's metadata and re-download if it has changed.

---

## 4. Observability & Failure Recovery

- **Structured Error Handling:** Server-side controllers should log errors with deep context (workspace ID, device ID) and return structured problem-detail responses (RFC 7807) so the client can react appropriately.
- **Business-Level Event Monitoring:** Critical workflows (upload confirmation, schedule dispatch) should emit business-level events for monitoring. E.g., if a scheduled message is in the queue 5 minutes after its due time, an alert should fire.
- **Mercure Fallback:** The Android Player should implement exponential back-off when connecting to Mercure, and gracefully fall back to HTTP polling `/api/devices/{id}/playlist` if the SSE connection repeatedly fails.

---

## 5. Scalability Considerations

- **Scheduling Large Delays:** If thousands of schedules are set days in advance, the message queue broker (RabbitMQ/Redis) must be sized correctly to hold many pending messages. Memory usage must be monitored.
- **Database Schema Contention:** Ensure `lastSeen` and `isOnline` updates do not cause lock contention on the primary `devices` table. Consider moving real-time status tracking to a lightweight table or purely into Redis.

---

### Final Verdict on V1 Architecture
The current blueprint is very solid, but the main robustness gaps fall into three buckets:
1. **Missing guardrails** – rate-limiting, expirations, single-use tokens, and input validation depth.
2. **Assuming the happy path** – lack of idempotency, no compensation for failed confirmations, and no status checking on delayed message jobs.
3. **Passive Receiver Player** – relying purely on Mercure pushes without a fallback fetch leaves the screen vulnerable to missed events.

Addressing these critical situations will make the V1 system significantly more resilient and lay the precise groundwork required for the V2 offline architecture.
