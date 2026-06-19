# Optimize for Variable Refresh Rate Displays
**WWDC21 · Session 10147** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10147/)

_Platforms:_ iPadOS 15, macOS Monterey 12

## Overview
This session covers two distinct variable refresh rate technologies: Adaptive-Sync displays on macOS (new in Monterey) and ProMotion displays on iPad Pro. Both require careful frame-pacing to deliver smooth, hitch-free rendering. On the Mac, Adaptive-Sync allows a frame's on-screen duration to vary dynamically (e.g., 8–25 ms on a 40–120 Hz display), eliminating double-frames from minor GPU overruns and enabling even presentation at sub-maximum rates. On iPad, ProMotion dynamically adjusts the display refresh rate to match content, conserving power — but system states like Low Power Mode (new in iPadOS 15) and thermal throttling can cap the maximum rate, requiring apps with custom drawing loops to adapt.

The Metal presentation APIs (`presentAfterMinimumDuration:`, `presentAtTime:`) pair naturally with Adaptive-Sync. For custom drawing on iPad, `CADisplayLink` is the recommended tool because it is synchronized to the display, automatically adjusts its callback rate when the display rate changes, and provides `targetTimestamp` for accurate frame-timing calculations.

The key principle on both platforms is never hard-code a frame rate — always query it at runtime and use timestamps from the display link or Metal command buffer completion handlers to drive timing decisions.

## Key Topics
- **Adaptive-Sync on macOS** — Frame on-screen duration varies within a display-defined range; eliminates hitches from small GPU overruns; requires full-screen mode and a supported display with variable refresh rate enabled in System Preferences.
- **Detecting Adaptive-Sync** — New `NSScreen.minimumRefreshInterval` and `NSScreen.maximumRefreshInterval` properties; compare for inequality to detect Adaptive-Sync mode; also check `NSWindow.styleMask` for full-screen.
- **Metal Presentation Strategies** — `presentDrawable:afterMinimumDuration:` for fixed user-chosen rate; rolling-average GPU time fed into `presentDrawable:afterMinimumDuration:` for self-tuning smooth pacing; avoid plain `presentDrawable:` for Adaptive-Sync workloads.
- **ProMotion on iPad Pro** — Up to 120 Hz; dynamically reduces refresh rate to save power; Low Power Mode (iPadOS 15 new) caps to 60 Hz; Accessibility "Limit Frame Rate" toggle also caps to 60 Hz; thermal state can apply further restrictions.
- **CADisplayLink Best Practices** — Query `maximumFramesPerSecond` and `link.duration` at runtime; use `preferredFramesPerSecond` hint; use `targetTimestamp` not `timestamp` for animation state; track `previousTargetTimestamp` to compute correct time delta across frame-rate changes and skipped callbacks.
- **CVDisplayLink vs CADisplayLink** — `CVDisplayLink` (CoreVideo, macOS); `CADisplayLink` (CoreAnimation, iOS/iPadOS/tvOS/Mac Catalyst).

## APIs & Frameworks
- **Metal** framework
  - `MTLCommandBuffer.presentDrawable(_:afterMinimumDuration:)` — Present drawable after a minimum on-screen duration (ideal for Adaptive-Sync)
  - `MTLCommandBuffer.presentDrawable(_:atTime:)` — Present drawable at an absolute time
  - `MTLCommandBuffer.presentDrawable(_:)` — Plain present (no built-in pacing; avoid for Adaptive-Sync)
  - `MTLCommandBuffer.addCompletedHandler(_:)` — Completion callback for measuring GPU time
  - `MTLCommandBuffer.GPUStartTime` — GPU work start timestamp
  - `MTLCommandBuffer.GPUEndTime` — GPU work end timestamp
  - `CAMetalLayer.nextDrawable()` — Acquire the next Metal drawable
- **AppKit / NSScreen** (macOS)
  - `NSScreen.minimumRefreshInterval` **[NEW]** — Shortest possible on-screen duration; equal to `maximumRefreshInterval` on fixed-rate displays
  - `NSScreen.maximumRefreshInterval` **[NEW]** — Longest possible on-screen duration before forced panel refresh
  - `NSWindow.styleMask` — Check for `NSFullScreenWindowMask` to confirm full-screen mode
- **CoreAnimation / UIKit** (iOS/iPadOS)
  - `CADisplayLink` — Display-synchronized callback timer
    - `CADisplayLink.preferredFramesPerSecond` — Hint for desired callback rate
    - `CADisplayLink.duration` — Current interval between callbacks (dynamically updated)
    - `CADisplayLink.timestamp` — Time the callback was invoked
    - `CADisplayLink.targetTimestamp` — Time the next frame will be composited; use this for animation state
    - `CADisplayLink.addToRunLoop(_:forMode:)` — Register display link
  - `UIScreen.maximumFramesPerSecond` — Maximum frames per second for the screen (120 on ProMotion, even under Low Power Mode)
  - `CACurrentMediaTime()` — Query current media time to measure remaining vsync budget
- **CoreVideo** (macOS)
  - `CVDisplayLink` — macOS equivalent of `CADisplayLink`

## Code Highlights
Detect Adaptive-Sync mode and full-screen state (Objective-C):

```objc
- (BOOL)isAdaptiveSyncSupported:(NSScreen *)screen {
    NSTimeInterval minInterval = screen.minimumRefreshInterval;
    NSTimeInterval maxInterval = screen.maximumRefreshInterval;
    return minInterval != maxInterval;
}
- (BOOL)isWindowFullscreen:(NSWindow *)window {
    return ([window styleMask] &= NSFullScreenWindowMask) == NSFullScreenWindowMask;
}
```

Self-tuning frame pacing using rolling-average GPU time:

```objc
NSTimeInterval averageGPUTime = screen.minimumRefreshInterval;
[commandBuffer presentDrawable:currentDrawable afterMinimumDuration:averageGPUTime];
[commandBuffer addCompletedHandler:^(id<MTLCommandBuffer> buffer) {
    const NSTimeInterval GPUTime = buffer.GPUEndTime - buffer.GPUStartTime;
    const double alpha = .25;
    averageGPUTime = (GPUTime * alpha) + (averageGPUTime * (1.0 - alpha));
}];
```

CADisplayLink callback using `targetTimestamp` and tracking previous target for correct delta:

```objc
- (void)displayLinkCallback:(CADisplayLink *)link {
    progress += link.targetTimestamp - previousTargetTimestamp;
    previousTargetTimestamp = link.targetTimestamp;
    [self renderAnimationWithProgress:progress withDeadline:link.targetTimestamp];
}
```

## Takeaways
- Adaptive-Sync on macOS requires full-screen mode plus a supported display with the variable refresh rate setting enabled; detect it via `NSScreen.minimumRefreshInterval != maximumRefreshInterval`.
- Use `presentDrawable:afterMinimumDuration:` rather than plain `presentDrawable:` to get smooth, even frame delivery on Adaptive-Sync displays.
- Always use `CADisplayLink.targetTimestamp` (not `timestamp`) as the basis for animation state updates to ensure hitch-free transitions during frame-rate changes.
- Track `previousTargetTimestamp` across callbacks to correctly compute the time delta even when callbacks are delayed or skipped.

---
_Source: WWDC21 Session 10147 page (abstract, chapter summaries, code samples, and resource links)._
