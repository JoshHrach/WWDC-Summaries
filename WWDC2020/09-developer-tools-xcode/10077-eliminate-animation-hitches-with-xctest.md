# Eliminate Animation Hitches with XCTest
**WWDC20 · Session 10077** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10077/)

_Platforms:_ iOS, iPadOS (Xcode 12 Performance XCTest)

## Overview
Animation hitches — frames that appear on screen later than expected — are a primary source of perceived jank in iOS and iPadOS apps. Xcode 12 extends the `XCTOSSignpostMetric` API to capture hitch-specific measurements during Performance XCTests, giving developers a way to catch animation regressions in CI before they reach users.

The session defines hitches and hitch ratio as a comparable metric, introduces the `animationBegin` os_signpost interval type for custom instrumentation, covers the predefined UIKit scroll and navigation sub-metrics, and walks through writing, configuring, and iterating on a performance test that catches a real scroll regression caused by main-thread image scaling.

## Key Topics

### What Is a Hitch?
A hitch occurs when a frame misses its expected VSYNC deadline and appears on screen later than expected. On 60 Hz displays (iPhone, most iPads) the VSYNC cadence is 16.67 ms; on 120 Hz iPad Pro it is 8.33 ms. **Hitch time** is the number of milliseconds a frame is late. **Hitch time ratio** (ms/s) normalizes total hitch time over test duration, making results comparable across tests of different lengths.

Target hitch time ratios (matching Xcode Organizer thresholds from session 10076):
- < 5 ms/s — good user experience
- 5–10 ms/s — noticeable; investigate
- >= 10 ms/s — critical; take immediate action

### New Animation Metrics in XCTOSSignpostMetric (Xcode 12)
When measuring an **animation** os_signpost interval (as opposed to a generic interval), `XCTOSSignpostMetric` now returns five values instead of just duration:
- **Hitch count** — number of hitches in the measured block
- **Hitch time** — total milliseconds spent hitching
- **Hitch time ratio** — ms of hitch per second of test duration **[NEW]**
- **Frame rate** — frames per second displayed
- **Frame count** — total frames displayed

### Instrumenting Code with Animation Signposts
Three approaches to get animation interval data:

1. **Custom animation signpost** — use `os_signpost(.animationBegin, ...)` instead of `.begin`. The `.end` call is the same. One line change converts any existing instrumentation to emit animation intervals.
2. **UIKit scroll sub-metrics** — predefined class properties on `XCTOSSignpostMetric`:
   - `XCTOSSignpostMetric.scrollDecelerationMetric` **[NEW]**
   - `XCTOSSignpostMetric.scrollDraggingMetric` **[NEW]**
3. **UIKit navigation sub-metrics**:
   - `XCTOSSignpostMetric.navigationTransitionMetric` **[NEW]**
   - `XCTOSSignpostMetric.customNavigationTransitionMetric` **[NEW]**

The measure block listens for matching os_signpost intervals emitted during execution and collects metrics only for those intervals. Multiple distinct interval types can be listened to in a single measure block (e.g., both `scrollDeceleration` and `scrollDragging` from a single `swipeUp()` call).

### Writing a Performance Test: Best Practices
- Use a **separate test scheme** for performance tests; never mix with regular test schemes.
- Set **release build configuration** and **disable the debugger** — debug builds and the debugger add overhead that skews results.
- **Disable automatic screenshot collection** and turn off code coverage.
- **Disable all diagnostic options** (Runtime Sanitization, Runtime API Checking, Memory Management) — these add significant overhead.
- Use **`XCTMeasureOptions.invocationOptions = [.manuallyStop]`** to call `stopMeasuring()` after the animation and reset the application state before each iteration, so each of the five runs measures the same content.
- Swipe velocity can now be specified: `swipeUp(velocity: .fast)` **[NEW in Xcode 12]**.

### Catching a Real Regression
The session demonstrates catching a regression in which main-thread `UIImageView.contentMode = .scaleAspectFit` causes hitches because resizing is CPU-bound and occurs on the main thread. Switching to Core Animation's `setContentMode` offloads the work to the GPU, using existing pixel data without CPU allocation — the hitch count drops to zero.

## APIs & Frameworks

### XCTest (Xcode 12)
- `os_signpost(.animationBegin, log:name:)` **[NEW]** — emits an animation os_signpost interval; provides hitch metrics when measured
- `XCTOSSignpostMetric` — existing class; new class properties:
  - `.scrollDecelerationMetric` **[NEW]**
  - `.scrollDraggingMetric` **[NEW]**
  - `.navigationTransitionMetric` **[NEW]**
  - `.customNavigationTransitionMetric` **[NEW]**
- `XCTMeasureOptions` — `.invocationOptions: [XCTMeasureOptions.InvocationOptions]`; `.manuallyStop` value
- `XCUIElement.swipeUp(velocity:)` — velocity parameter **[NEW]**; `.fast`, `.slow`, `.default`
- `measure(metrics:options:block:)` — Performance XCTest entry point; `stopMeasuring()` available inside block
- Results visible in Xcode Report Navigator; baseline values can be set per metric

## Code Highlights

Measuring scroll deceleration with state reset between iterations:
```swift
func testScrollingAnimationPerformance() throws {
    app.launch()
    app.staticTexts["Meal Planner"].tap()
    let foodCollection = app.collectionViews.firstMatch

    let measureOptions = XCTMeasureOptions()
    measureOptions.invocationOptions = [.manuallyStop]

    measure(metrics: [XCTOSSignpostMetric.scrollDecelerationMetric],
            options: measureOptions) {
        foodCollection.swipeUp(velocity: .fast)
        stopMeasuring()
        foodCollection.swipeDown(velocity: .fast) // reset state
    }
}
```

Custom animation os_signpost interval (one line change from a generic interval):
```swift
os_signpost(.animationBegin, log: logHandle, name: "performAnimationInterval")
// ... animation work ...
os_signpost(.end, log: logHandle, name: "performAnimationInterval")
```

Predefined UIKit sub-metrics available in Xcode 12:
```swift
extension XCTOSSignpostMetric {
    open class var navigationTransitionMetric: XCTMetric { get }
    open class var customNavigationTransitionMetric: XCTMetric { get }
    open class var scrollDecelerationMetric: XCTMetric { get }
    open class var scrollDraggingMetric: XCTMetric { get }
}
```

## Takeaways
- Hitch time ratio (ms/s) is a stable, comparable metric for animation quality; target < 5 ms/s, investigate 5–10 ms/s, treat >= 10 ms/s as critical.
- Use `os_signpost(.animationBegin, ...)` to get hitch count, hitch time, and hitch time ratio in any custom instrumented code block.
- Use the new UIKit predefined sub-metrics (`scrollDecelerationMetric`, `scrollDraggingMetric`, etc.) to measure scroll and navigation performance without any app-side instrumentation.
- Always run performance tests in a dedicated scheme with a release build, no debugger, and no diagnostic options — these factors dramatically affect measurement accuracy.
- Reset app state between iterations using `manuallyStop` and `stopMeasuring()` to ensure each of the five runs measures the same content.

---
_Source: WWDC20 Session 10077 page (abstract, transcript, and code samples)._
