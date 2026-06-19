# Binary Frameworks in Swift
**WWDC19 · Session 416** · [Watch](https://developer.apple.com/videos/play/wwdc2019/416/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, watchOS 6, tvOS 13

## Overview
Xcode 11 introduces full support for creating and distributing binary Swift frameworks via the new XCFramework bundle format. An XCFramework can bundle variants for multiple platforms and architectures (device, Simulator, AppKit-based Mac, Catalyst Mac) into a single distributable package, solving the long-standing problem of having to ship separate fat frameworks per destination.

The session also explains Swift Module Interfaces — a new textual module format that replaces the binary compiled module format — which enables binary frameworks to remain importable across future Swift compiler versions. The second half covers how framework authors should design their APIs to maintain binary compatibility across releases and how to trade flexibility for client-side performance using `@inlinable`, `@frozen` enums, and `@frozen` structs when justified by profiling.

## Key Topics

**XCFramework Bundle Format**
- Single bundle contains variants for any combination of platforms (iOS device, iOS Simulator, macOS AppKit, macOS Catalyst/UIKit)
- Supports frameworks and static libraries with headers
- Xcode sets up client search paths automatically on drag-and-drop
- Generated interface viewable via Jump to Definition in Xcode

**Building an XCFramework**
1. Enable `BUILD_LIBRARIES_FOR_DISTRIBUTION = YES` build setting
2. Run `xcodebuild archive` per destination (device, simulator, Mac) with `SKIP_INSTALL=NO`
3. Run `xcodebuild -create-xcframework` referencing all per-destination `.framework` paths

**Swift Module Interfaces (`.swiftinterface`)**
- New textual format replacing the binary compiled module (`.swiftmodule`)
- Lists all public APIs (types, methods, conformances, synthesized members, deinits) in source-like text
- Future compiler versions can import interfaces from older versions — eliminates version lock
- Generated automatically when `BUILD_LIBRARIES_FOR_DISTRIBUTION = YES`

**Semantic Versioning for Frameworks**
- Patch version: bug fixes, private implementation changes
- Minor version: additive public API (new methods, new enum cases, new struct stored properties, new conformances)
- Major version: source- or binary-breaking changes (changed function signature, case added to frozen enum, etc.)
- Communicated via `CFBundleShortVersionString` in framework's `Info.plist`

**ABI Flexibility vs. Optimizability**
- By default `BUILD_LIBRARIES_FOR_DISTRIBUTION` maximizes flexibility (resilience): clients can handle new cases, new stored properties, new methods without recompiling
- `@inlinable` — copies function body into module interface; enables client-side inlining/optimization; body becomes part of public API and must not change observable behavior
- `@usableFromInline` — marks internal declarations accessible from `@inlinable` code without making them public
- `@frozen` enum — promises no new cases; clients skip default case and runtime size negotiation; binary-breaking to add cases afterward
- `@frozen` struct — promises stored properties won't change; clients can manipulate properties directly; requires all stored properties to be public or `@usableFromInline`

**Framework Author Responsibilities**
- Trust and security: framework shares app's entitlements and privacy permissions
- Dependencies: must also be built with `BUILD_LIBRARIES_FOR_DISTRIBUTION`; binary frameworks cannot depend on Swift Packages
- Minimize entitlements and dependencies; handle permission denial gracefully
- Objective-C compatibility: disable `SWIFT_INSTALL_OBJC_HEADER` and `DEFINES_MODULE` if no Obj-C API is published

## APIs & Frameworks

### Swift Language Attributes (some NEW)
- `@inlinable` **[NEW effective for binary distribution]** — makes function body part of module interface
- `@usableFromInline` **[NEW]** — exposes internal declaration to `@inlinable` code
- `@frozen` (enum) **[NEW]** — seals enum cases; eliminates resilience overhead; breaking to change
- `@frozen` (struct) **[NEW]** — seals stored property layout; enables in-place client access; breaking to change
- `@unknown default` in switch — handles exhaustive switching over non-frozen enums

### Xcode 11 Build Settings (NEW)
- `BUILD_LIBRARIES_FOR_DISTRIBUTION` **[NEW]** — activates module interface generation, resilience mode
- `SKIP_INSTALL` — set to `NO` when archiving frameworks for XCFramework creation
- `SWIFT_INSTALL_OBJC_HEADER` — disable to suppress generated Obj-C header
- `DEFINES_MODULE` — disable to suppress Obj-C module map

### xcodebuild Commands (NEW)
- `xcodebuild archive -scheme <name> -destination <...> SKIP_INSTALL=NO` **[NEW workflow]** — archive per-platform
- `xcodebuild -create-xcframework -framework <path> ... -output <path>.xcframework` **[NEW]** — bundle variants

### File Formats (NEW)
- `.xcframework` bundle **[NEW]** — multi-platform binary framework bundle
- `.swiftinterface` **[NEW]** — textual Swift module interface file

### Xcode UI
- Jump to Definition on a framework name → shows generated interface from `.swiftinterface`
- Frameworks, Libraries, and Embedded Content section in target General tab — drag-and-drop XCFramework

## Code Highlights

Building an XCFramework (shell):
```bash
# Archive for each destination
xcodebuild archive -scheme FlightKit \
    -destination "generic/platform=iOS" \
    -archivePath FlightKit-iOS.xcarchive \
    SKIP_INSTALL=NO BUILD_LIBRARIES_FOR_DISTRIBUTION=YES

xcodebuild archive -scheme FlightKit \
    -destination "generic/platform=iOS Simulator" \
    -archivePath FlightKit-iOS_Sim.xcarchive \
    SKIP_INSTALL=NO BUILD_LIBRARIES_FOR_DISTRIBUTION=YES

# Create XCFramework
xcodebuild -create-xcframework \
    -framework FlightKit-iOS.xcarchive/Products/Library/Frameworks/FlightKit.framework \
    -framework FlightKit-iOS_Sim.xcarchive/Products/Library/Frameworks/FlightKit.framework \
    -output FlightKit.xcframework
```

Using `@frozen` and `@inlinable`:
```swift
@frozen public enum FlightPlan {
    case oneWay
    case roundTrip
    // Adding cases now is binary-breaking
}

@inlinable public func canCarry(_ cargo: Cargo) -> Bool {
    // Body becomes public API; do not change observable behavior
    return capacity >= cargo.weight
}

@usableFromInline internal var capacity: Int
```

## Takeaways
- XCFramework is the only supported way to distribute binary Swift frameworks in Xcode 11+; replace fat frameworks immediately.
- Enabling `BUILD_LIBRARIES_FOR_DISTRIBUTION` is mandatory — it generates the `.swiftinterface` file that makes the framework importable by future Swift compiler versions.
- Default resilience is the right choice for most APIs; only reach for `@frozen` or `@inlinable` after profiling proves a performance need, because they are binary-breaking to undo.
- Binary frameworks cannot depend on Swift Packages; all dependencies must themselves be built with `BUILD_LIBRARIES_FOR_DISTRIBUTION`.

---
_Source: WWDC19 Session 416 page (abstract, full transcript, and resource links)._
