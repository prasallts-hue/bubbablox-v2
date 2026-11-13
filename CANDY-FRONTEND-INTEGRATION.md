# CANDY — BubbaBlox v2 / Frontend Integration Notes

Backend repository: `prasallts-hue/bubbablox-v2`
Preparation branch: `candy-security-prep`
Frontend repository: `prasallts-hue/4ndyy-frontend`

## Current state

The CANDY frontend already contains:

- `assets/js/candy-config.js`
- `assets/js/candy-api.js`

The frontend config intentionally has an empty `apiBaseUrl` and null endpoint paths. Do not invent endpoint paths before checking the backend controllers and session middleware.

The backend is an ASP.NET Core .NET 6 website. Its `FrontendProxyMiddleware` serves a legacy Next.js frontend through a configurable `FrontendProxy:BaseUrl`. The example value `http://127.0.0.1:3000` is local development only; it does not connect to the separate static CANDY frontend repository by itself.

## Changes already prepared

- TLS certificate validation bypass removed.
- Startup SQL that deleted rows from the game-server tables removed.
- Frontend proxy origin moved into `FrontendProxy:BaseUrl` configuration and validated as an absolute HTTP(S) URL.
- Old `zawg.ca` and insecure HTTP CSP origins removed.
- Example config now uses CANDY placeholders and generic paths rather than another developer's Windows directories.
- The unsafe all-in-one launcher is disabled pending a safe, host-specific launch plan.

## Target architecture (to decide after domain/hosting purchase)

Prefer same-origin deployment where possible:

- CANDY site: `https://<CANDY-domain>`
- API: `https://<CANDY-domain>/apisite/...` or a separately configured HTTPS API origin.
- Prefer secure, HttpOnly, SameSite cookies if compatible with the backend's existing session implementation.
- Keep tokens and credentials out of local storage and source control.
- Configure exact trusted origins; do not use wildcard CORS with credentials.

Do not connect the frontend adapter until the actual login, registration, logout, current-user, CSRF, and session contracts have been confirmed.

## Pending configuration

After the domain and host are known:

1. Verify the README's legacy 10-character domain requirement against the exact client/RCC binaries.
2. Set `BaseUrl`, `AssetUrl`, `CorsMiddlewareUrl`, Discord callback URLs, CDN settings, and the relevant client/RCC settings to CANDY-owned HTTPS origins.
3. Set `FrontendProxy:BaseUrl` only if intentionally using the backend's legacy frontend proxy. Otherwise deploy the static frontend through the selected web server/reverse proxy and route API requests separately.
4. Update `assets/js/candy-config.js` with the actual API base URL and verified endpoint paths.
5. Generate fresh, unique database, session, authorization, bot, webhook, Discord, and CAPTCHA secrets. Keep private values in local config or environment variables, never Git.

## Validation checklist

- [ ] Backend builds in an isolated environment.
- [ ] Database schema imports into a disposable database.
- [ ] Startup does not delete user or game-server data.
- [ ] TLS certificate validation remains enabled.
- [ ] Login/register/logout/current-user endpoints are mapped and tested.
- [ ] Cookie flags and CSRF flow are verified.
- [ ] Exact CORS/CSP origins are configured.
- [ ] Frontend adapter uses the real API origin and route paths.
- [ ] Static assets, thumbnails, and game services use the intended HTTPS host.
- [ ] No secrets, cookies, private keys, or production config are committed.
- [ ] RCC, renderer, and asset validation services run under isolated, least-privilege accounts.
