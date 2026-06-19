# What's New in Web Inspector
**WWDC20 · Session 10646** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10646/)

_Platforms:_ macOS Big Sur 11 (Safari 14)

## Overview
Web Inspector in Safari 14 received a sweeping overhaul: the Resources Tab and Debugger Tab were merged into a new unified Sources Tab, three entirely new tabs (Graphics, Layers, and a revamped Timelines timeline) were introduced, and debugging capabilities expanded significantly. Local Overrides let developers intercept and replace any network response — including HTTP status codes and headers — entirely within Web Inspector without touching source code. New JavaScript debugger actions (Step expression-level), script blackboxing, and new global breakpoints for microtasks, animation frames, timeouts, and intervals make pinpointing JavaScript issues faster. Cookie editing, inline-script pretty-printing, heap instance querying, and ITP debug logging round out a release that touches nearly every part of the tool.

## Key Topics

### New Sources Tab (Merges Resources + Debugger)
The Sources Tab consolidates the former Resources Tab and Debugger Tab:
- Lists all resources loaded since Web Inspector opened (WebSockets, XHRs, fetch calls, scripts, stylesheets, HTML, images)
- Provides alternate response representations: DOM Tree for HTML, Object Tree for JSON
- Contains all JavaScript debugger stepping controls and breakpoint management
- Is the home for the new Local Overrides feature

### Local Overrides
Any resource loaded over the network can be intercepted and replaced with a local copy:
- Click **Create Override** on any resource to copy its current response into an editable local file
- Overrides persist across inspector and page sessions
- Edit the override's content, HTTP status code, and HTTP headers via right-click → Edit Local Override
- The overridden resource icon changes throughout Web Inspector to indicate replacement
- Works for HTML, JavaScript, CSS, images, and any other resource type

### Inspector Bootstrap Script
A special JavaScript file that runs before any page code — before even `<script>` tags:
- Added via the add-resource button in the Sources Tab sidebar
- Use it to swizzle built-in functions, install global logging hooks, or set up debug state that must exist before page initialization
- Persists across inspector and page sessions, just like Local Overrides

### New Debugger Features

**Step (Expression-Level) Action** — New stepping granularity between Step In and Step Over:
- Steps to the next *expression* in the current call frame, not the next *statement*
- Allows moving through chained calls like `a() || b() || c()` one call at a time without Step In/Out gymnastics

**Script Blackboxing** — Instructs the debugger to skip-over designated scripts when pausing:
- Hover over any script in the Sources sidebar and click the blackbox button
- Configure regex patterns in Settings to blackbox multiple scripts at once (e.g., all jQuery files)
- Paused frames inside blackboxed scripts are shown greyed out in the call stack but not jumped to

**New Global Breakpoints:**
- Debugger Statements — toggleable independently of all other breakpoints
- All Microtasks — pause before any Promise or `queueMicrotask()` callback
- All Animation Frames — pause before `requestAnimationFrame` callbacks
- All Timeouts — pause before `setTimeout` callbacks
- All Intervals — pause before `setInterval` callbacks
- All Events — pause before any event listener callback (combined with script blackboxing to skip framework wrappers)

**HTML Pretty Printing** — Web Inspector now formats HTML and XML including inline `<script>` and `<style>` blocks; automatically detects minified HTML and toggles pretty printing, enabling breakpoints and stepping in inline scripts.

### Timelines Tab: Media & Animations Timeline
A new timeline in the Timelines Tab captures:
- All `<video>` and `<audio>` element lifecycle events (play, pause, buffering, etc.)
- CSS animation and CSS transition creation, playback, and destruction
- Enables correlation of media/animation state changes with other timeline activity (layout, scripting)
- Each animation/transition has its own row with links to the associated DOM node, timing parameters, and keyframes

Timeline recordings can be exported and imported for sharing or later analysis.

### Graphics Tab (Replaces Canvas Tab)
Expands the old Canvas Tab:
- Canvases section: all WebGL/Canvas 2D contexts with shader inspection and JavaScript API call recordings
- Animations section: all Web Animations, CSS animations, and CSS transitions with timing parameters, keyframes, and (for JS-created animations) creation backtrace
- **Log Animation** context menu action saves the JavaScript Web Animation object to a Console temporary variable for direct API manipulation

### Layers Tab
Live view of the compositing layer tree:
- Lists each layer with its memory cost and paint count
- Shows why each layer was created (e.g., animated by CSS, `will-change` property)
- 3D orbit visualization: click-drag to rotate horizontally/vertically, scroll to zoom, right-click-drag to pan

### Storage Tab Improvements
- Filter bar added to cookies, LocalStorage, SessionStorage, IndexedDB, and other storage views
- **Cookie editing**: double-click any cookie field to open an edit popover for all cookie attributes; changes apply on dismiss
- **Add Cookie**: button creates a new cookie with a required name; persists beyond Web Inspector

### Console: New Heap Querying Functions
- `queryInstances(constructor)` — scans the JavaScript heap and returns all objects that are instances of the given constructor (including subclass instances); also accepts a prototype
- `queryHolders(object)` — scans the JavaScript heap and returns all objects that hold a strong reference to the given object; useful for diagnosing memory leaks

### ITP and Ad Click Attribution Debug Logging
When Web Inspector is open and ITP Debug Mode is enabled (Develop menu), all Intelligent Tracking Prevention debug logs appear in the Web Inspector Console (and system Console.app). Ad Click Attribution debug logs appear similarly when the experimental feature is enabled.

### UI / Accessibility Improvements
- Toolbar and dashboard merged into the tab bar (saves vertical space)
- Tightened spacing throughout for more visible content area
- Dark Mode variants for all icons; independent Dark Mode toggle in Settings Tab
- Improved accessibility for screen reader navigation
- Network Tab now shows unique domain count, total transfer size, and redirect count below the main table

## APIs & Frameworks

### Web Inspector Tools (No Runtime API)
- Sources Tab **[NEW — replaces Resources + Debugger tabs]**
- Local Overrides **[NEW]** — per-resource response interception with editable content, status, and headers
- Inspector Bootstrap Script **[NEW]** — pre-page JavaScript injection persistent across sessions
- Step (expression-level) debugger action **[NEW]**
- Script Blackboxing **[NEW]** — per-script and regex-pattern-based debugger skip
- Global breakpoints: Debugger Statements, All Microtasks, All Animation Frames, All Timeouts, All Intervals, All Events **[NEW]**
- HTML/XML pretty printing with inline script/style support **[NEW]**
- Media & Animations timeline **[NEW]**
- Graphics Tab **[NEW — replaces Canvas Tab]** with Web Animations support
- Layers Tab **[NEW]** — live compositing layer tree with 3D visualization
- Cookie editing and filtering in Storage Tab **[NEW]**
- `queryInstances(constructor)` Console function **[NEW]**
- `queryHolders(object)` Console function **[NEW]**
- ITP / Ad Click Attribution debug logging in Console **[NEW]**

## Code Highlights

Inspector Bootstrap Script example — swizzle `fetch` before page code runs:
```javascript
// Runs before any page script
const originalFetch = window.fetch;
window.fetch = function(...args) {
    console.log('[Bootstrap] fetch called with:', args[0]);
    return originalFetch.apply(this, args);
};
```

Console heap queries:
```javascript
// Find all instances of Pet (including subclasses)
queryInstances(Pet)   // returns [buddy]

// Find all objects holding a strong reference to john
queryHolders(john)    // returns [alice] (alice.parent === john)
```

List with `onDelete` (shown as companion SwiftUI code in session):
```swift
// watchOS/SwiftUI pattern referenced in related session
List {
    ForEach(model.locations) { ClockCell(location: $0) }
    .onDelete { deleteClock(index: $0) }
}
```

## Takeaways

- Local Overrides are the headline feature: intercept any network response, modify its content, status code, and headers, and reload — no server changes required, persists across sessions.
- The new Step action fills a critical gap in JavaScript debugging: expression-level stepping through chained calls avoids the Step In / Step Out dance that was error-prone and slow.
- Script blackboxing makes global breakpoints (All Events, All Microtasks) practical in apps that use frameworks: the debugger skips library code and pauses in your code instead.
- `queryInstances` and `queryHolders` bring heap-level memory investigation directly into the Console without requiring a full heap snapshot capture.
- The Graphics, Layers, and Media & Animations views together provide a complete picture of compositing, animation performance, and media lifecycle in a single inspector.

---
_Source: WWDC20 Session 10646 page (abstract, transcript, and resource links)._
