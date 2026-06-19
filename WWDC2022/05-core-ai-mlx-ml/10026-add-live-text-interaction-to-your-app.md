# Add Live Text Interaction to Your App
**WWDC22 · Session 10026** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10026/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13

## Overview
Live Text analyzes images to extract text and data detectors, allowing users to select and copy text, activate data detectors (maps, phone numbers, URLs), translate, look up, and interact with QR codes — all from still images or paused video frames. This session introduces the new VisionKit API that lets any app integrate these capabilities with minimal code.

The API centers on four classes: `ImageAnalyzer` performs the async image analysis using the Apple Neural Engine, `ImageAnalysis` carries the results, and either `ImageAnalysisInteraction` (iOS/iPadOS) or `ImageAnalysisOverlayView` (macOS) bridges the analysis to the UI. On iOS 16 devices with an Apple Neural Engine, and all devices running macOS Ventura, this feature is available.

The session also covers customizing the supplementary interface (the Live Text button and Quick Actions), resolving gesture conflicts, optimizing performance, and leveraging automatic Live Text support already built into AVKit, WebKit, and Quick Look.

## Key Topics

### Core Live Text API
- Create one shared `ImageAnalyzer` per app
- Configure analysis with `ImageAnalyzer.Configuration` specifying `.text` and/or `.machineReadableCode`
- Call `analyzer.analyze(_:configuration:)` async; handle errors
- Set `interaction.analysis` and `interaction.preferredInteractionTypes` once analysis completes and the image hasn't changed

### Interaction Types
- `.automatic` — text selection + data detectors (recommended; matches system behavior)
- `.textSelection` — text selection only, no data detectors; state doesn't change with Live Text button
- `.dataDetectors` — data detectors only, no text selection; Live Text button hidden
- Empty set — disables interaction entirely
- `allowLongPressForDataDetectorsInTextMode` — controls long-press activation of data detectors in text mode (default `true`)

### Supplementary Interface Customization
- `isSupplementaryInterfaceHidden` — hide the Live Text button and Quick Actions
- `supplementaryInterfaceContentInsets` — adjust insets to avoid overlapping app chrome
- `supplementaryInterfaceFont` — adopt custom app font/weight for the Live Text button and Quick Actions (point size ignored for sizing consistency)

### Gesture Conflict Resolution
- Delegate method `interactionShouldBeginAtPoint(_:for:)` — return `false` to suppress interaction at a point
- Check `interaction.hasInteractiveItem(atPoint:)` and `interaction.hasActiveTextSelection` before returning
- `gestureRecognizerShouldBegin` — use same checks to prevent app gesture recognizers from stealing events
- `hitTest(_:with:)` override for hit-testing customization

### ContentsRect / Non-UIImageView Usage
- Implement `contentsRectForInteraction(_:)` delegate method to return a unit-coordinate CGRect describing how image content maps to the interaction view's bounds
- Call `setContentsRectNeedsUpdate()` when contentsRect changes without a bounds change
- `UIImageView` calculates `contentsRect` automatically based on its `contentMode`

### AVKit Integration (New in iOS 16)
- `AVPlayerView` and `AVPlayerViewController` have `allowsVideoFrameAnalysis` property (default `true`) for automatic Live Text on paused frames
- For `AVPlayerLayer`: use `currentlyDisplayedPixelBuffer` to get the current frame (only valid when `rate == 0`, shallow copy, not write-safe)
- Only available for non-FairPlay protected content

## APIs & Frameworks

**VisionKit** **[NEW]**
- `ImageAnalyzer` **[NEW]** — performs async image analysis using Neural Engine
  - `ImageAnalyzer.Configuration` **[NEW]** — init with `[InteractionTypes]` (`.text`, `.machineReadableCode`)
  - `analyze(_:configuration:)` async throws → `ImageAnalysis` **[NEW]**
  - Supports `UIImage`, `CGImage`, `CIImage`, `CVPixelBuffer`, `URL` (`CVPixelBuffer` most efficient)
- `ImageAnalysis` **[NEW]** — result object; set on interaction
- `ImageAnalysisInteraction` (iOS/iPadOS) **[NEW]** — `UIInteraction` subclass
  - `.analysis: ImageAnalysis?`
  - `.preferredInteractionTypes: InteractionTypes` — `.automatic`, `.textSelection`, `.dataDetectors`, `[]`
  - `.isSupplementaryInterfaceHidden: Bool`
  - `.supplementaryInterfaceContentInsets: NSDirectionalEdgeInsets`
  - `.supplementaryInterfaceFont: UIFont?`
  - `.allowLongPressForDataDetectorsInTextMode: Bool`
  - `.hasInteractiveItem(atPoint:) -> Bool`
  - `.hasActiveTextSelection: Bool`
  - `.setContentsRectNeedsUpdate()`
  - `ImageAnalysisInteractionDelegate` protocol — `contentsRectForInteraction(_:)`, `interactionShouldBeginAtPoint(_:for:)`
- `ImageAnalysisOverlayView` (macOS) **[NEW]** — `NSView` overlay equivalent

**AVKit (updates)**
- `AVPlayerView.allowsVideoFrameAnalysis: Bool` **[NEW]** (macOS)
- `AVPlayerViewController.allowsVideoFrameAnalysis: Bool` **[NEW]** (iOS/iPadOS)
- `AVPlayerLayer.currentlyDisplayedPixelBuffer: CVPixelBuffer?` **[NEW]**

**Automatic Live Text support (no adoption needed)**
- `UITextField` / `UITextView` — camera keyboard input
- `WKWebView` (WebKit)
- `QLPreviewController` (Quick Look)

## Code Highlights

Minimal Live Text adoption for a UIImageView:
```swift
import VisionKit

let analyzer = ImageAnalyzer()
let interaction = ImageAnalysisInteraction()

override func viewDidLoad() {
    super.viewDidLoad()
    imageView.addInteraction(interaction)
}

func analyzeCurrentImage() {
    guard let image = image else { return }
    Task {
        let config = ImageAnalyzer.Configuration([.text, .machineReadableCode])
        do {
            let analysis = try await analyzer.analyze(image, configuration: config)
            if image == self.image {
                interaction.analysis = analysis
                interaction.preferredInteractionTypes = .automatic
            }
        } catch { /* handle */ }
    }
}
```

## Takeaways
- Live Text integration requires as few as ~15 lines of code when using `UIImageView`; VisionKit handles `contentsRect`, supplementary interface positioning, and gesture management automatically.
- Use one shared `ImageAnalyzer` per app; pass the native image type to avoid conversions; begin analysis only when the image is on-screen.
- For scrolling content (timelines, feeds), defer analysis until scrolling stops to optimize system resource usage.
- AVKit's `allowsVideoFrameAnalysis` enables Live Text on paused video frames out of the box — no additional integration needed for standard player views.

---
_Source: WWDC22 Session 10026 page (abstract, chapter summaries, code samples, and resource links)._
