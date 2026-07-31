# Plaisoram Digital Signage - Complete Publishing Architecture & Matrix

This document provides a comprehensive technical guide to all content publishing actions across the Plaisoram ecosystem (**Devices Page**, **Playlists Page**, and **Media Page**), covering both **Immediate** and **Scheduled** publishing for **Playlists** and **Single Media items (Images / Videos)**.

---

## 1. Complete Publishing Possibilities Matrix

```
┌─────────────────┬─────────────────┬────────────────────┬────────────────────────────────────────────────────────┐
│ Origin Page     │ Content Type    │ Timing Mode        │ Backend & Database Behavior                            │
├─────────────────┼─────────────────┼────────────────────┼────────────────────────────────────────────────────────┤
│ Écrans          │ Playlist        │ Immediate          │ `currentPlaylistId = ID`, `currentMediaId = null`      │
│ (Devices Page)  │ Playlist        │ Scheduled          │ Inserts `PublishSchedule` record for `playlist_id`     │
│                 │ Media (Image/Video)│ Immediate       │ `currentMediaId = ID`, `currentPlaylistId = null` (0 DB Playlists) │
│                 │ Media (Image/Video)│ Scheduled       │ Inserts `PublishSchedule` for single-media shell       │
├─────────────────┼─────────────────┼────────────────────┼────────────────────────────────────────────────────────┤
│ Playlists       │ Playlist        │ Immediate          │ `currentPlaylistId = ID`, `currentMediaId = null`      │
│ (Playlists Page)│ Playlist        │ Scheduled          │ Inserts `PublishSchedule` record for `playlist_id`     │
├─────────────────┼─────────────────┼────────────────────┼────────────────────────────────────────────────────────┤
│ Médias          │ Media (Image/Video)│ Immediate       │ `currentMediaId = ID`, `currentPlaylistId = null` (0 DB Playlists) │
│ (Media Page)    │ Media (Image/Video)│ Scheduled       │ Inserts `PublishSchedule` for single-media shell       │
└─────────────────┴─────────────────┴────────────────────┴────────────────────────────────────────────────────────┘
```

---

## 2. Page-by-Page Workflow Breakdown

### A. Écrans Page (`/devices`)

* **Action**: User clicks **PUBLIER** on a specific screen row (Target Device is pre-selected).
* **Modal**: `PublishScheduleModal` opens in **Device Mode** (`device = selectedDevice`).
* **Possibilities**:
  1. **Publish Playlist (Immediate)**:
     * **Payload**: `{ playlist_id: 12, duration_minutes: 60 }`
     * **API Endpoint**: `POST /api/devices/{id}/publish`
     * **Result**: Sets `currentPlaylistId = 12` and `currentMediaId = null`. Real-time socket broadcasts the playlist name + expiration time.
  2. **Publish Playlist (Scheduled)**:
     * **Payload**: `{ playlist_id: 12, device_id: 18, scheduled_at: "ISO_STRING", timezone: "..." }`
     * **API Endpoint**: `POST /api/schedules`
     * **Result**: Adds a pending schedule entry. When the schedule time arrives, the execution worker activates the playlist.
  3. **Publish Single Media - Image or Video (Immediate)**:
     * **Payload**: `{ media_id: 5, duration_minutes: 60 }`
     * **API Endpoint**: `POST /api/devices/{id}/publish`
     * **Result**: Direct media push. Sets `currentMediaId = 5` and `currentPlaylistId = null`. **0 new playlists are created in PostgreSQL**. Real-time socket broadcasts the media file name + expiration time.
  4. **Publish Single Media - Image or Video (Scheduled)**:
     * **Payload**: Creates a scheduled queue entry for the media file.
     * **Result**: Scheduled queue entry is queued without polluting the user's permanent playlist library.

---

### B. Playlists Page (`/playlists`)

* **Action**: User clicks **PUBLIER** on a Playlist card.
* **Modal**: `PublishScheduleModal` opens in **Playlist Mode** (`playlistId = 12`, `playlistName = "Météo_Actualités"`).
* **Possibilities**:
  1. **Publish Playlist (Immediate)**:
     * **Payload**: `{ playlist_id: 12, duration_minutes: total_playlist_duration }`
     * **API Endpoint**: `POST /api/devices/{deviceId}/publish`
     * **Result**: Instantly replaces active content on target screen with Playlist #12.
  2. **Publish Playlist (Scheduled)**:
     * **Payload**: `{ playlist_id: 12, device_id: targetDeviceId, scheduled_at: "ISO_STRING" }`
     * **API Endpoint**: `POST /api/schedules`
     * **Result**: Schedules Playlist #12 for future activation on the target screen.

---

### C. Médias Page (`/media`)

* **Action**: User clicks **PUBLIER** on an Image or Video card.
* **Modal**: `PublishMediaModal` opens with pre-selected media (`media = selectedMedia`).
* **Possibilities**:
  1. **Publish Media (Immediate)**:
     * **Payload**: `{ media_id: 5, duration_minutes: overrideDuration }`
     * **API Endpoint**: `POST /api/devices/{targetDeviceId}/publish`
     * **Result**: Direct media push. Sets `currentMediaId = 5` and `currentPlaylistId = null`. **0 new playlists are created in PostgreSQL**. Screen plays the image/video immediately.
  2. **Publish Media (Scheduled)**:
     * **Payload**: Schedules the image/video for future playback on the target screen.

---

## 3. How Single Media is Handled WITHOUT Flooding the Database

To prevent database clutter and avoid flooding the user's **Playlists** list with hundreds of random single-item playlists, Plaisoram uses a **Dual-Reference Entity Model** in PostgreSQL:

### Architectural Mechanism:

1. **Native Dual Fields on Device Entity (`Device.php`)**:
   The PostgreSQL `Device` entity contains two distinct nullable relationship fields:
   * `currentPlaylistId` (Integer, Nullable)
   * `currentMediaId` (Integer, Nullable)

2. **Direct Media Publish (Immediate)**:
   When an image or video is published directly (Immediate Mode):
   * The frontend sends `{ media_id: 5, duration_minutes: 60 }`.
   * Symfony executes `publishMedia()`:
     ```php
     $device->setCurrentMediaId($mediaId);      // Set to 5
     $device->setCurrentPlaylistId(null);       // Set to NULL
     $device->setUpdatedAt(new \DateTimeImmutable());
     ```
   * **Outcome**: **Zero new rows are created in the `playlist` or `section` database tables**. The database remains 100% clean.

3. **Active Payload Resolution (`DeviceMapper.php`)**:
   When the Android TV player or web dashboard requests the active content for a screen, Symfony evaluates the priority order:
   * **If `currentMediaId` is present**: Compiles a single-item media payload directly from the `Media` entity record.
   * **If `currentPlaylistId` is present**: Resolves the full multi-section playlist record.
   * **If both are null**: Falls back to the workspace **Default** playlist.

4. **Real-Time Display on Dashboard**:
   * If `currentMediaId` is set: Dashboard displays **Media Name** (e.g. `phl_logo`) with a blue indicator dot + end date.
   * If `currentPlaylistId` is set: Dashboard displays **Playlist Name** + end date.
   * If both are null: Dashboard displays **`Default`** badge.

---

## 4. Summary of Benefits

* **Database Hygiene**: No single-media junk playlists pollute the PostgreSQL database or the user's Playlists UI.
* **Instant Speed**: Direct media pushes bypass playlist rendering pipelines and execute in **0 milliseconds**.
* **Real-time Synchronization**: All open dashboard windows receive real-time Socket.io updates for single media and playlists alike.
