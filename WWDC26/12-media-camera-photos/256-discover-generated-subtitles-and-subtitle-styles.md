# Discover Generated Subtitles and Subtitle Styles
**WWDC26 · Session 256** · [Watch](https://developer.apple.com/videos/play/wwdc2026/256/)

_Platforms:_ iOS, iPadOS, macOS, tvOS

## Overview
WWDC26 brings two accessibility and localization improvements to video playback: on-device AI-generated subtitles and a subtitle style preview API. Generated subtitles require no changes to media authoring or app code for apps using `AVPlayerViewController` or `AVPlayerView` — the system generates and displays them automatically for supported content.

Generated subtitles use on-device models (no server round-trip) and work in two modes: transcription (speech → text in the source language) and translation (existing subtitles → a different language). They are surfaced to users with a sparkle badge distinguishing them from manually authored captions.

The subtitle style preview API lets apps show users a live preview of their selected caption style before they commit, using `AVPlayerLayer` and `MediaAccessibility` — enabling a more polished accessibility customization experience than the system Settings panel alone provides.

## Key Topics

### Media Authoring
The standard authoring pipeline — video + audio + manually created subtitle tracks assembled into HLS or file-based containers — remains unchanged. Generated subtitles are overlaid by the playback system at runtime without any authoring-time work.

### Subtitle Generation Methods
Two on-device generation modes:
1. **Speech transcription** — the model listens to the audio track and generates timed subtitle text in the source language.
2. **Language translation** — the model translates an existing subtitle track into another language. Initial support targets English source subtitles with translation into multiple languages.

### Availability and Support
Generated subtitles work for HTTP Live Streaming (VOD), local file-based content, and progressive download. Supported on recent device generations with sufficient on-device model support. English transcription and multi-language translation are the initial supported configurations.

### Presenting Subtitles in Your App
- `AVPlayerViewController` / `AVPlayerView` — built-in subtitle selection UI; no extra work needed.
- Custom player UI with `AVPlayerLayer` — use `AVMediaSelectionGroup` and `AVMediaSelectionOption` to build a subtitle track picker; the system populates generated subtitle options automatically.

### Subtitle Style Preview
Users can customize caption appearance (font, size, color, background) in Settings > Accessibility > Subtitles & Captioning. Apps can surface these styles in-context using:
- `MACaptionAppearanceCopyProfileIDs()` — retrieve the list of available style profile IDs.
- `AVPlayerLayer.setCaptionPreviewProfileID(_:position:text:)` — overlay a preview of the given style on the player layer.
- `AVPlayerLayer.stopShowingCaptionPreview()` — remove the preview.
- `MACaptionAppearanceSetActiveProfileID(_:)` — commit the selected style as the active system style.

## APIs & Frameworks

### AVKit
- `AVPlayerViewController` — automatic generated subtitle support; no API changes needed
- `AVPlayerView` (macOS) — same automatic support

### AVFoundation
- `AVPlayerLayer`
  - `setCaptionPreviewProfileID(_:position:text:)` — **[NEW]** show a caption style preview overlay
  - `stopShowingCaptionPreview()` — **[NEW]** remove the preview
- `AVCaptionRenderer` — used for custom caption rendering
- `AVMediaSelectionGroup` — subtitle/caption track group on an `AVAsset`
- `AVMediaSelectionOption` — individual subtitle option (includes generated options at runtime)
- `AVPlayer` — standard playback engine

### MediaAccessibility
- `MACaptionAppearanceCopyProfileIDs()` — returns `[String]` list of user caption style profile IDs
- `MACaptionAppearanceSetActiveProfileID(_:)` — sets the active caption style profile (takes a `CFString`)

### HTTP Live Streaming
- [What's new in HTTP Live Streaming](https://developer.apple.com/streaming/Whats-new-HLS.pdf) — updated spec covering generated subtitle track signaling in HLS playlists

## Code Highlights

Full subtitle style preview implementation:
```swift
import AVFoundation
import MediaAccessibility

func updateProfileList() {
    subtitleStyleProfileIDs = MACaptionAppearanceCopyProfileIDs() as? [String] ?? []
}

func showPreviewStyle(subtitleStyleProfileID: String) {
    playerLayer.setCaptionPreviewProfileID(subtitleStyleProfileID, position: .zero, text: nil)
}

func stopPreviewStyle() {
    playerLayer.stopShowingCaptionPreview()
}

func setSubtitleStyle(subtitleStyleProfileID: CFString) {
    MACaptionAppearanceSetActiveProfileID(subtitleStyleProfileID)
}
```

## Takeaways
- Generated subtitles require zero code changes for apps using `AVPlayerViewController` or `AVPlayerView` — the system handles generation and display automatically for supported content.
- Generated subtitle options appear in the standard subtitle track picker alongside manual tracks; no special handling is required by the app.
- The new `AVPlayerLayer` caption preview APIs enable apps to provide an in-context style customization experience rather than sending users to Settings.
- Generated subtitles are on-device only — no network request, no server infrastructure, and no privacy concerns around subtitle data leaving the device.

---
_Source: WWDC26 Session 256 page (abstract, chapter summaries, code samples, and resource links)._
