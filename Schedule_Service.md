# Plaisoram Scheduling System Architecture

This document outlines the complete, end-to-end architecture and implementation details for the Scheduling and Publishing system in Plaisoram.

## 1. Frontend Procedure (Next.js)

The user interface strictly separates the concept of **saving** a layout from **scheduling/publishing** it.

### A. Saving a Playlist (`/playlists/editLayout`)
- The "Edit Layout" page is strictly for drafting and updating.
- Clicking the **Save** button triggers either a `POST` (for new playlists) or `PUT` (for existing playlists) request to `/api/playlists`.
- There is **no publish button** on the edit page.

### B. Scheduling & Publishing (`/playlists`)
- The **Schedule & Publish** action is available on the main Playlists list page via a dedicated action button on each playlist card.
- Clicking this opens the `PublishScheduleModal`.
- **Validation**: 
  - The modal enforces the selection of a valid target device.
  - If "Custom" is selected, the user must pick a date and exact time (HH:mm). 
  - Past dates are strictly forbidden by frontend validation.
- **Execution**: 
  - `Immediate`: Sends a request to `/api/devices/{id}/publish`.
  - `Custom`: Sends a `POST` payload to `/api/schedules` containing the `playlist_id`, `device_id`, UTC `scheduled_at` string, and timezone.

### C. Calendar View (`/schedules`)
- A full interactive calendar grid visualizes all upcoming schedules by fetching from `GET /api/schedules`.

---

## 2. Backend Server Logic (Symfony)

The backend handles the persistence, validation, and execution of schedules.

### A. Data Persistence (`PublishSchedule` Entity)
- A Doctrine entity that links a `Device` and a `Playlist`.
- Stores `scheduledAt` (UTC timestamp), `timezone`, and `status` (`pending`, `completed`, `failed`).
- Optimized with an index on `[status, scheduled_at]` to prevent full table scans when the worker polls for due schedules.

### B. API Controllers
- **`ScheduleController`**: 
  - `POST /api/schedules`: Validates inputs. Enforces that `scheduled_at` cannot be in the past (allowing a 1-minute network buffer). Creates a `pending` schedule.
  - `GET /api/schedules`: Retrieves all schedules for the user's workspace.

### C. Execution Engine (Symfony Messenger)
- **Architecture**: The polling loop has been replaced by a Push-Based Delayed Message architecture using `symfony/messenger`.
- **Procedure**:
  1. When a schedule is created, `ScheduleController` dispatches a `PublishPlaylistMessage` with a `DelayStamp` specifying the exact milliseconds until execution.
  2. The message is serialized and stored in the Doctrine transport (`messenger_messages` table).
  3. A long-running worker process (`run-worker.bat` running `php bin/console messenger:consume async`) picks up the message at the exact millisecond it becomes due.
  4. The `PublishPlaylistMessageHandler` receives the message, validates the schedule is still `pending`, updates the target `Device` entity to point to the new `currentPlaylist`, and marks the schedule as `completed`.
  5. The handler then triggers the real-time Mercure update.
- **Resilience**: If the Mercure push fails (or any other exception occurs), Messenger automatically catches it, rolls back the transaction, and retries the message using exponential backoff. If it fails repeatedly, it goes to the `failed` transport (Dead Letter Queue). This guarantees zero stuck `processing` zombie rows.

### D. Real-time Screen Updates (Mercure)
- Immediately after updating the database, the worker publishes a JSON update payload to the **Mercure Hub**.
- The Android players are subscribed to their specific SSE channel (`/device/{id}/updates`).
- Upon receiving the signal, the player immediately syncs its local storage and transitions to the new scheduled playlist without manual intervention.
