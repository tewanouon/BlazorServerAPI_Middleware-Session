.Net Core 8 & Blazor server

# BlazorServerAPI_Middleware-Session
BlazorServerAPI_Middleware with Session 

BlazorServerAPI_Middleware-Session  — Design Summary
========================================

1) Overall Architecture
- Two projects in one solution:
  • ApiHost (ASP.NET Core Web API)
  • BlazorUi (Blazor Server)
- BlazorUi calls ApiHost via REST with JWT Bearer auth.

2) Authentication & Security
- JWT access tokens only (no refresh tokens).
- Access token lifetime: 20 minutes.
- ApiHost generates a NEW random 64-character signing key at startup and keeps it in memory only.
  => After API restarts, existing tokens are invalid.
- Claims included: name, role, approval_level, scope
- Policies:
  • CanApprove: requires approval_level = manager or admin
  • ApiScope: requires scope = api.read

3) CORS
- Allowed origins are restricted.
- Default in appsettings.json: https://localhost:5201
- Can be overridden with env var CORS_ORIGINS (comma-separated).

4) Configuration (Environment Variables)
- On ApiHost:
  • CORS_ORIGINS: set allowed UI origins (optional if default fits)
- On BlazorUi:
  • API_BASE_URL: where the API lives; defaults to https://localhost:5101

5) UI (Blazor Server)
- Stores token in sessionStorage only (no localStorage, no local DB).
- Pages:
  • /login: obtains JWT and stores it in sessionStorage.
  • /     : demo secured action (POST /api/internal/products/approve)
  • /fetch: demo proxy call (GET /api/proxy/sap/items)

6) API Endpoints
- POST /auth/token     -> returns { access_token, expires_at, token_type }
- GET  /api/internal/products               (Authorize)
- POST /api/internal/products/approve       (Authorize + CanApprove policy)
- GET  /api/proxy/sap/items                 (Authorize + ApiScope policy)

7) Key Traits
- Stateless auth; no refresh persistence.
- Random in-memory signing key per API run.
- Separation of concerns: API handles auth/authorization; UI manages session + calls API.
- Environment-driven configuration for CORS and API base URL.


ASCII Diagram (High-Level)
--------------------------

+----------------------+           +-----------------------+             +--------------------+
|      User Browser    |  SignalR  |     BlazorUi         |   HTTPS     |      ApiHost       |
|  (sessionStorage)    |<--------->|   (Blazor Server)    |<----------->|  (ASP.NET Core API)|
+----------+-----------+           +-----------+-----------+             +---------+----------+
           |                                   |                                   |
           | 1) /login (UI)                    |                                   |
           |---------------------------------->|                                   |
           |                                   | 2) POST /auth/token (username,pw) |
           |                                   |---------------------------------->|
           |                                   |                                   |
           |                                   | 3) { access_token, expires_at }   |
           |                                   |<----------------------------------|
           |                                   |                                   |
           | 4) UI stores access_token in sessionStorage                           |
           |                                   |                                   |
           | 5) User clicks Approve (/), or Load Items (/fetch)                    |
           |---------------------------------->|                                   |
           |                                   | 6) Call secured API with Bearer   |
           |                                   |   Authorization: Bearer <token>   |
           |                                   |---------------------------------->|
           |                                   |                                   |
           |                                   | 7) API validates JWT using        |
           |                                   |    in-memory random signing key   |
           |                                   |    and policies (role/scope/etc.) |
           |                                   |                                   |
           |                                   | 8) Response (200/403/401)         |
           |                                   |<----------------------------------|
           |                                   |                                   |
           | 9) If 401 (expired), UI asks user to login again                      |
           |                                   |                                   |


Mermaid Diagram (copy into Markdown that supports Mermaid)
----------------------------------------------------------
(See separate file: Diagram.mmd)

