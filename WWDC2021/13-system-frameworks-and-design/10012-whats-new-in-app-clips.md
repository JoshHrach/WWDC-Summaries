# What's New in App Clips
**WWDC21 · Session 10012** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10012/)

_Platforms:_ iOS 15, iPadOS 15

## Overview
App Clips receive three key enhancements in iOS 15. First, the App Clip Card can now appear as a full-sized overlay directly within a web page in Safari or embedded `SFSafariViewController`, making App Clip discovery far more prominent than the previous slim Smart App Banner. Second, developers can create local experiences on a test device to exercise the full card-to-launch flow without registering an experience in App Store Connect. Third, App Clip Codes — Apple-designed NFC/scan-only visual codes introduced in iOS 14.3 — gain new tooling via a command-line generator with customizable templates and colors.

The session also highlights community App Clips from Firi Games, TikTok, Panera Bread, Honk, and Primer AR Home Design as examples of well-designed experiences that surface through Safari, Messages, Maps, Spotlight, Siri Suggestions, QR/NFC codes, and App Clip Codes.

## Key Topics

### App Clip Card in Safari and SFSafariViewController
Adding `app-clip-display=card` to the existing `apple-itunes-app` meta tag upgrades the Smart App Banner to a full-sized App Clip Card that appears in the center of the web page. This works in both Safari and apps that embed web content via `SFSafariViewController`. If the user taps "View in Safari," Safari remembers the preference and falls back to the regular banner on subsequent loads.

### Testing with Local Experiences
Developers can configure a local experience on their device via Developer Settings > Local Experiences without having any registered App Clip experience in App Store Connect. A local experience requires the app clip's bundle ID, a URL prefix, title, subtitle, and a local photo library image for the header. It supports QR codes, NFC tags, App Clip Codes, Safari, and Messages as invocation methods, but does not appear in Maps, location-based Siri Suggestions, or Spotlight Search (those require registered experiences). Local experiences only work for Xcode-installed or beta-testing app clips.

### App Clip Codes
App Clip Codes are available on the approximately 1 billion devices running iOS 14.3+. Two variants exist: NFC-integrated (tap or scan) and Scan Only. Codes are generated either by downloading from App Store Connect (for registered URLs) or using the `AppClipCodeGenerator` command-line tool (for local testing, bulk generation, and customization). The tool supports multiple templates, foreground/background color customization, optional logo hiding, and produces SVG output that scales to any print size. App Clip Codes can also serve as ARKit image anchors for AR-based experiences.

## APIs & Frameworks

**App Clip / WebKit**
- `apple-itunes-app` HTML meta tag — existing
- `app-clip-display=card` key in `apple-itunes-app` meta tag **[NEW]** — triggers full-sized card mode in Safari/SFSafariViewController
- `SFSafariViewController` — used to render web content in apps with App Clip card support

**App Clips Framework**
- App Clip local experience configuration via Developer Settings **[NEW]** — no API; device-only developer tool
- URL-based invocation handling (unchanged) — existing URL handling works for App Clip Codes without modifications

**App Clip Code Generator (command-line tool)**
- `AppClipCodeGenerator templates` — lists available visual templates
- `AppClipCodeGenerator generate` — generates an SVG App Clip Code
  - `--type` — `nfc` or `scanOnly`
  - `--url` — registered or test URL to encode
  - `--template` — template number (1–N)
  - `--output` — output SVG filename
- SVG output format for scalable print production **[NEW improved tooling]**

**ARKit**
- App Clip Code as ARKit image anchor **[NEW]** — enables position tracking of a physical App Clip Code in the real world (see WWDC21 Session 10073 "Explore ARKit 5")

**App Store Connect**
- Local experience configuration (Developer Settings) **[NEW]**
- App Clip Code download for registered URLs **[NEW]**

## Code Highlights

Enabling the full-sized App Clip Card in a web page:
```html
<meta name="apple-itunes-app"
      content="app-id=XXXXXXXXX, app-clip-bundle-id=com.example.app.clip,
               app-clip-display=card">
```

Generating an App Clip Code via command line:
```bash
AppClipCodeGenerator generate \
  --type nfc \
  --url https://fruta.example \
  --template 4 \
  --output fruta.svg
```

## Takeaways
- Adding `app-clip-display=card` to the `apple-itunes-app` meta tag is a one-line change that dramatically improves App Clip discovery inside third-party apps using `SFSafariViewController`.
- Local experiences let developers test the complete App Clip Card flow on-device without any App Store Connect configuration.
- App Clip Codes (available since iOS 14.3) are the recommended physical-world invocation method and support NFC, scanning, AR anchoring, and full SVG customization via the command-line generator.
- App Clips appear in iOS 15 Spotlight search results and can be proactively suggested by Siri based on location proximity.

---
_Source: WWDC21 Session 10012 page (abstract, chapter summaries, code samples, and resource links)._
