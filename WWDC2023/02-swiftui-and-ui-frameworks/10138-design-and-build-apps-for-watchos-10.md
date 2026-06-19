# Design and Build Apps for watchOS 10
**WWDC23 · Session 10138** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10138/)

_Platforms:_ watchOS 10

## Overview
watchOS 10 is the most significant redesign of the Apple Watch UI since the original launch. This joint design-and-engineering session (presented by an Apple designer and a SwiftUI engineering manager) walks through the watchOS 10 design principles and shows how to implement them with the updated SwiftUI APIs.

The session covers four areas: the core design philosophy ("Apple Watch Moments"), updated navigation paradigms (`NavigationSplitView`, `TabView` with vertical pages, `NavigationStack`), a new layout grid system and toolbar placements, and the full-screen color and materials system that brings vibrancy and sense of place to every app.

## Key Topics

### Design Principles: The "Apple Watch Moment"
Apple Watch is a timekeeping device best suited for quick, focused interactions. The core design goal is surfacing the single most relevant piece of information the moment the wearer raises their wrist.

- **Brief and focused**: If you had ten seconds of someone's attention, what would you show? Design for that. News on Watch shows five top stories, not the full multi-tab experience from iPhone.
- **Digital Crown as primary navigation input**: watchOS 10 emphasizes the Digital Crown for getting to and navigating within apps—scroll, paginate, and make precise adjustments. Touch always backs up crown interactions.
- **Start with the Smart Stack widget**: Design the most timely, glanceable representation of your app as a Smart Stack widget first, then architect the app around those moments.

### Navigation Paradigms

**`NavigationSplitView`** (recommended for source list + detail apps)
- Borrowed from the two-column iPad layout. On Watch, the source list sits "tucked beneath" the detail view—just a tap away.
- Always initialize with a selection value so SwiftUI launches directly to the detail view.
- Source list has a shorter navigation bar (no title, no close button) to show more comparative data.
- The swipe-up gesture from the bottom edge reveals the source list with a smooth built-in transition.
- Best for apps like Weather (city list → city detail) or Stocks (stock list → price detail).

**`TabView` with vertical page style** (new in watchOS 10)
- Tabs scroll vertically; each tab can expand and resize inline as needed (supports large type, long localized strings).
- New blur transition between tabs plus `.containerBackground(_:for:)` creates a seamless visual blend.
- `TabView` auto-detects scrolling content in a tab and expands to accommodate it—just add a `List` inside a tab.
- **Tab-selection-driven animations**: drive SwiftUI animations (including `matchedGeometryEffect`) from the `TabView` selection value. Activity uses this to animate rings from a tab's content position to the toolbar as you scroll to the next tab.

**`NavigationStack`**
- Still the right choice for hierarchical navigation (Workout, Calendar, Music).
- Updated push animation: highlights and moves the selected row.
- Use a large title on the root view; no title on subviews with a back button.

### Layout System
watchOS 10's layout grids are derived mathematically from the curvature of the display. Three foundational layouts:
- **Dial-based Views**: dense information, full-screen color/imagery, up to four corner controls.
- **Infographic Views**: charts, graphs, data visualizations paired with text and metrics.
- **Lists**: scrolling content for browsing.

All layouts adapt automatically to all Apple Watch sizes supported by watchOS 10. Layout grids are available in the Apple Design Resources.

**Key SwiftUI layout APIs**:
- `.scenePadding(.horizontal)` — insets a view to align with the dial layout grid's outer margins.
- `.edgesIgnoringSafeArea(.vertical)` — allows dial content to extend into vertical safe areas for proper centering.
- Toolbar placements:
  - `.topBarLeading` / `.topBarTrailing` **[NEW]** — adds controls to the top bar; the system time moves to center when a top bar item is present.
  - `.bottomBar` — adds controls to the bottom bar, auto-aligned using the layout grid.
- `.controlSize(_:)` — adjust button prominence (e.g., `.large` for the primary action in Now Playing).

### Color and Materials
watchOS 10 introduces a full-screen color and vibrancy system:

**Full-screen background materials** (four levels): `.ultraThin`, `.thin`, `.regular`, `.thick` — blur the background at different intensities. Applied via the existing SwiftUI materials API.

**`containerBackground(_:for:)`** **[NEW]** — sets a full-screen background gradient tinted with any color. Presentations throughout the system use a thin material, giving a "hint" of the underlying view color.

**Foreground styles (vibrant)**:
- `.primary` — solid foreground for primary labels and key chart bars.
- `.secondary`, `.tertiary`, `.quaternary` — progressively more transparent/vibrant; let background color show through. Use to create information hierarchy without adding visual noise.
- All system colors have new vibrant variants that ensure legibility over full-screen backgrounds.

**Navigation bar**: new variable blur provides a gentle transition as content scrolls underneath navigation items.

## APIs & Frameworks

**SwiftUI (watchOS 10 — New/Updated)**
- `NavigationSplitView` — two-column source list + detail for watchOS (selection always initialized)
- `TabView(.page(indexDisplayMode: .never))` + `.tabViewStyle(.verticalPage)` **[NEW]** — vertical paging tabs with blur transitions
- `TabView` selection-driven animations **[NEW]** — animate with `matchedGeometryEffect` bound to tab selection
- `.containerBackground(_:for:)` **[NEW]** — full-screen background color/gradient behind content
- `.scenePadding(.horizontal)` — align to dial layout grid margins
- `.edgesIgnoringSafeArea(.vertical)` — extend dial views into vertical safe areas
- Toolbar placement `.topBarLeading` / `.topBarTrailing` **[NEW]** — top bar controls with auto-centered time
- Toolbar placement `.bottomBar` — grid-aligned bottom controls
- `.controlSize(.large)` — larger prominent button style
- Vibrant foreground styles: `.primary`, `.secondary`, `.tertiary`, `.quaternary` over material backgrounds
- Full-screen materials: `.ultraThinMaterial`, `.thinMaterial`, `.regularMaterial`, `.thickMaterial`

## Code Highlights

Dial-based view using scene padding and safe area:
```swift
struct DialBasedView: View {
    var body: some View {
        ZStack {
            Rectangle()
                .foregroundStyle(Color.clear)

            Circle()
                .foregroundStyle(Color.red)
                .scenePadding(.horizontal)
        }
        .edgesIgnoringSafeArea(.vertical)
    }
}
```

Container background for full-screen color:
```swift
// Noise app: green gradient background
SomeView()
    .containerBackground(Color.green.gradient, for: .navigation)
```

NavigationSplitView always launching to detail:
```swift
NavigationSplitView {
    List(cities, selection: $selectedCity) { city in
        Text(city.name)
    }
} detail: {
    if let city = selectedCity {
        CityDetailView(city: city)
    }
}
.onAppear {
    selectedCity = cities.first  // launch directly to detail
}
```

TabView with vertical page style and blur transition:
```swift
TabView(selection: $selectedTab) {
    MoveDetailView()
        .tag(Tab.move)
    ExerciseDetailView()
        .tag(Tab.exercise)
    StandDetailView()
        .tag(Tab.stand)
}
.tabViewStyle(.verticalPage)
.containerBackground(Color.red.gradient, for: .tabView)
```

## Resources
- [Human Interface Guidelines: watchOS](https://developer.apple.com/design/human-interface-guidelines/designing-for-watchos)
- [Apple Design Resources](https://developer.apple.com/design/resources/)
- Related: "Meet watchOS 10" (WWDC23 10026)
- Related: "Update your app for watchOS 10" (WWDC23 10031)
- Related: "Build widgets for the Smart Stack on Apple Watch" (WWDC23 10029)
- Related: "What's new in SwiftUI" (WWDC23 10148)

## Takeaways
- The "Apple Watch Moment" framing—surface the single most relevant piece of information in the fewest interactions—should guide every watchOS app design decision.
- Choose `NavigationSplitView` for source-list + detail patterns, vertical `TabView` for a few focused views, and `NavigationStack` for deeper hierarchies; mixing paradigms is not recommended.
- `.containerBackground(_:for:)` is the single most impactful API for adopting watchOS 10's full-screen color aesthetic; pair with `.secondary` foreground style for supportive elements to let background color show through.
- Tab-selection-driven animations with `matchedGeometryEffect` enable sophisticated transitions (like Activity's ring animation) with minimal code.

---
_Source: WWDC23 Session 10138 page (abstract, transcript, chapter summaries, code samples, and resource links)._
