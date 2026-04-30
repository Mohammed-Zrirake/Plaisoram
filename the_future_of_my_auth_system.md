# The Future of the Plaisoram Auth System

The current authentication architecture is **Valid, Framework-compatible, and Production-usable**. We made the correct architectural decisions to keep the system lean and maintainable by avoiding over-engineered features like dedicated session tables, token family tracking, and complex device-level revocation before they are truly needed.

However, as the platform scales, there are a few lightweight, high-value fixes we should implement to harden the security model without exploding the schema complexity.

---

## 1. Refresh Token Hashing (Security)

> [!WARNING]
> Currently, the `refresh_tokens` table stores the tokens in plain text (`varchar(128)`). If the database leaks, an attacker instantly gains access to active refresh tokens.

**The Fix:**
Configure the `gesdinet/jwt-refresh-token-bundle` to hash the token before storing it in the database. 
- **Impact:** Zero new tables, no schema explosion.
- **Benefit:** If the database is compromised, the tokens are completely useless to the attacker.

## 2. Refresh Token Audit Timestamps (Debugging)

> [!NOTE]
> The `refresh_tokens` table currently only tracks the `valid` datetime (expiration). It does not track when the token was actually created.

**The Fix:**
Add a `createdAt` datetime column to the `RefreshToken` entity.
- **Impact:** Minor schema update.
- **Benefit:** Allows us to audit session age, debug login anomalies, and write future background cleanup jobs.

## 3. Real-Time Device State (`isOnline`)

> [!TIP]
> Relying on a static `isOnline` boolean in the database is prone to state staleness. If an Android TV crashes or loses internet unexpectedly, it cannot tell the backend to set `isOnline = false`.

**The Fix:**
Instead of a simple boolean toggle, transition to a `lastSeenAt` timestamp.
- **Impact:** Requires a heartbeat polling mechanism from the Android TVs.
- **Benefit:** The frontend can reliably infer `isOnline` by checking if `lastSeenAt` is within the last 2 minutes, completely eliminating "ghost" online TVs.

## 4. Database Indexing (Performance)

> [!IMPORTANT]
> As the user base grows, database lookups during the authentication flow must remain lightning fast.

**The Fix:**
Ensure Doctrine migrations generate explicit database indexes on high-frequency lookup columns:
- `user.email`
- `refresh_tokens.refresh_token`
- `refresh_tokens.username`
- **Impact:** Near-zero development time.
- **Benefit:** Prevents the database from doing full-table scans during login and token refresh, keeping the API highly responsive under load.
