# 🚀 Plaisoram System — Verified Tech Stack & Dependency Matrix

> **Last Audited:** July 23, 2026  
> **Repository Root:** `c:\Users\zrirak\Desktop\Software\Plaisoram`

---

## 🛠️ Multi-Project Architecture

```
                  ┌──────────────────────────────┐
                  │   Next.js 16 Web Dashboard   │ (plaisoram_web)
                  └──────────────┬───────────────┘
                                 │ REST API (JWT)
                                 ▼
                  ┌──────────────────────────────┐
                  │    Symfony 8 PHP Backend     │ (Plaisoram_Server)
                  └──────┬────────────────┬──────┘
                         │                │
          Mercure SSE    │                │ HTTP REST / Sockets
          (Android Player)                ▼
                         │     ┌─────────────────────┐
                         │     │ Node.js Realtime 2.0│ (Plaisoram_Realtime)
                         │     └──────────┬──────────┘
                         ▼                │ Socket.IO (Web UI)
              ┌─────────────────────┐     │
              │ Android TV Player 1.0│◄────┘
              └─────────────────────┘ (Plaisoram_Player)
```

---

## 1. 🌐 Web Dashboard (`plaisoram_web`)
> **Path:** `c:\Users\zrirak\Desktop\Software\Plaisoram\plaisoram_web`

| Technology / Library | Version | Description |
|---|---|---|
| **Framework** | **Next.js 16.2.11** | React Framework (App Router) |
| **Core UI Library** | **React 19.2.8** & **React-DOM 19.2.8** | Component Engine |
| **Styling System** | **Tailwind CSS v4** (`@tailwindcss/postcss ^4`) | Modern Utility CSS |
| **Language** | **TypeScript ^7.0** | Static Type Safety |
| **Linter / Formatter** | **Biome 2.5.5** | High-performance JS/TS Linter |
| **Data Fetching & Cache** | **`@tanstack/react-query` ^5.101.4** | Server State Management |
| **Real-time Client** | **`socket.io-client` ^4.8.3** | WebSocket Connection to Realtime Gateway |
| **Internationalization** | **`next-intl` ^4.13.3** | Multi-language Routing & Translations (FR/EN) |
| **Icons & UI** | **`lucide-react` ^1.26.0** & **`sonner` ^2.0.7** | Lucide Icon Library & Toast Notifications |
| **Hosting & CI/CD** | **Vercel** | Edge Network Deployment |

---

## 2. ⚡ Backend API (`Plaisoram_Server`)
> **Path:** `c:\Users\zrirak\Desktop\Software\Plaisoram\Plaisoram_Server`

| Technology / Library | Version | Description |
|---|---|---|
| **Runtime Language** | **PHP >= 8.4** | High-Performance Server Language |
| **Framework** | **Symfony 8.0.\*** | Modular Hexagonal API Architecture |
| **Database & ORM** | **Doctrine ORM ^3.6** (`doctrine-bundle ^3.2`) | Database Layer & Entity Mapping |
| **Authentication** | **Lexik JWT ^3.2** & **Refresh Token ^2.0** | Dual Access Token Security |
| **Object Storage** | **Flysystem AWS S3 v3 ^3.32** | Backblaze B2 S3-Compatible Storage Adapter |
| **Real-time Push Engine** | **`symfony/mercure-bundle` ^0.4.2** | Mercure Hub SSE Dispatcher |
| **Dual Notifier** | **`MercureDeviceNotifier`** | Simultaneous Mercure SSE + Socket.IO Event Forwarding |
| **Testing** | **PHPUnit ^13.0** & **Dama Doctrine Test Bundle ^8.6** | Automated Test Framework |
| **Hosting** | **DigitalOcean App Platform** | Production Containerized PHP Environment |

---

## 3. 📺 Android TV Player (`Plaisoram_Player`)
> **Path:** `c:\Users\zrirak\Desktop\Software\Plaisoram\Plaisoram_Player`

| Technology / Library | Version | Description |
|---|---|---|
| **Language** | **Kotlin 2.0.21** (JDK 17) | Primary Android Development Language |
| **Android SDK** | **Target / Compile SDK 35** (Min SDK 24) | Android TV Kiosk Platform |
| **Build System** | **Android Gradle Plugin 8.7.2** (KSP 2.0.21) | Gradle Kotlin DSL (`build.gradle.kts`) |
| **Media Playback Engine** | **Jetpack Media3 / ExoPlayer 1.4.1** | Hardware-Accelerated Video & Image Playback |
| **UI Framework** | **Jetpack Compose (BOM 2024.11.00)** | Modern Declarative UI |
| **Local Database & Cache** | **Android Room 2.6.1** | SQLite Local Storage for Offline Playback |
| **Background Scheduler** | **Android WorkManager 2.10.0** | Delta Sync & Media Download Worker |
| **Dependency Injection** | **Google Dagger Hilt 2.52** | Native Android Dependency Injector |
| **HTTP & REST** | **Retrofit 2.11.0** & **OkHttp 4.12.0** | Network Request & Image Loader (Coil 2.6.0) |
| **Sockets & SSE** | **Socket.io Java Client 2.1.1** & **OkHttp SSE** | Dual Real-time Connection Engine |

---

## 4. 🔄 Real-time Gateway (`Plaisoram_Realtime`)
> **Path:** `c:\Users\zrirak\Desktop\Software\Plaisoram\Plaisoram_Realtime`

| Technology / Library | Version | Description |
|---|---|---|
| **Runtime & Manager** | **Node.js >= 18.0.0** (`pnpm 10.5.2`) | Asynchronous Event-Driven JS Engine |
| **Framework** | **Express ^4.21.0** | Microservice HTTP Router |
| **WebSocket Engine** | **Socket.IO ^4.8.0** | Full-duplex Bi-directional Communication |
| **Language** | **TypeScript ^5.7.0** | Strict Type Checking (`tsx ^4.19.0`) |
| **Logging** | **Pino ^9.0.0** (`pino-pretty ^11.0.0`) | Zero-Overhead JSON Logger |
| **Security** | **JsonWebToken ^9.0.2** | Gateway Token Verification |

---

## ☁️ Shared Infrastructure Services

| Service | Provider | Production Configuration |
|---|---|---|
| **Frontend Web Hosting** | **Vercel** | Production Next.js 16 Deployment |
| **Backend API Server** | **DigitalOcean** | Production Symfony 8 Environment |
| **Realtime Gateway** | **DigitalOcean** | Production Socket.IO Container |
| **Object Storage** | **Backblaze B2** | S3-Compatible Bucket for High-Res Media & Videos |
| **Database** | **Managed MySQL** | Relational Database Instance |
| **Realtime Hub** | **Mercure Hub** | Open-source SSE Hub |
