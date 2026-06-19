# Distribute Apps in Xcode with Cloud Signing
**WWDC21 · Session 10204** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10204/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, tvOS 15, watchOS 8

## Overview
This session covers the end-to-end app distribution workflow in Xcode 13, with a focus on three major new features: cloud signing, in-Xcode app record creation, and automatic build number management. Together these changes eliminate all pre-distribution setup requirements—no certificates, no App Store Connect prep, and no build number bookkeeping needed before hitting Upload.

The session walks through uploading an iOS app to App Store Connect for TestFlight, then surveys every other distribution method available in Xcode 13 (Mac App Store, Developer ID, Ad Hoc, Enterprise/Custom Apps). It closes with a detailed look at automating distribution via `xcodebuild` and Xcode Cloud, including the new App Store Connect API key authentication for headless CI environments.

## Key Topics
- **Cloud Signing (NEW):** Certificates and private keys are stored and used server-side. Xcode generates a partial signature (content hashes), sends them to Apple servers, receives the completed signature back, and inserts it into the IPA—no local certificate setup required. Supported for all distribution methods (App Store, Developer ID, Enterprise). Admins/Account Holders get cloud signing by default; Developers need the "Access to Cloud Managed Distribution Certificate" checkbox in App Store Connect.
- **App Record Creation in Xcode (NEW):** The distribution assistant offers to create a new App Store Connect app record (name, bundle ID, primary language) directly from the archive before the first upload, removing the prior requirement to set it up on the website first.
- **Build Number Management (NEW):** If Xcode detects a build number already used on App Store Connect or a non-incrementing value, it offers to automatically increment the build number in the archive to a valid value.
- **Distribution Methods:** App Store Connect (Upload or Export to IPA), Mac App Store (shared record if same bundle ID), Developer ID + Notarization, Ad Hoc (up to 100 registered devices per type), Enterprise/Custom Apps.
- **Automating with xcodebuild:** `xcodebuild -exportArchive` with an export options plist; App Store Connect API key flags for headless auth (`-authenticationKeyIssuerID`, `-authenticationKeyID`, `-authenticationKeyPath`).
- **Xcode Cloud Integration:** Automated distribution to App Store Connect as part of CI workflows.

## APIs & Frameworks
- **Xcode 13 Organizer** – Archive management, app analytics (crashes, energy, insights, metrics)
- **Cloud Signing** **[NEW]** – Server-side certificate/key storage and signing for automatic signing mode
- **App Record Creation in Distribution Assistant** **[NEW]** – Creates App Store Connect app record directly from Xcode
- **Build Number Management** **[NEW]** – Auto-increments build number in archive during distribution
- **xcodebuild** command-line tool:
  - `xcodebuild archive` – Creates a developer-signed archive
  - `xcodebuild -exportArchive` – Exports/uploads from an archive
  - `-archivePath` – Path to the `.xcarchive`
  - `-exportOptionsPlist` – Path to the export options property list
  - `-allowProvisioningUpdates` – Allows communication with Apple Developer website
  - `-authenticationKeyIssuerID` **[NEW]** – App Store Connect API key Issuer ID
  - `-authenticationKeyID` **[NEW]** – App Store Connect API key Key ID
  - `-authenticationKeyPath` **[NEW]** – Path to downloaded `.p8` API key file
- **Export Options Plist** – Records all distribution choices (method, signing, bitcode, symbols) for reproducible automation
- **App Store Connect API Keys** – Used as credentials for headless xcodebuild authentication **[NEW]**
- **TestFlight** – Beta distribution via App Store Connect; newly available on macOS Monterey
- **Developer ID + Notarization** – Signs with Developer ID certificate, uploads for malware scanning
- **IPA format** – App package format for distribution
- **Bitcode** – Optional: embed bitcode in IPA for App Store re-compilation
- **Xcode Cloud** – CI/CD service with built-in distribution to App Store Connect

## Code Highlights
Full `xcodebuild` command for automated distribution with App Store Connect API key authentication:
```bash
xcodebuild -exportArchive \
  -archivePath /path/to/Baker.xcarchive \
  -exportOptionsPlist /path/to/ExportOptions.plist \
  -allowProvisioningUpdates \
  -authenticationKeyIssuerID "ISSUER_ID" \
  -authenticationKeyID "KEY_ID" \
  -authenticationKeyPath /path/to/AuthKey_KEY_ID.p8
```

Minimal `ExportOptions.plist` for App Store upload:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "...">
<plist version="1.0">
<dict>
  <key>method</key><string>app-store</string>
  <key>uploadBitcode</key><true/>
  <key>signingStyle</key><string>automatic</string>
</dict>
</plist>
```

## Takeaways
- Cloud signing removes the most common distribution blocker (missing local certificates/keys); any team member with the right App Store Connect role can now distribute without machine setup.
- The new in-Xcode app record creation and build number auto-increment together mean a brand-new app can go from first archive to first TestFlight build in one uninterrupted workflow.
- App Store Connect API keys now enable fully headless `xcodebuild` distribution without pre-signing into Xcode, making clean CI environments practical.
- All distribution methods—App Store, Developer ID, Ad Hoc, Enterprise—benefit from cloud signing, making the workflow consistent regardless of target channel.

---
_Source: WWDC21 Session 10204 page (abstract, chapter summaries, code samples, and resource links)._
