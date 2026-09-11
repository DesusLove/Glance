<div align="center">

<img src="glance/Assets.xcassets/appicon.imageset/appicon.png" alt="Glance" width="140">

# Glance

**Face unlock for your Mac.**

Unlock with a glance — no typing, no reaching for Touch ID. Everything runs
on-device with Vision & Core ML, wrapped in a Dynamic Island-style notch UI.

[![MIT License](https://img.shields.io/badge/license-MIT-black.svg)](LICENSE)
![macOS](https://img.shields.io/badge/macOS-15%2B-black.svg)
![Swift](https://img.shields.io/badge/Swift-SwiftUI-black.svg)
![Model](https://img.shields.io/badge/Core%20ML-ArcFace-black.svg)

<a href="https://github.com/DesusLove/Glance/releases/latest"><img width="200" src="https://github.com/user-attachments/assets/cdb8af97-1ee2-4669-b7cb-dcfb56c9dd61" alt="Download for Mac" /></a>

[Installation](#-installation) · [How It Works](#-how-it-works) · [Features](#-features) · [Privacy & Security](#-privacy--security) · [Building](#-building-from-source)

https://github.com/user-attachments/assets/77438826-80a9-4ab2-9fc3-42407a2d0adb

</div>

> [!WARNING]
> ### Glance is not as secure as Apple's Face ID or Touch ID
>
> MacBooks don't have the depth sensors that make iPhone Face ID trustworthy. An
> iPhone builds a 3D map of your face; a MacBook webcam sees a flat 2D image. That means:
>
> - ✅ Defeats, with reasonable confidence, a **printed photo** and a **photo on a phone screen** — Heavy liveness detection must be turned on
> - ❌ Does **not** reliably defeat a **video** of you
> - 🔑 macOS offers no API for a third-party app to authorize a login, so Glance unlocks by **typing your stored password** on the lock screen
>
> Glance is a **convenience feature, not a security upgrade.** Only continue if you accept the tradeoff.

## 📦 Installation

**Requirements:** macOS 15 Sequoia or later · Apple Silicon or Intel Mac

1. Download the latest **Glance.dmg** from [Releases](https://github.com/DesusLove/Glance/releases/latest).
2. Open the `.dmg` and drag **Glance** into `/Applications`.
3. Launch the app and follow the onboarding.

## 📖 How It Works

1. **Enroll** — Onboarding guides you through capturing your face while turning your head in nine directions. Each frame becomes a 512-number *embedding* — a mathematical fingerprint — and the image is thrown away.
2. **Authorize** — Enter your Mac password once. It's encrypted behind Touch ID and never leaves the device.
3. **Lock** — When your Mac locks or wakes from sleep, the animation appears in the notch and starts searching for a face.
4. **Unlock** — If it's you — and the liveness checks agree you're a real person — Glance types the password and you're in.

## ✨ Features

| Feature | Description |
|---|---|
| **Face unlock** | Triggers on wake, on lock, or on pressing space at the lock screen. Pick any combination. |
| **Multiple identities** | Enroll several people, or several versions of yourself — with glasses, a beard, different lighting. Toggle any of them off without deleting. |
| **Liveness checks** | Watches for the motion and reflections that separate a real face from a photo. *Light* or *Heavy* strictness, or off. |
| **Notch UI** | A closed pill that expands into a scan animation with success and failure states. Hover to retry — or turn animations off entirely and Glance stays invisible. |
| **Camera & display** | Choose which camera to use, including different cameras for the built-in display vs. an external monitor. |
| **Auto-locking sessions** | The Touch ID session re-locks itself after an idle period you choose, so an unattended Mac doesn't stay authorized forever. |
| **Trackpad haptics** | Hovering over the notch will trigger haptics. |
| **Notchless Mac support** | On Macs without a notch, the UI becomes a pill-shaped, Dynamic Island-style design. |
| **Your data, your call** | Edit or delete your enrolment or stored password at any time. The encrypted files are removed immediately. |

---

# 🔒 Privacy & Security

Glance is designed to keep biometric data and credentials on-device.

### Face data

Glance never stores camera images. During enrollment, each captured face is converted into a **512-dimensional embedding** using an ArcFace-based Core ML model. The original frame is then discarded.

Embeddings are stored locally and encrypted with **AES-GCM**.

### Credentials

Your Mac password is stored as encrypted data and is never written to disk in plaintext. The encryption key is a **256-bit AES key stored in the macOS Keychain**, protected by `userPresence` — requiring Touch ID or your device password.

The key is only held in memory while an authorized Glance session is active.

### Unlock pipeline

Glance won't type your password simply because a face matches. An unlock requires **all** of the following:

1. A valid Glance session is authorized.
2. The Mac is actually at the lock screen.
3. An enabled identity matches above the configured similarity threshold.
4. Liveness checks accept the detected face.
5. Accessibility permission is available to enter the password.

Face recognition and liveness detection run independently and must both succeed before the password is entered.

### How it tells a face from a photo

Five independent cues over a rolling ~2s window, in two roles:

| Cue | Role | What it looks for |
|---|---|---|
| **Gloss / glare** | 🚫 Deny | Large flat specular highlight — glass or screen glare rather than skin's small scattered shine. |
| **Device detected** | 🚫 Deny | A device-shaped rectangle overlapping the face — a phone or tablet held up. |
| **Flat vs 3D** | ✅ Confirm | Held-out nose points miss the plane fit — the face has real depth. |
| **Depth / pose** | ✅ Confirm | Nose offset tracks head yaw — the nose sits off the eye plane, so this isn't flat. |
| **Blink** | ✅ Confirm | Eye aspect ratio dipped and recovered — a photo cannot blink. |

- **Deny cues** are evidence of a spoof. Either one fails the scan outright and overrides anything else.
- **Confirm cues** are evidence of a real face. Any one is enough — and their absence is never a failure, since a live person can sit still and not blink.

*Light* detection only runs deny cues. *Heavy* detection runs both deny and confirm cues.

### Local by design

Face recognition, enrollment, and liveness detection run entirely on-device using Vision and Core ML. Glance does not send face data, camera frames, or credentials to any server.

### Permissions

| Permission | Why |
|---|---|
| **Camera** | To see your face. Frames are processed in memory and never written to disk. |
| **Accessibility** | To type your password into the lock screen. |
| **Touch ID** | Gates the key that encrypts your face data and password. |

### Face Lab

> [!TIP]
> Face Lab is a hidden debug console for testing face recognition and liveness detection with real values.
>
> **To open it:** Settings → About → click the app icon 5 times. A debug section appears in the sidebar.

### Tech stack

| Layer | Technology |
|---|---|
| UI | SwiftUI, notch overlay + Dynamic Island-style animations |
| Face detection | Apple Vision framework |
| Recognition | ArcFace (MobileFaceNet) via Core ML — 512-d embeddings |
| Liveness | Custom cue engine: geometry, parallax, blink, glare, device detection |
| Crypto | CryptoKit AES-GCM + Keychain (`userPresence`) |
| Unlock | CGEvent keystroke injection (Accessibility) |
| Updates | Sparkle |

---

## 🛠 Building from Source

### Prerequisites

- macOS 15+
- Xcode 26+

### Steps

1. Clone the repository:
   ```bash
   git clone https://github.com/DesusLove/Glance.git
   cd Glance
   ```
2. Open in Xcode:
   ```bash
   open glance/glance.xcodeproj
   ```
3. Build and run:
   - Press `Cmd + R` (or click ▶️ Run).

> [!NOTE]
> On first build, Xcode resolves the Sparkle package automatically. The ArcFace Core ML model ships in the repo at `glance/Models/ArcFace.mlpackage` — no separate download needed.

## 🤝 Contributing

Not currently accepting PRs — feel free to fork this project.

App feedback goes to [tryglance.app/feedback](https://tryglance.app/feedback).

## 🙏 Acknowledgements

- **[The Boring Notch](https://github.com/TheBoredTeam/boring.notch)** — for the notch window physics.
- **[InsightFace](https://github.com/deepinsight/insightface)** — the ArcFace model doing the recognition.

## 📄 License

[MIT](LICENSE) © Kunta Solomon Dongo
