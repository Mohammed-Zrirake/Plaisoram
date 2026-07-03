# Immediate Override & Schedule Shifting Architectural Concerns

**Date:** 2026-07-03  
**Status:** Temporary Development Logic (Not Production Ready)  
**Module:** `DeviceManager` & `ScheduleManager` (Backend)

---

## 1. Current Dev Implementation Overview
To support ad-hoc immediate media and playlist overrides during development while preserving the relative playback order of pre-programmed playlists, the following adjustments were made:

1. **Disabled 1-Hour Verification Timer:**  
   In `ScheduleManager::createSchedule()`, the dispatching of the `VerifyDeviceStatusMessage` (scheduled 1 hour before launch) has been commented out to prevent premature status checks and queue clutter during dynamic time shifting.

2. **Immediate Override Shifting (`applyImmediateOverrideShift`):**  
   When an immediate publish action occurs with a duration of $D$ minutes (`durationMinutes > 0`):
   * **Rerun Current Playlist:** If the device was already playing a playlist (`currentPlaylistId !== null`), a new `PublishSchedule` is created to rerun that exact playlist at `now + D minutes`.
   * **Shift Future Timelines:** All existing upcoming schedules (`scheduledAt > now`, status = `pending`) for the device have their `scheduledAt` database timestamps shifted forward by $+D$ minutes (`$oldDate->modify("+{$durationMinutes} minutes")`).

---

## 2. Major Production Architectural Hazards (Why This Must Be Changed Before Prod)

While shifting dates satisfies basic sequential ordering in development, it introduces severe operational and system risks that will break production signage networks:

### A. Real-World Time Synchronization Loss (Schedule Drift)
In commercial digital signage (pharmacies, banks, retail chains), campaigns are strictly bound to real-world business hours (e.g., breakfast promos 08:00–11:00, lunch promos 12:00–14:00, closing warnings 18:30–19:00).
* **The Hazard:** Shifting schedules by $+D$ minutes causes cumulative drift. If 3 staff members push 10-minute immediate broadcasts throughout the day, the evening closing announcement will shift by $+30$ minutes, playing to an empty store after doors are locked.

### B. Recurrence Rule & Cron Collisions
`PublishSchedule` supports recurring rules (e.g., daily at 14:00).
* **The Hazard:** If today's 14:00 schedule is shifted to 14:20, what happens tomorrow? If tomorrow's schedule spawns at 14:00, or if today's evening schedule drifts past midnight into tomorrow morning, overlapping schedule collisions will corrupt database queries and screen playback.

### C. Multi-Screen De-synchronization (Visual Chaos)
Venues often deploy multiple screens side-by-side or across zones programmed to play synchronized marketing videos at exact timestamps.
* **The Hazard:** If a manager pushes a quick override to Screen #1 only, Screen #1's future timeline shifts permanently while Screens #2, #3, and #4 remain locked to the real-world clock. The venue loses visual synchronization.

### D. Message Broker Queue Desynchronization (The Silent Killer)
When a schedule is initially created, Symfony Messenger dispatches a `PublishPlaylistMessage` with a `DelayStamp` calculated as `($scheduledAt - time()) * 1000`.
* **The Hazard:** Shifting `scheduledAt` in PostgreSQL **does not alter or cancel the pending timer residing in the asynchronous message queue (e.g., RabbitMQ/Redis/DB queue)**.
* In production, the old queue message will fire at the **original unshifted time**, while the database timestamp indicates the shifted time. This results in race conditions where the server attempts to publish playlists at unintended times, causing screen flickering and state corruption.

---

## 3. Recommended Production Architecture: The "Preemption & Fallback" Layer

Before releasing to production, the timeline shifting logic must be replaced with an industry-standard **Time-Boxed Overlay Model**:

1. **Immutable Future Timelines:** Never alter upcoming `scheduledAt` dates in the database when an immediate override is published.
2. **Fallback State Restoration:** When an immediate publish is triggered for $D$ minutes, record the active `currentPlaylistId` in a temporary fallback column or memory state.
3. **Timer Expiration Handling:** When the $D$-minute override expires:
   * Check if a scheduled program launched during the window. If so, let the scheduled program continue.
   * If not, restore the fallback playlist without shifting any future schedules.
4. **Hard Preemption:** If an immediate broadcast is playing when the clock strikes a programmed `scheduledAt` boundary, the scheduled program must hard-preempt (cut off) the immediate broadcast to maintain clock synchronization.
