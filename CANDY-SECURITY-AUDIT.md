# CANDY — Preliminary Security Audit (BubbaBlox v2)

Repository: `prasallts-hue/bubbablox-v2`
Audited ref: `2021`
Preparation branch: `candy-security-prep`
Date: 2026-11-13

## Verdict

**Not ready to deploy or connect to a public frontend yet.** The source has a recognizable revival-server architecture, but this review found high-risk security and operational defects that must be corrected and tested first.

This is a static, partial review of selected text files. No source, installer, dependency lifecycle script, or binary was executed. The repository is large (about 763 MB); binary contents, the complete dependency graph, and every source file were not exhaustively inspected. Therefore this is **not a malware-free certification**.

## Findings

### Critical — TLS certificate validation is explicitly disabled

File: `Roblox/Roblox.Website/Program.cs`

The code assigns a `ServicePointManager.ServerCertificateValidationCallback` that always returns `true`. This accepts invalid/untrusted TLS certificates for requests using that callback and can expose credentials and data to interception.

**Required:** remove the permissive callback and use normal platform certificate validation. Do not replace it with another "accept all" callback.

### High — database rows are deleted automatically at web startup

File: `Roblox/Roblox.Website/Program.cs`

A background task opens the configured PostgreSQL database and executes:

- `DELETE FROM asset_server_player;`
- `DELETE FROM asset_server;`

This is destructive startup behavior. It may be intentional cleanup for this fork, but it must not run against a live database without an explicit, reviewed migration/maintenance decision and backup.

**Required:** disable or redesign this cleanup so normal application startup never silently deletes production data.

### High — deployment script uses a developer-specific absolute path and kills RCC processes

File: `runall.bat`

The script changes to a hardcoded `C:\Users\Admin\...` directory and force-kills every process named `RCCService.exe`. It is not portable and could interrupt unrelated RCC workloads.

**Required:** replace with explicit configurable paths and scoped process management. Do not run this script as-is.

### High — example configuration contains insecure/foreign defaults

File: `Roblox/Roblox.Website/appsettings.example.json`

Examples include:
- `AssetUrl` points to an external `http://bt.zawg.ca` host.
- Several URLs and callbacks use plain HTTP or placeholders.
- Directory paths point to another developer's Windows machine.
- Placeholder secrets are present for database, Discord, webhook, session, authorization, and captcha settings.

**Required:** use CANDY-owned HTTPS origins, local paths, and newly generated secrets. Never copy placeholder values into production. Keep real configuration and credentials out of Git.

### Medium — Git ignore file contains unresolved merge-conflict markers

File: `.gitignore`

The file contains literal `<<<<<<< HEAD`, `=======`, and `>>>>>>> ...` markers. This is a repository hygiene issue and suggests a merge was not fully resolved.

**Required:** reconcile the ignore rules and remove conflict markers.

### Medium — frontend proxy is hardwired to localhost

File: `Roblox/Roblox.Website/Program.cs` and `Roblox/Roblox.Website/Middleware/FrontendProxyMiddleware.cs`

The `FrontendProxy` HttpClient uses `http://localhost:3000`. This assumes a local Next.js frontend process. It does not automatically connect to the separate CANDY static frontend repository.

**Required:** choose one architecture: serve the CANDY frontend from the same origin/reverse proxy, or configure a separate HTTPS frontend origin and API origin. Avoid broad wildcard CORS.

### Medium — domain length constraint and legacy client patching

The README says the domain must be exactly 10 characters because legacy client/RCC binaries contain hardcoded domain strings. This constraint may affect client compatibility and must be verified before choosing a CANDY domain.

### Medium — old framework/runtime baseline

The website targets `.NET 6`; the README asks for Node.js 18.16.1 and Go 1.20+. These are old baselines. Upgrade planning and dependency review are required before public deployment.

### Low/Review — installation script changes system services

File: `AssetValidationServiceV2/install.sh`

The script uses `sudo` to remove/copy a systemd service definition, enable it, and start it. It must be reviewed and adapted to the intended host before use.

## RAT / malicious-code assessment

- The inspected `Inventory.cs` contains ordinary inventory-owner lookup logic; no reverse-shell or RAT behavior was present in that file.
- The inspected `Program.cs` contains a dangerous TLS-validation bypass and destructive database startup cleanup. These are confirmed risks, but by themselves they do **not** establish that the project contains a RAT.
- No confirmed RAT was identified in the selected files.
- The repository's full source, binaries, archives, and dependency install scripts have not been exhaustively analyzed. Unknown executables and archives must remain untrusted and must not be run until independently verified.

## Safe next steps

1. Keep the repository isolated; do not expose it to the internet yet.
2. Remove the TLS validation bypass.
3. remove/review destructive database cleanup.
4. Reconcile `.gitignore` and hardcoded paths.
5. Replace legacy domains and all secrets with CANDY-owned HTTPS config and fresh secrets.
6. Review binary artifacts and dependency lifecycle scripts.
7. Build and test in an isolated environment with a disposable database before any production deployment.
8. Only then connect the CANDY frontend and test login, registration, session cookies, CORS, CSRF, and API routes.

## Scope and limitations

This report records the exact issues observed during a partial static inspection. It does not claim the repository is clean, safe to execute, or ready for production.
