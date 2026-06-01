# Plaisoram DevOps & Deployment Strategy

This document outlines the updated deployment architecture for the Plaisoram system. This strategy balances ease-of-use by using a managed Platform-as-a-Service (PaaS) for the backend API, while keeping costs low by using a raw Virtual Machine for the broadcasting service.

---

## 1. The Final Technology Stack

| Component | Provider / Technology | Purpose |
| :--- | :--- | :--- |
| **Backend API** | **DigitalOcean App Platform** | A fully managed PaaS. Automatically builds and runs your Symfony code directly from GitHub, exactly like Upsun, meaning zero server maintenance for you. |
| **Broadcasting** | **DigitalOcean Droplet (VM)** | Runs the standalone Mercure Hub. |
| **File Storage** | **DigitalOcean Spaces** | S3-compatible Object Storage for all media/video assets. |
| **Frontend** | **Vercel** | Hosts the Next.js Web Dashboard. |
| **Database** | **Neon** | Serverless PostgreSQL Database. |

---

## 2. Opinion on Broadcasting Resources (Mercure)

You asked for my opinion on which DigitalOcean compute resource is best for the Mercure VM. 

**Recommendation:** A **Basic Shared CPU Droplet ($4 to $6/month)**.

**Why?**
Mercure is written in Go and uses Server-Sent Events (SSE). It is incredibly lightweight and efficient. A basic $6 Droplet with 1GB of RAM can easily handle thousands of simultaneous active connections from your Android displays without breaking a sweat. You do not need a CPU-Optimized or Dedicated Droplet for this. 

---

## 3. The Execution Plan

We will execute this deployment in three phases:

### Phase 1: Infrastructure Provisioning (DigitalOcean)
1. **App Platform (API):** Connect your GitHub repository to DigitalOcean App Platform to deploy the Symfony backend. It will detect PHP automatically.
2. **Droplet (Mercure):** Spin up a Basic Ubuntu Droplet ($6/mo).
3. **Spaces:** Create a DigitalOcean Spaces bucket and generate Access Keys.

### Phase 2: Mercure Setup
Instead of a complex Docker setup for the backend, we only need to configure the lightweight Mercure hub on the VM.
1. SSH into the Droplet.
2. Download and run the pre-compiled Mercure binary.
3. Use a lightweight reverse proxy (like Caddy) to instantly provide HTTPS for the Mercure hub.

### Phase 3: Adapting Configuration
Update the `.env` variables in your App Platform dashboard so the Symfony backend can talk to the other services:
```env
# Database (Neon)
DATABASE_URL=postgres://...

# Storage (DO Spaces)
B2_BUCKET_NAME=your-do-space-name
B2_KEY_ID=your-do-access-key
B2_APP_KEY=your-do-secret-key
B2_ENDPOINT=https://fra1.digitaloceanspaces.com

# Mercure (Droplet)
MERCURE_URL=https://mercure.yourdomain.com/.well-known/mercure
MERCURE_PUBLIC_URL=https://mercure.yourdomain.com/.well-known/mercure
MERCURE_JWT_SECRET=your-secret-key
```

### Phase 4: Frontend Updates
Once the DigitalOcean App Platform assigns a URL to your backend (e.g., `https://plaisoram-api.ondigitalocean.app`), update Vercel with the new `NEXT_PUBLIC_API_URL` and `NEXT_PUBLIC_MERCURE_URL`.
