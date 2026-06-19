# Design Widgets for the Smart Stack on Apple Watch
**WWDC23 · Session 10309** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10309/)

_Platforms:_ watchOS 10

## Overview
watchOS 10 introduces the Smart Stack — a new surface accessed by turning the Digital Crown from any watch face, showing a dynamically ordered stack of widgets. This session by Apple designer Ed covers how to design widgets that are glanceable, distinctive, and relevance-aware for this new context. It complements the engineering code-along "Build widgets for the Smart Stack on Apple Watch" (10029).

The Smart Stack solves a real tension in Apple Watch design: some users want clean, expressive watch faces without complications, while others want maximal utility. The Smart Stack lets both audiences access timely app information without cluttering the watch face. For developers, this means widgets are now a first-class watchOS feature, not just an iOS port.

## Key Topics

### Design Intent and Interaction
- The Smart Stack appears when the user turns the Digital Crown from any watch face.
- Widgets are intelligently reordered by relevance; users can also manually pin, add, or remove widgets.
- Design should assume the user spends no more than ~10 seconds engaging with glanceable watch content.

### Six Standard Design Layouts **[NEW]**
Apple provides six recommended layout templates (available in Apple Design Resources) to ensure visual consistency across the stack:
1. **Three-line text** — headline + two lines of text; ideal for news, messages, descriptions.
2. **Three-line text with color code** — adds a colored left-edge stripe for category/calendar color coding.
3. **Bar gauge + text** — horizontal progress bar with supporting text; good for progress tracking (e.g., audiobook position).
4. **Circular graphic + text** — icon/ring graphic with supporting text; ideal for Activity rings, status indicators.
5. **Large text** — single large number or keyword at a glance; useful for counts, states ("High", "Low").
6. **Chart** — data-over-time visualization; for widgets that primarily express trends.

Custom layouts are permitted when standard templates don't suit the content, but even custom layouts should use system text styling classes for type size, weight, and margins.

### Combination Widget (Combo) **[NEW]**
A system-generated widget slot that holds three circular complications of the user's choosing. Apps that provide circular complications can appear here. Best used when three related complications form a meaningful set (e.g., temperature + UV index + AQI, or three family member Contact complications).

### Color and Iconography
- Default widget appearance: dark material background with white text.
- Widgets in the Smart Stack are NOT tinted to match the watch face color (unlike watch face complications) — developers have full color control.
- Use a colored or dynamic background to reinforce app recognition or convey state (e.g., Weather gradient, Stocks red/green, Audiobook blurred cover art).
- Use SF Symbols or vector icons in front of content to establish context.
- Avoid raster images for icons; prefer vector assets.

### Sessions
A **Session** is a clearly defined active state with a start and end (music playback, timer, stopwatch, workout). The system auto-generates a **Session Control widget** that floats to the top of the Smart Stack when a Session is active. Developers should NOT duplicate this with their own widget. Instead, design the app widget to complement Sessions — suggest content leading into a Session or show post-Session summaries — so the widget remains useful even during an active Session.

### Relevancy Signals **[NEW]**
Widgets declare relevancy signals so the Smart Stack can intelligently reorder them. Key signals:
- **Time/date** — widget becomes relevant as an event, alarm, or reminder approaches.
- **Location (GPS-based)** — widget becomes relevant when the user arrives at a specific location.
- **Location (inferred)** — Home, Work, School — no GPS permission required.
- **Headphones connected** — ideal for audio apps (e.g., surface Audiobooks widget when AirPods connect).
- **Wake/sleep** — morning or bedtime relevance.
- **Workout start/end** — surface fitness-related widgets when workout activity is detected.

## APIs & Frameworks

### WidgetKit **[NEW watchOS support]**
- `Widget` protocol — defines a widget for Smart Stack **[NEW on watchOS]**
- `WidgetConfiguration` — `StaticConfiguration` or `IntentConfiguration` for watchOS widgets **[NEW on watchOS]**
- `WidgetFamily.accessoryRectangular` — the primary Smart Stack widget family **[NEW]**
- `WidgetFamily.accessoryCircular` — circular complication family (appears in Combo widget)
- `TimelineProvider` — supplies widget entries with timestamps
- `TimelineEntry` — data snapshot at a point in time
- `TimelineReloadPolicy` — controls how often the widget timeline refreshes
- `WidgetRelevanceProvider` / `RelevantContext` — declares relevancy signals to the system **[NEW]**
  - `.date(_:)` — time-based relevancy **[NEW]**
  - `.location(_:)` — GPS location-based relevancy **[NEW]**
  - `.inferredLocation(_:)` — Home/Work/School inferred location relevancy **[NEW]**
  - `.headphonesConnected` — audio session trigger **[NEW]**
  - `.workout` — workout state relevancy **[NEW]**
  - `.sleep` — wake/sleep relevancy **[NEW]**
- Session Control widget — system-generated; automatically appears for active sessions (music, workout, timer)
- Smart Stack pin/reorder — user-controlled widget ordering on top of system relevancy
- Text styling classes — system-defined type sizes, weights, and margins for consistent widget layouts

### Related Documentation
- [WidgetKit documentation](https://developer.apple.com/documentation/WidgetKit)
- [Keeping a widget up to date](https://developer.apple.com/documentation/WidgetKit/Keeping-a-Widget-Up-To-Date)
- [Human Interface Guidelines: Widgets](https://developer.apple.com/design/human-interface-guidelines/widgets)

## Code Highlights
No code samples in this design session. See "Build widgets for the Smart Stack on Apple Watch" (10029) for the code-along implementation.

## Takeaways
- Use one of the six standard design layouts for most widgets; reach for custom layouts only when the content truly cannot fit the standard templates.
- Assign color to your widget background to aid recognition — Smart Stack widgets are not watch-face-tinted, so you have full creative control.
- Declare relevancy signals thoughtfully: the more signals apps provide, the smarter and more useful the Smart Stack becomes for users.
- Do not replicate Session Control widgets; design your widget to be useful before and after an active session instead.

---
_Source: WWDC23 Session 10309 page (abstract, chapter summaries, code samples, and resource links)._
