# Platforms State of the Union 5-Minute Recap
**WWDC24 · Session 111977** · [Watch](https://developer.apple.com/videos/play/wwdc2024/111977/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia 15, visionOS 2, watchOS 11

## Overview
This short recap condenses the full Platforms State of the Union into a five-minute highlight reel, touching on the most consequential announcements for developers across all Apple platforms. It serves as a quick orientation to the breadth of WWDC24 — from Apple Intelligence to Swift's 10th anniversary, Xcode 16, and the widening reach of visionOS.

The session emphasizes that WWDC24 marks the beginning of a new chapter centered on Apple Intelligence — a personal intelligence system bringing generative models to iOS, iPadOS, and macOS. It also highlights the maturation of Swift, SwiftUI, and the developer toolchain, and teases significant gaming and spatial computing advances.

## Key Topics

**Apple Intelligence**
Writing Tools become available automatically in any app using standard text UI frameworks. New APIs allow per-app customization of Writing Tools behavior. Genmoji and Image Playground APIs deliver generative image experiences. A new, larger cloud-based model powers Swift Assist in Xcode.

**Swift & SwiftUI**
Swift celebrates its 10th birthday. C and C++ interoperability are highlighted as making Swift the best successor to C++. SwiftUI gains new previews architecture (dynamic linking using the same build artifacts as build-and-run), new customization APIs, and improved cross-framework foundation sharing.

**Xcode 16**
On-device code completion powered by a new engine trained on project symbols. Swift Assist cloud model enables natural-language coding requests. New single-view backtrace, Instruments flame graph view, and enhanced localization catalogs.

**Controls & Widgets**
New Controls API lets apps expose toggles, actions, and deep links to the Lock Screen and Action button. App icons and widgets now support Light, Dark, and Tinted appearances.

**Gaming**
Game Porting Toolkit 2 adds AVX2 support and ray tracing compatibility, enabling more Windows games to be evaluated on Apple silicon. Every Apple silicon Mac, M-series iPad, and iPhone 15 Pro can now run console-quality games.

**visionOS & Spatial Computing**
Community highlighted as rapidly building spatial apps. djay cited as example of SwiftUI iPad app extended to visionOS with minimal effort.

## APIs & Frameworks
- **Writing Tools** — automatic adoption in apps using standard text views; `UITextViewDelegate` API for customization **[NEW]**
- **Image Playground API** — new, consistent generative image creation **[NEW]**
- **Genmoji** — new expressive emoji generation capability **[NEW]**
- **Controls API** — Lock Screen and Action button controls **[NEW]**
- **SwiftUI** — dynamic linking previews architecture, new customizations, interoperability improvements **[NEW]**
- **Xcode 16** — on-device ML code completion engine, Swift Assist cloud model **[NEW]**
- **Game Porting Toolkit 2** — AVX2, ray tracing, improved compatibility **[NEW]**
- **Metal** — Apple silicon gaming pipeline; iPhone 15 Pro, M-series iPad support

## Code Highlights
No code samples are included in this recap session — it is a high-level overview.

## Takeaways
- Adopt standard UIKit/AppKit text views to get Writing Tools for free in iOS 18.
- Use the new Controls API to surface app actions on the Lock Screen and Action button.
- Explore Game Porting Toolkit 2 to evaluate existing Windows/console-quality titles on Apple silicon.
- Watch the full Platforms State of the Union (Session 102) for complete technical depth.

---
_Source: WWDC24 Session 111977 page (abstract, transcript summary, and resource links)._
