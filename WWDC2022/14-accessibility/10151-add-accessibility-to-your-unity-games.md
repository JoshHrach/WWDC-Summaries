# Add Accessibility to Your Unity Games
**WWDC22 · Session 10151** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10151/)

_Platforms:_ iOS 16, iPadOS 16, tvOS 16, macOS Ventura 13

## Overview
This session introduces the open source Apple Accessibility plug-in for Unity, enabling developers to add VoiceOver, Switch Control, Dynamic Type, and UI accommodation support to Unity-based games on Apple platforms. The plug-in is part of Apple's broader suite of Unity plug-ins available on GitHub.

The session walks through a sample card game, demonstrating how to define accessibility elements with labels, traits, and values — making the game fully navigable via VoiceOver and Switch Control. It then covers Dynamic Type integration for responsive text sizing and shows how to query UI accommodation settings like Reduce Transparency, Increase Contrast, and Reduce Motion.

The result is a game that is accessible to millions of users who rely on assistive technologies, achieved with minimal code changes thanks to the plug-in's Unity-native component model.

## Key Topics

### Accessibility Elements
- `AccessibilityNode` Unity component marks game objects as accessible elements
- Each element gets a **Label** (read by VoiceOver), optional **Traits** (Button, Static Text, etc.), and an optional **Value** (dynamic content like card face)
- Standard Unity UI controls (Text, Button) get default labels automatically; custom objects require explicit labels
- `accessibilityValueDelegate` closure provides dynamic values at runtime

### Dynamic Type
- `AccessibilitySettings.PreferredContentSizeMultiplier` — float multiplier matching the user's preferred text size
- `AccessibilitySettings.PreferredContentSizeCategory` — enum mapping to slider ticks (e.g., `.AccessibilityMedium`)
- `AccessibilitySettings.onPreferredTextSizesChanged` — event fired when user changes text size in Settings or Control Center
- Can be applied to both text elements (scale font size) and non-text assets (swap to large-print materials)

### UI Accommodations
- `AccessibilitySettings.IsReduceTransparencyEnabled` — check to remove blur/transparency effects
- `AccessibilitySettings.IsIncreaseContrastEnabled` — check to use higher-contrast UI assets
- `AccessibilitySettings.IsReduceMotionEnabled` — check to disable animations (e.g., card flip coroutines)

## APIs & Frameworks

**Apple Accessibility Unity Plug-in** (`Apple.Accessibility` namespace) **[NEW]**
- `AccessibilityNode` (MonoBehaviour component) **[NEW]**
  - `.Label` — string property for the accessible label
  - `.Traits` — enum flags (`.Button`, `.StaticText`, `.None`, and more)
  - `.accessibilityValueDelegate` — `Func<string>` delegate for dynamic values
- `AccessibilitySettings` (static class) **[NEW]**
  - `.PreferredContentSizeMultiplier` — `float` **[NEW]**
  - `.PreferredContentSizeCategory` — `ContentSizeCategory` enum **[NEW]**
  - `.onPreferredTextSizesChanged` — event **[NEW]**
  - `.IsReduceTransparencyEnabled` — `bool` **[NEW]**
  - `.IsIncreaseContrastEnabled` — `bool` **[NEW]**
  - `.IsReduceMotionEnabled` — `bool` **[NEW]**
- `ContentSizeCategory` enum **[NEW]** — values including `.AccessibilityMedium` and other size categories

**Platform Frameworks (underlying)**
- `Accessibility` (Apple framework, bridged via plug-in)
- VoiceOver (UIAccessibility)
- Switch Control (UIAccessibility)

## Code Highlights

Defining an accessible card with dynamic value:
```csharp
using Apple.Accessibility;
public class AccessibleCard : MonoBehaviour {
    public PlayingCard cardType;
    public bool isCovered;
    void Start() {
        var node = GetComponent<AccessibilityNode>();
        node.accessibilityValueDelegate = () => {
            if (isCovered) return "covered";
            if (cardType == PlayingCard.AceOfSpades) return "Ace of Spades";
            // ...
        };
    }
}
```

Responding to Dynamic Type changes:
```csharp
public class DynamicTextSize : MonoBehaviour {
    int originalSize;
    void Start() { originalSize = GetComponent<Text>().textSize; }
    void OnEnable() { AccessibilitySettings.onPreferredTextSizesChanged += _settingsChanged; }
    void _settingsChanged() {
        GetComponent<Text>().textSize = (int)(originalSize *
            AccessibilitySettings.PreferredContentSizeMultiplier);
    }
}
```

Disabling animations for Reduce Motion:
```csharp
public void Flip() {
    if (!AccessibilitySettings.IsReduceMotionEnabled)
        StartCoroutine(Animate());
    else
        transform.rotation = Quaternion.identity;
}
```

## Takeaways
- The open source Apple Accessibility Unity plug-in (on GitHub at `apple/unityplugins`) makes it straightforward to support VoiceOver, Switch Control, Dynamic Type, and UI accommodations in Unity games.
- `AccessibilityNode` component requires no subclassing — attach it to any GameObject and configure Label, Traits, and Value via the Inspector or scripting.
- Dynamic Type support should cover both text scaling (via `PreferredContentSizeMultiplier`) and asset swapping (via `PreferredContentSizeCategory`) for a complete low-vision experience.
- Always honor Reduce Motion by providing a non-animated fallback — users with motion sensitivity will appreciate it.

---
_Source: WWDC22 Session 10151 page (abstract, chapter summaries, code samples, and resource links)._
