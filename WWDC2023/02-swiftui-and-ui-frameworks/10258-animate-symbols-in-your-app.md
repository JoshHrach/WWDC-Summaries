# Animate Symbols in Your App
**WWDC23 · Session 10258** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10258/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17, watchOS 10, visionOS 1

## Overview
iOS 17 and macOS Sonoma introduce animated SF Symbols through a new **Symbols framework** with a unified API for creating and configuring symbol effects across SwiftUI, UIKit, and AppKit. This session covers all seven new built-in symbol animations — Bounce, Pulse, Variable Color, Scale, Appear, Disappear, and Replace — along with the conceptual model of four animation behaviors (discrete, indefinite, transition, content transition) that determine which APIs can be used with each effect.

The Symbols framework uses a dot-chained Swift API style with no strings: effects like `.bounce`, `.variableColor.iterative.reversing`, and `.replace.offUp` are real Swift code with full Xcode autocomplete and compile-time validation. Effects can be applied and combined on any symbol image, including custom symbols, making the system universally applicable.

The session also covers special topics including effect propagation in SwiftUI view hierarchies, applying effects without animation, and the automatic cross-fade behavior when changing variable value symbols in iOS 17.

## Key Topics

### Symbol Effect Behaviors
- **Discrete** — one-shot animation triggered by an event (Bounce, Pulse discrete, Variable Color discrete)
- **Indefinite** — changes appearance and holds until explicitly removed (Scale, Variable Color, Appear, Disappear)
- **Transition** — animates insertion/removal from the view hierarchy (Appear, Disappear)
- **Content Transition** — animates between two different symbol images (Replace)

### SwiftUI APIs
- `.symbolEffect(_:)` — applies indefinite or discrete effects; takes an optional `isActive` boolean for indefinite effects and a `value` parameter for discrete effects
- `.contentTransition(.symbolEffect(.replace))` — for Replace content transitions
- `.transition(.symbolEffect(.disappear))` — for hierarchy-changing transition effects
- `.symbolEffectsRemoved()` — prevents a view from inheriting symbol effects from parent views
- `withTransaction` with `disablesAnimations: true` — applies effects without animation on initial appearance

### UIKit / AppKit APIs
- `UIImageView.addSymbolEffect(_:)` — adds indefinite or discrete effects
- `UIImageView.addSymbolEffect(_:options:)` — with repeat count options
- `UIImageView.addSymbolEffect(_:animated:)` — applies effect without animation when `animated: false`
- `UIImageView.addSymbolEffect(_:completion:)` — completion handler after effect finishes (useful for removing views from hierarchy)
- `UIImageView.removeSymbolEffect(ofType:)` — removes indefinite effect
- `UIImageView.setSymbolImage(_:contentTransition:)` — Replace content transition and automatic variable value crossfade
- `UIBarButtonItem` — same `addSymbolEffect` / `removeSymbolEffect` methods available
- `UIControl.isSymbolAnimationEnabled` / `UIBarButtonItem.isSymbolAnimationEnabled` — controls built-in symbol animations **[NEW]**
- Built-in `UISlider` symbol animations (bounce at track ends) in iOS 17 **[NEW]**

### Variable Value
- Automatic cross-fade between variable values in SwiftUI when state changes (`Image(systemName:variableValue:)`)
- `UIImageView.setSymbolImage(_:contentTransition: .automatic)` — detects variable value change and crossfades in UIKit

## APIs & Frameworks

### Symbols Framework **[NEW]**
- `Symbols` — new framework; included automatically with SwiftUI, UIKit, AppKit
- `SymbolEffect` protocol — base protocol for all effects
- `DiscreteSymbolEffect` protocol — effects that play one-off animations
- `IndefiniteSymbolEffect` protocol — effects that hold state until removed
- `TransitionSymbolEffect` protocol — effects that animate view hierarchy insertion/removal
- `ContentTransitionSymbolEffect` protocol — effects that animate between symbol images

### Effect Types (all **[NEW]**)
- `.bounce` — discrete; animates symbol up or down
- `.bounce.up` / `.bounce.down` — directional configuration
- `.pulse` — discrete or indefinite
- `.variableColor` — discrete or indefinite
- `.variableColor.iterative` — iterative fill mode
- `.variableColor.reversing` — reverses after filling
- `.variableColor.iterative.reversing` — combined configuration
- `.variableColor.cumulative` — cumulative fill mode
- `.scale` — indefinite
- `.scale.up` / `.scale.down` — direction configuration
- `.appear` — indefinite or transition
- `.appear.up` / `.appear.down` — directional
- `.disappear` — indefinite or transition
- `.disappear.down`
- `.replace` — content transition
- `.replace.offUp` / `.replace.downUp` — directional configuration

### SwiftUI
- `.symbolEffect(_:)` modifier — indefinite/discrete **[NEW]**
- `.symbolEffect(_:isActive:)` — conditional indefinite effect **[NEW]**
- `.symbolEffect(_:options:value:)` — discrete with repeat count and trigger value **[NEW]**
- `.contentTransition(.symbolEffect(_:))` — Replace content transition **[NEW]**
- `.transition(.symbolEffect(_:))` — transition behavior **[NEW]**
- `.symbolEffectsRemoved()` — stops effect inheritance **[NEW]**
- `Image(systemName:variableValue:)` — automatic variable value animation when value changes **[NEW]**
- `SymbolEffectOptions.repeat(_:)` — repeat count option **[NEW]**

### UIKit / AppKit
- `UIImageView.addSymbolEffect(_:)` **[NEW]**
- `UIImageView.addSymbolEffect(_:options:)` **[NEW]**
- `UIImageView.addSymbolEffect(_:animated:)` **[NEW]**
- `UIImageView.addSymbolEffect(_:completion:)` **[NEW]**
- `UIImageView.removeSymbolEffect(ofType:)` **[NEW]**
- `UIImageView.setSymbolImage(_:contentTransition:)` **[NEW]**
- `NSImageView.addSymbolEffect(_:)` **[NEW]**
- `NSImageView.removeSymbolEffect(ofType:)` **[NEW]**
- `UIBarButtonItem.addSymbolEffect(_:)` **[NEW]**
- `UIControl.isSymbolAnimationEnabled` **[NEW]**
- `UIBarButtonItem.isSymbolAnimationEnabled` **[NEW]**
- `SymbolEffectCompletion` / completion handler context — `context.sender`, `context.isFinished`

## Code Highlights

Combining indefinite effects in SwiftUI:
```swift
Image(systemName: "wifi.router")
    .symbolEffect(.variableColor.iterative.reversing)
    .symbolEffect(.scale.up)
```

Discrete bounce triggered by state change:
```swift
Image(systemName: "antenna.radiowaves.left.and.right")
    .symbolEffect(.bounce, options: .repeat(2), value: bounceValue)
```

Replace content transition:
```swift
Image(systemName: isPaused ? "pause.fill" : "play.fill")
    .contentTransition(.symbolEffect(.replace.offUp))
```

UIKit Disappear with completion to remove from hierarchy:
```swift
imageView.addSymbolEffect(.disappear) { context in
    if let imageView = context.sender as? UIImageView, context.isFinished {
        imageView.removeFromSuperview()
    }
}
```

## Takeaways
- The new **Symbols framework** unifies symbol animation across SwiftUI, UIKit, and AppKit with a type-safe dot-chain API — no strings, full autocomplete.
- Understanding the four behaviors (discrete, indefinite, transition, content transition) is the key to picking the right API for each use case.
- `UIControl` and `UIBarButtonItem` gain built-in symbol animation support and an `isSymbolAnimationEnabled` toggle.
- SwiftUI automatically animates `variableValue` changes in iOS 17; UIKit needs `setSymbolImage(_:contentTransition: .automatic)`.

---
_Source: WWDC23 Session 10258 page (abstract, chapter summaries, code samples, and resource links)._
