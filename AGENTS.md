# AGENTS.md

## Active Roadmap — Read First

Before starting feature work on this repository, **read [`PLAN.md`](./PLAN.md)** and [`REPORT.md`](./REPORT.md).

- **Current state (v1):** Single-file Python 3 CLI (`faceguard.py`) for macOS using OpenCV + `face_recognition`/`dlib`.
- **Planned direction (v2):** Rebuild as a **native macOS menu-bar app** (Swift + SwiftUI + Apple Vision + CoreML) distributed as a `.dmg` with:
  - Armed / Auto-Arm on idle modes (note: MacBook camera LED is hardware-wired and cannot be hidden while the camera is active)
  - Native Settings window + guided enrollment (rejecting multi-face frames)
  - Easy Telegram bot setup (guided BYO-Bot wizard with automatic `getUpdates` chat-ID capture, Keychain storage)
  - No Windows/Linux scope until macOS v2 ships

## Non-Negotiable Rules

1. **Never gate screen locking on network I/O** — always lock first, then send the Telegram photo asynchronously.
2. **Never commit credentials** — `.env` is gitignored; v2 must store Telegram tokens in macOS Keychain.
3. **Keep docs in sync with code** — see known v1 doc drift in `REPORT.md` §10.
4. **Preserve privacy posture** — all face detection and recognition must run 100% locally on-device.

See [`CLAUDE.md`](./CLAUDE.md) for v1 CLI commands, architecture, and tuned constants.
