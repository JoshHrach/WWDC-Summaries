# VoiceOver Efficiency with Custom Rotors
**WWDC20 · Session 10116** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10116/)

_Platforms:_ iOS 14, iPadOS 14

## Overview
Custom rotors extend VoiceOver's built-in rotor mechanism to filter and group interface elements in ways that parallel how sighted users visually parse the screen. The VoiceOver rotor (activated by twisting two fingers) lets users select a navigation mode; a swipe down or up then jumps to the next or previous item in that mode. Custom rotors add developer-defined categories to this list—turning what would otherwise be a sequential swipe-through-everything experience into a purposeful scan of related elements.

The session uses two concrete examples. In a map app showing Apple Stores and parks, custom rotors sort each category by distance from the user, replicating the visual affordance of color-coded markers. A separate brochure text view uses a custom text rotor to jump between lines marked with a custom `NSAttributedString.Key` alert attribute, letting users hear only critical alerts without reading every line. The key insight is that both examples share the same closure-based `UIAccessibilityCustomRotor` API; the difference lies in whether `UIAccessibilityCustomRotorItemResult` wraps an element reference (map) or a `UITextRange` (text).

## Key Topics
- **`UIAccessibilityCustomRotor`** — declares a named rotor; initialized with a localized name and a predicate closure
- **Predicate closure** — receives `UIAccessibilityCustomRotorSearchPredicate` with `currentItem` and `searchDirection`; must return `UIAccessibilityCustomRotorItemResult?` or `nil` at boundaries
- **`searchDirection`** — `.previous` (swipe up) / `.next` (swipe down); drives index increment/decrement or text search direction
- **`UIAccessibilityCustomRotorItemResult`** — wraps `targetElement` (for view-based rotors) or `targetRange: UITextRange?` (for text rotors)
- **`accessibilityCustomRotors` property** — set on the host view; VoiceOver discovers rotors from this array when the view receives focus
- **Map rotor pattern** — filter `MKAnnotationView` items by POI type; sort by distance; return annotation view as `targetElement`
- **Text rotor pattern** — enumerate `NSAttributedString` for a custom attribute; return matched range as a `UITextRange` via `UITextInput`; `targetElement` must conform to `UITextInput`

## APIs & Frameworks

**UIKit — Accessibility (VoiceOver Custom Rotors)**
- `UIAccessibilityCustomRotor` — custom rotor type
  - `init(name: String, itemSearchBlock: (UIAccessibilityCustomRotorSearchPredicate) -> UIAccessibilityCustomRotorItemResult?)` — primary initializer
  - `init(attributedName: NSAttributedString, itemSearchBlock:)` — variant for rich rotor name text
- `UIAccessibilityCustomRotorSearchPredicate` — closure argument
  - `currentItem: UIAccessibilityCustomRotorItemResult` — the currently focused element/range
  - `searchDirection: UIAccessibilityCustomRotor.Direction` — `.previous` or `.next`
- `UIAccessibilityCustomRotorItemResult` — result returned from the closure
  - `init(targetElement: NSObjectProtocol?, targetRange: UITextRange?)` — view-based result (pass `nil` for `targetRange`)
  - `init(targetElement:targetRange:)` — text-based result (pass text-input view as `targetElement`, non-nil `UITextRange` for `targetRange`)
  - Return `nil` from the block to signal boundary (first/last item)
- `UIAccessibilityElement.accessibilityCustomRotors: [UIAccessibilityCustomRotor]?` — property on `UIView`, `UIAccessibilityElement`, etc.; assign to register rotors
- `UIView.accessibilityCustomRotors` — same property available on `UIView`

**UIKit — Text Input (for text rotors)**
- `UITextInput` protocol — `targetElement` must conform; provides `textRange(from:to:)` and `beginningOfDocument`
- `UITextRange` — opaque range type within a `UITextInput`; required for text `targetRange`

**MapKit (referenced in demo)**
- `MKAnnotationView` — used as `targetElement` in the map rotor result
- `mapView.accessibilityCustomRotors` — assigned an array of custom rotors for the map view

**Foundation**
- `NSAttributedString.enumerateAttribute(_:in:options:using:)` — used in text rotor to find ranges of the custom alert attribute
- `NSAttributedString.Key` (custom) — developer-defined attribute key used to mark alert spans

## Code Highlights

Map rotor — filter annotation views by POI type and navigate by distance:
```swift
mapView.accessibilityCustomRotors = [customRotor(for: .stores), customRotor(for: .parks)]

func customRotor(for poiType: POI) -> UIAccessibilityCustomRotor {
    UIAccessibilityCustomRotor(name: poiType.rotorName) { [unowned self] predicate in
        let currentElement = predicate.currentItem.targetElement as? MKAnnotationView
        let annotations = self.annotationViews(for: poiType)  // sorted by distance
        let currentIndex = annotations.firstIndex { $0 == currentElement }
        let targetIndex: Int
        switch predicate.searchDirection {
        case .previous:
            targetIndex = (currentIndex ?? 1) - 1
        case .next:
            targetIndex = (currentIndex ?? -1) + 1
        }
        guard 0..<annotations.count ~= targetIndex else { return nil } // boundary
        return UIAccessibilityCustomRotorItemResult(targetElement: annotations[targetIndex],
                                                    targetRange: nil)
    }
}
```

Text rotor — jump between spans marked with a custom `NSAttributedString.Key`:
```swift
textView.accessibilityCustomRotors = [customRotor(for: .alertAttribute)]

func customRotor(for attribute: NSAttributedString.Key) -> UIAccessibilityCustomRotor {
    UIAccessibilityCustomRotor(name: attribute.rotorName) { [unowned self] predicate in
        var targetRange: UITextRange?
        let beginningRange = self.textRange(from: self.beginningOfDocument,
                                            to: self.beginningOfDocument)
        guard let currentRange = predicate.currentItem.targetRange ?? beginningRange else {
            return nil
        }
        let searchRange: NSRange, searchOptions: NSAttributedString.EnumerationOptions
        switch predicate.searchDirection {
        case .previous:
            searchRange = self.rangeOfAttributedTextBefore(currentRange)
            searchOptions = [.reverse]
        case .next:
            searchRange = self.rangeOfAttributedTextAfter(currentRange)
            searchOptions = []
        }
        self.attributedText.enumerateAttribute(
            attribute, in: searchRange, options: searchOptions) { value, range, stop in
            guard value != nil else { return }
            targetRange = self.textRange(from: range)
            stop.pointee = true
        }
        return UIAccessibilityCustomRotorItemResult(targetElement: self,
                                                    targetRange: targetRange)
    }
}
```

## Takeaways
- Assign a `[UIAccessibilityCustomRotor]` to any view's `accessibilityCustomRotors` property; VoiceOver automatically adds them to the rotor dial when that view is focused, requiring no further integration work.
- The closure receives `searchDirection` (`.previous`/`.next`) and the currently focused item; return `nil` to signal a boundary so VoiceOver can inform the user they have reached the first or last item.
- For text-based rotors, `targetElement` must conform to `UITextInput` and `targetRange` must be a `UITextRange`; use `NSAttributedString.enumerateAttribute` to locate custom-keyed spans.
- Audit the most visually complex areas of an app—maps, rich text, custom collection layouts—with VoiceOver to identify where sequential navigation is impractical and a custom rotor would restore parity with the visual experience.

---
_Source: WWDC20 Session 10116 page (transcript, code samples, and resource links)._
