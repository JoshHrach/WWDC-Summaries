# Creating Swift Packages
**WWDC19 · Session 410** · [Watch](https://developer.apple.com/videos/play/wwdc2019/410/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, watchOS 6, tvOS 13

## Overview
This session teaches developers how to author their own Swift packages — both local packages for code organization within a workspace and published packages for sharing with a team or the open-source community. It introduces first-class Swift Package Manager support in Xcode 11, enabling a workflow that goes from creating a package to tagging and pushing a semantic version on GitHub, all without leaving Xcode.

The session also covers the full `Package.swift` manifest API: targets, products, dependencies, version requirements, C/Swift target configuration, platform deployment targets, and Swift tools version semantics. It closes with an overview of the Swift PM open-source project, the `libSwiftPM` library, and the community evolution process.

## Key Topics

- **Local packages** — Created via File > New > Swift Package; function like subprojects in a workspace; platform-independent by default (build for whatever client platform needs them); not versioned until published.
- **Publishing a package** — Drag package out of project, flesh out README, write tests, create a local Git repository, push to GitHub, and tag a semantic version — all from within Xcode source control UI.
- **Semantic versioning** — Major (breaking changes), minor (additive, backwards-compatible), patch (bug fixes). Major version 0 for initial development. Pre-release identifiers for beta testing.
- **Package manifest (`Package.swift`) structure** — Swift tools version comment, `import PackageDescription`, single `Package(...)` initializer with `name`, `targets`, `products`, `dependencies`, `platforms`.
- **Standard directory layout** — `Sources/<TargetName>/` for library targets; `Tests/<TargetName>/` for test targets; Xcode auto-discovers source files, no need to list them individually.
- **Products** — `library(name:targets:)` exports targets to dependents; `library(name:type:.dynamic,targets:)` for dynamic linking.
- **Target configuration** — `target(name:dependencies:path:sources:publicHeadersPath:cSettings:swiftSettings:linkerSettings:)` for fine-grained control; `cSettings: [.define("MACRO")]` for C macros.
- **Version requirements** — `.upToNextMajor(from:)` (recommended), `.upToNextMinor(from:)`, `.exact(_:)`, `.branch(_:)`, `.revision(_:)`; branch/revision not allowed in published packages.
- **Platform deployment targets** — `platforms: [.macOS(.v10_15), .iOS(.v13)]` in the manifest; does not restrict platforms, only sets minimum deployment target.
- **Editing dependencies** — Add a local checkout of a package dependency to override the remote version (match on last path component); enables editing app and package simultaneously.
- **Swift PM open source** — `libSwiftPM` library; SourceKit-LSP for Language Server Protocol support; community proposals for resources (images, data files) in progress.

## APIs & Frameworks

### Swift Package Manager (`PackageDescription` module)
- `Package(name:platforms:products:dependencies:targets:swiftLanguageVersions:)` **[NEW Xcode 11 integration]**
- `Target.target(name:dependencies:path:exclude:sources:publicHeadersPath:cSettings:swiftSettings:linkerSettings:)` **[NEW]**
- `Target.testTarget(name:dependencies:path:exclude:sources:)` **[NEW]**
- `Product.library(name:type:targets:)` — exports targets; `type` can be `.static` or `.dynamic`
- `Package.Dependency.package(url:from:)` — shorthand for `.upToNextMajor(from:)`
- `Package.Dependency.package(url:_:)` — accepts `Range<Version>` or `ClosedRange<Version>`
- `.upToNextMajor(from:)` — recommended version requirement
- `.upToNextMinor(from:)` — conservative version requirement
- `.exact(_:)` — pins to specific version
- `.branch(_:)` — branch-based (not for published packages)
- `.revision(_:)` — revision-based (not for published packages)
- `CSetting.define(_:_:)` — define C preprocessor macro
- `SwiftSetting.define(_:_:)` — define Swift compilation condition
- `platforms: [.macOS(.v10_15), .iOS(.v13), .watchOS(.v6), .tvOS(.v13)]`
- Swift tools version comment: `// swift-tools-version:5.1`

### Swift PM Command-Line Tools
- `swift build` — build the package
- `swift run` — run an executable product
- `swift test` — run tests
- `swift package` — package management operations (init, resolve, update, generate-xcodeproj, etc.)

### Xcode 11 Integration
- File > New > Swift Package — create a local package **[NEW]**
- File > Swift Packages > Add Package Dependency — add a remote package **[NEW]**
- Source Control menu — Create Repositories, Create Remote (GitHub), Tag, Push with tags **[NEW]**
- Swift Package Dependencies section in Project Navigator **[NEW]**
- Frameworks, Libraries, and Embedded Content phase — link package library products

### SourceKit-LSP
- Language Server Protocol implementation for Swift and C-based languages built on `libSwiftPM` **[NEW]**

## Code Highlights

Minimal `Package.swift` with two targets and one library product:

```swift
// swift-tools-version:5.1
import PackageDescription

let package = Package(
    name: "FoodAndStuff",
    products: [
        .library(name: "FoodAndStuff", targets: ["FoodAndStuff"])
    ],
    targets: [
        .target(name: "FoodAndStuff"),
        .testTarget(name: "FoodAndStuffTests", dependencies: ["FoodAndStuff"])
    ]
)
```

Package with a C target, custom paths, and a Swift bridge:

```swift
let package = Package(
    name: "MenuDownloader",
    products: [
        .library(name: "MenuDownloader", targets: ["MenuDownloaderSwift"]),
        .library(name: "MenuDownloaderC", type: .dynamic, targets: ["MenuDownloaderC"])
    ],
    dependencies: [
        .package(url: "https://github.com/jpsim/Yams.git", from: "2.0.0")
    ],
    targets: [
        .target(name: "MenuDownloaderC",
                path: "CLegacy",
                cSettings: [.define("LOAD_SECRET_MENU")]),
        .target(name: "MenuDownloaderSwift",
                path: "Swift",
                dependencies: ["MenuDownloaderC", "Yams"])
    ]
)
```

## Takeaways

- Use local packages to refactor shared code in a workspace without adding versioning overhead; they build for all client platforms automatically with no extra configuration.
- Prefer `.upToNextMajor(from:)` for version requirements — it balances stability with flexibility and reduces package graph conflicts.
- Overriding a remote dependency with a local checkout (same last path component) is the cleanest way to develop a package and consuming app simultaneously without changing your dependency list.
- Swift PM's open-source `libSwiftPM` and the evolution process mean features like package resources are coming through community proposals — watch and participate at swift.org.

---
_Source: WWDC19 Session 410 page (abstract, full transcript, and resource links)._
