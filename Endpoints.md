# Plaisoram System Endpoints

> **Last Updated:** 2026-07-01
> **Scope:** Complete mapping of routes and API endpoints across the three main pillars of the Plaisoram ecosystem: **Plaisoram_Server** (Symfony 8 API Backend), **plaisoram_web** (Next.js Frontend & Proxy), and **Plaisoram_Player** (Android Kotlin Digital Signage Client).

---

## 1. Plaisoram_Server (Symfony API Backend)

Core REST API endpoints served by the Symfony backend.
Endpoints are explicitly demarcated as **[🌐 Public]** (accessible without JWT authentication, typically used for device pairing/bootstrapping) or **[🔒 Private]** (requires a valid JWT Bearer token and active workspace context).

### **Device Management (`/api/devices`)**
*Note: While firewall routing permits public access to certain Android Player endpoints (such as init, pairing status, and active playlist retrieval), internal controller logic strictly secures management endpoints (listing, pairing, deleting, power toggling) by enforcing authentication and workspace verification.*

- `GET    /api/devices` — Get all devices registered in the authenticated user's workspace. **[🔒 Private]**
- `POST   /api/devices/init` — Initialize a new Android device connection (receives `androidId`, `width`, `height`; returns pairing code and status; protected by global panic switch & exponential rate limiting). **[🌐 Public]**
- `POST   /api/devices/pair` — Pair a device using a 6-character pairing code (protected by exponential backoff per user). **[🔒 Private]**
- `GET    /api/devices/{code}/pair` — Check pairing token status via code (polled by Android Player; returns 410 Gone with reboot command if code expired). **[🌐 Public]**
- `GET    /api/devices/{id}/status` — Get connection status and token info for a specific device by ID. **[🌐 Public]**
- `POST   /api/devices/{id}/status` — Update connection status (heartbeat reporting `online` or `offline` status). **[🌐 Public]**
- `POST   /api/devices/{id}/publish` — Publish and push an active playlist to a device. **[🔒 Private]**
- `PUT/PATCH /api/devices/{id}` — Update a device's configuration settings (e.g., custom device name). **[🔒 Private]**
- `DELETE /api/devices/{id}` — Unpair and delete a device from the workspace. **[🔒 Private]**
- `POST   /api/devices/{id}/toggle-power` — Send a remote power toggle command to an active device via Mercure SSE. **[🔒 Private]**
- `POST   /api/devices/{id}/refresh` — Force a remote device to refresh its playlist and data via Mercure SSE. **[🔒 Private]**
- `GET    /api/devices/{id}/playlist` — Get the active playlist configuration, layout zones, and media assets for a specific device. **[🌐 Public]**

### **Media Management (`/api/media`)**
- `GET    /api/media` — Get all media assets in the workspace (optional query parameter `?folder_id={id|0}` to filter by root or specific folder). **[🔒 Private]**
- `POST   /api/media/upload` — Upload a new media file directly to server storage (optional `folder_id` in form data). **[🔒 Private]**
- `GET    /api/media/presigned-url` — Generate a pre-signed cloud upload URL (`?filename=&fileType=`). **[🔒 Private]**
- `POST   /api/media/confirm-upload` — Confirm successful cloud upload and register media entity in database (`filename`, `type`, `duration`, `folder_id`). **[🔒 Private]**
- `DELETE /api/media/{id}` — Delete a media asset from storage and workspace. **[🔒 Private]**
- `GET    /api/media/serve/{filename}` — Serve/stream a media file directly from storage with proper MIME type headers. **[🌐 Public]**

### **Media Folders (`/api/folders`)**
- `GET    /api/folders` — Get all media folders within the current workspace (`api_folders_index`). **[🔒 Private]**
- `POST   /api/folders` — Create a new media folder with name, description, and initial media items (`api_folders_create`). **[🔒 Private]**
- `PUT    /api/folders/{id}` — Update an existing media folder (`api_folders_update`). **[🔒 Private]**
- `DELETE /api/folders/{id}` — Delete a media folder (`api_folders_delete`). **[🔒 Private]**

### **Playlist Management (`/api/playlists`)**
- `GET    /api/playlists` — Get all playlists in the workspace (`api_playlists_index`). **[🔒 Private]**
- `GET    /api/playlists/{id}` — Get detailed configuration, layout, zones, and media items for a specific playlist (`api_playlists_show`). **[🔒 Private]**
- `POST   /api/playlists` — Create a new playlist (`api_playlists_create`). **[🔒 Private]**
- `PUT    /api/playlists/{id}` — Update an existing playlist's structure and zone assignments (`api_playlists_update`). **[🔒 Private]**
- `DELETE /api/playlists/{id}` — Delete a playlist from the workspace (`api_playlists_delete`). **[🔒 Private]**
- `POST   /api/playlists/{id}/duplicate` — Clone/duplicate an existing playlist within the workspace (`api_playlists_duplicate`). **[🔒 Private]**

### **Scheduling (`/api/schedules`)**
- `GET    /api/schedules` — Get all scheduled publications for the workspace. **[🔒 Private]**
- `POST   /api/schedules` — Create a new scheduled publication linking a playlist to a device with timezone and recurrence rules (`scheduled_at`, `recurrence_rule`). **[🔒 Private]**
- `DELETE /api/schedules/{id}` — Cancel and delete a scheduled publication. **[🔒 Private]**

### **User Profile & Workspace (`/api/profile`)**
- `GET    /api/profile` — Get current user profile details, workspace configuration, and API secret key (`api_profile_show`). **[🔒 Private]**
- `PUT    /api/profile` — Update user profile details (`firstName`, `lastName`, `email`, `phone`, `password`). **[🔒 Private]**
- `PUT    /api/profile/workspace` — Update workspace branding and localization settings (`companyName`, `timezone`, `logoUrl`). **[🔒 Private]**
- `GET    /api/profile/usage` — Get workspace resource metrics and storage usage statistics (`api_profile_usage`). **[🔒 Private]**

### **Authentication & Registration**
- `POST   /api/login_check` — Authenticate user credentials and receive a JWT access token & refresh token. **[🌐 Public]**
- `POST   /api/token/refresh` — Refresh an expired JWT access token using a valid refresh token (`api_refresh_token`). **[🌐 Public]**
- `POST   /api/register` — Register a new user account and initial workspace (`api_register`). **[🌐 Public]**
- `ALL    /logout` — Security route loader for session/token invalidation (`_security_logout`). **[🔒 Private]**

### **Real-Time Hub (Mercure SSE)**
- `GET    /.well-known/mercure` — Mercure Server-Sent Events (SSE) Hub endpoint for pushing live updates (device power toggle, refresh commands, playlist updates, and online/offline status changes) to web dashboards and connected players. **[🔒 Protected via JWT Topics]**

---

## 2. plaisoram_web (Next.js Frontend)

The web dashboard routes, server actions, and internal API proxy endpoints.

### **Internal API & Proxy Routes (`src/app/api`)**
- `POST   /api/logout` — Invalidation route that clears authentication cookies (`token` and `refresh_token`). **[🔒 Private]**
- `ALL    /api/[...slug]` — Next.js catch-all proxy route that transparently forwards requests to the Symfony Backend while attaching the JWT token (from `x-auth-token` header or `token` cookie) and `Accept-Language` localization header. **[🔒 Private]**

### **Server Actions (`src/actions`)**
- `loginAction(formData)` — Authenticates via `/api/login_check` and sets secure HTTP-only cookies (`token` for 15 mins, `refresh_token` for 14 days).
- `signupAction(formData)` — Registers a new user account via `/api/register`.
- `logoutAction()` — Server action to delete authentication cookies.
- `getProfileAction()` / `updateProfileAction(data)` / `updateWorkspaceAction(data)` / `uploadLogoAction(formData)` — Server actions interacting with `/api/profile` and `/api/media`.
- `setLocaleAction(locale)` — Sets the `NEXT_LOCALE` cookie for multi-language support (English/French).

### **Dashboard Pages (Client Routes)**
- `/` — Main Dashboard Overview
- `/login` — User Login Page
- `/signup` - User Registration Page
- `/devices` — Device Management List
- `/devices/add` — Add New Device Wizard & Pairing Code Input
- `/media` — Media Asset & Folder Library
- `/playlists` — Playlists Management List
- `/playlists/editLayout` — Interactive Multi-Zone TV Canvas Editor
- `/schedules` — Scheduling Calendar & Publication Overview
- `/apps` — Integrations and Apps Marketplace
- `/canvas` — Freeform Canvas Designer
- `/settings` — Main Settings Overview
- `/settings/profile` — User Profile Settings
- `/settings/companyinformations` — Organization & Workspace Settings
- `/settings/billing` — Billing & Subscription Management
- `/settings/logo` — Custom Branding & Logo Management
- `/settings/about` — About Plaisoram System
- `/settings/support` — Help & Technical Support

---

## 3. Plaisoram_Player (Android Kotlin Client)**

The Android digital signage player acts as a remote client utilizing Clean Architecture (Retrofit / Room / WorkManager). It interacts with the backend via the following REST endpoints defined in `PlaisoramApi.kt`:

### **Consumed API Endpoints**
- `POST   /api/devices/init` — Invoked on initial boot to register device hardware specs (`androidId`, `width`, `height`) and obtain a 6-character pairing code. **[🌐 Public]**
- `GET    /api/devices/{id}/status` — Polled continuously during the pairing screen flow to check if the device has been linked to a workspace by a user. **[🌐 Public]**
- `GET    /api/devices/{deviceId}/playlist` — Fetches the active playlist layout, zone definitions, and media asset URLs for background downloading and offline-resilient screen rendering. **[🌐 Public]**
