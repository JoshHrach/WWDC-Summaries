# Build Scalable Enterprise App Suites
**WWDC20 · Session 10142** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10142/)

_Platforms:_ iOS 14, iPadOS 14

## Overview
This session uses Apple Retail's suite of enterprise apps — covering store reservations, product catalogs, store layout management, and employee communication — as a case study for building focused, maintainable enterprise applications. The session covers three pillars: architecting for code sharing via App Groups and Swift packages, testing strategy (unit, UI, and performance), and managing deployed apps through configuration-driven user experiences that require no code changes or resubmissions.

Apple Retail's mobile engineering team manages over 500 retail store apps and has structured their codebase around shared Swift packages for authentication, image caching, UIKit component libraries, shared model objects, and barcode/OCR scanning. The shared scanning package — built on Vision and AVFoundation — provides a common API for barcode scanning (PDF417, DataMatrix, QR) and real-time OCR across all apps.

Pınar's portion of the session focuses on the production management layer: client-based configuration files hosted on a server (PLIST→JSON format), server-based feature flags controlled by business teams, configuration versioning for backward compatibility, and forced update policies when server/client schema versions diverge.

## Key Topics

**App Groups for Data Sharing**
App Groups allow multiple apps in the same developer portal to share `UserDefaults` and files through a common container. Enabled in Xcode via Signing and Capabilities → + Capability → App Groups. Once set up, `UserDefaults(suiteName:)` with the app group identifier provides a shared key-value store accessible from all member apps.

**Swift Packages for Shared Code**
Extracting common functionality into Swift packages reduces duplication, accelerates onboarding, and enables a dedicated team to focus on cross-app concerns. Apple Retail's key packages include:
- Authentication (built on `URLSession`, hides auth from call-site engineers)
- Image caching and fetching (disk caching, queue management)
- Shared UIKit components (custom table view cells, product detail views)
- Shared model objects (Customer, Reservation, Product)
- Scanning package (Vision + AVFoundation; barcode + OCR, with supplementary overlay UI)

**Scanning Package Pattern**
The scanning package uses a configuration/initializer pattern: a configuration struct (`BarcodeScanOptions`, `OCRScanOptions`) captures all parameters, a view controller class is initialized with that struct, and a delegate receives success/failure callbacks. This allows easy reuse and configuration sharing across apps while hiding Vision pipeline complexity.

**Testing Strategy**
- Unit tests (XCTest): written by engineers alongside each class/function to verify code paths and return values.
- UI tests (XCUITest): encode manual QA test cases in Swift; prioritized into check-in, daily, and weekly runs to balance speed and coverage.
- Performance tests (Instruments): CPU/GPU profiling for rendering performance (smooth scrolling, drag responsiveness) and battery analysis (critical for devices that run all day on a retail floor).

**Config-Driven User Experience (Client-Based)**
A PLIST file with UI configuration attributes (form fields, feature flags, layout parameters) is hosted on the server in JSON format. On app launch, the app downloads the config and applies it — enabling, disabling, or reconfiguring UI elements without any code changes or new App Store builds. Changes propagate on next app launch.

**Server-Based Feature Flags**
Business and operations teams control feature flags directly on the server. The app fetches these flags on startup via a service API and uses them to show or hide features at the global, market, or store level. Developers are not involved in changing flag values after deployment.

**Backward Compatibility and Forced Updates**
Config versioning tracks which version of server and client config is in use. On startup the app compares versions to determine which UX to show. When a new client attribute has no server mapping (or vice versa), the app requires a forced update — preventing users from getting into an incompatible state.

## APIs & Frameworks

### Foundation
- `UserDefaults(suiteName:)` — shared `UserDefaults` instance for an App Group container
- `UserDefaults.set(_:forKey:)` — writes a value to the shared store
- `UserDefaults.string(forKey:)`, `integer(forKey:)`, `bool(forKey:)` — typed reads from the shared store
- `Bundle.main.url(forResource:withExtension:)` — locates bundled configuration files
- `JSONDecoder` — decodes downloaded server configuration JSON into Swift model types
- `URLSession` — HTTP networking for fetching remote configurations and feature flags

### App Groups (Xcode Capability)
- App Group identifier: `group.com.<yourcompany>.<groupname>` format
- Enabled via: Signing and Capabilities → App Groups capability
- Applies to: all apps sharing the same developer portal and app group identifier

### Vision (in scanning package)
- `VNDetectBarcodesRequest` — detects barcodes in image buffers (supports PDF417, DataMatrix, QR, etc.)
- `VNRecognizeTextRequest` — real-time OCR on camera frames
- `VNImageRequestHandler` — executes Vision requests on a `CVPixelBuffer` or `CIImage`
- `VNBarcodeObservation` — result object with `payloadStringValue`, `symbology`
- `VNRecognizedTextObservation` — result object for OCR, with `topCandidates(_:)`

### AVFoundation (in scanning package)
- `AVCaptureSession` — manages camera input and output pipelines
- `AVCaptureVideoDataOutput` — delivers camera frames as `CMSampleBuffer` for Vision processing
- `AVCaptureDevice` — torch (flash) control via `torchMode`

### XCTest / Testing
- `XCTestCase` — base class for unit tests
- `XCUIApplication` — UI test app driver
- `XCUITest` — framework for UI testing (encodes manual test cases in Swift)
- `XCTMetric`, `XCTPerformanceMetric` — performance testing metrics (CPU, memory, clock time)
- Instruments — profiling for CPU, GPU, memory, battery (Time Profiler, Core Animation, Energy Log)

## Code Highlights

Shared `UserDefaults` via App Group:
```swift
let sharedDefaults = UserDefaults(suiteName: "group.com.apple.myappgroup")!
sharedDefaults.set("My Cool Value", forKey: "MyKeyName")
let myKeyNameValue = sharedDefaults.string(forKey: "MyKeyName")
```

Barcode scanner initialization (shared scanning package):
```swift
import RetailScanner

let scanOptions = BarcodeScanOptions()
scanOptions.scanRegion = .regular
scanOptions.shouldAddSupplementaryView = false
scanOptions.shouldShowBarcodeDetector = true

let barcodeViewController = BarcodeScannerViewController(scanOptions: scanOptions)
barcodeViewController.delegate = self
```

OCR scanner initialization (shared scanning package):
```swift
import RetailScanner

let scanOptions = OCRScanOptions(
    scanRegion: .custom(CGSize(width: 400, height: 100)),
    accessibilityBehavior: .vibrate,
    shouldAddSupplementaryView: true,
    validation: nil,
    shouldShowResultView: true
)
scanOptions.recognitionLevel = .fast

let ocrViewController = OCRScannerViewController(scanOptions: scanOptions)
ocrViewController.delegate = self
```

Config-driven table view cell (client-side pattern):
```swift
func configuredCellForLabel(for customerInfoField: CustomerInfoField,
                             at indexPath: IndexPath) -> UITableViewCell { ... }

func configuredCellForPhoneNumber(for customerInfoField: CustomerInfoField,
                                   at indexPath: IndexPath) -> UITableViewCell { ... }
// Fields driven by PLIST config; removing a field from config removes it from UI on next launch
```

## Takeaways
- Enable App Groups early when building an enterprise app suite — shared `UserDefaults` and file containers cost nothing and provide a clean, system-supported data-sharing mechanism without custom IPC.
- Extract shared concerns into Swift packages as soon as they appear in two apps: authentication, networking, image loading, model objects, and scanning are universally good candidates; dedicated package teams can iterate independently of app teams.
- Use a configuration/initializer pattern for shared view controllers (pass a config struct, receive results via delegate) — this keeps the shared package's API stable while letting each app customize behavior without subclassing.
- Drive UI shape from a server-hosted configuration file rather than code: field visibility, feature enablement, and layout parameters can be changed in production without a code change or App Store review, as long as configuration versioning and backward-compatibility rules are in place.

---
_Source: WWDC20 Session 10142 page (transcript, code samples, and resource links)._
