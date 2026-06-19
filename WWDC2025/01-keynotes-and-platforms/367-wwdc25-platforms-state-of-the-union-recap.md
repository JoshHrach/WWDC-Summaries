# WWDC25 Platforms State of the Union Recap
**WWDC25 · Session 367** · [Watch](https://developer.apple.com/videos/play/wwdc2025/367/)

_Platforms:_ iOS 26, iPadOS 26, macOS Tahoe 26, tvOS 26, visionOS 26, watchOS 26

## Overview
This short recap session distills the highlights of the WWDC25 Platforms State of the Union into a fast-paced overview for developers who want a high-level orientation before diving into individual sessions. It covers the new design language (Liquid Glass), Apple Intelligence features for developers, Xcode 26, Swift 6.2, SwiftUI updates, Metal 4, and a rapid-fire lightning round of additional platform improvements.

It does not contain chapter summaries or code samples — it is a curated highlight reel intended to be watched before the deeper technical sessions.

## Key Topics

### Liquid Glass design system
A new dynamic material called **Liquid Glass** underpins the visual redesign across all Apple platforms. It combines optical qualities of glass with a sense of fluidity, adding depth and vitality to UI elements. A new tool, **Icon Composer**, lets developers create layered, translucent app icons with blurring, translucency control, specular highlights, and tint preview.

### Apple Intelligence and Foundation Models
The **Foundation Models framework** is introduced — a new API for on-device inference using Apple's on-device model, specialized for everyday tasks such as text extraction, summarization, and structured outputs. All processing stays on-device, no server required, at no cost to developers or users.

### Xcode 26 and generative coding models
Xcode 26 integrates large language model support. Built-in support for ChatGPT. New **coding tools** in the source editor (analogous to writing tools) offer suggested actions: generating previews, playgrounds, and fixing issues. Developers can also ask for specific code changes inline.

### Swift 6.2
Focus areas: performance, concurrency, and interoperability. Key change: modules or individual files can be configured to run on the **main actor by default**, eliminating many `@MainActor` annotations needed in Swift 6.1 and reducing friction when writing single-threaded code.

### SwiftUI
New **web APIs**, **3D chart support**, performance improvements, and **rich text editing**. Rich text editing: change a `TextEditor`'s text binding from `String` to `AttributedString` to gain a fully styled editor.

### Metal 4 and games
Metal 4 is designed for Apple Silicon with new features including **Metal FX frame interpolation** and **denoising APIs**. **Game Porting Toolkit 3** adds more optimization tools and support for Windows upscaling technologies. **PlayStation VR2 Sense controller** support enables new input on Apple Vision Pro.

### Lightning round highlights
- Terminal: 24-bit color, new themes, Powerline fonts
- visionOS: 180°, 360°, wide-field-of-view media and Apple Immersive Video support via Apple Projected Media Profile
- CarPlay: Live Activities for timely in-car updates
- New **declared age range API** for appropriate content experiences
- Accessibility nutrition labels
- Updates to Family Sharing and Assistive Access

## APIs & Frameworks

### Foundation Models
- **`Foundation Models`** framework **[NEW]** — on-device language model API for text extraction, summarization, structured output, and more

### Xcode 26
- Integrated LLM coding support **[NEW]** — ChatGPT built in
- **Coding tools** in source editor **[NEW]** — inline code generation, preview generation, fix suggestions

### Swift 6.2
- **Main actor by default** module/file configuration **[NEW]** — eliminates repeated `@MainActor` annotations
- Concurrency and interoperability improvements

### SwiftUI
- Rich text editing via `AttributedString` binding on `TextEditor` **[NEW]**
- New web APIs **[NEW]**
- 3D chart support **[NEW]**

### Metal 4
- **Metal FX frame interpolation** **[NEW]**
- **Metal FX denoising** **[NEW]**
- Designed for Apple Silicon

### Game Porting Toolkit 3
- Additional optimization tooling **[NEW]**
- Windows upscaling technology support **[NEW]**

### Game Controller / visionOS
- **PlayStation VR2 Sense controller** support on Apple Vision Pro **[NEW]**

### Media (visionOS)
- **Apple Projected Media Profile** — 180°, 360°, wide-FOV immersive video **[NEW]**

### CarPlay
- Live Activities in CarPlay **[NEW]**

### System
- Declared age range API **[NEW]**
- Accessibility nutrition labels **[NEW]**

## Code Highlights
No code samples in this session — see individual WWDC25 sessions for implementation details.

## Takeaways
- The Liquid Glass design system is automatic for apps linked against the iOS 26/macOS Tahoe SDKs; audit for layout and color issues, and update app icons with Icon Composer.
- The Foundation Models framework is the key new on-device AI API — start with the "Meet the Foundation Models framework" session to understand capabilities and safety guardrails.
- Swift 6.2's main-actor-default module setting is the most impactful concurrency ergonomics improvement since Swift 5; consider enabling it for new modules or feature files.
- Metal 4 + Game Porting Toolkit 3 together lower the bar for porting PC/console titles to Apple platforms; frame interpolation and denoising can directly improve frame rate and visual quality.

---
_Source: WWDC25 Session 367 page (abstract and transcript)._
