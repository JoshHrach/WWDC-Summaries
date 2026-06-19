# Power down: Improve battery consumption
**WWDC22 · Session 10083** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10083/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, watchOS 9, tvOS 16

## Overview
This session from Apple's Software Power team presents four concrete actions developers can take to reduce battery drain in their apps: adopting Dark Mode, auditing animation frame rates, limiting background service usage, and deferring time-insensitive work. Each area is grounded in how the hardware actually consumes power — OLED pixels require less energy for darker colors, ProMotion displays scale power with refresh rate, and background location/audio services keep the system awake unnecessarily when left running idle.

The session demonstrates practical tooling: Xcode gauges, Instruments' CoreAnimation FPS instrument, and MetricKit for measuring `cumulativeBackgroundLocationTime`. It also covers new iOS 16 Control Center visibility into apps actively using location services in the background, which gives users an unexpected way to discover apps with runaway location sessions.

## Key Topics

### Dark Mode and OLED Power Savings
- On OLED displays (iPhone 13/13 Pro and later), darker pixels consume less power — up to **70% display power savings** for heavily dark-background apps
- Savings scale with screen brightness; highest brightness = highest savings
- Adopt dynamic colors (`UIColor.systemBackground`, semantic color assets in Xcode) so light/dark modes switch automatically
- Support alternate images for each mode in the asset catalog
- For web content in Safari: implement `color-scheme` CSS property + stylesheet variables for color references; apply alternate image/media variants
- Safari does NOT auto-darken web content — developers must opt in

### Auditing Frame Rates (ProMotion Displays)
- The **highest-rate animation on screen** determines the display's refresh rate for the entire frame
- Secondary animations (e.g., a scrolling text banner) can silently force the whole screen to a higher rate than the primary content requires
- Reducing a secondary animation from 60 fps to 30 fps can save **up to 20%** battery drain
- Use **Instruments > CoreAnimation FPS** instrument to see frame rate timeline and find offending secondary elements
- Use `CADisplayLink.preferredFrameRateRange` (minimum/maximum/preferred) to tell the system what rate your content needs; system picks the nearest available rate

### Limiting Background Location and Audio
- **Location**: Call `stopUpdatingLocation()` as soon as location data is no longer needed; do not leave sessions running idle in the background
- Debugging location overuse:
  - **Xcode gauges** — real-time location usage and energy timeline
  - **MetricKit** — `MXAppRunTimeMetric.cumulativeBackgroundLocationTime` for aggregate background location time over a day of use
  - **iOS 16 Control Center** — new indicator showing apps currently using location; users can tap to see detail
- **Audio**: Enable `AVAudioEngine.isAutoShutdownEnabled = true` — engine detects idle state and shuts down audio hardware automatically; restarts on demand; enforced on watchOS
- General principle: always tell the system when you are done with a service

### Deferring Work with Background APIs
- **`BGProcessingTask`** — defer database cleanup, backups, ML training to charging time
  - `BGProcessingTaskRequest(identifier:)` — specify task; set `requiresExternalPower`, `requiresNetworkConnectivity`
  - System grants several minutes of runtime at an opportune time
- **Discretionary `URLSession`** — defer non-user-initiated networking (telemetry, prefetching) to charging + Wi-Fi
  - `URLSessionConfiguration.isDiscretionary = true`
  - `timeoutIntervalForResource` / `timeoutIntervalForRequest` — prevent indefinite retry
  - `task.earliestBeginDate` — delay start until a future time
  - `task.countOfBytesClientExpectsToSend` / `countOfBytesClientExpectsToReceive` — help system load-balance
- **Push priority** — low-priority pushes (`apns-priority: 5`) delay delivery until device is already awake; high-priority (`apns-priority: 10`) wakes the device immediately; use low priority for passive/non-urgent notifications

## APIs & Frameworks

**UIKit / AppKit**
- `UIColor` dynamic color assets — respond to light/dark mode changes
- `UITraitCollection.userInterfaceStyle` — `.light` / `.dark`

**CoreAnimation**
- `CADisplayLink` — timer synchronized with display refresh
  - `preferredFrameRateRange: CAFrameRateRange` — **[NEW in iOS 15, reinforced here]** set minimum/maximum/preferred frame rates
  - `CAFrameRateRange(minimum:maximum:preferred:)` — **[NEW]**
  - `add(to: .current, forMode: .defaultRunLoopMode)` — activate

**CoreLocation**
- `CLLocationManager.stopUpdatingLocation()` — stop location streaming

**AVFoundation**
- `AVAudioEngine.isAutoShutdownEnabled: Bool` — enable idle auto-shutdown; enforced on watchOS
- `AVAudioEngine.stop()` — explicit shutdown when done

**BackgroundTasks**
- `BGProcessingTaskRequest(identifier:)` — **[NEW]** (introduced iOS 13, usage reinforced here)
  - `.requiresExternalPower: Bool`
  - `.requiresNetworkConnectivity: Bool`
- `BGTaskScheduler.shared.submit(_:)` — submit deferred processing task

**Foundation / URLSession**
- `URLSessionConfiguration.background(withIdentifier:)` — background session
  - `.isDiscretionary: Bool` — defer to optimal time
  - `.timeoutIntervalForResource: TimeInterval`
  - `.timeoutIntervalForRequest: TimeInterval`
- `URLSessionTask`
  - `.earliestBeginDate: Date?` — minimum delay before start
  - `.countOfBytesClientExpectsToSend: Int64` — workload hint
  - `.countOfBytesClientExpectsToReceive: Int64` — workload hint

**MetricKit**
- `MXAppRunTimeMetric.cumulativeBackgroundLocationTime: Measurement<UnitDuration>` — total background location time in a diagnostic period

**APNs push payload**
- `apns-priority: 5` — low priority (deferred delivery)
- `apns-priority: 10` — high priority (immediate, wakes device)

**Instruments**
- CoreAnimation FPS instrument — frame rate timeline, identifies offending animations

## Code Highlights

CADisplayLink with frame rate range:
```swift
func createDisplayLink() {
    let displayLink = CADisplayLink(target: self, selector: #selector(step))
    displayLink.preferredFrameRateRange = CAFrameRateRange(minimum: 10, maximum: 60, preferred: 30)
    displayLink.add(to: .current, forMode: .defaultRunLoopMode)
}
```

Discretionary URLSession for deferred download:
```swift
let config = URLSessionConfiguration.background(withIdentifier: "com.app.attachments")
config.isDiscretionary = true
config.timeoutIntervalForResource = 24 * 60 * 60
config.timeoutIntervalForRequest = 60
let session = URLSession(configuration: config, delegate: ..., delegateQueue: ...)

let task = session.downloadTask(with: request)
task.earliestBeginDate = Date(timeIntervalSinceNow: 2 * 60 * 60) // defer 2 hours
task.countOfBytesClientExpectsToSend = 160
task.countOfBytesClientExpectsToReceive = 4096
task.resume()
```

## Takeaways
- Dark Mode adoption on OLED iPhones can reduce display power consumption by up to 70% for content-heavy screens; it should be treated as a power optimization, not just a cosmetic preference.
- A single secondary animation running at a higher frame rate than the primary content forces the entire display to that higher refresh rate — audit with CoreAnimation FPS in Instruments and constrain with `CADisplayLink.preferredFrameRateRange`.
- Always call `stopUpdatingLocation()` when done, and enable `AVAudioEngine.isAutoShutdownEnabled` — background location and audio running idle are leading causes of unexpected battery drain.
- Use `BGProcessingTask`, discretionary `URLSession`, and low-priority APNs pushes (`apns-priority: 5`) to shift non-urgent work to charging/Wi-Fi windows.

---
_Source: WWDC22 Session 10083 page (abstract, chapter summaries, code samples, and resource links)._
