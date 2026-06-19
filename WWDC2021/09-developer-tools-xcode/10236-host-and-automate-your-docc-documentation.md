# Host and Automate Your DocC Documentation
**WWDC21 · Session 10236** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10236/)

_Platforms:_ macOS Monterey 12 (Xcode 13)

## Overview
This session covers two complementary topics for DocC: hosting a generated `.doccarchive` on a web server, and automating documentation builds on the command line with `xcodebuild`'s new `docbuild` action. Together, these capabilities allow frameworks and packages to publish documentation to a public website and keep it continuously updated via CI/CD.

## Key Topics

**DocC Archive Structure**
A `.doccarchive` is a self-contained container holding all data needed both for reading documentation in Xcode and for hosting online. It is structured as a single-page Vue.js web app. The archive contains an `index.html` entry point, CSS/JS assets, and a `data/` directory with per-symbol JSON content files, plus images, downloads, and videos.

**Web Server Configuration**
Two routing rules are needed to host a `.doccarchive`:
1. Any request whose path starts with `/documentation/` or `/tutorials/` should be served with the archive's `index.html` — the single-page app takes over and loads the correct content.
2. Requests matching the top-level directories within the archive (`css/`, `js/`, `data/`, `images/`, `downloads/`, `favicon.ico`, `favicon.svg`, `img/`, `theme-settings.json`, `videos/`) should be routed to the corresponding files inside the `.doccarchive`. This works with Apache `.htaccess`, nginx, GitHub Pages, etc.

**Automating with xcodebuild docbuild**
Xcode 13 adds a `docbuild` action to `xcodebuild`. It works like the standard `build` action but additionally invokes the Swift compiler to generate symbol graphs (`.symbols.json`) and runs the DocC compiler to produce `.doccarchive` outputs in the derived data folder. Dependencies (other Swift frameworks/packages) are also documented in the same pass.

After the build, find archives with `find <derivedDataPath> -name "*.doccarchive"` and copy them to the hosting directory. This script can be integrated as a post-merge CI hook to keep hosted documentation always current.

**Build Pipeline Internals**
The Swift compiler generates a symbol graph containing public symbols, their relationships, and in-source documentation comments. The DocC compiler combines the symbol graph with any documentation catalog (`.docc` — articles, media, tutorials) to produce the archive. Both Xcode's GUI build and `xcodebuild docbuild` follow this same pipeline.

## APIs & Frameworks

- **DocC** (Documentation Compiler) **[NEW]** — integrated into Xcode 13
- `xcodebuild docbuild` action **[NEW]** — command-line documentation build
  - `-scheme <name>` — target scheme to document
  - `-derivedDataPath <path>` — custom output directory (optional but recommended)
  - `-sdk`, `-destination`, `-configuration` — standard build flags (optional)
  - `-list` — list available schemes in a project
- `.doccarchive` file format — self-contained documentation container (Xcode + web hosting)
- Symbol Graph format (`.symbols.json`) — Swift compiler output, input to DocC compiler
- DocC catalog (`.docc` directory) — optional articles, media, and tutorials source
- Apache `.htaccess` `RewriteRule` — server-side routing for `/documentation/` and `/tutorials/` paths
  - `RewriteRule ^(documentation|tutorials)\/.*$ <archive>/index.html [L]`
  - `RewriteRule ^(css|js|data|images|downloads|...)\/.*$ <archive>/$0 [L]`
- Vue.js single-page app — rendering engine embedded in the archive
- `find ... -name "*.doccarchive"` — shell command to locate built archives in derived data

## Code Highlights

Apache `.htaccess` routing for hosting a DocC archive:
```apache
# Enable custom routing.
RewriteEngine On

# Route documentation and tutorial pages.
RewriteRule ^(documentation|tutorials)\/.*$ SlothCreator.doccarchive/index.html [L]

# Route files within the documentation archive.
RewriteRule ^(css|js|data|images|downloads|favicon\.ico|favicon\.svg|img|theme-settings\.json|videos)\/.*$ SlothCreator.doccarchive/$0 [L]
```

Automation script to build and deploy DocC documentation:
```bash
#!/bin/sh
# Build documentation
xcodebuild docbuild \
  -scheme "SlothCreator" \
  -derivedDataPath MyDerivedDataPath

# Copy all built archives to the web hosting directory
find MyDerivedDataPath \
  -name "*.doccarchive" \
  -exec cp -R {} ~/www \;
```

## Takeaways

- A `.doccarchive` is both an Xcode-importable documentation bundle and a self-contained web app — the same artifact serves both use cases.
- Two server routing rules are all that's needed to host DocC documentation on any standard web server: one for page requests (`/documentation/`, `/tutorials/`) and one for asset requests (mapped to the archive's file structure).
- `xcodebuild docbuild` is the correct action for CI documentation builds — it's identical to a normal build but adds symbol graph generation and DocC compilation.
- Integrate `xcodebuild docbuild` as a post-merge CI hook to ensure hosted documentation is always in sync with the latest code changes.

---
_Source: WWDC21 Session 10236 page (abstract, chapter summaries, code samples, and resource links)._
