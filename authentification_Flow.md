# Authentication Flow Architecture

This document maps out the complete authentication lifecycle between the Plaisoram Next.js Web Frontend and the Symfony Server Backend. It details the steps for logging in, protecting routes, making authenticated requests, and rotating refresh tokens invisibly.

## Sequence Diagram

```mermaid
sequenceDiagram
    autonumber
    actor User as User (Browser)
    participant NextJS_Middleware as Proxy<br/>(src/proxy.ts)
    participant NextJS_Page as Auth Page<br/>(login/page.tsx)
    participant NextJS_Action as Server Action<br/>(src/actions/auth.ts)
    participant Symfony_Login as Symfony API<br/>(/api/login_check)
    participant Symfony_Refresh as Symfony API<br/>(/api/token/refresh)
    participant NextJS_API as API Interceptor<br/>(src/lib/api.ts)
    participant Symfony_Protected as Symfony API<br/>(/api/*)
    participant Symfony_Register as Symfony API<br/>(/api/register)

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

    Note over User, Symfony_Protected: 3. Accessing Protected Pages
    User->>NextJS_Middleware: Requests private page (e.g. '/')
    NextJS_Middleware->>NextJS_Middleware: Checks for 'token' or 'refresh_token' cookie
    alt Has Cookies
        NextJS_Middleware-->>User: Allows request (NextResponse.next())
    else Missing Cookies
        NextJS_Middleware-->>User: Redirects to /login
    end

    Note over User, Symfony_Protected: 4. Making Authenticated API Calls & Token Rotation
    NextJS_API->>Symfony_Protected: fetchApi('/api/data') + Bearer Token
    alt Token is Valid
        Symfony_Protected-->>NextJS_API: Returns 200 OK + Data
    else Token Expired (401)
        Symfony_Protected-->>NextJS_API: Returns 401 Unauthorized
        NextJS_API->>Symfony_Refresh: POST /api/token/refresh + refresh_token
        Symfony_Refresh-->>NextJS_API: Returns NEW JWT & NEW Refresh Token
        NextJS_API->>User: Updates HttpOnly Cookies
        NextJS_API->>Symfony_Protected: Retries original request with NEW JWT
        Symfony_Protected-->>NextJS_API: Returns 200 OK + Data
    end
```

## Files Involved

### Plaisoram Web (Next.js)
- **`src/app/(auth)/login/page.tsx` & `src/app/(auth)/signup/page.tsx`**: The client-side UI where the user inputs their credentials. They invoke the secure Server Actions instead of making direct HTTP requests to the backend.
- **`src/actions/auth.ts`**: The Backend-For-Frontend (BFF) Server Actions. These run on the Node.js server, call the Symfony backend directly, and set the strict `HttpOnly`, `Secure` cookies.
- **`src/proxy.ts`**: Edge proxy that intercepts every request to the Next.js app to ensure the user possesses an active authentication cookie before allowing them to access private routes.
- **`src/lib/api.ts`**: The `fetchApi` wrapper intended for Next.js Server Components. It automatically extracts the JWT from the cookies, injects it into the `Authorization` header, and handles the transparent 401 retry-with-refresh logic.

### Plaisoram Server (Symfony)
- **`src/Controller/RegistrationController.php`**: Handles the initial `POST /api/register`. It securely hashes the password, automatically provisions a dedicated `Workspace` for multi-tenant data isolation, and bonds the new user to this Workspace.
- **`src/Controller/*Controller.php`**: Protected endpoints (Devices, Media) automatically extract the `Workspace` from the authenticated user's JWT to guarantee data silos.
- **`config/packages/security.yaml`**: The security nerve center. Configures the `login` firewall (for `json_login`), the `api_token_refresh` firewall, and the stateless `api` firewall. Enforces rate limits (`login_throttling`).
- **`config/packages/gesdinet_jwt_refresh_token.yaml`**: Configures the long-lived refresh tokens with `single_use: true` (token rotation).
- **`src/Entity/User.php` & `src/Entity/RefreshToken.php`**: The Doctrine entities mapping user identities and valid refresh tokens to the database.
