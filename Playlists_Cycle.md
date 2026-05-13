# The Plaisoram Playlist Cycle

This document explains the complete lifecycle of a Playlist in the Plaisoram ecosystem. It specifically clarifies how layout dimensions (width, height, x, and y coordinates) are preserved and respected by the digital player, even though the player's local Room database (`playlist_items`) does not store them.

## 1. Creation on the Server
- **Action:** A user creates a Playlist on the Plaisoram Web Dashboard.
- **Storage:** The layout is defined by adding **Zones**. Each Zone stores its positional data (`xPercent`, `yPercent`, `widthPercent`, `heightPercent`) in the Server's MySQL database (`Plaisoram_Server`).
- **Media Assignment:** Media items are linked to specific Zones via the `PlaylistMedia` and `Zone` entities.

## 2. Assignment & Real-Time Notification
- **Action:** The user assigns the newly created Playlist to a specific Device (Screen).
- **Server DB Update:** The Server updates the `currentPlaylistId` of the Device.
- **Mercure Broadcast:** The Server immediately broadcasts a real-time Server-Sent Event (SSE) via **Mercure** (`event: 'PlaylistUpdated'`) targeting the specific Device's topic.

## 3. Player Receives the Notification
- **SSE Listener:** The Android Player is always listening to its Mercure stream in the background (`PlayerViewModel.kt`).
- **Trigger Sync:** Upon receiving the `PlaylistUpdated` event, the Player immediately triggers the `SyncEngine` to download the new playlist data.

## 4. Fetching the Complete Layout Data
- **API Request:** The Player makes a `GET /api/devices/{id}/playlist` request to the Server.
- **JSON Payload:** The Server's `DeviceController` dynamically constructs and returns a comprehensive JSON payload mapped to a `PlaylistLayoutDto`. 
- **Payload Contents:** This JSON contains the `layoutType` and an array of `zones`. Crucially, **each zone in the JSON includes its exact `xPercent`, `yPercent`, `widthPercent`, and `heightPercent`**, along with the nested `media` details.

## 5. Dual Storage Strategy on the Player
This step explains why the Room database lacks layout columns. The Player uses a "Dual Storage" strategy to handle the JSON response (`PlaylistRepositoryImpl.kt`):

1. **Layout Storage (SharedPreferences):** 
   - The Player takes the entire raw `PlaylistLayoutDto` JSON response and serializes it directly into Android's **SharedPreferences** under the key `active_layout`. 
   - It is loaded into memory as a reactive `StateFlow` (`activeLayout`). SharedPreferences is perfect for this because the layout is a lightweight JSON object that dictates the structure of the screen.

2. **Media Synchronization Storage (Room DB SQLite):** 
   - The Player extracts just the media items from the zones and inserts them into the Room database's `playlist_items` table.
   - **Why?** The Room database is used *strictly* as a download queue for the `SyncEngine`. It tracks the heavy lifting: physical file downloads, local file paths (`localPath`), and download states (`PENDING`, `DOWNLOADING`, `COMPLETED`). It doesn't need to care about where the video will be placed on the screen; it only cares about successfully downloading the `.mp4` or `.jpg` file to the local disk.

## 6. Rendering on the Screen
- **Drawing the Grid:** The Player's UI components (`MultiZoneLayout`, `ZonedLayoutRenderer`) observe the `activeLayout` StateFlow from SharedPreferences. They iterate through the JSON zones, applying the exact `widthPercent`, `heightPercent`, `xPercent`, and `yPercent` to draw the layout grid dynamically.
- **Filling the Grid:** For each drawn zone, the UI checks the assigned `mediaId`. It queries the Room database to get the `localPath` of the downloaded file. If the file is `COMPLETED`, the media is rendered inside that specific zone.

---

### Summary
Your observation was completely correct: the Room database doesn't store layout information. 

Instead, the Player relies on a highly efficient separation of concerns:
- **SharedPreferences** handles the **Layout State** (Where things go).
- **Room Database** handles the **Media Sync State** (Are the files downloaded?).
