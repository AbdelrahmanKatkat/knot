---
name: package-desktop
description: Build and release the installable laptop app — Tauri shell, PyInstaller backend sidecar, installers, auto-update, CI. Use for packaging, release and first-run issues.
---

# Package the desktop app

1. Freeze backend: `pyinstaller` entry `knot.cli:main` → `desktop/src-tauri/binaries/knot-backend-<target-triple>` (Tauri sidecar naming). Include sqlite-vec extension, faster-whisper/ctranslate2 libs; do not bundle model weights.
2. Build UI: `pnpm --dir ui build`; Tauri `frontendDist` points at it.
3. Tauri launches the sidecar with a random port + one-time token, waits for `/health`, and kills it on exit. Declare mic permission (macOS `NSMicrophoneUsageDescription`, entitlements).
4. Bundle ffmpeg per platform as a sidecar/resource. Node (for the Notion MCP server) is checked in the wizard or bundled.
5. Installers: `.msi/.exe` (Windows), `.dmg` (macOS, sign + notarize), `.AppImage/.deb` (Linux). CI matrix builds on tag `v*` and attaches artifacts to the release.
6. Updater: Tauri updater with signed manifest; never auto-update during an active recording or job.
7. Smoke test on a clean VM: install → wizard → import fixture → notes appear → uninstall leaves user data prompt.
8. App data lives in the OS app-data dir (`KNOT_DATA_DIR` default), not the install dir.
