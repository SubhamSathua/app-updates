# App Updates Manifest Registry

**Centralized static update manifest and release orchestration hub for ScreenHarmony and Android client applications.**

This repository hosts and delivers serverless in-app update manifests (`update.json`), release channel configurations (`stable`, `alpha`), emergency kill-switches, and cryptographic integrity digests via GitHub Pages and edge CDN.

**Use Case:**
Android and multi-platform client applications evaluate static manifest endpoints to determine update availability, enforce minimum supported build floors, deliver instant emergency lockdowns, and download signed release APKs without requiring dedicated backend servers.

---

## # Index
- [Features](#-features)
- [Directory Structure](#-directory-structure)
- [Manifest Schema (RFC-042)](#-manifest-schema-rfc-042)
- [Evaluation States & Decision Engine](#-evaluation-states--decision-engine)
- [How to add a new app](#-how-to-add-a-new-app)
- [Deployment & GitHub Pages](#-deployment--github-pages)
- [Security & Integrity](#-security--integrity)
- [License & Legal](#-license--legal)
- [Links](#-links)
- [Contact](#-contact)

---

## # Features
- **Serverless & Edge-Delivered:** 100% static JSON payload served via GitHub Pages and raw GitHub CDN.
- **Multi-Ring Distribution:** Independent release definitions for `stable` (production) and `alpha` (testing/nightly) channels.
- **Semantic Version Progression:** Standardized semantic string evaluation (`version: "2.9.0"`, `minVersion: "2.0.0"`).
- **Mandatory Floor Deprecation:** Halts obsolete app builds below `minVersion` with customizable deprecation notices.
- **Emergency Kill-Switches:** Remotely lock app operations globally or per-channel during scheduled maintenance or security incidents.
- **SHA-256 Cryptographic Verification:** Protects against corrupted downloads and MitM injection attacks.
- **Multi-Distribution Links:** Direct APK, GitHub Releases, and one-tap Obtainium deep-link integration.

---

## # Directory Structure

```text
app-updates/
├── index.html                     # Central interactive dashboard & simulator
├── screen-harmony-flex/
│   ├── index.html                 # App-specific manifest simulator
│   └── update.json                # Live ScreenHarmony Flex manifest payload
├── LICENSE                        # Apache License 2.0
└── README.md                      # Documentation & protocol specification
```

---

## # Manifest Schema (RFC-042)

Each app directory contains an `update.json` file formatted according to the RFC-042 specification:

```json
{
  "killSwitch": {
    "global": {
      "enabled": false,
      "title": "Service Temporarily Suspended",
      "message": "All app operations are temporarily paused for backend maintenance."
    },
    "alpha": {
      "enabled": false,
      "title": "Alpha Testing Ring Paused",
      "message": "Alpha usage is halted while an emergency build is prepared."
    },
    "stable": {
      "enabled": false,
      "title": "Production Service Paused",
      "message": "Production builds are temporarily paused for maintenance."
    }
  },
  "stable": {
    "version": "2.9.0",
    "minVersion": "2.0.0",
    "releaseDate": "2026-09-18",
    "updateType": "OPTIONAL",
    "deprecationMessage": "Your installed version has reached end-of-life. Please update to continue.",
    "changelog": [
      "Added ScreenHarmony Cloud login and BYOB support.",
      "Added 12-Word Secret Recovery Phrase generator.",
      "Added Android 12-14+ Exact Alarms engine and multi-OEM anti-tamper."
    ],
    "downloads": {
      "directApk": "https://github.com/SubhamSathua/screen-harmony-flex/releases/latest/download/app-prod-universal-release.apk",
      "github": "https://github.com/SubhamSathua/screen-harmony-flex/releases/latest",
      "obtainium": "obtainium://add/https://github.com/SubhamSathua/screen-harmony-flex",
      "fdroid": "https://f-droid.org/packages/com.prism.screenharmony.flex"
    },
    "sha256": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"
  },
  "alpha": {
    "version": "2.9.0-alpha",
    "minVersion": "2.4.0",
    "releaseDate": "2026-09-18",
    "updateType": "OPTIONAL",
    "deprecationMessage": "Experimental build expired. Download the newest nightly iteration.",
    "changelog": [
      "Experimental remote manifest update engine.",
      "Real-time diagnostics logging improvements."
    ],
    "downloads": {
      "directApk": "https://github.com/SubhamSathua/screen-harmony-flex/releases/latest/download/app-alpha-universal-release.apk",
      "github": "https://github.com/SubhamSathua/screen-harmony-flex/releases",
      "obtainium": "obtainium://add/https://github.com/SubhamSathua/screen-harmony-flex"
    },
    "sha256": ""
  }
}
```

---

## # Evaluation States & Decision Engine

1. **Global Kill-Switch (`killSwitch.global.enabled == true`):**
   - Non-dismissible full-screen lock overlay.
   - Halts all application activity across all distribution channels.
2. **Channel Kill-Switch (`killSwitch[channel].enabled == true`):**
   - Blocks users on the targeted testing or production ring while leaving unaffected channels active.
3. **Floor Deprecation (`localVersion < minVersion`):**
   - Non-dismissible compulsory update screen.
   - User cannot dismiss or use older app versions until upgraded.
4. **Update Available (`remoteVersion > localVersion`):**
   - **`CRITICAL`**: Non-dismissible update prompt.
   - **`RECOMMENDED`**: Advisory banner with dismissible reminder on subsequent launches.
   - **`OPTIONAL`**: Standard update dialog with "Update Now" and "Later" options.
5. **Up to Date (`remoteVersion <= localVersion`):**
   - App remains in idle status with no user interruption.

---

## # How to add a new app

1. Create a new directory under root matching your application's slug:
   ```bash
   mkdir my-new-app
   ```
2. Copy the template `update.json` into the folder:
   ```bash
   cp screen-harmony-flex/update.json my-new-app/update.json
   ```
3. Update version numbers, download URLs, changelogs, and SHA-256 hashes.
4. Commit and push to `main` branch. GitHub Pages will serve it at:
   `https://<username>.github.io/app-updates/my-new-app/update.json`

---

## # Deployment & GitHub Pages

This repository is configured to deploy directly via **GitHub Pages**:

* **Root Dashboard:** `https://<username>.github.io/app-updates/`
* **App Endpoints:** `https://<username>.github.io/app-updates/<app-slug>/update.json`
* **Raw Fallback Endpoint:** `https://raw.githubusercontent.com/<username>/app-updates/main/<app-slug>/update.json`

---

## # Security & Integrity

- **Cryptographic Digest (SHA-256):** When using in-app APK downloaders, client applications compute the SHA-256 digest of the downloaded binary before opening Android's `PackageInstaller`. If a checksum mismatch occurs, the file is safely deleted to protect user devices.
- **Zero Secrets / Zero Backend:** This repository contains strictly public release metadata. No private keys, keystores, or API secrets are stored.

---

## # License & Legal
This project is licensed under the **MIT License**.

**Liability Protection:** The author provides this software and manifest registry "as is" without warranties. By using this service, you agree that the author is not liable for any damages, service downtime, or issues resulting from its use.

---

## # Links
- [Main App Repository](https://github.com/SubhamSathua/screen-harmony-flex) - ScreenHarmony Flex source code.
- [MIT License](LICENSE) - View the full license terms.

---

## # Contact
**Author:** Subham Kumar Sathua
**GitHub:** [@SubhamSathua](https://github.com/SubhamSathua)
**Repository:** [app-updates](https://github.com/SubhamSathua/app-updates)

---
Copyright © 2026 Subham Kumar Sathua. Licensed under the MIT License.
