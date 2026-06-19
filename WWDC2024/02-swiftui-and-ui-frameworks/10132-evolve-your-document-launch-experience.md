# Evolve Your Document Launch Experience
**WWDC24 · Session 10132** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10132/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia 15, visionOS 2

## Overview
iOS 18 replaces the plain document browser with a rich launch screen designed for document-centric apps. The new `DocumentGroupLaunchScene` (SwiftUI) and `UIDocumentViewController.launchOptions` (UIKit) let apps add branded backgrounds, foreground accessory views, and a "New Document" button with document-type intent routing. The session builds the Storybook sample app as a running example, showing both frameworks in parallel.

The launch experience is separate from the document browser — apps can still open the browser for file management — but the launch screen is the new first thing users see.

## Key Topics

### DocumentGroupLaunchScene (SwiftUI)
`DocumentGroupLaunchScene` is a new `Scene` type placed alongside `DocumentGroup` in the app's `body`. It accepts a `NewDocumentButton` and an `overlayAccessoryView` closure. The closure receives a `geometry` value with `geometry.titleViewFrame`, enabling apps to place decorative elements relative to the title. The scene handles all document type routing automatically.

### NewDocumentButton
`NewDocumentButton(_:for:action:)` creates a prominent new-document CTA. Pass a `UIDocument.Type` (or protocol-conforming type) and an optional action closure that runs after document creation. Multiple `NewDocumentButton` variants can be stacked for multi-document-type apps.

### Document Creation Intent (UIKit)
`UIDocument.CreationIntent` is a new enum that distinguishes system-initiated document creation from user-initiated. Access the active intent via `browser.activeDocumentCreationIntent` in `UIDocumentPickerDelegate`. Use this to customize the initial document content or template.

### UIDocumentViewController.launchOptions (UIKit)
`UIDocumentViewController` gains a `launchOptions` property with:
- `background.image: UIImage?` — full-bleed background for the launch screen
- `foregroundAccessoryView: UIView?` — decorative view layered above the background, below the system chrome
- These replace custom `UIDocumentBrowserViewController` appearance code

### Async Document Creation
When document creation requires async work (e.g., network fetch for a template), use `CheckedContinuation<StoryDocument?, Error>` inside a `Task` to bridge the completion-handler-based `UIDocument` creation lifecycle into Swift concurrency.

## APIs & Frameworks

**SwiftUI**
- `DocumentGroupLaunchScene` **[NEW]** — new `Scene` type for document app launch screens
  - `init(content:) { NewDocumentButton(...) } overlayAccessoryView: { geometry in ... }` **[NEW]**
- `NewDocumentButton(_:for:action:)` **[NEW]**
- `DocumentLaunchSceneGeometry` **[NEW]**
  - `.titleViewFrame: CGRect` **[NEW]**
- `DocumentGroup` (existing, used alongside `DocumentGroupLaunchScene`)

**UIKit**
- `UIDocumentViewController` (existing)
  - `.launchOptions: UIDocumentViewController.LaunchOptions` **[NEW]**
- `UIDocumentViewController.LaunchOptions` **[NEW]**
  - `.background.image: UIImage?` **[NEW]**
  - `.foregroundAccessoryView: UIView?` **[NEW]**
- `UIDocument.CreationIntent` **[NEW]** enum
  - `.none`, and custom intent values **[NEW]**
- `UIDocumentBrowserViewController` (existing)
  - `browser.activeDocumentCreationIntent: UIDocument.CreationIntent?` **[NEW]**

**Swift Concurrency**
- `CheckedContinuation<T, Error>` (existing, highlighted for async document creation bridge)
- `withCheckedThrowingContinuation { continuation in ... }` (existing)

## Code Highlights

```swift
// SwiftUI launch scene
@main struct StorybookApp: App {
    var body: some Scene {
        DocumentGroup(newDocument: { StoryDocument() }) { config in
            ContentView(document: config.$document)
        }
        DocumentGroupLaunchScene {
            NewDocumentButton("New Story", for: StoryDocument.self) {
                StoryDocument(template: .blank)
            }
        } overlayAccessoryView: { geometry in
            AppLogoView()
                .position(x: geometry.titleViewFrame.midX,
                          y: geometry.titleViewFrame.maxY + 20)
        }
    }
}

// UIKit launch options
class StoryViewController: UIDocumentViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        launchOptions.background.image = UIImage(named: "LaunchBackground")
        launchOptions.foregroundAccessoryView = AppLogoView()
    }
}

// Async document creation
func createDocument() async throws -> StoryDocument {
    return try await withCheckedThrowingContinuation { continuation in
        fetchTemplate { template, error in
            if let error { continuation.resume(throwing: error); return }
            continuation.resume(returning: StoryDocument(template: template!))
        }
    }
}
```

## Takeaways
- Replace `UIDocumentBrowserViewController` customization code with `UIDocumentViewController.launchOptions.background.image` and `.foregroundAccessoryView` — fewer override points, same visual result.
- Use `geometry.titleViewFrame` in `overlayAccessoryView` to position artwork relative to the system title — this prevents clipping across device sizes.
- Use `UIDocument.CreationIntent` to apply different templates (blank vs. from network) when the user taps "New" vs. the system auto-creates a document.
- Bridge async template fetching into document creation with `withCheckedThrowingContinuation` rather than forcing a synchronous first-save cycle.

---
_Source: WWDC24 Session 10132 page (abstract, chapter summaries, code samples, and resource links)._
