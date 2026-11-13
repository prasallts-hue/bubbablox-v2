# CANDY — Security Audit and Remediation Status (BubbaBlox v2)

Repository: `prasallts-hue/bubbablox-v2`
Original source branch reviewed: `2021`
Preparation branch: `candy-security-prep`
Review date: 2026-11-13

## Current verdict

**Improved, but not ready for public deployment.** The high-risk issues identified in the first pass have been corrected on this preparation branch. The fixes have been verified by re-reading the committed files and checking the JSON syntax. A full build, integration test, binary scan, and exhaustive malware review have not been performed.

## Remediations committed on `candy-security-prep`

1. `Roblox/Roblox.Website/Program.cs`
   - Removed the callback that accepted every TLS certificate.
   - Removed explicit legacy TLS 1.0/1.1 enablement. The runtime's normal secure TLS defaults and certificate validation now apply.
   - Removed automatic startup SQL commands that deleted every row from `asset_server_player` and `asset_server`.
   - Made the frontend proxy base URL configurable with `FrontendProxy:BaseUrl`; validates that it is an absolute HTTP(S) URL. The example uses loopback for local development only.

2. `Roblox/Roblox.Website/Middleware/CorsMiddleware.cs`
   - Removed hardcoded `zawg.ca` and insecure HTTP origins from the Content Security Policy.
   - Removed the HTTP localhost script exception. The configured CANDY host and the required hCaptcha/CDN origins remain.

3. `Roblox/Roblox.Website/appsettings.example.json`
   - Removed the old external `bt.zawg.ca` asset URL and developer-specific Windows paths.
   - Replaced them with explicit CANDY placeholders and a generic local directory layout.
   - Added `FrontendProxy:BaseUrl`.
   - The file parses as valid JSON. It is still a template: replace all placeholders and generate fresh secrets before deployment.

4. `.gitignore`
   - Removed unresolved merge-conflict markers and consolidated ignore rules for local configuration, secrets, dependencies, build outputs, and IDE files.

5. `runall.bat`
   - Disabled automatic launch in this branch. The old script used an absolute path from another developer's computer and force-killed all processes named `RCCService.exe`.
   - It now explains why automatic startup is disabled. Services must be reviewed and started individually after the host is configured.

## Remaining blockers

- No CANDY domain or hosting environment has been supplied yet. Do not fill in the domain placeholders until the real HTTPS domain is known.
- The README states a 10-character domain requirement for legacy client/RCC binaries. Verify this against the exact binaries and client builds before choosing the domain.
- The website targets `.NET 6`, an old runtime baseline. Plan an upgrade only after compatibility and dependency testing.
- The app still needs real local configuration files, a PostgreSQL schema/database, Redis, renderer configuration, game-server configuration, and the required service components.
- The CANDY frontend adapter still has empty API URL and endpoint settings. The real login, registration, logout, current-user, session, CSRF, and cookie contracts must be mapped before connection.
- The frontend proxy is configurable but its default is local development only; it is not yet wired to the separate CANDY static frontend.
- Startup still assumes all required configuration keys/files exist. Build/run validation with a disposable database is required.

## RAT / malicious-code assessment

- No confirmed RAT was identified in the selected source files.
- The inspected `Inventory.cs` contains ordinary inventory-owner lookup logic.
- The earlier confirmed TLS bypass and destructive database cleanup were serious security defects, but are not proof of a RAT.
- The repository is large (about 763 MB). All binaries, archives, source files, dependencies, and lifecycle scripts have not been exhaustively inspected. Do not execute unknown binaries or installers until independently verified.
- This is a partial static review, not a malware-free certification.

## Validation performed

- Re-fetched the changed files from the preparation branch.
- Confirmed the TLS certificate bypass is absent from `Program.cs`.
- Confirmed the automatic destructive SQL statements are absent from `Program.cs`.
- Confirmed the proxy base URL is read from configuration and validated.
- Confirmed `appsettings.example.json` parses as valid JSON.
- Confirmed `.gitignore` has no merge-conflict markers.
- Confirmed `runall.bat` no longer contains `taskkill`.

## Validation not performed

- No .NET, Node, Go, or Python build/test was run.
- No executable, archive, installer, or dependency lifecycle script was run.
- No full antivirus/YARA/static binary analysis or dynamic sandbox analysis was performed.
- No production database or public server was used.

## Safe next steps

1. Keep the source isolated until the remaining review and build tests are complete.
2. Supply the final HTTPS domain and hosting details only after purchase.
3. Generate unique secrets and configure local-only files; never commit them.
4. Build and test with a disposable database and least-privilege service accounts.
5. Map the backend auth/session/API routes to the CANDY frontend and test CSRF, cookie flags, CORS, CSP, and assets.
6. Perform a complete review of binary artifacts and third-party dependencies before deployment.
