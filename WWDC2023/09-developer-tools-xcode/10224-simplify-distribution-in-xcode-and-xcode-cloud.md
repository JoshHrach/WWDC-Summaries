# Simplify distribution in Xcode and Xcode Cloud
**WWDC23 · Session 10224** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10224/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, Xcode 15, Xcode Cloud

## Overview
Xcode 15 significantly streamlines the process of distributing apps to TestFlight, the App Store, and direct distribution (notarization). The Organizer's new "streamlined distribution" options collapse the multi-step export workflow into a single click with sensible defaults pre-selected. A brand-new "TestFlight Internal Only" option enables rapid team iteration on development branches without risk of those builds ever reaching the App Store.

For teams using Xcode Cloud, this session shows two key automation improvements: automatically populating TestFlight "What to Test" notes from Git commit messages via a `ci_post_xcodebuild.sh` script, and a new Notarize post-action that lets Xcode Cloud handle the full notarization flow for direct Mac app distribution without any manual steps.

## Key Topics

### Streamlined Distribution in Xcode 15
The Xcode Organizer (Product > Archive, then "Distribute App") now offers pre-configured streamlined options:

- **TestFlight & App Store** — full TestFlight functionality with ability to submit to the App Store
- **TestFlight Internal Only** (**[NEW]**) — share with team members; build can never be submitted to the App Store; ideal for feature branch iteration
- **Debugging** — export signed build installable on registered devices (debug signing)
- **Release Testing** — export signed build mimicking App Store signing, installable on registered devices
- **Custom** — full manual configuration as before

All streamlined options automatically apply recommended settings: automatic signing/re-signing, embedded app symbols for server-side crash log symbolication, auto-incremented build numbers, and stripped Swift dylib symbol information.

### Archiving from Simulator Selection
Xcode 15 archives correctly even when a simulator is selected as the destination — it builds for all necessary device CPU architectures automatically.

### TestFlight "Ready to Test" Notification (New)
After an uploaded build finishes processing in App Store Connect, a new system notification alerts the developer that the build is ready to test in TestFlight.

### Xcode Cloud: Automated TestFlight Distribution
Configure an Xcode Cloud workflow with:
1. Archive action set to "TestFlight (Internal Testing Only)" — prevents accidental App Store submission.
2. A "TestFlight Internal" post-action targeting a specific tester group.

A `ci_post_xcodebuild.sh` script can automatically populate the TestFlight "What to Test" field from the most recent Git commit message, surfacing it per-locale (e.g., `WhatToTest.en-US.txt`) when Xcode Cloud uploads the build.

### Xcode Cloud: Notarize Post-Action (New)
A new "Notarize" post-action in Xcode Cloud workflows automates the full notarization flow for Mac apps:
1. Archive and test actions run on the configured start condition (e.g., push to release branch).
2. The Notarize post-action submits the archive to Apple's notary service, receives the ticket, and staples it.
3. The notarized `.app` can be downloaded directly from Xcode Cloud's build report.

Requires Admin or App Manager role to configure notarization workflows.

### Notarization Overview
Notarization verifies apps against Apple's malicious content scanner. The notary service issues a ticket that is stapled to the app. At first launch on a customer's Mac, macOS verifies both the stapled ticket and the notary service ticket to confirm the app was cleared by Apple.

## APIs & Frameworks

- **Xcode 15**
  - **Xcode Organizer** — window for managing archives and distribution
  - **Streamlined Distribution** — new one-click distribution UI **[NEW]**
    - "TestFlight & App Store" option
    - "TestFlight Internal Only" option **[NEW]**
    - "Debugging" option
    - "Release Testing" option
    - "Custom" option (unchanged)
  - Automatic signing re-sign
  - Auto-increment build number
  - Embedded app symbols (dSYMs for server-side symbolication)
  - Stripped Swift embedded dylib symbols
  - "Ready to Test" push notification after build processing **[NEW]**
  - **Integrate menu** — new top-level menu for Xcode Cloud workflow management **[NEW]**
    - "Manage Workflows" — open workflow editor
    - "Commit" — commit and push changes
- **Xcode Cloud**
  - **Archive action** — "TestFlight (Internal Testing Only)" deployment option **[NEW]**
  - **TestFlight Internal post-action** — deploy build to a specific internal tester group
  - **Notarize post-action** — fully automated Mac app notarization **[NEW]**
  - `ci_post_xcodebuild.sh` — custom build script hook for post-archive automation
    - `$CI_APP_STORE_SIGNED_APP_PATH` — environment variable indicating a signed build exists
    - `TestFlight/WhatToTest.en-US.txt` — file path for automatic "What to Test" notes
  - Download Notarized App — button in build report to download stapled app
- **TestFlight** — beta distribution; feedback (screenshots + text) viewable in Xcode Organizer > Feedback tab
- **App Store Connect** — Notarization and TestFlight management portal
- **Notarization** / **Apple Notary Service** — server-side malicious content scanning for Mac apps
- **Ticket stapling** — embedding the notarization ticket into the app bundle for offline verification

## Code Highlights

`ci_post_xcodebuild.sh` — populate TestFlight "What to Test" from Git commit message:
```zsh
#!/bin/zsh
# ci_post_xcodebuild.sh

if [[ -d "$CI_APP_STORE_SIGNED_APP_PATH" ]]; then
  TESTFLIGHT_DIR_PATH=../TestFlight
  mkdir $TESTFLIGHT_DIR_PATH
  git log -1 --pretty=format:"%s" >! $TESTFLIGHT_DIR_PATH/WhatToTest.en-US.txt
fi
```

## Takeaways

- Xcode 15's streamlined distribution options reduce multi-step export flows to a single click; use "TestFlight Internal Only" for feature branch builds that should never reach the App Store.
- The new Integrate menu in Xcode 15 brings Xcode Cloud workflow management directly into the IDE — no more switching to App Store Connect to manage CI workflows.
- Use `ci_post_xcodebuild.sh` with the `$CI_APP_STORE_SIGNED_APP_PATH` environment variable to automatically populate TestFlight "What to Test" notes from Git commit messages.
- The new Xcode Cloud Notarize post-action fully automates Mac app notarization; add it to a release workflow so every push to the release branch produces a downloadable, notarized `.app`.

---
_Source: WWDC23 Session 10224 page (abstract, chapter summaries, code samples, and resource links)._
