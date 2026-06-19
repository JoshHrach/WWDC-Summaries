# Mastering Xcode Previews
**WWDC19 · Session 233** · [Watch](https://developer.apple.com/videos/play/wwdc2019/233/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, watchOS 6, tvOS 13

## Overview
Xcode 11 introduces live previews directly inside the source editor, transforming the edit-build-run cycle into a continuous feedback loop. Previews render SwiftUI views (and UIKit/AppKit views wrapped with representable protocols) without requiring a full simulator launch. Changes to source code are reflected almost instantly in the canvas alongside the editor.

The session covers how the preview system works — a separate compilation step that builds a special preview executable — and how to author rich, multi-device, multi-configuration preview declarations using `PreviewProvider`. It also explains how to preview `UIViewController` and `UIView` subclasses by wrapping them with `UIViewControllerRepresentable` and `UIViewRepresentable`, and how to use environment modifiers to test Dark Mode, Dynamic Type sizes, locales, and more without changing simulator settings.

Practical techniques include grouping multiple previews, pinning a preview while editing other files, using `PreviewDevice` to render on a specific simulated device, and creating preview-friendly factory methods to inject mock data without altering production code paths.

## Key Topics
- **Xcode Previews canvas** — live rendering panel beside the source editor; automatic refresh on code change; Pin Preview to keep a preview active while navigating other files
- **`PreviewProvider` protocol** — defines one or more previews returned from a static `previews` property; Xcode discovers all types conforming to this protocol in the target
- **Wrapping UIKit views** — `UIViewRepresentable` and `UIViewControllerRepresentable` adapters let UIKit components appear in SwiftUI previews
- **Environment overrides** — `.environment(\.colorScheme, .dark)`, `.environment(\.locale, Locale(identifier:))`, `.environment(\.dynamicTypeSize, .xxxLarge)` and others let you test multiple configurations in a single file
- **Multi-device previews** — `Group { ... }` containing views modified with `.previewDevice(PreviewDevice(rawValue:))` renders multiple device targets simultaneously
- **Preview layouts** — `.previewLayout(.sizeThatFits)` and `.previewLayout(.fixed(width:height:))` resize the preview canvas for component-level testing
- **Preview display name** — `.previewDisplayName("...")` labels each preview in the canvas for clarity
- **Mock data and preview helpers** — create static factory methods or dedicated preview-only extensions to supply realistic sample data without polluting production code

## APIs & Frameworks
- **SwiftUI**
  - `PreviewProvider` protocol **[NEW]**
    - `static var previews: some View { }` — required computed property
  - `#if DEBUG` / preview-conditional compilation
  - `.previewDevice(_:)` modifier **[NEW]** — specify simulator device by name
  - `PreviewDevice` struct **[NEW]** — wraps device name string
  - `.previewLayout(_:)` modifier **[NEW]**
    - `PreviewLayout.device` (default)
    - `PreviewLayout.sizeThatFits`
    - `PreviewLayout.fixed(width:height:)`
  - `.previewDisplayName(_:)` modifier **[NEW]**
  - `.environment(_:_:)` modifier — inject environment values for preview configuration
  - `EnvironmentValues` keys used in previews:
    - `\.colorScheme` — `.light` / `.dark`
    - `\.locale`
    - `\.dynamicTypeSize` (or `\.sizeCategory` in iOS 13)
    - `\.layoutDirection` — `.leftToRight` / `.rightToLeft`
  - `Group { }` — render multiple previews in the canvas simultaneously
- **UIKit integration**
  - `UIViewRepresentable` protocol — wrap `UIView` for SwiftUI previews **[NEW]**
    - `makeUIView(context:) -> UIViewType`
    - `updateUIView(_:context:)`
  - `UIViewControllerRepresentable` protocol — wrap `UIViewController` **[NEW]**
    - `makeUIViewController(context:) -> UIViewControllerType`
    - `updateUIViewController(_:context:)`
- **AppKit integration**
  - `NSViewRepresentable` — wrap `NSView` for macOS previews **[NEW]**
  - `NSViewControllerRepresentable` — wrap `NSViewController` **[NEW]**
- **Xcode 11**
  - Xcode canvas (preview panel) — live preview alongside editor
  - Pin Preview — keeps a specific file's preview visible while editing other files
  - Preview pause / resume
  - Resume shortcut (Option+Command+P)
  - Debug Preview — attach debugger to the preview process

## Code Highlights

```swift
// Basic SwiftUI PreviewProvider
struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```

```swift
// Multiple device previews
struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        Group {
            ContentView()
                .previewDevice(PreviewDevice(rawValue: "iPhone SE (2nd generation)"))
                .previewDisplayName("iPhone SE")
            ContentView()
                .previewDevice(PreviewDevice(rawValue: "iPhone 11 Pro Max"))
                .previewDisplayName("iPhone 11 Pro Max")
        }
    }
}
```

```swift
// Dark mode and large text previews
struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        Group {
            ContentView()
                .environment(\.colorScheme, .dark)
                .previewDisplayName("Dark Mode")
            ContentView()
                .environment(\.sizeCategory, .accessibilityExtraExtraExtraLarge)
                .previewDisplayName("AX3 Text")
        }
    }
}
```

```swift
// Wrapping a UIViewController for preview
struct MyViewController_Preview: PreviewProvider {
    static var previews: some View {
        MyViewControllerRepresentable()
            .previewLayout(.sizeThatFits)
    }
}

struct MyViewControllerRepresentable: UIViewControllerRepresentable {
    func makeUIViewController(context: Context) -> MyViewController {
        MyViewController()
    }
    func updateUIViewController(_ vc: MyViewController, context: Context) {}
}
```

## Takeaways
- Xcode Previews eliminate the need to launch the simulator for most UI iterations — previews render in seconds directly in the editor canvas.
- `PreviewProvider` can host any number of views in a `Group`, enabling side-by-side comparison across devices, color schemes, and text sizes in one file.
- UIKit and AppKit views and view controllers participate in previews via their respective Representable protocols with minimal wrapper code.
- Supply mock data through static factory methods or preview-specific extensions to avoid polluting production initializers.

---
_Source: WWDC19 Session 233 page (abstract, chapter summaries, code samples, and resource links)._
