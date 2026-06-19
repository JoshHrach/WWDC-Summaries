# Create Rich Documentation with Swift-DocC
**WWDC23 · Session 10244** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10244/)

_Platforms:_ iOS, iPadOS, macOS, watchOS, tvOS, visionOS (Xcode 15 / Swift-DocC)

## Overview
Xcode 15 elevates Swift-DocC documentation authoring with a brand-new Documentation Preview editor that renders documentation in real time alongside source code, plus expanded authoring capabilities: grid-based layouts, video support, custom page icons, featured content sections, and fully custom website themes.

The session follows the SlothCreator Swift package through a complete documentation overhaul: documenting Swift extensions (new in Xcode 15), iterating with the live preview editor, enriching article pages with new directives (`@Row`/`@Column`, `@TabNavigator`, `@Video`, `@Links`, `@CallToAction`, `@PageKind`, `@PageImage`, `@PageColor`), applying a custom site theme via `theme-settings.json`, and navigating the redesigned web experience with quick navigation (Shift-Command-O).

## Key Topics

### Documentation Preview Editor (New in Xcode 15)
Activated via the Assistant editor jump bar → Documentation Preview. Stays active as you move between `.swift`, Objective-C header, and `.md` files. Renders the selected symbol or article's documentation in real time with every keystroke—no separate build step required. Images are picked up automatically from the documentation catalog in light/dark variants using the base filename.

### Swift Extension Documentation (New in Xcode 15)
Xcode 15 + Swift-DocC can now generate documentation pages for extensions you write to types from other frameworks. Extensions appear as a separate extended module on the top-level page (e.g., a SwiftUI extended module grouping). Symbols in extensions are documented the same way as owned symbols using `///` documentation comments and `.md` sidecar files.

### Authoring Directives

**Layout**
- `@Row { @Column(size:) { … } }` — grid-based layout. `size` sets column span in an implicit 4-column grid; default is 1. Mix text and images in columns.
- `@TabNavigator { @Tab("Label") { … } }` — collapses multiple elements (e.g., localized screenshots) into clickable tabs.
- `@Video(poster:source:alt:)` — embed video with poster image into an article page.

**Metadata (placed in `@Metadata { … }` at page top)**
- `@CallToAction(purpose: link | download, url: "…")` — prominent CTA button on the page.
- `@PageKind(sampleCode | article)` — marks page as sample code (adds curly-brace icon and "Sample Code" heading); `article` is default.
- `@PageImage(purpose: card | icon, source: "filename", alt: "…")` — card image shown when page is featured in `@Links`; icon image shown in sidebar and page header.
- `@PageColor(green | blue | orange | purple | yellow | …)` — accent color for top-level page.

**Featured Content**
- `@Links(visualStyle: list | compactGrid | detailedGrid) { - <doc:PageName> }` — featured link section above Topics; renders page cards using their `@PageImage(purpose: card)`.

### Topic Organization
Curate symbols and articles into Topic groups using third-level headings under a `## Topics` heading in the top-level `.md` file. Symbol links use double-backtick syntax: `` ``TypeName`` ``. Article links use `<doc:ArticleName>`.

### Custom Themes (`theme-settings.json`)
Place a file named exactly `theme-settings.json` in the documentation catalog. The JSON structure uses `"theme"` → `"color"` and `"typography"` sub-objects. Color variables like `"standard-green"` accept hex values; typography variables like `"html-font"` accept CSS font family values. Themes are deployment-specific: they apply to the web build only—Xcode's documentation window always uses the Xcode theme.

### Publishing and Navigation
- Use `xcodebuild docbuild` in CI (Xcode Cloud or any CI) to build and deploy documentation; compatible with GitHub Pages, Netlify, and any static host.
- Web documentation built with Xcode 15 gains **Quick Navigation**: press Shift-Command-O to open a fuzzy search popover across all pages, with live preview on the right. Community-driven feature.
- Navigation sidebar shows topic groups with disclosure triangles for browsing symbol hierarchies.

## APIs & Frameworks

**Swift-DocC (Xcode 15 — New Features)**
- Documentation Preview editor **[NEW]** — real-time rendered preview in Xcode's Source editor
- Swift extension documentation support **[NEW]** — document extensions to external types
- `@Row` / `@Column(size:)` directives **[NEW]** — grid-based page layouts
- `@TabNavigator` / `@Tab("Label")` directives **[NEW]** — tabbed content
- `@Video(poster:source:alt:)` directive **[NEW]** — video in documentation pages
- `@CallToAction(purpose:url:)` directive **[NEW]** — CTA button in `@Metadata`
- `@PageKind(sampleCode)` directive **[NEW]** — sample code page type with special styling
- `@PageImage(purpose:source:alt:)` directive **[NEW]** — card and icon images for pages
- `@PageColor(_:)` directive **[NEW]** — top-level page accent color
- `@Links(visualStyle:)` directive **[NEW]** — featured link cards section
- `theme-settings.json` **[NEW]** — site-wide custom color and typography theming
- Quick Navigation (Shift-Command-O) **[NEW]** — fuzzy search across all web documentation pages

## Code Highlights

Documenting a Swift extension with code example and image:
```swift
import SwiftUI

/// An extension that facilitates the display of sloths in user interfaces.
public extension Image {
    /// Create an image from the given sloth.
    ///
    /// Use this initializer to display an image representation of a given sloth.
    ///
    /// ```swift
    /// let iceSloth = Sloth(name: "Super Sloth", color: .blue, power: .ice)
    /// var body: some View {
    ///     Image(iceSloth).resizable().aspectRatio(contentMode: .fit)
    ///     Text(iceSloth.name)
    /// }
    /// ```
    ///
    /// ![A screenshot of an ice sloth, with the text Super Sloth underneath.](iceSloth)
    ///
    /// This initializer is useful for displaying static sloth images.
    /// To create an interactive view containing a sloth, use ``SlothView``.
    init(_ sloth: Sloth) {
        self.init("\(sloth.power)-sloth")
    }
}
```

Grid-based layout with asymmetric columns:
```
@Row {
    @Column(size: 2) {
        Paragraph text describing the UI.
    }
    @Column {
        ![Screenshot alt text](image-filename)
    }
}
```

Tab navigator for localized screenshots:
```
@TabNavigator {
    @Tab("English") {
        ![English screenshot](slothy-localization_eng)
    }
    @Tab("Chinese") {
        ![Chinese screenshot](slothy-localization_zh)
    }
    @Tab("Spanish") {
        ![Spanish screenshot](slothy-localization_es)
    }
}
```

Sample code page metadata with call-to-action:
```
@Metadata {
    @CallToAction(purpose: link, url: "https://example.com/slothy-repository")
    @PageKind(sampleCode)
}
```

Featured links with detailed grid style:
```
@Links(visualStyle: detailedGrid) {
    - <doc:GettingStarted>
    - <doc:SlothySample>
}
```

Custom theme file (`theme-settings.json`):
```json
{
    "theme": {
        "color": {
            "standard-green": "#83ac38"
        },
        "typography": {
            "html-font": "serif"
        }
    }
}
```

## Resources
- [DocC Documentation](https://developer.apple.com/documentation/docc)
- Related: "What's new in Xcode 15" (WWDC23 10165)
- Related: "Improve the discoverability of your Swift-DocC content" (WWDC22 110369)
- Related: "Build interactive tutorials using DocC" (WWDC21 10235)

## Takeaways
- The Documentation Preview editor in Xcode 15 is the fastest way to author DocC content: changes render live without leaving the source file.
- New layout directives (`@Row`, `@Column`, `@TabNavigator`, `@Video`) make articles more engaging; metadata directives (`@PageKind`, `@PageImage`, `@PageColor`, `@CallToAction`) add polish and discoverability.
- `@Links(visualStyle: detailedGrid)` with `@PageImage(purpose: card)` is the recommended pattern for featuring sample code articles on top-level pages.
- Custom themes in `theme-settings.json` are web-only—Xcode always uses its own theme—so you can brand the site freely without affecting the in-editor reading experience.

---
_Source: WWDC23 Session 10244 page (abstract, transcript, chapter summaries, code samples, and resource links)._
