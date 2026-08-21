# TCL-PRO-2026

Static site hosted on Firebase Hosting.

- **Live URL:** https://tcl-pro-live.web.app
- **Firebase project:** tcl-pro-live
- **Hosting root:** `public/` (contains `index.html`, `404.html`)

## Deploying

1. Make changes to `public/index.html`.
2. Run:
   ```
   firebase deploy
   ```

## Switching Firebase project (if needed)

```
firebase use tcl-pro-live
```

## PWA

The site is a PWA — installable on Android/desktop Chrome.

- `public/manifest.json` — app name, icons, theme/background color
- `public/sw.js` — service worker (required for installability)
- `public/icons/` — placeholder app icons (192px, 512px, 512px maskable) — swap these for real branding when ready

## Android App (TWA)

The Android app is a **Trusted Web Activity (TWA)** — a thin native wrapper that opens the live hosted site (`https://tcl-pro-live.web.app`) fullscreen, with no browser UI. It contains almost no baked-in content itself.

Project location: `android-twa/`

- `android-twa/twa-manifest.json` — Android app identity (package id, name, colors, icon URLs, signing key reference)
- `android-twa/android.keystore` — **release signing key. Back this up. Losing it means the app can never be updated on Play Store again.**
  - Keystore password / key password: `<YOUR_KEYSTORE_PASSWORD>`
  - Key alias: `android`
- Package ID: `com.tcl.tenniscricketleague`
- Local toolchain (not committed): JDK 17 at `~/.bubblewrap/jdk/jdk-17.0.11+9`, Android SDK at `~/android-sdk`

### Two kinds of updates

**1. Regular content/feature changes (the common case)**
Just deploy the web app as usual:
```
firebase deploy
```
No APK rebuild needed — the installed Android app loads the live site every time it opens.

**2. Native/app-level changes** (icon, app name, splash/theme colors, package id, permissions, or a Play Store–required version bump)
Requires rebuilding the APK:

```
cd android-twa

# 1. Edit twa-manifest.json as needed (bump appVersionCode/appVersion if required)

# 2. Rebuild (from PowerShell, not git-bash — gradlew.bat path resolution issue in bash)
$env:JAVA_HOME = "C:\Users\admin\.bubblewrap\jdk\jdk-17.0.11+9"
$env:ANDROID_HOME = "C:\Users\admin\android-sdk"
.\gradlew.bat assembleRelease --stacktrace

# 3. Zipalign + sign (from git-bash)
BUILD_TOOLS="$HOME/android-sdk/build-tools/35.0.0"
"$BUILD_TOOLS/zipalign.exe" -v -p 4 app/build/outputs/apk/release/app-release-unsigned.apk app-release-aligned.apk
"$BUILD_TOOLS/apksigner.bat" sign --ks android.keystore --ks-pass pass:<YOUR_KEYSTORE_PASSWORD> --key-pass pass:<YOUR_KEYSTORE_PASSWORD> --ks-key-alias android --out app-release-signed.apk app-release-aligned.apk
```

Output: `android-twa/app-release-signed.apk` — install this on a device to test, or use it for Play Store submission (as an AAB for production — see `app/build/outputs/bundle/release/`).
