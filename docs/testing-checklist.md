# Testing Checklist

Use this checklist to validate the dashboard end-to-end.

## Authentication flows
- [ ] config.json present with valid tenantId/clientId
- [ ] SPA redirect URI matches local origin (e.g., http://localhost:8080)
- [ ] Sign in succeeds; username displayed in header
- [ ] Logout clears session; buttons behave as expected
- [ ] Missing/invalid clientId disables Sign in and Collect with friendly message

## API data collection
- [ ] Collect Data shows progress overlay and completes without fatal errors
- [ ] Graph: secureScores returns recent timeline entries
- [ ] Graph: secureScoreControlProfiles returns controls metadata
- [ ] MDE: machines list populated; charts (onboarding, last seen, OS) render
- [ ] MDE: software list populated; top rows sorted by exposed machines
- [ ] MDE: vulnerabilities populated; critical (CVSS>=7) filtered table populated
- [ ] MDE: browser extensions export populated (if TVM license available)
- [ ] Azure secure score: populated via proxyBaseUrl when configured
- [ ] Debug Log contains useful messages; no secrets/tokens logged

## Retry and error handling
- [ ] Rate-limit scenarios (429/503) recover with exponential backoff
- [ ] Network failures show user-friendly status and log details
- [ ] Missing scopes are logged clearly in Debug Log

## Compliance Manager CSV
- [ ] Upload valid CSV parses successfully; summary metrics shown
- [ ] Top gaps table appears with control, category, status, score
- [ ] Malformed CSV shows an error and does not crash the app

## PowerPoint export
- [ ] PPTX downloads successfully
- [ ] Executive summary shows current overall Secure Score and chart image
- [ ] Device posture slide includes onboarding and OS charts
- [ ] Critical vulnerabilities table shows top items by CVSS
- [ ] M365 Secure Score recommendations show top controls
- [ ] Azure secure score slide displays summary (if proxy data present)
- [ ] Compliance Manager slide shows metrics and gaps when CSV uploaded

## UI/UX and usability
- [ ] Buttons enable/disable correctly during collection
- [ ] Global search filters visible table rows
- [ ] Tables sort by clicking headers
- [ ] Layout is responsive across common viewport sizes

## Cross-browser validation
- [ ] Latest Chrome
- [ ] Latest Edge
- [ ] Latest Firefox
- [ ] Safari (macOS/iOS) as applicable

## Security & privacy
- [ ] No tokens or PII in Debug Log or exports
- [ ] Local storage only; exports are user-initiated JSON/PPTX files
- [ ] .gitignore prevents committing config.json and generated exports

