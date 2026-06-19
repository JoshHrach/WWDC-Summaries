# Adopting Swift Packages in Xcode
**WWDC19 · Session 408** · [Watch](https://developer.apple.com/videos/play/wwdc2019/408/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, watchOS 6, tvOS 13

## Overview
Xcode 11 introduces native integration of the Swift Package Manager directly into Xcode projects, enabling developers to use community-developed and internally shared packages when building apps for all Apple platforms. This session provides a practical walkthrough of adding a third-party package (Yams, a YAML parser) to a SwiftUI app via the Xcode UI, then dives deep into how packages are structured, how version resolution works, and how to keep packages up to date.

The session also covers an advanced scenario: diagnosing and resolving a package version conflict caused by incompatible transitive dependencies. The key mental model is to always look at the entire dependency graph — direct and indirect — rather than a single package requirement in isolation.

## Key Topics

### Adding a Package Dependency (Xcode UI)
- File > Swift Packages > Add Package Dependency opens a search/URL dialog.
- GitHub accounts linked in Xcode Preferences surface all repositories and starred repos.
- After entering a URL or selecting a repo, Xcode presents available versions and a Version Rule picker.
- Default rule: "Up to Next Major Version" — starts from the latest release, excludes the next major bump.
- After resolving, select the package's **Products** (libraries) to link into target(s).
- The package appears in both the **Swift Packages** tab of the Project Editor and the **Swift Package Dependencies** section of the Navigator.
- Full code completion and Quick Help (from documentation comments in the package source) work immediately after `import`.

### Package Structure
- A Swift Package is a directory containing `Package.swift` (the manifest), `Sources/`, and `Tests/`.
- `Sources/<TargetName>/` — one subdirectory per buildable target; can contain Swift, C, Objective-C, or Objective-C++ files.
- `Tests/<TestSuiteName>/` — one subdirectory per test suite.
- `Package.swift` sections: `swift-tools-version`, `import PackageDescription`, `name`, `products`, `targets`, and optionally `dependencies`, `swiftLanguageVersions`.
- **Products** are what the package vends to clients; a library product exposes one or more targets.
- **Targets** are the individually buildable units; targets can declare dependencies on other targets within the same package.
- Package libraries are **static by default** in Xcode 11 — all code is linked into the app binary.
- Xcode recompiles package source for each consuming app/target (platform, architecture, etc.).

### Semantic Versioning
- **Major** version: incremented on breaking API changes — clients may need source modifications.
- **Minor** version: incremented when new functionality is added in a backward-compatible way.
- **Patch** version: incremented for backward-compatible bug fixes only.
- Version rules in Xcode: "Up to Next Major" (most common), "Up to Next Minor" (conservative), "Exact", "Branch", "Commit".

### Package Resolution
- Resolution selects one version per package that satisfies all constraints in the dependency graph (direct + transitive).
- Only one version of any given package can exist in a workspace.
- The **Swift Packages tab** in Project Editor shows direct project→package dependencies.
- Transitive dependencies are visible by browsing the `Package.swift` of dependent packages in the Navigator.
- **Package.resolved** (inside `.xcodeproj/project.xcworkspace/xcshareddata/swiftpm/`) records the exact resolved version of every package.
- Check in `Package.resolved` so all team members get identical versions at the same commit.

### Updating Packages
- File > Swift Packages > Update to Latest Package Versions re-resolves and fetches newer versions within constraints.
- After updating, commit and push `Package.resolved` to share the update with the team.
- Updating across a major version boundary may introduce source-breaking API changes — be prepared to adapt.

### Resolving Version Conflicts
- Conflicts occur when two requirements on the same package cannot simultaneously be satisfied by a single version.
- Debugging approach: look at the full dependency graph, not just the new requirement.
- Common fix: update the version rule of an intermediate dependency so its sub-dependency constraint is compatible with your new requirement.
- Changing a package's version rule is done by selecting it in the Swift Packages tab and editing the rule.

## APIs & Frameworks

**Swift Package Manager / PackageDescription** (all **[NEW]** Xcode 11 integration)
- `Package.swift` manifest — `Package(name:products:dependencies:targets:swiftLanguageVersions:)`
- `Product.library(name:targets:)` — vend a library product to clients
- `Target.target(name:dependencies:)` — declare a buildable target
- `Target.testTarget(name:dependencies:)` — declare a test target
- `.package(url:from:)` / `.package(url:.upToNextMajor(from:))` / `.package(url:.upToNextMinor(from:))` / `.package(url:.exact(_:))` — dependency declaration in `Package.swift`
- `Package.resolved` file — records exact resolved versions for reproducibility
- Xcode Project Editor **Swift Packages tab** **[NEW]** — add/remove/edit package dependencies
- Xcode Navigator **Swift Package Dependencies** section **[NEW]** — browse package source and manifests
- File > Swift Packages > **Add Package Dependency** **[NEW]**
- File > Swift Packages > **Update to Latest Package Versions** **[NEW]**
- File > Swift Packages > **Reset Package Caches** **[NEW]**

## Code Highlights

```swift
// Package.swift example (simplified from Yams)
// swift-tools-version:5.0
import PackageDescription

let package = Package(
    name: "Yams",
    products: [
        .library(name: "Yams", targets: ["Yams"])
    ],
    targets: [
        .target(name: "CYaml"),
        .target(name: "Yams", dependencies: ["CYaml"]),
        .testTarget(name: "YamsTests", dependencies: ["Yams"])
    ],
    swiftLanguageVersions: [.v4, .v4_2, .v5]
)

// App code: using the package after adding it
import Yams

let yamlString = try String(contentsOf: fileURL)
let decoder = YAMLDecoder()
let menu = try decoder.decode(Menu.self, from: yamlString)
```

## Takeaways
- Xcode 11's native SPM integration means packages are a first-class dependency management tool for all Apple platform apps — no third-party tools required.
- Always check in `Package.resolved` — omitting it leads to non-reproducible builds across the team.
- The "Up to Next Major Version" rule is the right default: it automatically picks up bug fixes and minor additions while protecting against breaking changes.
- When debugging resolution conflicts, draw out the full dependency graph; the fix is usually updating a transitive dependency to a newer major version that aligns requirements.

---
_Source: WWDC19 Session 408 page (abstract, chapter summaries, code samples, and resource links)._
