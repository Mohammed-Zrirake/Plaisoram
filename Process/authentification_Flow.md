# Authentication Flow Architecture

This document maps out the complete authentication lifecycle between the Plaisoram Next.js Web Frontend and the Symfony Server Backend. It details the steps for logging in, protecting routes, making authenticated requests, and proactively rotating refresh tokens invisibly.

## Sequence Diagram

```mermaid
sequenceDiagram
    autonumber
    actor User as User (Browser)
    participant NextJS_Middleware as Next.js Middleware<br/>(src/middleware.ts)
    participant NextJS_Page as Auth Page<br/>(login/page.tsx)
    participant NextJS_Client as Client Page<br/>(e.g. DevicesPage)
    participant NextJS_Action as Server Action<br/>(src/actions/auth.ts)
    participant NextJS_Proxy as BFF API Proxy<br/>(api/[...slug]/route.ts)
    participant Symfony_Register as Symfony API<br/>(/api/register)
    participant Symfony_Login as Symfony API<br/>(/api/login_check)
    participant Symfony_Refresh as Symfony API<br/>(/api/token/refresh)
    participant Symfony_Protected as Symfony API<br/>(/api/*)

    Note over User, Symfony_Protected: 1. Registration & Workspace Provisioning
    User->>NextJS_Page: Submits Details (Name, Email, Org, Password)
    NextJS_Page->>NextJS_Action: Calls signupAction(formData)
    NextJS_Action->>Symfony_Register: POST /api/register
    Symfony_Register->>Symfony_Register: Provisions secure Workspace
    Symfony_Register->>Symfony_Register: Hashes Password & Creates User
    Symfony_Register-->>NextJS_Action: Returns 201 Created
    NextJS_Action-->>NextJS_Page: Redirects to /login

    Note over User, Symfony_Protected: 2. Login Flow
    User->>NextJS_Page: Submits Email & Password
    NextJS_Page->>NextJS_Action: Calls loginAction(formData)
    NextJS_Action->>Symfony_Login: POST /api/login_check
    Symfony_Login-->>NextJS_Action: Returns JWT & Refresh Token
    NextJS_Action->>User: Sets HttpOnly Cookies (token, refresh_token)
    NextJS_Action-->>NextJS_Page: Returns { success: true }
    NextJS_Page->>User: router.push('/')

    Note over User, Symfony_Protected: 3. Accessing Protected Pages (Middleware)
    User->>NextJS_Middleware: Requests private page (e.g. '/')
    NextJS_Middleware->>NextJS_Middleware: Checks for 'token' or 'refresh_token' cookie
    alt Has Cookies
        NextJS_Middleware-->>User: Allows request (NextResponse.next())
    else Missing Cookies
        NextJS_Middleware-->>User: Redirects to /login
    end

    Note over User, Symfony_Protected: 4. Authenticated API Calls & Proactive Token Rotation
    NextJS_Client->>NextJS_Proxy: fetchClient('/api/devices')
    NextJS_Proxy->>NextJS_Proxy: Reads HttpOnly 'token' & 'refresh_token'<br/>Decodes JWT to check 'exp' claim
    alt Token Expired (Race Condition Deduplication)
        NextJS_Proxy->>NextJS_Proxy: Locks refreshPromise to deduplicate requests
        NextJS_Proxy->>Symfony_Refresh: POST /api/token/refresh + refresh_token
        Symfony_Refresh-->>NextJS_Proxy: Returns NEW JWT & NEW Refresh Token
        NextJS_Proxy->>User: Appends Set-Cookie headers for new tokens
    end
    NextJS_Proxy->>Symfony_Protected: Proxies original request with valid Bearer Token
    alt Request Succeeds
        Symfony_Protected-->>NextJS_Proxy: Returns 200 OK + Data
        NextJS_Proxy-->>NextJS_Client: Returns 200 OK + Data
    else Refresh Token Dead / Server Error
        Symfony_Protected-->>NextJS_Proxy: Returns 401 Unauthorized
        NextJS_Proxy-->>NextJS_Client: Returns 401 Unauthorized
        NextJS_Client->>User: fetchClient catches 401 -> window.location.href = '/login'
    end
```

## Files Involved

### Plaisoram Web (Next.js)
- **`src/app/(auth)/login/page.tsx` & `src/app/(auth)/signup/page.tsx`**: The client-side UI where the user inputs their credentials. They invoke secure Server Actions instead of making direct HTTP requests to the backend.
- **`src/actions/auth.ts`**: The Backend-For-Frontend (BFF) Server Actions. These run on the Node.js server, call the Symfony backend directly to authenticate, and set the strict `HttpOnly`, `Secure` session cookies.
- **`src/middleware.ts`**: Next.js Middleware that intercepts requests to page routes, ensuring the user possesses an active authentication cookie before allowing them to access private dashboards. *(Ensure this file is correctly named and placed in `src/` for Next.js to detect it).*
- **`src/app/api/logout/route.ts`**: API endpoint called by the client to physically delete the HttpOnly cookies when a session dies, ensuring a clean redirect.
- **`src/lib/fetchClient.ts`**: Client-side fetch wrapper that intercepts 401 Unauthorized responses, calls `/api/logout`, and automatically redirects the user back to the login page to prevent broken UI states or infinite redirect loops.
- **`src/app/api/[...slug]/route.ts`**: The Backend-For-Frontend (BFF) API Proxy. Intercepts all client-side calls. It is responsible for:
  - Reading the `HttpOnly` token cookies securely.
  - Proactively checking the JWT expiration (`exp`) claim.
  - Transparently exchanging the `refresh_token` for a new JWT if the current token is expired.
  - **Deduplication:** Uses a global `refreshPromise` lock to ensure multiple simultaneous requests wait for a single refresh, preventing race conditions with `single_use` tokens.
  - Forwarding the request to Symfony (`http://localhost:8000/api/*`) with the valid `Authorization: Bearer <token>` header.

### Plaisoram Server (Symfony)
- **`src/Controller/RegistrationController.php`**: Handles the initial `POST /api/register`. It securely hashes the password, automatically provisions a dedicated `Workspace` for multi-tenant data isolation, and bonds the new user to this Workspace.
- **`src/Controller/*Controller.php`**: Protected endpoints (Devices, Media) automatically extract the `Workspace` from the authenticated user's JWT to guarantee data silos.
- **`config/packages/security.yaml`**: The security nerve center. Configures the `login` firewall (for `json_login`), the `api_token_refresh` firewall, and the stateless `api` firewall. Enforces rate limits (`login_throttling`).
- **`config/packages/gesdinet_jwt_refresh_token.yaml`**: Configures the long-lived refresh tokens with `single_use: true` (token rotation).
- **`src/Entity/User.php` & `src/Entity/RefreshToken.php`**: The Doctrine entities mapping user identities and valid refresh tokens to the database.
