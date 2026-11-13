# CANDY — BubbaBlox v2 / Frontend Integration Notes

Backend repository: `prasallts-hue/bubbablox-v2`
Backend ref: `candy-security-prep`
Frontend repository: `prasallts-hue/4ndyy-frontend`

## Current state

The CANDY frontend already contains a small API adapter:

- `assets/js/candy-config.js`
- `assets/js/candy-api.js`

The config currently has an empty `apiBaseUrl` and null endpoint paths. This is intentional until the backend origin and actual endpoint contract are confirmed.

The backend's website is an ASP.NET Core .NET 6 application. Its controllers expose legacy Roblox-compatible routes (including `/apisite/`), and its frontend proxy currently forwards website routes to `http://localhost:3000`. That proxy is for a local Next.js frontend and is not, by itself, an integration with the separate CANDY static frontend.

## Integration architecture to finalize

Prefer a same-origin deployment where possible:

- CANDY website: `https://<CANDY-domain>`
- Backend API: `https://<CANDY-domain>/apisite/...` or a separately configured HTTPS API origin.
- Session authentication: use secure, HttpOnly, SameSite cookies if the existing backend session flow supports them.
- Keep credentials out of browser local storage and out of source control.
- Configure exact allowed origins; do not use wildcard CORS with credentials.

Do not configure the frontend adapter with guessed routes. Login, registration, logout, current-user, CSRF, and session behavior must be mapped to the actual backend controllers and middleware first.

## Required configuration after deployment details are known

Update `assets/js/candy-config.js` in the frontend repository with:
- the final HTTPS API base URL;
- verified login/register/logout/current-user endpoint paths;
- the correct credential mode for the chosen same-origin or cross-origin architecture.

Backend configuration must use the same CANDY HTTPS origin for `BaseUrl`, `AssetUrl`, Discord callbacks, CORS/CSP, and client/RCC settings where applicable. Do not use the example secrets.

## Blocking items

- Decide the CANDY HTTPS domain. The README documents a 10-character domain constraint for legacy binaries; verify this against the exact client/RCC builds before committing to a domain.
- Fix the TLS certificate-validation bypass in `Program.cs`.
- Remove or explicitly gate the database deletion task that runs on website startup.
- Replace the localhost-only frontend proxy architecture or deliberately deploy the frontend on localhost:3000.
- Confirm the backend auth/session/CSRF contract before configuring the frontend adapter.

## Validation checklist

- [ ] Backend builds successfully in an isolated environment.
- [ ] Database schema imports into a disposable database.
- [ ] No startup operation deletes user or asset data unexpectedly.
- [ ] TLS validation is enabled.
- [ ] Login/register/logout/current-user endpoints are confirmed.
- [ ] Cookie flags, CSRF token flow, and CORS are tested.
- [ ] Frontend API adapter is configured with real endpoint paths.
- [ ] Static assets and thumbnails load through the intended HTTPS origin.
- [ ] No secrets, cookies, or private keys are committed.
