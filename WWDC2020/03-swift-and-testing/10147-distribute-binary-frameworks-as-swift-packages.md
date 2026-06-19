# Distribute Binary Frameworks as Swift Packages
**WWDC20 · Session 10147** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10147/)

_Platforms:_ iOS, iPadOS, macOS, watchOS, tvOS (Xcode 12 / Swift Package Manager)

## Overview
Xcode 12 extends Swift Package Manager with support for binary dependencies via a new `binaryTarget` target type in `Package.swift`. Previously, Swift packages could only distribute source code; closed-source library authors had to rely on manual XCFramework distribution or CocoaPods. Now, XCFrameworks can be wrapped in a Swift package and consumed identically to source-based packages — through File > Swift Packages > Add Package Dependency or a `Package.swift` dependency declaration.

The session covers three workflows: using a binary dependency as a consumer, authoring a Swift package that distributes an XCFramework, and creating XCFrameworks with Xcode. Integrity is ensured via a SHA-256 checksum in the package manifest; Xcode verifies the checksum at download time and rejects mismatches.

## Key Topics

### Consuming Binary Dependencies
- Add via Xcode UI (File → Swift Packages → Add Package Dependency) just like source packages.
- Specify repository URL and version requirement; Xcode downloads and links the XCFramework.
- The XCFramework appears under "Referenced Binaries" in the project navigator.
- Binary targets are handled identically to XCFrameworks added directly to an app.
- The dependency entry in `Package.swift` uses the same `.package(url:from:)` syntax as source packages.

### Authoring a Binary Swift Package
- Requires `swift-tools-version:5.3` **[NEW]** or later.
- New target type: `.binaryTarget(name:url:checksum:)` **[NEW]**
  - `name` — must match the XCFramework's module name.
  - `url` — HTTPS URL to a `.zip` containing the `.xcframework`.
  - `checksum` — SHA-256 hash of the zip archive; computed with `swift package compute-checksum <archive>`.
- Alternative: `.binaryTarget(name:path:)` — local path; intended for development only (not for distribution since large binaries should not be committed to Git).
- Products reference binary targets just like source targets: `.library(name:targets:)`.
- The binary artifact is downloaded separately from the Git checkout — repository history stays clean.

### Versioning and Naming Rules
- Each binary framework should have one canonical package; do not bundle other authors' XCFrameworks.
- Names must be unique across the dependency graph.
- Follow semantic versioning: breaking API changes (renamed types/methods) → major version bump.
- Version the XCFramework itself via the `CFBundleShortVersionString` / bundle version string in the framework's `Info.plist`.

### Creating XCFrameworks (Xcode 12)
1. Enable "Build Libraries for Distribution" build setting on the framework/library target.
2. Archive each platform/architecture variant: `xcodebuild archive ...`
3. Bundle variants: `xcodebuild -create-xcframework ...`
- Each XCFramework contains one module; it can bundle frameworks or dynamic/static libraries.
- Platform variants are organized by target triple subdirectories inside the `.xcframework` bundle.

### Trade-offs of Binary Dependencies
- Debugging is harder (no source available).
- Cannot make your own fixes to the binary.
- Platform support is limited to what the author builds and includes.

## APIs & Frameworks

### Swift Package Manager (Xcode 12 / swift-tools-version:5.3)
- `Target.binaryTarget(name:url:checksum:)` **[NEW]** — HTTPS binary target
- `Target.binaryTarget(name:path:)` **[NEW]** — local path binary target (development only)
- `swift package compute-checksum <path-to-zip>` — CLI command to generate SHA-256 checksum
- `swift-tools-version:5.3` — minimum tools version required for binary targets
- `Product.library(name:targets:)` — expose a binary target as a library product (unchanged syntax)
- `.package(url:from:)` — consumer dependency declaration (unchanged syntax)
- XCFramework format (introduced in Xcode 11) — see WWDC19 "Binary Frameworks in Swift"

## Code Highlights

Package manifest declaring a binary target:
```swift
// swift-tools-version:5.3
import PackageDescription

let package = Package(
    name: "Emoji",
    products: [
        .library(name: "Emoji", targets: ["Emoji"])
    ],
    targets: [
        .binaryTarget(
            name: "Emoji",
            url: "https://example.com/Emoji/Emoji-1.0.0.xcframework.zip",
            checksum: "6d988a1a27418674b4d7c31732f6d60e60734ceb11a0ce9b54d1871918d9c194"
        )
    ]
)
```

Computing the checksum from the command line:
```bash
swift package compute-checksum Emoji-1.0.0.xcframework.zip
```

Consumer app's Package.swift adding the binary dependency:
```swift
dependencies: [
    .package(url: "https://github.com/JohnnyAppleseed2020/BinaryEmoji", from: "1.0.0"),
],
targets: [
    .target(name: "package", dependencies: ["Emoji"])
]
```

## Takeaways
- Xcode 12 adds `.binaryTarget` to Swift Package Manager, enabling closed-source XCFramework distribution via the same familiar package workflow.
- Use `swift package compute-checksum` to generate the required SHA-256 checksum; Xcode verifies it at download time to guarantee binary integrity.
- Binary targets require `swift-tools-version:5.3`; use a local `path`-based target during development and switch to HTTPS + checksum for distribution.
- Apply semantic versioning to both the Swift package version and the XCFramework's bundle version string; never bundle other authors' frameworks in your own package.

---
_Source: WWDC20 Session 10147 page (abstract, transcript, code samples, and resource links)._
