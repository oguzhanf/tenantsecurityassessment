# Authentication Guide for the M365 Security Assessment Dashboard

This app is a single-page HTML application. It supports two operation modes:

- User sign-in (interactive) with MSAL.js (recommended)
- Managed identity / app-only via an optional proxy (for automation/app-only access)

## 1) App registration (Microsoft Entra ID)
1. Register a new application
   - Platform: Single-page application (SPA)
   - Redirect URIs: add the file origin you will use (for local testing, use http://localhost if served, or a static hosting origin). Opening a file:// URI does not work for auth; serve locally (e.g., `npx http-server`).
2. Expose public client (no client secret) and enable Authorization Code Flow with PKCE (default for SPA).
3. API permissions (request these delegated permissions):
   - Microsoft Graph: SecurityEvents.Read.All
   - Microsoft Defender for Endpoint (MDE):
     - Machine.Read
     - Software.Read
     - Vulnerability.Read
     - AdvancedHunting.Read (optional, only if using Advanced Hunting)
   - Click “Grant admin consent” if your tenant requires admin pre-consent.
4. Optional application (app-only) permissions for server-side use (proxy):
   - MDE: Machine.Read.All, Software.Read.All, Vulnerability.Read.All, AdvancedHunting.Read.All
   - Graph: SecurityEvents.Read.All

## 2) User interactive mode (MSAL.js in browser)
- Flow: Authorization Code + PKCE
- Library: `@azure/msal-browser` via CDN
- Steps in app:
  1) Initialize `PublicClientApplication` with your tenant and clientId
  2) Call `loginPopup` or `loginRedirect`
  3) Call `acquireTokenSilent` (fallback to `acquireTokenPopup`) per resource to get an access token
  4) Call APIs with Authorization: Bearer {token}
- Scopes for token acquisition:
  - Graph: `https://graph.microsoft.com/.default` won’t work for delegated. Use resource scopes such as `SecurityEvents.Read.All` or use `graphScopes = ["https://graph.microsoft.com/SecurityEvents.Read.All"]`
  - MDE: use `https://api.security.microsoft.com/.default` for delegated? No: for delegated, request specific scopes (e.g., `Machine.Read`). The `.default` pattern is for app-only. For delegated via MSAL.js, request resource-specific delegated scopes as configured in your app registration.
  - Practical approach: Split login into 2 resource requests if needed (Graph scopes and MDE scopes).

## 3) Managed identity / app-only mode (via proxy)
Browsers cannot safely perform app-only (client credentials) because:
- They cannot protect client secrets or private keys for certificate auth
- Azure Managed Identity only exists in Azure-hosted services, not the local browser

Recommended architecture:
- Deploy a minimal Azure Function/App Service with:
  - Managed identity (or certificate-based client credentials)
  - Outbound calls to Graph, MDE, and ARM (Defender for Cloud)
  - Caches & rate-limit policies
  - Exposes a read-only REST surface, e.g., `/proxy/graph/...`, `/proxy/mde/...`, `/proxy/arm/...`
- Configure CORS on the proxy to allow your SPA’s origin
- In the HTML app, set `window.CONFIG.proxyBaseUrl = "https://<your-function>.azurewebsites.net"` to enable proxy mode

## 4) Token caching and logout
- The app uses MSAL session storage/local storage per MSAL configuration
- Provide a Logout button that clears MSAL account and local IndexedDB cache (optional)

## 5) Permissions consent and troubleshooting
- If calls return 401/403, verify:
  - Correct scopes are requested for that resource
  - Admin consent granted
  - The signed-in user has the appropriate Defender/Graph data access roles
- For Defender for Endpoint, ensure the tenant has MDE/TVM licenses and API access enabled

## 6) Local development
- Serve the HTML via a local static server (for example):
  - `npx http-server -p 8080`
  - Configure SPA redirect URI to `http://localhost:8080`
- Update configuration in `config.json` (do not edit the HTML):
  - `tenantId`, `clientId`, optional `authority`
  - `graphScopes` and `mdeScopes`
  - optional `proxyBaseUrl` if using a server-side proxy
- Use `config.example.json` as a template and keep `config.json` out of source control (.gitignore)

## 7) Security hardening
- Use `loginRedirect` in production; `loginPopup` for local ease only
- Use per-resource scope requests to limit token claims
- Store cached datasets with timestamps and optionally encrypt at rest with Web Crypto (user passphrase)
- Do not log tokens; scrub PII in logs

## 8) ARM (Azure secure score) from browser
- ARM often blocks cross-origin browser calls

## 9) Configuration with config.json
- Location: alongside the HTML file (fetched as `config.json`)
- Keys:
  - `tenantId`, `clientId`, `authority`
  - `graphScopes`, `mdeScopes`
  - `proxyBaseUrl` (for proxy-based app-only/managed-identity scenarios)
  - `endpoints` (override Graph/MDE base URLs if needed)
  - `features` (feature flags)
- A `config.example.json` template is provided; copy to `config.json` and edit.
- `.gitignore` excludes `config.json` to prevent committing secrets.

## 10) Troubleshooting (common issues)
- Buttons disabled with message "Config required":
  - Ensure `config.json` exists and `clientId` is set
- Sign-in popup fails:
  - Check SPA redirect URI in Entra app registration matches your local origin (e.g., http://localhost:8080)
- 401/403 when calling APIs:
  - Verify delegated scopes are requested and admin consent is granted
  - Ensure the signed-in user has MDE/Graph roles and license (TVM for vulnerabilities/export)
- Missing data or partial charts:
  - See Debug Log panel in the Main tab for per-endpoint errors
  - Rate limits (429/503) are retried with backoff; try again later
- Azure Secure Score empty:
  - Set `proxyBaseUrl` to your proxy endpoint exposing `/arm/secureScores` and allow CORS

## 11) PowerPoint export
- Click "Export to PowerPoint" to generate `M365-Security-Assessment.pptx` with slides:
  - Executive summary (overall Secure Score + charts)
  - Device security posture (onboarding, OS)
  - Critical vulnerabilities (top CVEs)
  - M365 Secure Score recommendations (top controls)
  - Azure Secure Score summary (if collected via proxy)
  - Browser extensions assessment summary
  - Compliance Manager placeholder
- Uses PptxGenJS (CDN). Charts are embedded as images from the dashboard canvases.

- Use the proxy pattern (managed identity or app-only service) for Azure secure score
- Alternatively, allow CSV/JSON upload from another job that exports Azure secure score periodically

