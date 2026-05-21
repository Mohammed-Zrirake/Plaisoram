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

### C. Execution Engine (Worker Process)
- **Command**: `ProcessSchedulesCommand` (`app:process-schedules`)
- This command is designed to be run on a Cron Job (e.g., every minute) or via a daemonized worker (like Symfony Messenger).
- **Procedure**:
  1. Queries the database for schedules where `status = 'pending'` and `scheduled_at <= NOW()`.
  2. For each schedule, it updates the target `Device` entity to point to the new `currentPlaylist`.
  3. Marks the schedule as `completed` to maintain an audit log.
  4. Flushes changes to the database.

### D. Real-time Screen Updates (Mercure)
- Immediately after updating the database, the worker publishes a JSON update payload to the **Mercure Hub**.
- The Android players are subscribed to their specific SSE channel (`/device/{id}/updates`).
- Upon receiving the signal, the player immediately syncs its local storage and transitions to the new scheduled playlist without manual intervention.
