# Web → iOS App (Capacitor + GitHub Actions)

This scaffold wraps an existing web app in a native iOS shell using
[Capacitor](https://capacitorjs.com/) and builds it into a signed `.ipa`
automatically on GitHub using a macOS runner.

## Files included

| File | Purpose |
|---|---|
| `package.json` | Node deps + build script for your web app |
| `capacitor.config.json` | Tells Capacitor which folder your built web app lives in |
| `.github/workflows/ios-build.yml` | GitHub Actions pipeline: builds web app → syncs iOS project → signs → archives → exports `.ipa` |
| `ios/ExportOptions.plist` | Tells `xcodebuild` how to sign/export (App Store, Ad Hoc, etc.) |
| `fastlane/Appfile` / `fastlane/Fastfile` | Optional cleaner signing + TestFlight upload via Fastlane |
| `.gitignore` | Keeps secrets, build output, and Pods out of git |

## One-time local setup (do this once on your Mac before pushing)

1. **Install Capacitor into your existing web project:**
   ```bash
   npm install @capacitor/core @capacitor/ios
   npm install -D @capacitor/cli
   npx cap init "YourApp" "com.yourcompany.yourapp" --web-dir=dist
   ```
   Replace `dist` with wherever your web build output goes (`build`, `out`, `public`, etc.), and match it in `capacitor.config.json`.

2. **Add the iOS platform** (this generates the `ios/` Xcode project — commit it to the repo):
   ```bash
   npx cap add ios
   npx cap sync ios
   ```

3. Open once in Xcode to confirm it builds locally:
   ```bash
   npx cap open ios
   ```

## Required GitHub Secrets

Go to **Settings → Secrets and variables → Actions** in your repo and add:

| Secret | How to get it |
|---|---|
| `BUILD_CERTIFICATE_BASE64` | Export your distribution `.p12` cert from Keychain Access, then `base64 -i cert.p12 \| pbcopy` |
| `P12_PASSWORD` | The password you set when exporting the `.p12` |
| `BUILD_PROVISION_PROFILE_BASE64` | Download the `.mobileprovision` from Apple Developer portal, then `base64 -i profile.mobileprovision \| pbcopy` |
| `KEYCHAIN_PASSWORD` | Any password you make up — used only to create a temporary CI keychain |

For TestFlight uploads (optional, via the commented-out step or Fastlane `beta` lane), you'll also need an [App Store Connect API key](https://appstoreconnect.apple.com/access/api).

## Updating placeholders

Before your first push, replace:
- `com.yourcompany.yourapp` → your real bundle ID (in `capacitor.config.json`, `fastlane/Appfile`)
- `YOUR_APPLE_TEAM_ID` → your Apple Developer Team ID (in `ios/ExportOptions.plist`)
- `you@example.com`, team IDs in `fastlane/Appfile`
- Xcode version in the workflow (`Xcode_15.4.app`) if the runner image has a different default — check the [macos-14 runner image notes](https://github.com/actions/runner-images) for available versions

## Triggering a build

Push to `main`, or run the workflow manually from the **Actions** tab (`workflow_dispatch` is enabled). The signed `.ipa` will appear as a downloadable artifact on the workflow run, or you can uncomment the TestFlight step to ship it automatically.
