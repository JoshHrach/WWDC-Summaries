# Understanding CPU Usage with Web Inspector
**WWDC19 · Session 513** · [Watch](https://developer.apple.com/videos/play/wwdc2019/513/)

_Platforms:_ macOS Catalina 10.15, Safari 13

## Overview
Safari 13 ships a new CPU Usage Timeline inside Web Inspector that gives web developers direct visibility into how their pages and embedded web content consume CPU — and by extension, battery life. The timeline shows an energy impact gauge, a per-category main thread breakdown (JavaScript, painting, layout, style recalculation), and a per-thread CPU usage graph, making it straightforward to find and fix power regressions during load, idle, and interaction.

The session argues that all known web performance best practices — fast page load, optimized JavaScript, CSS animations and transitions — are also power-saving best practices. The CPU Usage Timeline provides the measurement infrastructure to quantify these improvements and identify unexpected regressions before they reach users.

A worked example using the WebKit Feature Status page demonstrates how to trace unexpected `requestAnimationFrame` work during scrolling back to unnecessary lazy-loading code, apply a conditional guard, and then replace the remaining polling loop entirely with the `IntersectionObserver` API — reducing average CPU from 16.3% to 9.5% during scroll.

## Key Topics

**CPU Usage Timeline**
A new timeline in Safari 13's Web Inspector Timelines tab. Records CPU usage across all threads, categorizes main-thread work, and computes an aggregate Energy Impact gauge (Low / Medium / High) over any selected time range.

**Energy Impact Gauge**
An interactive score representing total average CPU usage across all cores for the selected recording window. Selecting different slices (load, idle, interactive) gives separate impact readings, helping isolate which phase of page life causes excessive power draw.

**Main Thread Breakdown**
Color-coded by category: JavaScript processing, layout, style recalculation, painting. The indicator strip under the graph shows exactly when each category occurred during the recording timeline.

**Statistics and Sources Panel**
Shows counts of timer entries, `requestAnimationFrame` fires, and DOM events within the selected range. Clicking a category filters the Sources list to the responsible code and provides a direct jump to the JavaScript Debugger.

**Recording Best Practices**
- Record at least 15 seconds to get statistically meaningful measurements.
- Use the Reload button to automatically capture page load alongside interaction.
- Capture three phases: load, idle (wait for page to settle), and interaction (scroll, click, type).

**Power-Saving Strategies**
- Use CSS animations/transitions instead of JavaScript-driven animation.
- Avoid doing JavaScript work during scroll; scroll handlers fire at high frequency.
- Use `IntersectionObserver` instead of `requestAnimationFrame`-based visibility polling.
- Guard feature-detection and lazy-load setup to only run when needed.

## APIs & Frameworks

**Web Inspector (Safari 13)**
- CPU Usage Timeline **[NEW]** — new timeline type in the Timelines tab
- Energy Impact gauge **[NEW]** — Low / Medium / High interactive score for selected time range
- Main thread category indicator strip **[NEW]** — paint, layout, style, script color coding
- Per-thread CPU usage graph **[NEW]** — all threads contributing to the web content process
- Statistics panel — timer count, RAF count, event count for selected range
- Sources panel — filterable to code responsible for a given timer or event category
- Integration with JavaScript Debugger — one-click jump to source from timeline events

**Web Platform APIs**
- `IntersectionObserver` — observe element visibility without polling; fires only on entry/exit from viewport
- `IntersectionObserver(callback, options)` — constructor; `callback` receives `IntersectionObserverEntry[]`
- `IntersectionObserverEntry.isIntersecting` — boolean indicating visibility state
- `requestAnimationFrame(callback)` — frame-synchronized callback; shown as an anti-pattern for visibility polling
- `scroll` event — shown as an anti-pattern for heavy per-frame work
- CSS `animation` / `transition` — compositor-friendly alternatives to JavaScript animation

**Safari / WebKit Built-in Power Features**
- Background tab timer throttling — automatic; reduces timer frequency for hidden pages
- Content blocker extensions — block third-party tracking and resource loading

## Code Highlights

Problematic pattern (polling in rAF on every scroll):

```javascript
// Bad: requestAnimationFrame fires every frame during scroll
function updateImages() {
    images.forEach(img => {
        if (inView(img)) loadImage(img);
    });
}
requestAnimationFrame(updateImages);
window.addEventListener('scroll', updateImages);
```

Fix step 1 — guard setup when no lazy images exist:

```javascript
if (document.querySelectorAll('img[data-src]').length > 0) {
    setupLazyLoader();
}
```

Fix step 2 — replace polling with IntersectionObserver:

```javascript
const observer = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
        if (entry.isIntersecting) {
            loadImage(entry.target);
            observer.unobserve(entry.target);
        }
    });
});
document.querySelectorAll('img[data-src]').forEach(img => observer.observe(img));
```

## Takeaways
- The CPU Usage Timeline in Safari 13 Web Inspector makes it possible to measure the energy cost of each phase of a page's life cycle (load, idle, interaction) with an actionable gauge and per-category breakdown.
- Unnecessary JavaScript work during scroll is a common, high-impact power problem; `IntersectionObserver` eliminates the need for scroll handlers or rAF polling entirely.
- Guarding feature setup behind a meaningful condition prevents running dead code on pages that do not need the feature.
- CSS animations and transitions offload rendering work to the compositor and remain the lowest-power option for visual motion effects.

---
_Source: WWDC19 Session 513 page (abstract, transcript, and resource links)._
