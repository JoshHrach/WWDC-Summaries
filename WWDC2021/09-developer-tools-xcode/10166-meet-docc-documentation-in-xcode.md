# Meet DocC Documentation in Xcode
**WWDC21 · Session 10166** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10166/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8, tvOS 15

## Overview
Xcode 13 introduces DocC, a fully integrated documentation compiler for Swift frameworks and packages. DocC builds documentation alongside code using Xcode's build system, surfacing docs in Quick Help, code completion, and the Developer Documentation window — the same window used for Apple platform libraries. Documentation archives can be exported as `.doccarchive` files for sharing with colleagues or hosting on the web.

DocC enables three documentation modes: reference documentation (API-level), articles (big-picture narratives), and tutorials (interactive step-by-step walkthroughs). All three use extended Markdown syntax. The session focuses on the basics: writing documentation comments in source, building docs with Xcode's Build Documentation action, browsing docs in the Developer Documentation window, and creating symbol links between API pages.

DocC will be released as open source, along with a web app for hosting documentation archives online.

## Key Topics

**Building Documentation**
- `Product > Build Documentation` — builds and opens documentation on demand in Xcode 13 **[NEW]**
- `Build Documentation during 'Build'` build setting — continuously builds docs on every compile
- `xcodebuild docbuild` — command-line documentation build for CI workflows

**Writing Documentation Comments**
Triple-slash `///` inline comments and `/** */` block comments above public/open declarations are compiled into documentation pages. Only `public` and `open` symbols generate documentation pages.

**Comment Structure**
- First line → summary
- Blank line + additional text → Discussion section (supports full Markdown)
- `- Parameters:` / `- Parameter name:` → parameter descriptions
- `- Returns:` → return value description
- Fenced code blocks (triple backticks) → code examples rendered in docs

**Symbol Links**
Double-backtick syntax ` ``SymbolName`` ` creates cross-reference links between documentation pages. Sibling symbols can be referenced by name; children of other types use `Type/member` syntax. Symbol links are also active in Quick Help.

**Quick Help Enhancements**
Option-click on a symbol to open Quick Help with the summary and discussion. New "Open in Developer Documentation" link jumps to the full page in the Developer Documentation window.

**Documentation Archives**
DocC compiles documentation to a `.doccarchive` — a self-contained bundle containing a single-page web app. Archives can be exported from the documentation window's navigator context menu, double-clicked to open in Xcode, and distributed to teammates or hosted on the web.

**Add Documentation Action**
Command-click on a method declaration → "Add Documentation" inserts the appropriate comment template with `- Parameters:` and `- Returns:` sections pre-populated.

## APIs & Frameworks

### DocC (Documentation Compiler — Xcode 13) **[NEW]**
- `/// ` (triple-slash) — inline documentation comment marker
- `/** */` — block-style documentation comment
- `- Parameter <name>:` — single parameter documentation
- `- Parameters:` / `  - <name>:` — multi-parameter documentation
- `- Returns:` — return value documentation
- ` ``SymbolName`` ` — symbol cross-reference link (double backtick syntax) **[NEW]**
- ` ``Type/member`` ` — cross-reference to a member of another type **[NEW]**
- Fenced code blocks (` ``` `) — inline code examples in documentation

### Xcode 13 Build System Integration **[NEW]**
- `Product > Build Documentation` — on-demand documentation build action
- `Build Documentation during 'Build'` — continuous build setting
- `xcodebuild docbuild` — command-line equivalent

### Xcode 13 Developer Documentation Window **[NEW]**
- Unified documentation window hosting Swift framework/package docs alongside Apple platform docs
- Jump bar navigation — shows full symbol hierarchy
- Search — full-text search across all loaded documentation
- Export context menu — export `.doccarchive` from navigator

### DocC Archive Format
- `.doccarchive` — self-contained documentation bundle **[NEW]**
  - Contains compiled reference, articles, and tutorials
  - Embeds a single-page web app for browser hosting
  - Opens directly in Xcode by double-clicking

## Code Highlights

Basic documentation comment with summary, discussion, and code example:
```swift
/// Food that a sloth can consume.
///
/// Sloths love to eat the leaves and twigs they find in the rainforest canopy.
///
/// ```swift
/// superSloth.eat(.twig)
/// ```
public struct Food { ... }
```

Method documentation with parameters, returns, and symbol links:
```swift
/// Eat the provided specialty sloth food.
///
/// When they eat food, a sloth's ``energyLevel`` increases by the food's ``Food/energy``.
///
/// - Parameters:
///   - food: The food for the sloth to eat.
///   - quantity: The quantity of the food for the sloth to eat.
/// - Returns: The sloth's energy level after eating.
mutating public func eat(_ food: Food, quantity: Int = 1) -> Int { ... }
```

## Takeaways
- DocC makes Swift documentation a first-class, compiler-integrated feature in Xcode 13, with docs browsable alongside Apple platform docs in the same window.
- Triple-slash `///` comments on `public`/`open` symbols are all that is needed to start building browsable, searchable documentation with no additional tooling setup.
- The double-backtick ` ``Symbol`` ` syntax for cross-reference links is the most impactful new authoring feature — it connects your API pages into a navigable web and works even in Quick Help.
- `.doccarchive` is a portable, web-hostable format that makes it easy to share framework documentation without requiring recipients to have the source code.

---
_Source: WWDC21 Session 10166 page (abstract, chapter summaries, code samples, and resource links)._
