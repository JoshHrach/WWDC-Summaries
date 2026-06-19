# Discover and curate Swift Packages using Collections
**WWDC21 · Session 10197** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10197/)

_Platforms:_ macOS Monterey 12, Xcode 13

## Overview
Xcode 13 introduces **Swift Package Collections**: curated, signed JSON files that bundle a list of packages with their metadata so teams, educators, and community members can share and discover vetted packages more easily. Collections power a richer "Add Package" workflow in Xcode—showing license, README, release history, and authors—and enable fix-its that suggest adding a missing package when an unknown module is imported.

The session covers what collections are, how Xcode's new default Apple open-source collection works, the JSON format and tooling for creating and signing your own collections, command-line usage via `swift package-collection`, and the end-to-end Xcode workflow for adding packages from a collection.

## Key Topics

### Swift Package Collections Overview **[NEW in Xcode 13]**
- A collection is a JSON file (typically hosted over HTTPS) containing a list of package URLs plus metadata: summary, keywords, versions, products, README URL, and license.
- Collections are stored in a local database managed by libSwiftPM, making them available from any tool that links libSwiftPM—Xcode, the Swift Package Manager CLI, and custom tooling.
- Xcode 13 ships with a **default Apple open-source collection** containing packages like Swift Argument Parser and Swift NIO; this collection is updated regularly and enables:
  - Module import autocompletion for packages in the collection.
  - Fix-its that offer to add a missing package when an unknown module is imported.

### Enhanced "Add Package" Workflow in Xcode 13
- New panel shows: latest version, all release history and release notes, authors, license, and README directly inside Xcode.
- Context menu on the project navigator ("Add Packages") provides quick access without navigating menus.
- Packages not in any collection can still be added by pasting a direct URL or by selecting a specific branch.

### Creating a Collection
Collections are authored as a simple input JSON file and processed by the open-source **Swift Package Collection Generator** tool (`package-collection-generate`):

- Input JSON specifies collection-level metadata (`name`, `overview`, `keywords`, `author`) and a list of package URLs.
- Optional per-package overrides: `summary`, `keywords`, `versions` (restrict which versions appear), `excludedProducts`, `readmeURL`.
- The generator fetches metadata from each package's repository automatically (supports a `--auth-token` flag for GitHub API access).
- Output is a fully-formed collection JSON ready for distribution.

### Signing a Collection
- Signing is optional but recommended: it verifies the author and protects the collection's integrity against modification.
- Use the `package-collection-sign` tool (part of the same GitHub project) with a valid, non-expired code signing certificate.
- SwiftPM displays the signer's identity and a verified signature indicator when users inspect the collection.

### Distributing and Using Collections
- Host the signed JSON on any HTTPS web server or share the file directly.
- CLI: `swift package-collection add <URL>` adds a collection to the local database; `swift package-collection describe` lists its packages and signature info; `swift package-collection describe <package-URL>` shows metadata for a specific package.
- In Xcode UI: File → Add Packages → click "+" → paste the collection URL → Load.

## APIs & Frameworks

**Swift Package Manager (libSwiftPM)**
- Swift Package Collections format — curated JSON list of packages with metadata **[NEW]**
- `swift package-collection add <url>` — add a collection to the local database **[NEW]**
- `swift package-collection describe [<url>]` — inspect a collection or individual package **[NEW]**

**Swift Package Collection Generator (open source, GitHub)**
- `package-collection-generate` — generates a collection JSON from an input file **[NEW]**
- `package-collection-sign` — signs a collection JSON with a code signing certificate **[NEW]**
- Repository: `https://github.com/apple/swift-package-collection-generator`

**Xcode 13**
- Default Apple open-source package collection (Swift Argument Parser, Swift NIO, Swift Numerics, etc.) **[NEW]**
- Module import fix-its for packages in collections **[NEW]**
- Enhanced "Add Package" dialog with license, README, release history **[NEW]**
- Context menu "Add Packages" shortcut in project navigator **[NEW]**

## Code Highlights

Minimal collection input JSON:
```json
{
  "name": "WWDC21 Demo Collection",
  "overview": "Packages to be used in our demo app",
  "keywords": ["wwdc21"],
  "author": { "name": "Boris Buegling" },
  "packages": [
    { "url": "https://github.com/apple/swift-format" },
    { "url": "https://github.com/Alamofire/Alamofire" }
  ]
}
```

Collection entry with per-package overrides:
```json
{
  "url": "https://github.com/apple/swift-format",
  "summary": "Formatting technology for Swift source code.",
  "keywords": ["formatting", "swift"],
  "versions": ["0.50400.0", "0.50300.0"],
  "excludedProducts": ["SwiftFormatConfiguration"],
  "readmeURL": "https://github.com/apple/swift-format/blob/main/README.md"
}
```

Generating, signing, and publishing a collection:
```bash
# Step 1: generate
package-collection-generate --verbose input.json collection.json --auth-token <token>

# Step 2: sign
package-collection-sign collection.json collection-signed.json developer-key.pem developer-cert.cer

# Step 3: distribute (upload collection-signed.json to an HTTPS server)

# Add from command line
swift package-collection add https://example.com/collection-signed.json

# Inspect the collection
swift package-collection describe

# Inspect a specific package
swift package-collection describe https://github.com/apple/swift-format
```

## Takeaways
- Swift Package Collections are the recommended way for teams, educators, and community authors to share curated, vetted package lists that integrate directly into Xcode's "Add Package" workflow.
- The default Apple collection in Xcode 13 removes the first-use friction for popular Apple open-source packages: just type the import, accept the fix-it.
- Signing a collection with a code signing certificate lets users verify the author and guarantees the collection has not been tampered with after publication.
- Collections are managed by libSwiftPM, so they work identically in Xcode and on the command line; no Xcode-specific configuration required.

---
_Source: WWDC21 Session 10197 page (abstract, full transcript, code samples, and resource links)._
