# Swift Packages: Resources and Localization
**WWDC20 · Session 10169** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10169/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, tvOS 14, watchOS 7

## Overview
Xcode 12 extends Swift packages to support bundled resources—images, asset catalogs, storyboards, Core Data models, and arbitrary files—alongside Swift or C-based source code. Resources are processed at build time into a per-module resource bundle that is automatically embedded in the consuming app bundle, making them available at runtime through Foundation's `Bundle` API without any additional setup.

Swift 5.3 (included in Xcode 12) is the minimum tools version required to use package resources. The `Package.swift` manifest must declare `// swift-tools-version:5.3` and, for any target that contains resources, a `defaultLocalization` parameter. Resources with an unambiguous purpose (asset catalogs `.xcassets`, storyboards, Core Data models) require no manifest declaration; files with ambiguous purposes (plain images, text files, directories) must be declared as either `.process` or `.copy` resources, or listed under `excludes`.

Localization support follows the standard `.lproj` directory convention. Packages declare a `defaultLocalization` language tag, then ship per-language `.lproj` directories containing `Localizable.strings` or `.stringsdict` files and language-specific resource overrides. At runtime, Foundation's bundle APIs match the user's preferred language automatically, and SwiftUI previews can be overridden with `.environment(\.locale, …)` to validate non-default localizations without leaving Xcode.

## Key Topics
- **Adding resources to a package target** — placing files next to source code and configuring the manifest
- **Automatic vs. explicit resource rules** — which file types are handled implicitly (`.xcassets`, `.storyboard`, `.xcdatamodel`) vs. those requiring `.process` or `.copy` rules
- **`.process` vs. `.copy` actions** — `.process` applies platform-appropriate transformations; `.copy` preserves exact directory structure
- **`excludes` list** — marking development-only files (notes, design sketches) so they are ignored at build time
- **Synthesized `Bundle.module` accessor** — Xcode generates a `module` static property on `Bundle` giving the target's resource bundle in both Swift and Objective-C
- **Passing the bundle to system APIs** — any API that accepts a `Bundle` parameter (e.g., `UIImage(named:in:compatibleWith:)`) works with `.module`
- **Localization with `.lproj` directories** — creating per-language string files and per-language resource variants inside the package
- **`defaultLocalization` manifest key** — required for packages with any localized content; specifies the fallback language

## APIs & Frameworks

**PackageDescription (Package.swift)**
- `// swift-tools-version:5.3` **[NEW]** — minimum tools version for resource support
- `Package(name:defaultLocalization:products:targets:)` **[NEW]** — `defaultLocalization` parameter added
- `Target.target(name:dependencies:path:exclude:sources:resources:)` **[NEW]** — `resources` and `excludes` parameters added
- `.process(_ path:localization:)` **[NEW]** — resource rule that applies platform-appropriate processing
- `.copy(_ path:)` **[NEW]** — resource rule that copies file/directory verbatim, preserving structure

**Foundation**
- `Bundle.module` **[NEW]** — synthesized static accessor returning the resource bundle for the current Swift package module
- `Bundle(for:)` / `Bundle.main` — existing bundle accessors; `Bundle.module` complements these for packages
- `Bundle.url(forResource:withExtension:)` — standard API to locate files in the resource bundle
- `Bundle.path(forResource:ofType:)` — Objective-C-compatible path lookup

**SwiftUI**
- `Image(_:bundle:)` — loads an image asset from a specified bundle (use `.module`)
- `Text(_:bundle:)` — loads a localized string from a specified bundle
- `.environment(\.locale, Locale(identifier:))` — preview modifier for testing localizations

**Xcode 12 tooling**
- SwiftUI previews in packages **[NEW]** — previews work without a client app in the workspace
- Synthesized `Bundle.module` in both Swift and Objective-C code modules **[NEW]**

## Code Highlights

Package manifest declaring resources and exclusions:
```swift
// swift-tools-version:5.3
import PackageDescription

let package = Package(name: "MyGame",
    products: [
        .library(name: "GameLogic", targets: ["GameLogic"])
    ],
    targets: [
        .target(name: "GameLogic",
            excludes: [
                "Internal Notes.txt",
                "Artwork Creation"],
            resources: [
                .process("Logo.png"),
                .copy("Game Data")]
        )
    ]
)
```

Accessing the synthesized resource bundle in Swift:
```swift
// In a SwiftUI view inside the package
Image("dice-1", bundle: .module)
Text("roll", bundle: .module)
```

Accessing resources in Objective-C:
```objc
NSBundle *bundle = SWIFTMODULE_MODULE_BUNDLE; // synthesized accessor
UIImage *img = [UIImage imageNamed:@"dice-1" inBundle:bundle compatibleWithTraitCollection:nil];
```

## Takeaways
- Resources in Swift packages require Swift 5.3 / Xcode 12; set the tools version in `Package.swift` and use `.process` for most resources, `.copy` only when directory structure must be preserved verbatim.
- Xcode synthesizes a `Bundle.module` static accessor for every package target that contains resources; use it wherever a `Bundle` parameter is accepted.
- Localization uses the existing `.lproj` convention; declare `defaultLocalization` in the manifest and place per-language `Localizable.strings` files in the appropriate `.lproj` directories.
- SwiftUI previews now work inside packages without a host app, and locale environment overrides let you validate localizations directly in the canvas.

---
_Source: WWDC20 Session 10169 page (abstract, chapter summaries, code samples, and resource links)._
