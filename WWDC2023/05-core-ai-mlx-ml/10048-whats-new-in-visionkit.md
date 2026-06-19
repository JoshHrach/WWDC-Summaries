# What's new in VisionKit
**WWDC23 · Session 10048** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10048/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14 (native + Catalyst)

## Overview
VisionKit gains three major new capabilities in iOS 17 and macOS Sonoma: Subject Lifting for interactive foreground subject extraction and sticker creation, Visual Look Up for identifying pets, landmarks, art, food, products, and signs, and a comprehensive native macOS API via `ImageAnalysisOverlayView`. Existing apps using `ImageAnalysisInteraction` get Subject Lifting automatically with no code changes.

Additional improvements include optical flow tracking in `DataScannerViewController` for smoother text tracking, a new currency recognition content type, expanded Live Text language support (Thai, Vietnamese), table structure detection, context-aware data detectors, full text selection APIs, and seamless contextual menu integration on macOS.

## Key Topics

**Subject Lifting**
- Long press lifts the foreground subject with animated glow; iOS 17 adds sticker creation with effects (shiny, puffy, etc.)
- Existing apps using `ImageAnalysisInteraction` with `.automatic` type get it for free — no code changes needed
- Analysis is deferred: happens after a few seconds on screen (iOS) or on first menu appearance (macOS) to preserve power
- New interaction types: `.imageSegmentation` (subject lifting only), `.automaticTextOnly` (iOS 16 text-only behavior without subject lifting)
- Subject-related menu items animate the glow preview when highlighted

**Visual Look Up**
- Identifies pets, nature, landmarks, art, media; iOS 17 adds food, products, signs and symbols, laundry symbols
- Two-stage: on-device bounding box + domain detection at analysis time; server lookup only when user requests it
- Add `.visualLookUp` to `ImageAnalyzerConfiguration` to enable
- Invoked automatically alongside Subject Lifting when a single correlated result is detected
- Modal badge mode: set `.visualLookUp` as `preferredInteractionType` to show badges over all results

**DataScannerViewController Updates**
- Optical flow tracking **[NEW]**: smoother, more stable text highlight tracking; free when scanning text without a specific content type and with `highFrameRateTracking` enabled (default)
- Currency content type **[NEW]**: `DataScannerViewController.RecognizedItem.TextContentType.currency` — transcript includes currency symbol and amount

**Live Text Updates**
- New languages: Thai, Vietnamese
- Table structure detection **[NEW]**: recognized tables can be copied/pasted preserving structure into Notes, Numbers
- Context-aware data detectors: when adding a contact from an email, surrounding data detector information (name, phone) is included
- New text selection APIs: `transcript` (plain text), attributed text, selected ranges, selected text properties **[NEW]**
- New delegate method: text selection change notification **[NEW]**
- Menu builder API: insert custom menu items alongside system items (copy, share)

**macOS Support**
- Catalyst: recompile to get Live Text, Subject Lifting, Visual Look Up (machine-readable codes are a no-op on macOS)
- Native macOS: `ImageAnalysisOverlayView` (NSView subclass) replaces `ImageAnalysisInteraction` (UIInteraction)
- `contentsRect` in unit coordinate space describes image placement within overlay bounds
- `trackingImageView` property auto-calculates `contentsRect` when using `NSImageView`
- `setContentsRectNeedsUpdate()` for manual refresh
- `delegate` method `contentsRect(for:)` for custom image views
- Contextual menu integration: `overlayView(_:updatedMenu:forEvent:atPoint:)` delegate method returns combined app+VisionKit menu
- `ImageAnalysisOverlayView.MenuTag` struct: named tags for VisionKit menu items (`copySubject`, `shareSubject`, `copyImage`, `lookUp`, `recommendedAppItems`)
- VisionKit forwards `NSMenuDelegate` callbacks via its own overlay delegate

## APIs & Frameworks

**VisionKit**
- `ImageAnalysisInteraction` — existing class, now supports Subject Lifting automatically
- `ImageAnalysisInteraction.InteractionType` updates:
  - `.imageSegmentation` **[NEW]** — subject lifting only
  - `.automaticTextOnly` **[NEW]** — text/data detectors without subject lifting
  - `.automatic` — now includes subject lifting (existing, updated behavior)
- `ImageAnalyzerConfiguration.AnalysisType`:
  - `.visualLookUp` **[NEW]**
- `ImageAnalysisInteraction.preferredInteractionType` — set to `.visualLookUp` for badge mode
- `DataScannerViewController.RecognizedItem.TextContentType.currency` **[NEW]**
- `DataScannerViewController` — optical flow tracking enabled by default when scanning text **[NEW behavior]**
- `ImageAnalysis.transcript` — existing; now also exposes attributed text, selected ranges, selected text **[NEW properties]**
- `ImageAnalysisInteractionDelegate` — new method for text selection changes **[NEW]**
- `ImageAnalysisOverlayView` **[NEW]** (macOS) — NSView subclass
  - `contentsRect: CGRect` **[NEW]**
  - `trackingImageView: NSImageView?` **[NEW]**
  - `setContentsRectNeedsUpdate()` **[NEW]**
  - `delegate: ImageAnalysisOverlayViewDelegate?`
- `ImageAnalysisOverlayViewDelegate` **[NEW]** (macOS)
  - `contentsRect(for:)` **[NEW]**
  - `overlayView(_:updatedMenu:forEvent:atPoint:)` **[NEW]**
- `ImageAnalysisOverlayView.MenuTag` **[NEW]** (macOS) — struct with tags: `copySubject`, `shareSubject`, `copyImage`, `lookUp`, `recommendedAppItems`
- `ImageAnalyzer` — same API on iOS and macOS

## Code Highlights

Currency scanning:
```swift
// In DataScannerViewController recognizedItems stream
for await item in dataScanner.recognizedItems {
    for recognized in item {
        if case .text(let text) = recognized {
            let transcript = text.transcript
            if transcript.contains(currencySymbol) {
                total += parseAmount(transcript)
            }
        }
    }
}
```

Adding a custom menu item to the Live Text menu (iOS):
```swift
// In UIMenuBuilder
let selectedText = interaction.selectedText
guard !selectedText.isEmpty else { return }
let command = UICommand(title: "Create Reminder", action: #selector(createReminder))
let menu = UIMenu(children: [command])
builder.insertSibling(menu, afterMenu: .share)
```

macOS contextual menu integration:
```swift
func overlayView(_ overlayView: ImageAnalysisOverlayView,
                 updatedMenu menu: NSMenu,
                 forEvent event: NSEvent,
                 atPoint point: CGPoint) -> NSMenu {
    // Add VisionKit's copySubject item to your app's menu
    let appMenu = buildAppMenu(for: event)
    if let copySubjectItem = menu.item(withTag: ImageAnalysisOverlayView.MenuTag.copySubject) {
        appMenu.addItem(copySubjectItem)
    }
    return appMenu
}
```

## Takeaways
- Existing `ImageAnalysisInteraction` apps with `.automatic` type get Subject Lifting for free in iOS 17 — test and ship.
- Add `.visualLookUp` to your analyzer configuration to enable the Look Up feature; use `preferredInteractionType` for the full badge-browsing experience.
- Use the new `currency` content type in `DataScannerViewController` to build receipt scanning, price capture, and expense tracking features.
- On macOS Sonoma, use `ImageAnalysisOverlayView` and the `updatedMenu` delegate method to integrate VisionKit content naturally into your app's right-click menus.

---
_Source: WWDC23 Session 10048 page (abstract, chapter summaries, code samples, and resource links)._
