# Plaisoram Critical Situations & Robustness Enhancements

This document outlines the critical vulnerabilities, bottlenecks, and edge cases in the current V1 architecture, along with strategies to mitigate them to ensure the system is robust against high traffic, concurrent actions, and malicious abuse.

## 1. Security & Abuse Prevention

### Registration (Process 1)
- **Email-existence leak:** Returning `409 Conflict` on duplicate emails tells an attacker which emails are registered. In a high-security context, mask this by accepting the request, returning `201 Created`, but sending a "you already have an account" email. At minimum, aggressive rate-limiting (per IP / fingerprint) and a CAPTCHA should be added.
- **No email verification:** Registrations should require a confirmation link before the account is usable. Otherwise, bots can pollute the database with spam Workspaces and devices.

### Authentication (Process 2)
- ~~**Token storage in LocalStorage:** This is an XSS vector. Use HttpOnly, Secure cookies with `SameSite=Strict` for the JWT.~~ [x] Handled
- ~~**Missing Refresh Mechanism:** Maintain a short-lived access token + refresh token pattern. If the token expires mid-session, the dashboard currently breaks.~~ [x] Handled
- ~~**Brute-force protection:** The `login_check` endpoint needs exponential backoff or account locking after N failed attempts.~~ [x] Handled

**Implemented Solution (Next.js Backend-For-Frontend Pattern):**
The authentication architecture utilizes a highly secure Backend-For-Frontend (BFF) pattern that shields the browser from accessing tokens directly.

**1. Login & Token Issuance:**
- The user logs in via the Next.js Server Action (`plaisoram_web/src/actions/auth.ts`). 
- This Server Action sends a backend request to Symfony's `POST /api/login_check`.
- Symfony validates the credentials. Brute-force attacks are mitigated natively by Symfony's `login_throttling` (configured in `Plaisoram_Server/config/packages/security.yaml` with max 5 attempts).
- Symfony returns a JSON payload containing the short-lived JWT and a long-lived Refresh Token.
- **Security Measure:** The Next.js Server Action converts these JSON tokens into `httpOnly`, `secure`, and `SameSite=Strict` cookies before sending the response to the user's browser. Client-side JavaScript NEVER touches the tokens, completely neutralizing XSS attacks.

**2. Transparent API Proxying:**
- When the Next.js frontend needs to fetch data, it hits its own proxy route (`plaisoram_web/src/app/api/[...slug]/route.ts`).
- This Next.js API route reads the `httpOnly` cookie, attaches it as an `Authorization: Bearer <token>` header, and forwards the request securely to Symfony.

**3. Proactive Token Refresh:**
- Every page request is intercepted by the Next.js Middleware (`plaisoram_web/src/proxy.ts`).
- The middleware decodes the JWT and checks its expiration timestamp. 
- If the token is within 30 seconds of expiring, the middleware pauses the user's request, proactively hits Symfony's `POST /api/token/refresh` (powered by `gesdinet/jwt-refresh-token-bundle`), and retrieves a new set of tokens.
- **Security Measure:** The middleware seamlessly overwrites the browser's `httpOnly` cookies with the new tokens and allows the user's original request to continue. This ensures the user's session never breaks unexpectedly without exposing refresh mechanics to the browser.

### Device Pairing (Process 3)
- ~~**TTL for 6-digit codes:** The code is a critical resource. The `initDevice` call should store an `expiresAt` (e.g., 5 minutes) and the code must be strictly single-use.~~ [x] Handled
- ~~**Rate-limiting:** Rate-limit `initDevice` per IP or device fingerprint to prevent attackers from exhausting the pairing-code space and creating thousands of pending devices.~~ [x] Handled
- ~~**Replay Attacks:** The confirmation step `POST /api/devices/{code}/pair` must invalidate the code immediately after successful pairing to prevent replays if the player crashes and retries.~~ [x] Handled

**Implemented Solution (Multi-Layered Architecture):**
1. **TTL and Replay Prevention:** `initDevice` assigns a strict 5-minute expiration to all pairing codes. Upon successful pairing, the code is immediately nullified in the database (`setPairingCode(null)`), making replay attacks impossible. Expired codes trigger a `410 Gone` response prompting the TV to reboot.
2. **Global Panic Switch:** A sliding-window rate limiter is implemented at the controller level capping global pairing requests to 50 per minute to protect the database against mass botnet Write Storms.
3. **Hardware Fingerprint Exponential Backoff:** A custom `ExponentialRateLimiter` service tracks the Android TV's `androidId` (`Settings.Secure.ANDROID_ID`) and exponentially delays subsequent initialization attempts (e.g., $3^{attempts}$ seconds), locking out spamming devices for hours. The `androidId` is also permanently saved to the `Device` entity for tracking.

### Heartbeat (Process 4)
- ~~**Database Write Storms:** Every heartbeat writes `isOnline` and `lastSeen` directly to the relational database. For thousands of devices pinging every 30s, this will crush the database.~~ [x] Handled
- ~~**Robust Alternative:** Write heartbeats to a fast key-value store (like Redis) with a TTL. A separate, lower-frequency background worker can bulk-update the SQL database and publish aggregate status changes to Mercure.~~ [x] Handled (Replaced by 3-Phase Ping)

**Implemented Solution (3-Phase Ping-Pong Architecture):**
To support Serverless Databases (e.g., Neon DB) which charge by compute hour, the continuous heartbeat has been completely eliminated in favor of a zero-CPU event-driven architecture:
1. **Graceful Shutdown (Zero CPU):** The Android TV maintains a silent SSE connection to Mercure. It only fires an HTTP `/status` update when it first boots up or when the app is gracefully shut down. 
2. **Hourly Batch Ping:** To catch ungraceful shutdowns (e.g., power outages) and generate historical uptime analytics, a Symfony Command (`app:device:batch-ping`) runs once per hour. It pushes a global `{"action": "ping"}` event via Mercure to all online devices. The devices instantly reply via HTTP. The server performs a single bulk update and then goes back to sleep for 59 minutes.
3. **Just-In-Time Ping:** When a user schedules a playlist on the dashboard, the backend instantly issues a targeted Mercure Ping to the specific TV to verify its real-time presence before finalizing the schedule.

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
