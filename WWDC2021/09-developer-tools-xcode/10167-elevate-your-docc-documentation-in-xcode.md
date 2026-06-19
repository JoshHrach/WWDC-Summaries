# Elevate Your DocC Documentation in Xcode
**WWDC21 · Session 10167** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10167/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8, tvOS 15

## Overview
This session is the second in the DocC series and focuses on enriching a Swift framework's documentation beyond auto-generated API reference pages. Using the SlothCreator sample framework, the speakers demonstrate how to create a Documentation Catalog in Xcode 13, author top-level and task articles in Markdown, organize symbols into meaningful topic groups, and use documentation extension files to separate structural metadata from source code.

The session establishes a clear mental model for three DocC page types—reference, articles, and tutorials—and explains when each is appropriate. Articles are the focus: they provide the "big picture" narrative that reference pages alone cannot convey. Top-level articles establish a framework's identity; task articles guide developers through specific workflows. Documentation extensions allow topic group customization without cluttering source code comment blocks.

## Key Topics
- **Documentation Catalog (NEW):** A new Xcode 13 file type (`.docc` folder) that consolidates all documentation assets—Markdown files, images, and extension files—in the project navigator alongside source code.
- **Three Page Types:** Reference (auto-generated from source comments, enriched with Markdown), Articles (free-form Markdown pages), Tutorials (step-by-step project walkthroughs covered in Session 10235).
- **Top-Level Article:** Provides a concise summary, Overview section, and image for the framework's landing page. Added as a `.md` file named after the module inside the documentation catalog.
- **Task Article:** A focused article that guides developers through completing a specific workflow using the framework's APIs.
- **Organizing Documentation with Topics Sections:** Adding a `## Topics` section to any article or container symbol page to define named groups (`### Group Name`) with bulleted lists of doc links (`- <doc:ArticleName>`) and symbol links (` ``SymbolName`` `). DocC uses these to order both the page and the documentation navigator.
- **Documentation Extensions:** A `.md` file linked to a symbol via ` # ``ModuleName/SymbolName`` ` syntax in the title. Merges with the source code comment at build time. Best practice: keep primary content (summary, discussion) in source; put topics sections in the extension file.
- **Image Best Practices:** Use 2x resolution images with `@2x` filename suffix; provide a Dark Mode variant with `~dark` suffix. DocC auto-selects the right image—only the base name is needed in Markdown.
- **DocC Diagnostics:** Link validation warnings appear in Xcode; autocomplete works for doc links.
- **Build Documentation:** `Product > Build Documentation` command rebuilds the documentation window; also exportable as a `.doccarchive` for offline distribution or web publishing.

## APIs & Frameworks

**DocC / Xcode 13**
- `Documentation Catalog` file type (`.docc`) **[NEW]** – Container for all documentation assets
- `Extension File` template **[NEW]** – Markdown file linked to a symbol for separate topic organization
- `## Topics` section – Groups symbols/articles on a documentation page
- `### Group Name` – Creates a named group within a topics section
- ` ``SymbolName`` ` syntax – Links to a symbol from Markdown
- ` ``Module/SymbolName`` ` syntax – Fully qualified symbol link
- `<doc:ArticleName>` syntax – Links to an article page
- Image Markdown syntax: `![Description](imageName.png)` (no need for @2x or ~dark suffix)
- `Product > Build Documentation` – Xcode menu command to compile and display documentation
- Export button in Documentation Navigator – Exports a `.doccarchive`

**DocC Page Types**
- Reference pages – Auto-generated from source code comments; customizable with Markdown
- Article pages – Free-form `.md` files in the documentation catalog
- Tutorial pages – Step-by-step interactive walkthroughs (see Session 10235)

## Code Highlights
Top-level framework article Markdown structure:
```markdown
# ``SlothCreator``

Catalog sloths you find in nature and create new adorable virtual sloths.

## Overview

SlothCreator provides models and utilities for creating, tracking, and caring
for sloths. The framework provides structures to model an individual ``Sloth``,
and identify them by key characteristics, including their ``Sloth/name`` and
special supernatural ``Sloth/power-swift.property``.

![A sloth hanging off a tree.](sloth.png)

## Topics

### Essentials

- <doc:GettingStarted>
- ``Sloth``
```

Documentation extension file linking to a symbol:
```markdown
# ``SlothCreator/Sloth``

## Topics

### Scheduling Sloth Activities

- ``Activity``
- ``Habitat``
```

## Takeaways
- A Documentation Catalog is now the recommended home for all supplemental documentation, keeping it in the project navigator alongside source code and making it easier to maintain.
- Articles solve the "blank framework landing page" problem: a top-level article gives new adopters the big picture before they dive into API reference.
- Topics sections control the order and grouping in both the documentation page and the navigator; organize from essential/simple to advanced, and group by feature theme rather than type.
- Documentation extension files are the right place for topics sections when you want cleaner source code—DocC merges them transparently at build time.

---
_Source: WWDC21 Session 10167 page (abstract, chapter summaries, code samples, and resource links)._
