# Plaisoram System Endpoints

This document maps out all the defined endpoints and routes across the three main pillars of the Plaisoram system: the Symfony Backend Server, the Next.js Web Dashboard, and the Android Player App.

---

## 1. Plaisoram_Server (Symfony API Backend)

These are the core REST API endpoints served by the Symfony backend.

### **Device Management (`/api/devices`)**
- `GET    /api/devices` - Get all devices for the current workspace.
- `POST   /api/devices/init` - Initialize a new device connection.
- `POST   /api/devices/pair` - Pair a device using a code.
- `GET    /api/devices/{code}/pair` - Check pairing status via code.
- `POST   /api/devices/{id}/heartbeat` - Send a heartbeat signal from the player.
- `GET    /api/devices/{id}/status` - Get the current connection status of a device.
- `POST   /api/devices/{id}/publish` - Publish a playlist to a device.
- `PATCH  /api/devices/{id}` - Update a device's settings (e.g., name, resolution).
- `DELETE /api/devices/{id}` - Remove a device from the workspace.
- `POST   /api/devices/{id}/toggle-power` - Send a power toggle command to a device.
- `POST   /api/devices/{id}/refresh` - Force a device to refresh its data.
- `POST   /api/devices/{id}/publish-media` - Instantly publish media to a device.
- `GET    /api/devices/{id}/playlist` - Get the active playlist data for a specific device.

### **Media Management (`/api/media`)**
- `GET    /api/media` - Get all media assets in the workspace.
- `POST   /api/media/upload` - Upload a new media file.
- `GET    /api/media/presigned-url` - Request a pre-signed URL for direct cloud upload.
- `POST   /api/media/confirm-upload` - Confirm a successful cloud upload.
- `DELETE /api/media/{id}` - Delete a media asset.
- `GET    /api/media/serve/{filename}` - Serve/stream a media file.

### **Media Folders (`/api/folders`)**
- `GET    /api/folders` - Get all media folders.
- `POST   /api/folders` - Create a new media folder.
- `PUT    /api/folders/{id}` - Update a media folder.
- `DELETE /api/folders/{id}` - Delete a media folder.

### **Playlist Management (`/api/playlists`)**
- `GET    /api/playlists` - Get all playlists in the workspace.
- `GET    /api/playlists/{id}` - Get detailed configuration, zones, and media for a specific playlist.
- `POST   /api/playlists` - Create a new playlist.
- `PUT    /api/playlists/{id}` - Update an existing playlist.
- `DELETE /api/playlists/{id}` - Delete a playlist.
- `POST   /api/playlists/{id}/duplicate` - Duplicate an existing playlist.

### **Scheduling (`/api/schedules`)**
- `GET    /api/schedules` - Get all scheduled publications.
- `POST   /api/schedules` - Create a new scheduled publication.

### **User Profile (`/api/profile`)**
- `GET    /api/profile` - Get current user profile details.
- `PUT    /api/profile` - Update user profile details.
- `PUT    /api/profile/workspace` - Update user's workspace settings.

### **Authentication & Registration**
- `POST   /api/login_check` (Provided by LexikJWTAuthenticationBundle)
- `POST   /api/token/refresh` (Provided by GesdinetJWTRefreshTokenBundle)
- `POST   /api/register` - Register a new user account.

### **Player Config**
- `GET    /api/player/config/{macAddress}` - Fetch remote configuration based on the player's MAC address.

---

## 2. plaisoram_web (Next.js Frontend)

The web dashboard routes and its internal API proxy endpoints.

### **Internal API Routes (`src/app/api`)**
- `POST   /api/logout` - Clears the authentication cookies.
- `ALL    /api/[...slug]` - Next.js proxy route that securely forwards all requests to the Symfony Server while attaching the JWT token from cookies.

### **Dashboard Pages (Client Routes)**
- `/` - Main Dashboard Overview
- `/login` - Login Page
- `/signup` - Registration Page
- `/devices` - Device Management List
- `/devices/add` - Add New Device Wizard
- `/media` - Media Asset Library
- `/playlists` - Playlists Management List
- `/playlists/editLayout` - Interactive Playlist / TV Canvas Editor
- `/schedules` - Scheduling Calendar / Overview
- `/apps` - Integrations and Apps
- `/canvas` - Freeform Canvas
- `/settings` - Main Settings Panel
- `/settings/profile` - User Profile Settings
- `/settings/companyinformations` - Organization Settings
- `/settings/billing` - Billing & Subscription
- `/settings/logo` - Custom Branding
- `/settings/about` - About Plaisoram
- `/settings/support` - Support and Help

---

## 3. Plaisoram_Player (Android App)

The Android digital signage player acts as a client. It primarily communicates with the server using the following endpoint definitions via Retrofit / OkHttp:

### **Consumed API Endpoints**
- `POST   /api/devices/init` - Used to register the device upon first boot and retrieve the pairing code.
- `GET    /api/devices/{id}/status` - Used to verify the connection status during the pairing flow.
- `GET    /api/devices/{id}/playlist` - Used to fetch the active playlist layout, zones, and media URLs to download and play.
- `POST   /api/devices/{id}/heartbeat` - Background service ping to notify the server that the device is online.
