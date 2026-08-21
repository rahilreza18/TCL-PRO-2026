# TCL Android App — Workflow Summary

## What this is
The Android app is a **TWA (Trusted Web Activity)** — a thin native wrapper that opens your live hosted website fullscreen, with no browser UI. It has no baked-in content; it always loads the live site.

## Key links & locations

| Item | Value |
|---|---|
| Live website | https://tcl-pro-live.web.app |
| Firebase project | `tcl-pro-live` |
| Android package ID | `com.tcl.tenniscricketleague` |
| App display name | Tennis Cricket League Live & Scoring (launcher name: "TCL") |
| Android project folder | `android-twa/` |
| Signed release APK (for device testing / sideloading) | `android-twa/app-release-signed.apk` |
| Signed production AAB (for Play Store upload) | `android-twa/app-release-bundle.aab` |

## Critical — signing key (back this up now)

| Item | Value |
|---|---|
| Keystore file | `android-twa/android.keystore` |
| Keystore password | `<YOUR_KEYSTORE_PASSWORD>` |
| Key password | `<YOUR_KEYSTORE_PASSWORD>` |
| Key alias | `android` |
| Cert valid until | 2054-01-06 |

**If this keystore or password is lost, the app can never be updated on Play Store again — it would have to be republished as a brand new app.** Back up `android.keystore` somewhere safe (cloud drive, password manager) outside this project folder.

The actual password is intentionally not written in this file (kept out of git). Store it in a password manager alongside the keystore backup.

## Local toolchain (installed on this machine, not in git)
- JDK 17: `C:\Users\admin\.bubblewrap\jdk\jdk-17.0.11+9`
- Android SDK: `C:\Users\admin\android-sdk`
- Bubblewrap CLI: installed globally via npm (`@bubblewrap/cli`)

---

## Workflow 1: Regular website/content updates (the common case)

No Android rebuild needed. The installed app always loads the live site.

```
# from project root
firebase deploy
```
Anyone with the app already installed sees the change next time they open it.

---

## Workflow 2: Native/app-level changes
Needed only for: app icon, app name, splash/theme colors, permissions, package id changes, or a Play Store–required version bump.

1. Edit `android-twa/twa-manifest.json` (bump `appVersionCode` and `appVersion` if this is a Play Store update).
2. Rebuild APK (PowerShell — not git-bash, due to a `gradlew.bat` path-resolution quirk in bash):
   ```powershell
   cd android-twa
   $env:JAVA_HOME = "C:\Users\admin\.bubblewrap\jdk\jdk-17.0.11+9"
   $env:ANDROID_HOME = "C:\Users\admin\android-sdk"
   .\gradlew.bat assembleRelease --stacktrace
   ```
3. Zipalign + sign the APK (git-bash):
   ```bash
   cd android-twa
   BUILD_TOOLS="$HOME/android-sdk/build-tools/35.0.0"
   "$BUILD_TOOLS/zipalign.exe" -v -p 4 app/build/outputs/apk/release/app-release-unsigned.apk app-release-aligned.apk
   "$BUILD_TOOLS/apksigner.bat" sign --ks android.keystore --ks-pass pass:<YOUR_KEYSTORE_PASSWORD> --key-pass pass:<YOUR_KEYSTORE_PASSWORD> --ks-key-alias android --out app-release-signed.apk app-release-aligned.apk
   ```
4. For a Play Store release, also build + sign the AAB:
   ```powershell
   # PowerShell
   .\gradlew.bat bundleRelease --stacktrace
   ```
   ```bash
   # git-bash — jarsigner, not apksigner, for AAB
   cd android-twa
   JARSIGNER="/c/Users/admin/.bubblewrap/jdk/jdk-17.0.11+9/bin/jarsigner.exe"
   cp app/build/outputs/bundle/release/app-release.aab app-release-bundle.aab
   "$JARSIGNER" -keystore android.keystore -storepass "<YOUR_KEYSTORE_PASSWORD>" -keypass "<YOUR_KEYSTORE_PASSWORD>" app-release-bundle.aab android
   ```

---

## Digital Asset Links (already set up — don't remove)
For the app to open fullscreen (instead of falling back to a browser-style Custom Tab with a URL bar), Android must verify the app and website are owned by the same party. This is done via:

`public/.well-known/assetlinks.json` — contains the app's SHA-256 signing certificate fingerprint, deployed at:
https://tcl-pro-live.web.app/.well-known/assetlinks.json

`firebase.json` has a rule (`"!/.well-known/**"`) to make sure Firebase's default dotfile-ignore doesn't exclude this folder from deploys. **Do not remove that rule.**

If the signing key ever changes, `assetlinks.json` must be regenerated with the new fingerprint:
```bash
KEYTOOL="/c/Users/admin/.bubblewrap/jdk/jdk-17.0.11+9/bin/keytool.exe"
"$KEYTOOL" -list -v -keystore android-twa/android.keystore -alias android -storepass "<YOUR_KEYSTORE_PASSWORD>" | grep -A1 "SHA256:"
```

---

## Where we are / what's left

- [x] Web app hosted on Firebase (`tcl-pro-live`)
- [x] PWA-ready (manifest, service worker, icons)
- [x] Android TWA app built, signed, tested on a real device — works fullscreen
- [x] Production AAB built and signed
- [ ] Replace placeholder icons/branding with real assets (optional but recommended before Play Store)
- [ ] Google Play Console developer account setup
- [ ] Store listing (screenshots, descriptions, privacy policy URL)
- [ ] Content rating questionnaire + Data safety form
- [ ] Upload AAB to Play Console, set up release track, submit for review
