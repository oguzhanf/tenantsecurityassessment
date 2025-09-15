# M365 Security Assessment Dashboard

A single‑file HTML dashboard that signs into Microsoft 365, collects security posture data, and presents executive‑friendly visuals. Configured via a separate config.json for portability.

## Features
- Sign in with MSAL.js (delegated) and collect:
  - Microsoft Graph: M365 Secure Score, control profiles
  - Microsoft Defender for Endpoint: devices, software, vulnerabilities, browser extensions
  - Azure Secure Score: via optional proxy (app‑only/managed identity)
- Offline‑friendly: cache and Export/Import JSON
- Robust collection: retries, progress overlay, scope checks, debug log
- PowerPoint export (PptxGenJS): executive summary + detailed slides
- Compliance Manager: upload CSV to visualize baseline and gaps; included in PPT
- Modern responsive layout; sortable tables and global search

## Prerequisites
- An Azure AD (Entra ID) App Registration (SPA) with delegated permissions:
  - Microsoft Graph: SecurityEvents.Read.All
  - Microsoft Defender for Endpoint: Machine.Read, Software.Read, Vulnerability.Read
  - Optional: TVM/Advanced Hunting licenses may be required for some datasets (e.g., vulnerabilities, browser extensions export)
- Admin consent granted for the delegated permissions
- A simple local HTTP server for development

## Quick Start
1. Clone the repository
2. Copy config.example.json to config.json
3. Edit config.json and set:
   - tenantId: your tenant GUID or domain
   - clientId: your App Registration Application (client) ID
   - authority: optional; defaults to https://login.microsoftonline.com/{tenantId}
4. Serve locally, e.g.:
   - `npx http-server -p 8080`
5. Open http://localhost:8080/m365-security-dashboard.html
6. Click Sign in → Collect Data → Export/Import JSON or Export to PowerPoint

Note: config.json is .gitignored; do not commit tenant secrets.

## Configuration (config.json)
- tenantId: tenant GUID (recommended) or "common" for multi‑tenant testing
- clientId: your SPA app’s Application (client) ID
- authority: optional explicit authority; if omitted, the app builds it from tenantId
- graphScopes: array of delegated Graph scopes (default includes SecurityEvents.Read.All)
- mdeScopes: array of delegated MDE scopes (Machine.Read, Software.Read, Vulnerability.Read)
- endpoints: base URLs for Graph and MDE (override if using sovereign clouds)
- proxyBaseUrl: optional proxy for Azure Secure Score (exposes /arm/secureScores)

How to obtain values:
- tenantId: Entra ID → Overview → Tenant ID
- clientId: App registrations → Your App → Application (client) ID
- Redirect URI: Add your local origin (e.g., http://localhost:8080) to SPA platform

## Local development
- Use a static server (file:// URLs won’t work with MSAL):
  - `npx http-server -p 8080`
- Ensure SPA redirect URI includes your local origin
- Update config.json; restart isn’t required, but you may refresh the page

## Using the dashboard
- Sign in: opens MSAL popup for delegated consent
- Collect Data: runs parallel calls with retries and progress overlay; see Debug Log
- Compliance Manager: upload CSV export (from M365 Compliance Manager) to populate the Compliance tab and PPT slide
- Export to PowerPoint: generates an executive‑ready PPTX with charts and summaries

## Troubleshooting
- Buttons disabled with message "Config required": set clientId in config.json
- Sign‑in issues: check redirect URI and authority/tenantId
- 401/403: verify scopes, admin consent, and user’s MDE/Graph roles/licenses
- Missing data: see Debug Log; retry later if rate‑limited; ensure defender/TVM licensing
- Azure Secure Score empty: configure proxyBaseUrl and allow CORS on your proxy

## File structure
- m365-security-dashboard.html — main single‑page app
- config.example.json — configuration template
- config.json — your local config (ignored by Git)
- docs/
  - authentication-guide.md — MSAL + proxy setup and troubleshooting
  - m365-api-research.md — API mappings and permissions (initial research)
  - testing-checklist.md — comprehensive validation scenarios

## Testing checklist
See docs/testing-checklist.md for a full end‑to‑end checklist covering auth, data collection, PPT export, error handling, and cross‑browser validation.

