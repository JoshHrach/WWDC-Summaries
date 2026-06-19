# What's new in Swift-DocC
**WWDC22 · Session 110368** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110368/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, tvOS 16, watchOS 9

## Overview
Swift-DocC received major updates in Xcode 14, expanding its documentation authoring capabilities beyond Swift frameworks to include app projects and Objective-C/C APIs. Developers can now write documentation inline for any project type, organize content with Documentation Catalogs, and benefit from DocC's Markdown-based toolchain across all target types.

Publishing documentation to the web became dramatically simpler: the DocC archive produced by Xcode 14 is directly compatible with static hosting services out of the box, including GitHub Pages, with only a single build setting required for non-root base paths. A new Swift Package Manager plug-in streamlines automated documentation builds for Swift packages.

The web browsing experience was overhauled with a new navigation sidebar that lets readers explore symbol hierarchies, expand and collapse nodes without leaving a page, and filter symbols by keyword — making large documentation sites far more navigable.

## Key Topics

### App Project Documentation Support
Swift-DocC now supports documenting app targets (not just frameworks/packages). Developers can open Product > Build Documentation on any app project and receive auto-generated stub pages for all APIs as a starting point.

### Objective-C and C API Documentation
Xcode 14 brings full DocC support to Objective-C and C code using the same triple-slash `///` Markdown comment syntax. Pages for mixed-language APIs display a language toggle letting readers switch between Swift and Objective-C views.

### Documentation Catalogs and Top-Level Pages
Adding a Documentation Catalog (`.docc` bundle) lets developers create custom top-level landing pages with summaries, overviews, and embedded images. The catalog's Markdown files complement inline source comments.

### Static Hosting and GitHub Pages Publishing
DocC archives are now directly deployable to most web servers without transformation. For hosting at a sub-path (e.g., GitHub Pages at `username.github.io/repo-name`), set the **DocC Archive Hosting Base Path** build setting to the repository name. Content can then be exported from Xcode and committed to a `docs/` directory.

### Swift-DocC Swift Package Manager Plug-in
A new official Swift-DocC SPM plug-in simplifies building and deploying documentation for Swift packages, enabling automated CI/CD pipelines targeting GitHub Pages and other hosting services.

### New Navigation Sidebar
The web documentation viewer gained a persistent navigation sidebar with a disclosure-triangle hierarchy for symbols, and a filter field at the bottom for quickly finding specific APIs by keyword.

## APIs & Frameworks

- **DocC / Swift-DocC** — documentation compiler and archive format **[NEW features]**
  - `///` triple-slash doc comment syntax (Swift and Objective-C/C) — unchanged syntax, now supported for all target types
  - `- Parameter <name>:` doc comment tag for parameter documentation
  - `- Returns:` doc comment tag
  - Double-backtick cross-reference links (e.g., ` ``SlothyApp`` `) with build-time link validation
  - Markdown fenced code blocks inside doc comments
  - Documentation Catalog (`.docc` bundle) — contains top-level `<TargetName>.md`, additional articles, and media assets
  - **DocC Archive** (`.doccarchive`) — portable, self-contained static website bundle **[compatible with static hosts NEW]**
- **Xcode 14 Build Settings**
  - `DOCC_ARCHIVE_HOSTING_BASE_PATH` — **[NEW]** specifies the base URL path for hosting on non-root paths (e.g., GitHub Pages)
- **xcodebuild** CLI
  - `xcodebuild docbuild` — command-line documentation build (introduced Xcode 13, enhanced Xcode 14)
- **Swift-DocC Swift Package Manager Plug-in** (`SwiftDocCPlugin`) **[NEW]**
  - Enables `swift package generate-documentation` and related commands for SPM packages
- **Xcode Documentation Window**
  - Export DocC Archive via context menu on any technology node in the documentation navigator
- **Web Viewer (DocC-Render)**
  - Navigation sidebar with collapsible symbol tree **[NEW]**
  - Filter field for symbol search within sidebar **[NEW]**
  - Language toggle for mixed Swift/Objective-C pages **[NEW]**

## Code Highlights

Documenting a Swift struct:
```swift
/// A view that displays a sloth.
///
/// This is the main view of ``SlothyApp``.
/// Create a sloth view by providing a binding to a sloth.
///
/// ```swift
/// @State private var sloth: Sloth
///
/// var body: some View {
///     SlothView(sloth: $sloth)
/// }
/// ```
struct SlothView: View { ... }
```

Documenting an initializer with parameters:
```swift
/// Creates a view that displays the specified sloth.
///
/// - Parameter sloth: The sloth the user will edit.
init(sloth: Binding<Sloth>) { ... }
```

Documenting an Objective-C class:
```objc
/// A sound that can be played.
///
/// - Parameters:
///   - name: The name of the sound.
///   - filePath: The path to the sound file on disk.
- (id)initWithName:(NSString *)name filePath:(NSString *)filePath;
```

## Takeaways
- Swift-DocC in Xcode 14 now covers all project types: Swift/ObjC apps, frameworks, and Swift packages.
- DocC archives are static-host-compatible by default; only the `DOCC_ARCHIVE_HOSTING_BASE_PATH` build setting is needed for GitHub Pages.
- The new SPM plug-in (`SwiftDocCPlugin`) makes CI/CD documentation automation straightforward for packages.
- The redesigned web navigation sidebar with hierarchical disclosure and a filter field dramatically improves discoverability for large documentation sites.

---
_Source: WWDC22 Session 110368 page (abstract, chapter summaries, code samples, and resource links)._
