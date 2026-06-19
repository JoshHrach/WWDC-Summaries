# Build Interactive Tutorials Using DocC
**WWDC21 · Session 10235** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10235/)

_Platforms:_ macOS Monterey 12, Xcode 13

## Overview
DocC is Apple's new documentation compiler introduced in Xcode 13, which compiles Swift framework and package documentation from source comments, articles, and now interactive tutorials. This session focuses specifically on the tutorial-authoring capability of DocC, showing how to create step-by-step, interactive learning experiences for framework adopters using an extended Markdown syntax called directives.

Tutorials built with DocC are centered around building real apps using the framework's API. The session walks through planning a tutorial collection for a fictional "SlothCreator" framework, organizing API into logical chapters, and then authoring the tutorials in Xcode 13 using DocC's directive syntax with rich media support.

DocC tutorials are previewed live within Xcode's Developer Documentation window, and can be bundled and hosted alongside reference documentation and articles, giving framework authors a single unified documentation system.

## Key Topics

### DocC Directives
DocC extends Markdown with a directive syntax (`@DirectiveName { }`) to provide structure for tutorial content. Key directives include:
- `@Tutorials` — top-level wrapper for the table of contents
- `@Intro` — introduction block with title, description, and image
- `@Chapter` — groups related tutorials together
- `@TutorialReference` — links to an individual tutorial page
- `@Tutorial` — top-level wrapper for a single tutorial page
- `@Section` — divides a tutorial into progressive segments
- `@ContentAndMedia` — pairs instructional text with an image
- `@Steps` — container for individual tutorial steps
- `@Step` — a single instructional step with text and media
- `@Code` — links a Swift source file and optional preview image for a code step
- `@Image` — embeds an image with an accessibility description

### Table of Contents Structure
The table of contents file organizes the tutorial collection. It specifies the framework name, provides an intro with an image, and organizes tutorials into named chapters using `@Chapter` and `@TutorialReference`.

### Tutorial Pages
Each tutorial page starts with a `@Tutorial` directive containing an estimated time, a title, an intro, and multiple `@Section` blocks. Sections contain `@Steps` blocks with individual `@Step` directives. Code steps use `@Code` to display the relevant Swift file, and DocC automatically diffs successive code files to highlight new lines.

### Planning a Tutorial Collection
A structured approach is recommended: identify the framework's key API, group them by functional area, design a sample app that uses those API groups, then outline tutorials for each group as chapters. This ensures coherent, progressive learning without redundancy.

### Accessibility
Tutorial images should include accessible `alt` descriptions in every `@Image` directive so screen reader users receive the full context of visual content.

## APIs & Frameworks

**DocC** — Documentation Compiler **[NEW]** (Xcode 13)
- `@Tutorials(name:)` — declares a tutorial collection with the framework name **[NEW]**
- `@Intro(title:)` — tutorial collection or tutorial page introduction **[NEW]**
- `@Chapter(name:)` — groups tutorials in the table of contents **[NEW]**
- `@TutorialReference(tutorial:)` — references a tutorial by doc: identifier **[NEW]**
- `@Tutorial(time:)` — defines a single tutorial page with estimated duration **[NEW]**
- `@Section(title:)` — a named section within a tutorial **[NEW]**
- `@ContentAndMedia(layout:)` — pairs content text with an image in a section intro **[NEW]**
- `@Steps` — container for step directives **[NEW]**
- `@Step` — an individual tutorial step **[NEW]**
- `@Code(name:file:)` — displays a Swift file in a code step; supports nested `@Image` for previews **[NEW]**
- `@Image(source:alt:)` — embeds an image with accessibility description **[NEW]**
- Documentation Catalog — a folder added to an Xcode project/package to hold `.md`, tutorial, and resource files **[NEW]**
- Resources folder (within Documentation Catalog) — stores image assets referenced by tutorials **[NEW]**

**Xcode 13**
- Product > Build Documentation (⌃⇧⌘D) — builds and opens documentation including tutorials **[NEW]**
- Developer Documentation window — displays compiled DocC content including tutorials **[NEW]**

## Code Highlights

Table of Contents directive structure:
```
@Tutorials(name: "SlothCreator") {
    @Intro(title: "Meet SlothCreator") {
        Create, catalog, and care for sloths using SlothCreator.
        @Image(source: slothcreator-intro.png, alt: "...")
    }
    @Chapter(name: "SlothCreator Essentials") {
        @Image(source: chapter1-slothcreatorEssentials.png, alt: "...")
        Create custom sloths and edit their attributes and powers.
        @TutorialReference(tutorial: "doc:Creating-Custom-Sloths")
    }
}
```

A code step with automatic diff highlighting and a preview image:
```
@Step {
    Import the `SlothCreator` package.
    @Code(name: "CustomizedSlothView.swift", file: 01-creating-code-02-02.swift) {
        @Image(source: preview-01-creating-code-02-01.png, alt: "...")
    }
}
```

## Takeaways
- DocC's directive syntax lets developers author rich, interactive tutorials using familiar Markdown extended with structured `@Directive` blocks.
- A Documentation Catalog in Xcode 13 unifies reference docs, articles, and tutorials in a single compilable package.
- Good tutorial design requires upfront planning: group API by functional area, design a realistic sample app, then map chapters to those groups.
- Accessibility is a first-class concern — every `@Image` directive should include a descriptive `alt` string.

---
_Source: WWDC21 Session 10235 page (abstract, chapter summaries, code samples, and resource links)._
