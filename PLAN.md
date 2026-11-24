# FaceGuard v2 Plan — Native macOS Menu-Bar App

> **Status:** Planned (not started yet — current codebase is the v1 Python CLI in `faceguard.py`).  
> **Target:** Turn FaceGuard from a developer-only Python CLI into a polished, downloadable macOS menu-bar `.app` / `.dmg` for everyday Mac users.

---

## Product Goal

Deliver a **local-first, zero-subscription macOS menu-bar app** that locks your Mac the moment an unrecognized face appears and sends you a photo alert on Telegram — installable by dragging to `/Applications`, with zero Terminal, Python, or `dlib` compilation.

### Scope Boundaries
- **macOS-only for v2** (Apple Silicon & Intel macOS 13+). Do **not** expand to Windows/Linux until macOS v2 is shipped and validated.
- **Keep the v1 Python CLI intact** (or move to `legacy/`) until v2 reaches parity on lock latency and accuracy.

---

## Core Architecture (v2)

- **UI:** Native **Swift + SwiftUI** menu-bar app (`MenuBarExtra` + native macOS `Settings` window).
- **Camera & Computer Vision:** `AVFoundation` + Apple **Vision** framework (`VNDetectFaceRectanglesRequest`) + on-device **CoreML** face embedding model (e.g., MobileFaceNet / ArcFace converted to `.mlpackage`) — eliminates Python, OpenCV, and `dlib` C++ build dependencies.
- **Storage & Security:**
  - Store face embeddings as signed/encrypted JSON or CoreData in Application Support (replace v1's unsafe `pickle`).
  - Store Telegram bot token in **macOS Keychain** (never plaintext `.env`).
  - Cap/prune saved intruder snapshots (e.g., keep last 50 or 30 days) with an in-app viewer & "Clear Snapshots" button.
- **Packaging & Distribution:** Unsigned open-source `.dmg` via GitHub Releases (linked from README; no paid Apple Developer account required — include a 1-line macOS first-open / quarantine note).

---

## Key UX & Feature Requirements

### 1. Menu-Bar Control & "Armed / Sentry Mode" (Camera LED Reality)
- **Hardware constraint:** On MacBooks, the green camera LED is wired in hardware to the camera sensor's power line (and macOS enforces the menu-bar privacy indicator). It **cannot** be turned off while the camera is active.
- **UX solution:** Provide clear operating modes from the menu bar instead of running the camera 24/7 while the user is working:
  - **Armed / Disarmed toggle** (1 click or global hotkey)
  - **Auto-Arm on Idle** (optional: automatically arm after N seconds of keyboard/mouse inactivity, disarm when you return)
  - **Quick Pause** (15 min / 1 hr / until unlocked)
  - **Recent Activity** (view last intruder snapshot directly from the menu bar)

### 2. Guided Face Enrollment Flow
- In-app camera preview sheet guiding the user through 6–10 angles (center, slight left/right/up/down, glasses on/off).
- Reject frames with >1 face during enrollment (fixes v1's bystander-enrollment bug documented in `REPORT.md` §3.1).
- Allow incremental "Add more angles" and one-click "Reset Face Data".

### 3. Easy Telegram Bot Connection (Zero Manual JSON / URLs)
Replace v1's 4-step manual `/getUpdates` flow with a guided in-app wizard:
- **Option A — Guided Bring-Your-Own-Bot (100% local, zero server, default):**
  1. In-app link opens `@BotFather` with a copyable `/newbot` guide.
  2. User pastes their bot token into Settings and clicks **"Link Telegram"**.
  3. App tells user to tap **Start** on their new bot while the app automatically polls `getUpdates` in the background, captures `chat_id`, saves the token to Keychain, and sends a **"FaceGuard connected!"** test message.
- **Option B — Optional 1-Click Shared Relay Bot (future/opt-in):**
  - One-click deep link `https://t.me/<OfficialBot>?start=<one-time-code>` via a tiny stateless worker. Only consider if BYO-Bot still causes onboarding friction, and keep BYO-Bot as the default for maximum privacy.

### 4. Native Settings Window
Expose clean controls (with sensible defaults carried over from v1's tuned behavior):
- **General:** Launch at login (`SMAppService`), Auto-arm on idle, Menu-bar status icon style
- **Detection:** Match strictness slider (mapped to embedding distance threshold), Owner grace window (default 5s), Consecutive unknown-frame streak (default 2), Cooldown (default 15s), Optional "Lock on walk-away" toggle
- **Alerts:** Telegram connection wizard, Include intruder photo toggle, Send test alert button, Snapshot retention limit
- **Permissions:** Live status indicators + 1-click "Open System Settings" buttons for **Camera** and (if needed) **Accessibility**

---

## Engineering Non-Negotiables (Carry Forward from v1 `REPORT.md`)

1. **Lock BEFORE network I/O:** Trigger screen lock immediately; upload Telegram alert asynchronously afterward so network latency never delays locking.
2. **Full-resolution verification on lock frame:** Keep v1's two-stage check (fast downscaled detection in idle loop → full-res re-verify before locking) to avoid false locks on partially occluded owner faces.
3. **Owner + guest suppression:** Never lock when the enrolled owner is in frame alongside a guest (plus the 5s owner grace window).
4. **Clean lock/unlock recovery:** Listen to `NSWorkspace.screensDidSleepNotification` / `com.apple.screenIsLocked` & `com.apple.screenIsUnlocked` distributed notifications to cleanly stop/restart the `AVCaptureSession` (no v1 `os.execv` hack).
5. **Clean snapshots:** Always save/send the raw unannotated frame.

---

## Milestones

- **M1 — Native Skeleton & Permissions:** SwiftUI menu-bar app, Camera permission onboarding, `AVCaptureSession` live preview, clean lock/unlock lifecycle handling, reliable macOS screen lock.
- **M2 — On-Device Face Enrollment & Recognition:** Vision face detection + CoreML face embeddings, guided enrollment UI (single-face guard), two-stage lock logic, Keychain/Application Support storage.
- **M3 — Easy Telegram Wizard & Settings:** Guided BYO-Bot linking (auto `getUpdates` polling + test message), full Settings window, Armed/Auto-Arm on idle, snapshot history & pruning.
- **M4 — Polish & Public Release:** App icon, README update + demo GIF + unsigned-install note, MIT `LICENSE`, GitHub Release with unsigned `.dmg`.
