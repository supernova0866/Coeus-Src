# Coeus-Src

The actual app pulled by the [Coeus installer](https://github.com/supernova0866/Coeus).

Coeus is a focus browser built on Electron. It intercepts every navigation request, enforces site blocklists and daily time limits, runs timed focus sessions with kiosk mode, and tracks usage patterns with a full dashboard.

## Stack

- **Electron** — browser shell, BrowserView for web content, kiosk mode
- **better-sqlite3** — synchronous SQLite for sessions, usage, rules, settings
- **JSZip** — data export zip builder
- **Plain JS + CSS** — no framework in the renderer, no build step

## Structure

```
src/           Node.js main process (full OS access)
renderer/      Browser context (zero Node, talks via IPC only)
shared/        Pure utils and constants used by both sides
data/          Runtime only — SQLite DB, exports, imports (gitignored)
assets/        Icons, sounds
```

## Dev setup

```bash
# Requires Python + Visual Studio C++ Build Tools for better-sqlite3
npm install
npm run rebuild    # compiles better-sqlite3 native module
npm start          # launches in dev mode
```

## Release

Push a version tag to trigger the GitHub Actions release workflow. It will:

1. Zip `src/`, `renderer/`, `shared/`, `assets/`, `package.json`
2. SHA-256 hash the zip
3. Sign it with the RSA private key (stored in `COEUS_SIGNING_KEY` Actions secret)
4. Verify the signature before uploading
5. Attach `coeus-app.zip` and `manifest.json` to the GitHub Release

```bash
git tag v0.2.0
git push origin v0.2.0
```

The Coeus installer picks this up automatically on next launch.

## IPC architecture

All communication between renderer and main process goes through `preload.js` via `contextBridge`. The renderer calls `window.ff.*` — a narrow typed API — and never has direct access to Node or the filesystem. All channel names are defined in `shared/constants.js` as the `IPC` object.

## Default PIN

`1234` — change it in Settings immediately after first run.

## Data

- **DB location**: `%APPDATA%/Coeus/coeus.db` (Windows) / `~/Library/Application Support/Coeus/coeus.db` (macOS)
- **Export**: Settings → Export → Data `.zip` or Settings `.json`
- **Import**: append-only for data, replace or merge for settings
