# Meet TestFlight on Mac
**WWDC21 · Session 10170** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10170/)

_Platforms:_ macOS Monterey 12, iOS 15

## Overview
TestFlight arrives on macOS, fulfilling one of the most requested developer features. Available as a Mac App Store download starting fall 2021, TestFlight on Mac brings the complete beta testing workflow — build distribution, automatic updates, screenshot feedback, and crash reporting — to native Mac apps and to iOS apps running on Apple Silicon Macs. The session also covers two additional improvements: multiple internal test groups and Xcode Cloud integration with Build Groups.

## Key Topics

**TestFlight on Mac: Tester Experience**
Testers install the TestFlight app from the Mac App Store. They accept an invite via email or public link, browse builds grouped by version, and click Install to download. Beta apps launch from TestFlight or from Dock, Launchpad, or Finder. A yellow dot distinguishes beta apps in the Dock and Launchpad; Finder labels them "Beta Application." Automatic updates keep testers on the latest build. Feedback is submitted by taking a screenshot, attaching it, and adding comments within TestFlight. Crashes are automatically captured; testers can add notes before submitting logs.

**Native Mac App Distribution**
Native Mac apps require a provisioning profile for TestFlight distribution. With Xcode Automatic Signing the profile is created and embedded automatically. Manual signing requires creating and uploading the profile via the Developer portal. Uploaded builds appear under the macOS platform in App Store Connect; groups, build metrics (invitations, installs, sessions, crashes, feedback), and feedback filtering by Mac device and macOS version all work identically to iOS.

**iOS Apps on Apple Silicon Macs**
Each tester group has a new toggle: "Enable TestFlight availability for iPhone and iPad apps on Apple Silicon Macs." When enabled, iOS builds in that group are installable on Apple Silicon Macs via TestFlight. Metrics from Apple Silicon Macs roll up into iOS platform counts. Feedback (screenshots and crashes) appears under the iOS platform filter in App Store Connect.

**Multiple Internal Groups**
Previously limited to a single internal group, TestFlight now supports multiple named internal groups. Each internal group can be configured independently:
- **Automatic distribution** — all current and future builds are available to this group immediately upon processing.
- **Manual distribution** — specific builds are added to the group by the developer.
- **Feedback** — enable or disable per group, just like external groups.

This enables separate groups for Dev Team (automatic, all builds) and QA Team (manual, stable builds only) without any overlap in build access.

**Xcode Cloud Integration: Build Groups**
Xcode Cloud workflows automatically distribute builds to TestFlight groups. Builds processed by Xcode Cloud are organized into Build Groups — groupings by Xcode Cloud workflow name and Git branch — visible in App Store Connect and the App Store Connect iOS app. Internal testers can browse builds by familiar branch names. Feedback in App Store Connect can be filtered by Build Group.

## APIs & Frameworks

### TestFlight on Mac **[NEW]**
- TestFlight Mac app — available on Mac App Store; supports native Mac apps and iOS apps on Apple Silicon Mac **[NEW]**
- Beta app visual indicator — yellow dot in Dock/Launchpad, "Beta Application" label in Finder **[NEW]**

### App Store Connect — macOS Platform Support **[NEW]**
- macOS platform filter in Crashes and Screenshots feedback sections **[NEW]**
- Build metrics for macOS: invited testers, installs, 7-day sessions, crashes, feedback count **[NEW]**
- Mac device and macOS version filter in feedback views **[NEW]**

### App Store Connect — iOS Apps on Apple Silicon Mac **[NEW]**
- Per-group toggle: "Enable TestFlight availability for iPhone and iPad apps on Apple Silicon Macs" **[NEW]**

### App Store Connect — Multiple Internal Groups **[NEW]**
- Create multiple named internal testing groups **[NEW]**
- Per-group automatic distribution toggle **[NEW]**
- Per-group feedback enable/disable **[NEW]**
- `BetaGroups` resource in App Store Connect API — manage internal groups programmatically **[NEW configuration options]**

### Xcode Cloud Integration **[NEW]**
- Automatic TestFlight distribution from Xcode Cloud workflows to specified groups **[NEW]**
- Build Groups — builds organized by Xcode Cloud workflow name and Git branch in App Store Connect **[NEW]**
- Build Group filter in App Store Connect feedback section **[NEW]**

## Code Highlights

No client-side code is required for TestFlight integration. The provisioning profile requirement for native Mac apps is the main technical step:

```bash
# Automatic signing: Xcode creates and embeds the profile automatically.
# Manual signing: create a macOS App Development provisioning profile in the
# Developer portal, download, and add it to the app target's build settings
# under PROVISIONING_PROFILE_SPECIFIER.
```

App Store Connect API — manage internal beta groups (BetaGroups resource):
```
POST /v1/betaGroups
{
  "data": {
    "type": "betaGroups",
    "attributes": {
      "name": "QA Team",
      "isInternalGroup": true,
      "hasAccessToAllBuilds": false,
      "feedbackEnabled": true
    },
    ...
  }
}
```

## Takeaways
- TestFlight on Mac requires no changes to app source code for native Mac apps built with Automatic Signing — the provisioning profile is handled automatically.
- The Apple Silicon Mac toggle per tester group gives granular control over which testers can run iOS builds on Mac, independent of who tests the native Mac build.
- Multiple internal groups replace the single internal group with per-group build distribution and feedback configuration, enabling separate Dev and QA workflows.
- Xcode Cloud's Build Groups surface builds by Git branch name in App Store Connect, making it easier for internal testers to find the right build without relying on build numbers.

---
_Source: WWDC21 Session 10170 page (abstract, chapter summaries, code samples, and resource links)._
