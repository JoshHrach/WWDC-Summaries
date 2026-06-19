# Become a Simulator Expert
**WWDC20 · Session 10647** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10647/)

_Platforms:_ iOS 14, iPadOS 14, tvOS 14, watchOS 7 (Simulator on macOS)

## Overview
Xcode 12 brings several Simulator quality-of-life improvements, and this session provides a practical tour of both the updated Simulator.app UI and the `simctl` command-line tool. Topics include the revamped screenshot workflow, full-screen support, pointer/trackpad capture for iPadOS apps, customizable capture shortcuts, Simulator preferences, and a set of powerful `simctl` sub-commands for managing privacy permissions, sending push notifications, recording video, overriding the status bar, and managing certificates in the keychain.

The session emphasizes that `simctl` lets you automate and script many tasks that previously required manual interaction through Simulator's UI — making it especially valuable for CI workflows and App Store screenshot generation.

## Key Topics
- **Screenshot improvements (Xcode 12)** — Screenshot button in toolbar produces a floating thumbnail; Control-click to save to disk, copy to clipboard, or open in an app; drag to insert into other apps; auto-saved to Desktop if ignored. Device mask can be transparent in screenshots (configurable in Preferences).
- **Full-screen mode (Xcode 12)** **[NEW]** — Simulator can enter full-screen mode alone or tiled side-by-side with Xcode.
- **Pointer and trackpad capture** — Clicking the pointer button in the toolbar enters pointer capture mode for iPadOS simulators; all Mac mouse/trackpad events (pinch, two-finger scroll, three-finger swipe) route to iPadOS; press Escape (or configured shortcut) to release. Keyboard-only capture available separately.
- **Auto-release on focus loss** — Xcode 12 automatically stops pointer capture when the Simulator window loses focus (e.g., hitting a breakpoint in Xcode); capture resumes when focus returns.
- **Simulator Preferences** — Customizable capture-stop shortcut (Escape / both Command keys / Control-Option); simulator lifetime settings (keep booted after quit); visual indicators for finger positions; device mask in screenshots.
- **Create Simulator from within the app (Xcode 12)** **[NEW]** — File > New Simulator lets you name, pick device type, and choose runtime without opening Xcode's Devices & Simulators window.
- **Window scaling modes** — Physical Size (exact on-screen dimensions), Point Accurate (all simulators same point size regardless of scale factor), Pixel Accurate (1:1 pixel mapping).
- **`simctl privacy`** — `grant`, `revoke`, `reset` privacy permissions per app or system-wide; supports `calendar`, `contacts`, `location`, `photos`, and more.
- **`simctl push`** **[NEW behavior]** — Send APNS push notifications to any booted simulator from the command line or by dragging an `.apns` JSON file onto the Simulator window; `Simulator Target Bundle` key in the JSON specifies the app.
- **`simctl io recordVideo`** — Records device screen to an MP4; options: `--codec` (HEVC default, h264), `--mask` (black default, ignored for full frame buffer), `--display` (internal or external); uses GPU acceleration; terminates on Ctrl-C.
- **`simctl status_bar`** — `override` and `clear` to customize time, cellular bars, data network type, Wi-Fi mode, and battery; ideal for App Store screenshot generation.
- **`simctl keychain`** — `add-root-cert` to install a CA certificate into the simulator's trusted root store (then manually trust in Settings > General > About > Certificate Trust Settings); `clear` to wipe saved passwords.

## APIs & Frameworks

This session covers Simulator.app UI and `simctl` CLI. No new Swift/Objective-C APIs are introduced. All commands use `xcrun simctl`:

### simctl Sub-commands
- **`simctl privacy <device> grant <service> <bundleID>`** — Grant permission; services: `calendar`, `contacts`, `location`, `photos`, `camera`, `microphone`, etc.
- **`simctl privacy <device> revoke <service|all> <bundleID>`** — Revoke permission
- **`simctl privacy <device> reset <service|all> [bundleID]`** — Reset to defaults (app-specific or system-wide)
- **`simctl push <device> [bundleID] <payload.json>`** — Send push notification; bundle ID may be in JSON as `Simulator Target Bundle` key
- **`simctl io <device> recordVideo [options] <output.mp4>`** — Record screen video; options: `--codec <hevc|h264>`, `--mask <black|white|ignored>`, `--display <internal|external>`, `--force`
- **`simctl status_bar <device> override [options]`** — Override status bar; options: `--time`, `--cellularBars`, `--dataNetwork`, `--wifiMode`, `--batteryLevel`, `--batteryState`
- **`simctl status_bar <device> clear`** — Remove status bar overrides
- **`simctl keychain <device> add-root-cert <path>`** — Add CA certificate to trusted root store
- **`simctl keychain <device> clear`** — Clear saved passwords

### Push Notification JSON Format
```json
{
  "Simulator Target Bundle": "com.example.MyApp",
  "aps": {
    "alert": {
      "title": "Push Notification",
      "subtitle": "New smoothies available",
      "body": "Check them out!"
    }
  }
}
```

## Code Highlights

Grant and revoke app permissions via simctl:
```shell
xcrun simctl privacy booted grant calendar com.example.MyApp
xcrun simctl privacy booted revoke all com.example.MyApp
xcrun simctl privacy booted reset all
```

Send a push notification from the command line:
```shell
xcrun simctl push booted com.example.MyApp payload.json
# Or with bundle ID in JSON:
xcrun simctl push booted payload.json
```

Record screen video (HEVC, with mask):
```shell
xcrun simctl io booted recordVideo video.mp4
# H.264, no device mask, external display:
xcrun simctl io booted recordVideo --codec h264 --mask ignored video.mp4
xcrun simctl io booted recordVideo --display external external.mp4
```

Override and clear status bar for App Store screenshots:
```shell
xcrun simctl status_bar booted override \
  --time 12:01 --cellularBars 1 --dataNetwork 3g --wifiMode failed
xcrun simctl status_bar booted clear
```

Add a CA certificate to the trusted root store:
```shell
xcrun simctl keychain booted add-root-cert myCA.pem
```

## Takeaways
- Xcode 12 adds full-screen Simulator support and a streamlined screenshot thumbnail workflow; screenshots can be multi-taken in rapid succession for App Store asset generation.
- The pointer capture mode turns the Mac trackpad into a full iPadOS pointer device (pinch, scroll, swipe) to test UIPointerInteraction and trackpad API without physical hardware.
- `simctl privacy grant/revoke/reset` enables automated testing of both the permitted and denied paths for any protected resource — run it in CI before each test suite.
- `simctl push` eliminates the need for a real APNS server during development; just drop a JSON file on the Simulator or invoke the command from a build script.
- `simctl status_bar override` combined with `simctl io recordVideo` produces perfectly controlled App Store screenshots and screen recordings from a single shell script.

---
_Source: WWDC20 Session 10647 page (abstract, transcript, and code samples)._
