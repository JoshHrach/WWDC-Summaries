# Getting to Know Swift Package Manager
**WWDC18 · Session 411** · [Watch](https://developer.apple.com/videos/play/wwdc2018/411/)

_Platforms:_ macOS, Linux (cross-platform Swift)

## Overview
This session introduces Swift Package Manager (SwiftPM) — its motivation, core concepts, command-line workflow, design philosophy, and future roadmap. Presenters Rick Ballard and Boris Buegling cover why Apple created a first-party package manager alongside Swift, walk through the three building blocks (dependencies, targets, products), demonstrate the full development workflow (init → build → test → edit mode), and outline aspirational features including resources, build settings, custom build tools, and a package index.

SwiftPM is itself a Swift package, open-source, and evolving via the Swift Evolution process on the Swift forums and bugs.swift.org.

## Key Topics

### Why Swift Package Manager

- Swift is cross-platform; a canonical cross-platform package manager enables consistent builds on macOS and Linux.
- A shared package manager creates a standard distribution channel: the ecosystem grows organically as great ideas ship as packages rather than waiting for inclusion in core libraries.
- Written in Swift — developers use the same language for both their packages and the manifest file.
- Uses llbuild (also used by Xcode's new build system) for fast, correct, parallelizable incremental builds.

### Core Concepts: Dependencies, Targets, Products

- **Package** — a directory with a `Package.swift` manifest at the root. The directory name is the default package name.
- **Dependencies** — other Swift packages, declared with a URL (git remote or local path) and a version requirement. SwiftPM resolves all transitive dependencies recursively and records the result in `Package.resolved`.
- **Targets** — the basic build units. Each target maps to a folder under `Sources/` or `Tests/`. A target compiles to a module or test suite. Targets can depend on other targets in the same package or on products from dependency packages.
- **Products** — declared outputs: `.library` (static or dynamic, or auto) or `.executable`. Products are what other packages consume.
- Sources are picked up automatically from the convention-based directory structure — no need to enumerate files in the manifest.

### Version Requirements and Dependency Resolution

- SwiftPM uses **Semantic Versioning**: major = breaking change; minor = backwards-compatible addition; patch = backwards-compatible bug fix.
- Version requirement syntaxes: `.exact("1.2.3")`, `.from("1.0.0")` (allows minor+patch updates), `.upToNextMinor(from:)` (patch only), `.upToNextMajor(from:)` (minor+patch).
- Resolution finds the latest version compatible with all constraints in the full graph.
- `Package.resolved` records exact resolved versions — commit this file so teammates and CI get identical builds. Run `swift package update` deliberately to upgrade.
- Branch and local-path dependencies are available for active co-development (development only; not for tagged releases).

### Command-Line Workflow

- `swift package init --type executable` — scaffold an executable package.
- `swift package init --type library` — scaffold a library package.
- `swift build` — build all targets.
- `swift run [target]` — build and run an executable target.
- `swift test [--parallel] [--filter <pattern>]` — run XCTest-based tests; `--parallel` runs tests concurrently for faster results; `--filter` runs a subset.
- `swift package update` — update dependencies to latest compatible versions, regenerate `Package.resolved`.
- `swift package edit <package>` — check out a dependency for local editing (overrides all transitive uses).

### Package.swift Manifest Design

- Uses the Swift language and SwiftPM's `PackageDescription` API.
- First line must be `// swift-tools-version:X.Y` — sets minimum required SwiftPM version and which `PackageDescription` API version is used.
- Follow Apple's API Design Guidelines in the manifest; prefer **declarative** syntax (constants, no side effects) so automated tools (e.g., libSyntax-based editors) can reliably parse and modify it.
- `swift-language-versions` — declare which Swift language versions the package's source supports; can be a list for multi-version compatibility using compiler directives.

### Build Isolation and Security

- SwiftPM does **not** allow arbitrary shell scripts or commands during the build — inputs/outputs must be fully declared so the build graph is statically knowable.
- Build sandbox (macOS) prevents manifest evaluation and build steps from writing to arbitrary filesystem locations or accessing the network.
- Packages that are not declared dependencies are not visible to your build.

### Future Directions (Aspirational)

- **Resources** — bundle images, data files, and other assets with products; would use Foundation's cross-platform resource API added in Spring 2018.
- **Build settings** — per-target compiler flags, linker flags, conditional settings.
- **Custom build tools** — safely-declared build plugins with explicit input/output dependencies.
- **Manifest editing API** — automated addition of dependencies and targets via libSyntax without requiring manual edits.
- **Semantic version analysis** — SwiftPM could detect API-breaking changes and suggest major-version bumps.
- **Package index** — a canonical namespace for discovery, quality metrics (test coverage), and trust evaluation.
- **Cross-platform sandboxing** — extend build sandbox security to non-macOS platforms.
- **Package mirroring / forking** — redirect dependency URLs to private mirrors.

## APIs & Frameworks

**PackageDescription (Package.swift API)**
- `Package(name:platforms:products:dependencies:targets:swiftLanguageVersions:)` — top-level manifest type
- `Target.target(name:dependencies:path:exclude:sources:publicHeadersPath:)` — library/executable target
- `Target.testTarget(name:dependencies:path:exclude:sources:)` — test target
- `Target.Dependency.product(name:package:)` — depend on a specific product from a dependency package
- `Target.Dependency.target(name:)` — depend on another target in the same package
- `Package.Dependency.package(url:from:)` — from version (minor+patch updates allowed)
- `Package.Dependency.package(url:upToNextMajor:)` — major-compatible range
- `Package.Dependency.package(url:upToNextMinor:)` — minor-compatible range (patch only)
- `Package.Dependency.package(url:exact:)` — pin to exact version
- `Package.Dependency.package(path:)` — local path dependency (dev only)
- `Product.library(name:type:targets:)` — library product; `type`: `.static`, `.dynamic`, or omit for auto
- `Product.executable(name:targets:)` — executable product
- `SwiftVersion` — `.v4`, `.v4_2`, `.v5` etc. for `swiftLanguageVersions`

**SwiftPM CLI**
- `swift package init [--type library|executable|system-module|manifest]`
- `swift build [-c release]`
- `swift run [--] [arguments]`
- `swift test [--parallel] [--filter <regex>]`
- `swift package update`
- `swift package resolve`
- `swift package edit <PackageName>`
- `swift package unedit <PackageName>`
- `swift package show-dependencies`
- `swift package generate-xcodeproj` (available in 2018 toolchain)

## Code Highlights

Minimal `Package.swift` for a command-line tool with a library dependency:
```swift
// swift-tools-version:4.2
import PackageDescription

let package = Package(
    name: "Dealer",
    products: [
        .library(name: "LibDealer", targets: ["LibDealer"]),
        .executable(name: "dealer", targets: ["dealer"]),
    ],
    dependencies: [
        .package(url: "https://github.com/nvzqz/RandomKit", from: "5.0.0"),
    ],
    targets: [
        .target(name: "LibDealer", dependencies: [
            .product(name: "RandomKit", package: "RandomKit"),
        ]),
        .target(name: "dealer", dependencies: ["LibDealer"]),
        .testTarget(name: "DealerTests", dependencies: ["LibDealer", "dealer"]),
    ]
)
```

Scaffolding and running a new package:
```bash
mkdir HelloWorld && cd HelloWorld
swift package init --type executable
swift run
# => Hello, world!
```

Running tests in parallel with a filter:
```bash
swift test --parallel --filter NeoTests.ConnectionTests
```

## Takeaways
- SwiftPM's three primitives — dependencies (packages you consume), targets (how you build), and products (what you expose) — cover the vast majority of real-world package structures without extra configuration.
- Always commit `Package.resolved` so your team and CI reproduce the exact same dependency graph; upgrade intentionally with `swift package update`.
- Prefer declarative syntax in `Package.swift` (no computed names, no side effects) to make the manifest amenable to automated tooling and easy auditing.
- Build sandboxing (no arbitrary shell scripts, no arbitrary file writes) is a feature, not a limitation — it enables fully correct incremental builds with a statically known dependency graph.

---
_Source: WWDC18 Session 411 page (abstract, full transcript, and resource links)._
