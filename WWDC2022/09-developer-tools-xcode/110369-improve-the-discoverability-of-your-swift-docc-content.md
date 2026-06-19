# Improve the Discoverability of Your Swift-DocC Content
**WWDC22 · Session 110369** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110369/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13 (developer tooling)

## Overview
This session focuses on best practices for organizing and structuring Swift-DocC documentation to maximize discoverability on the web. It introduces the new web-based navigation experience added to Swift-DocC in 2022, including a persistent sidebar navigator with a filter bar, and provides practical design guidance for structuring topic groups, ordering content, and writing clear group titles.

The session uses the SlothCreator sample framework as a worked example, demonstrating a three-step process: identifying high-level themes, organizing content by importance and specificity, and writing clear and mutually exclusive group titles.

## Key Topics
- **New web navigation experience** — Swift-DocC documentation sites now feature a two-panel layout: a left-side navigator/sidebar with a filter bar, and a right-side content view. The navigator shows the full API hierarchy with disclosure indicators. The layout adapts to different screen sizes.
- **Filter bar** — allows readers to type a search term and filter the navigator to matching pages in real time; supports tags for filtering by content type (Articles, Tutorials) and for hiding deprecated pages
- **Three-step content organization process:**
  1. **Define main high-level themes** — identify 3–10 overarching topic groups for the top-level documentation page; the first thing developers see when landing on docs
  2. **Organize by importance and specificity** — put beginner/essentials content first; order groups so broad topics come before specific ones; nest more specific groups deeper in the hierarchy
  3. **Optimize group titles** — titles should be clear, descriptive, and mutually exclusive; avoid generic terms like "Management" in favor of specific ones like "Care and Feeding"
- **Automatic organization** — Swift-DocC provides automatic grouping by type (protocols, structures, articles, tutorials) as a baseline when no manual organization is specified
- **Encouraging serendipity** — placing related topic groups near each other helps developers discover relevant APIs they weren't specifically looking for
- **SwiftDocCPlugin** — allows publishing DocC documentation sites (referenced in resources)

## APIs & Frameworks
**Swift-DocC / DocC**
- DocC documentation catalog — `.docc` bundle containing markdown and resources
- `Topics` section in documentation markdown **[NEW behavior]** — the mechanism for manually defining topic groups in documentation pages
- Navigator sidebar **[NEW]** — persistent sidebar with full API hierarchy and disclosure indicators on published documentation websites
- Filter bar **[NEW]** — real-time text filter + tag-based filter (Articles, Tutorials, deprecated) in the navigator
- Automatic organization by type — fallback grouping when no explicit `Topics` section is defined
- `SwiftDocCPlugin` — Swift Package Manager plugin for building and publishing DocC documentation

**DocC Markdown Directives (used for structure)**
- `## Topics` — section header in documentation markdown that defines manual topic groups
- `### <Group Title>` — topic group heading under the `Topics` section
- Symbol link syntax (`` `SymbolName` ``) — links to API symbols within topic groups

## Code Highlights
No code samples in this session. The content organization is done in documentation Markdown:

```markdown
## Topics

### Essentials
- <doc:GettingStartedWithSloths>
- ``Sloth``

### Sloth Creation
- ``SlothGenerator``
- ``PowerType``

### Sloth Views
- ``SlothView``
- ``HabitatMapView``

### Care and Feeding
- ``Activity``
- ``CareSchedule``
- ``FoodGenerator``
- ``Sloth/Food``
```

## Takeaways
- The new web navigator with filter bar makes it much easier for readers to browse and search documentation — well-organized content makes full use of it.
- Keep top-level topic groups to under 10; order them from broadest/most introductory to most specific.
- Topic group titles should be self-explanatory in isolation and clearly distinct from each other (mutually exclusive) to avoid reader confusion.
- Good documentation organization enables "serendipitous discovery" — developers find related APIs they didn't know to look for.

---
_Source: WWDC22 Session 110369 page (abstract, chapter summaries, code samples, and resource links)._
