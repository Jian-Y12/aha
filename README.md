# Aha Toolbox

**An offline-first Android toolbox for students** — timetable OCR, homework & grade tracking,
flashcards, notes, an encrypted photo vault, and one-tap JSON backup.
All data stays on your device: no account, no cloud, no analytics.



## Features

- **Timetable** — multi-semester weekly grid, odd/even-week courses, tap to edit, export as image,
  and **screenshot OCR import** (Tesseract.js + bundled Chinese model, runs 100% offline)
- **Study** — question bank with text & photo questions, practice mode with mastery tracking,
  homework list, grade records, vocabulary review
- **Daily** — expense tracker with budget alerts, parcel pickup codes, BMI & weight log,
  habit tracker, Pomodoro timer, random picker & lucky wheel
- **Favorites** — links, formulas, trips and activities
- **Notes** — text & image notes, screenshot annotation
- **Goals** — progress tracking with archive
- **Encrypted vault** — password-protected photo album (original image bytes, AES-GCM) and
  password list
- **Data safety** — versioned JSON backup with attachments, restore preview & one-step undo,
  rollback on storage-quota failure, read-only mode when local data is corrupted
- **Offline-first** — installable PWA with a service worker; the Android app bundles the OCR engine

## Repository layout

```
web/        Web app source (HTML / CSS / vanilla JS) loaded by the Capacitor WebView
android/    Custom native plugin (system "Save as" via SAF) + Capacitor plugin registry
tools/      Patch-style build pipeline: javac + d8 -> dex, repack APK, zipalign, sign
baseline/   Placeholder for the stock APK the pipeline patches (binary not committed)
docs/       Build notes and changelog
```

## How it works

The UI is a single-page web app (`web/index.html`, loaded by Capacitor 8 on Android).
Data lives in `localStorage` (structured data) and IndexedDB (question images, notes,
encrypted photo blobs). OCR uses Tesseract.js with a bundled `chi_sim` model, so it works
without network. Vault items are encrypted with Web Crypto AES-GCM; backups are versioned
JSON files with SHA-256 integrity checks and one-step undo.

The Android side adds one custom plugin, **AhaDocuments**, which lets the web layer save
exports through the system file picker (`ACTION_CREATE_DOCUMENT`) in 192 KiB chunks.
It is registered through `android/capacitor.plugins.json` — Capacitor loads plugin
classes from that asset at startup, so no `MainActivity` change is required.

## Build

> The build is **patch-based**: the Android shell is a prebuilt APK, and the pipeline
> replaces its `assets/public` web layer, appends the plugin dex and re-signs.
> That is why `baseline/` needs the stock APK.

Prerequisites: Python 3.10+, JDK 21, Android SDK Build-Tools 35, a debug keystore
(`%USERPROFILE%\.android\debug.keystore`, alias `androiddebugkey`).

1. Put the stock APK at `baseline/aha-toolbox-b34-fix5.apk`.
2. Compile the native plugin to dex:

```bash
python tools/compile_plugin.py --src android/AhaDocumentsPlugin.java --output build/dex/classes2.dex
```

3. Repack, align and sign:

```bash
python tools/repack_apk.py ^
  --baseline baseline/aha-toolbox-b34-fix5.apk ^
  --assets web ^
  --dex build/dex/classes2.dex ^
  --output build/aha-toolbox.apk
```

The tool scripts contain machine-specific JDK / SDK paths near the top — adjust them
(or set them to your local Android toolchain) before building.

## Changelog

See [docs/CHANGELOG.md](docs/CHANGELOG.md). Latest build: **b34-fix6** — web-layer bug fixes
(question display, practice grading, batch import atomicity, original-quality vault images,
cross-page Pomodoro, backup validation/rollback, navigation/offline fixes) plus the native
file-save plugin.

## Privacy

- No account, no sign-up, no telemetry.
- All data is stored locally on the device.
- Backups are plain JSON files exported by the user.
- OCR runs entirely on-device.

## License

MIT License — see [LICENSE](LICENSE).
