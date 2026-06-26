# Plaisoram System Endpoints

This document maps out all the defined endpoints and routes across the three main pillars of the Plaisoram system: the Symfony Backend Server, the Next.js Web Dashboard, and the Android Player App.

---

## 1. Plaisoram_Server (Symfony API Backend)

These are the core REST API endpoints served by the Symfony backend. 
Endpoints are explicitly marked as **[🌐 Public]** (no authentication required) or **[🔒 Private]** (requires a valid JWT Bearer token).

### **Device Management (`/api/devices`)**
*Note: While the firewall allows public access to these routes for Android Player communication, internal controller logic secures sensitive endpoints (like listing devices) by requiring a valid user/workspace context.*
- `GET    /api/devices` - Get all devices for the current workspace. **[🌐 Public]**
- `POST   /api/devices/init` - Initialize a new device connection. **[🌐 Public]**
- `POST   /api/devices/pair` - Pair a device using a code. **[🌐 Public]**
- `GET    /api/devices/{code}/pair` - Check pairing status via code. **[🌐 Public]**
- `POST   /api/devices/{id}/heartbeat` - Send a heartbeat signal from the player. **[🌐 Public]**
- `GET    /api/devices/{id}/status` - Get the current connection status of a device. **[🌐 Public]**
- `POST   /api/devices/{id}/publish` - Publish a playlist to a device. **[🌐 Public]**
- `PATCH  /api/devices/{id}` - Update a device's settings (e.g., name, resolution). **[🌐 Public]**
- `DELETE /api/devices/{id}` - Remove a device from the workspace. **[🌐 Public]**
- `POST   /api/devices/{id}/toggle-power` - Send a power toggle command to a device. **[🌐 Public]**
- `POST   /api/devices/{id}/refresh` - Force a device to refresh its data. **[🌐 Public]**
- `POST   /api/devices/{id}/publish-media` - Instantly publish media to a device. **[🌐 Public]**
- `GET    /api/devices/{id}/playlist` - Get the active playlist data for a specific device. **[🌐 Public]**

### **Media Management (`/api/media`)**
- `GET    /api/media` - Get all media assets in the workspace. **[🔒 Private]**
- `POST   /api/media/upload` - Upload a new media file. **[🔒 Private]**
- `GET    /api/media/presigned-url` - Request a pre-signed URL for direct cloud upload. **[🔒 Private]**
- `POST   /api/media/confirm-upload` - Confirm a successful cloud upload. **[🔒 Private]**
- `DELETE /api/media/{id}` - Delete a media asset. **[🔒 Private]**
- `GET    /api/media/serve/{filename}` - Serve/stream a media file. **[🌐 Public]**

### **Media Folders (`/api/folders`)**
- `GET    /api/folders` - Get all media folders. **[🔒 Private]**
- `POST   /api/folders` - Create a new media folder. **[🔒 Private]**
- `PUT    /api/folders/{id}` - Update a media folder. **[🔒 Private]**
- `DELETE /api/folders/{id}` - Delete a media folder. **[🔒 Private]**

### **Playlist Management (`/api/playlists`)**
- `GET    /api/playlists` - Get all playlists in the workspace. **[🔒 Private]**
- `GET    /api/playlists/{id}` - Get detailed configuration, zones, and media for a specific playlist. **[🔒 Private]**
- `POST   /api/playlists` - Create a new playlist. **[🔒 Private]**
- `PUT    /api/playlists/{id}` - Update an existing playlist. **[🔒 Private]**
- `DELETE /api/playlists/{id}` - Delete a playlist. **[🔒 Private]**
- `POST   /api/playlists/{id}/duplicate` - Duplicate an existing playlist. **[🔒 Private]**

### **Scheduling (`/api/schedules`)**
- `GET    /api/schedules` - Get all scheduled publications. **[🔒 Private]**
- `POST   /api/schedules` - Create a new scheduled publication. **[🔒 Private]**
- `DELETE /api/schedules/{id}` - Delete a scheduled publication. **[🔒 Private]**

### **User Profile (`/api/profile`)**
- `GET    /api/profile` - Get current user profile details. **[🔒 Private]**
- `PUT    /api/profile` - Update user profile details. **[🔒 Private]**
- `PUT    /api/profile/workspace` - Update user's workspace settings. **[🔒 Private]**
- `GET    /api/profile/usage` - Get workspace resource usage metrics. **[🔒 Private]**

### **Authentication & Registration**
- `POST   /api/login_check` - Authenticate and receive a JWT. **[🌐 Public]**
- `POST   /api/token/refresh` - Refresh an expired JWT. **[🌐 Public]**
- `POST   /api/register` - Register a new user account. **[🌐 Public]**

---

## 2. plaisoram_web (Next.js Frontend)

The web dashboard routes and its internal API proxy endpoints.

### **Internal API Routes (`src/app/api`)**
- `POST   /api/logout` - Clears the authentication cookies. **[🔒 Private]**
- `ALL    /api/[...slug]` - Next.js proxy route that securely forwards all requests to the Symfony Server while attaching the JWT token from cookies. **[🔒 Private]**

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
- `POST   /api/devices/init` - Used to register the device upon first boot and retrieve the pairing code. **[🌐 Public]**
- `GET    /api/devices/{id}/status` - Used to verify the connection status during the pairing flow. **[🌐 Public]**
- `GET    /api/devices/{id}/playlist` - Used to fetch the active playlist layout, zones, and media URLs to download and play. **[🌐 Public]**
- `POST   /api/devices/{id}/heartbeat` - Background service ping to notify the server that the device is online. **[🌐 Public]**
