# Translate Your App Using Agents in Xcode

**WWDC26 · Session 213** · [Watch](https://developer.apple.com/videos/play/wwdc2026/213/)

_Platforms:_ Xcode 27 · iOS · macOS · iPadOS · tvOS · watchOS · visionOS

## Overview

This session demonstrates how Xcode 27's coding agents automate the end-to-end localization workflow for Apple platform apps. Rather than manually editing String Catalog files or writing XLIFF export/import scripts, a developer can ask the agent to "translate my app" and watch it automatically prepare the project, build all targets to discover every localizable string, and populate translated String Catalogs for multiple languages — all in one agent session.

The review and iteration phase shows practical techniques for examining translation output: previewing the running app in a different locale, identifying truncation and layout issues caused by longer translated strings, and asking the agent to re-translate specific strings with additional context or constraints. The session also explains how machine-translated strings are marked in the XLIFF format with a `state-qualifier="leveraged-mt"` attribute, which is important for professional translator handoffs.

A best practices chapter explains the two developer responsibilities that maximize translation quality: ensuring all user-facing strings are properly marked as localizable using the correct API, and providing clear, context-rich comments and optional glossaries that guide the agent's translation decisions.

## Key Topics

### Add Translations
- Ask an agent to translate the app; Xcode handles the full pipeline automatically:
  1. Project preparation (adds target languages)
  2. Full build of all targets to discover every string
  3. Populates translated String Catalogs for each target language
- No manual XLIFF export/import required.

### Review and Iterate
- Preview the app running in a translated locale directly from Xcode.
- Identify truncation and layout breakage caused by string length differences.
- Re-translate specific strings by prompting the agent with additional context.
- Understand machine-translation markers in XLIFF (`state-qualifier="leveraged-mt"`) for professional translator handoffs.

### Best Practices
- Mark all user-facing strings as localizable at the call site.
- Provide `comment:` arguments with meaningful context on every string.
- Provide custom terminology glossaries or rules to guide agent translation decisions.

## APIs & Frameworks

**Xcode 27 / Localization**
- **[NEW]** Agent-driven String Catalog population (automatic build + translate pipeline)
- String Catalogs (`.xcstrings` files) — agent reads, writes, and populates
- XLIFF export/import — agent handles internally; `state-qualifier="leveraged-mt"` marks machine translations
- Xcode locale preview — preview app in any target locale without device

**SwiftUI Localization**
- `Text(_:comment:)` — localizable string with translator context
- `Text(_:tableName:comment:)` — localizable string from a named table

**Foundation Localization**
- `String(localized:comment:)` — localizable string in non-SwiftUI code
- `LocalizedStringResource(_:bundle:comment:)` — localizable string resource type

**Resources**
- [Localizing your app using agents](https://developer.apple.com/documentation/Xcode/localizing-your-app-using-agents)
- [Expanding Your App to New Markets](https://developer.apple.com/localization/)

## Code Highlights

SwiftUI localizable string with translator comment:

```swift
// Always provide a comment for context-aware translation
Text("Hello, world!", comment: "A standard greeting")

// Use tableName to organize strings into separate catalogs
Text("Hello, world!", tableName: "Greetings", comment: "A standard greeting")
```

Foundation localizable strings outside SwiftUI:

```swift
// Non-SwiftUI localized string
String(localized: "Hello, world!", comment: "A standard greeting")

// Localizable resource type (pass across API boundaries)
LocalizedStringResource("Hello World!", bundle: #bundle, comment: "A standard greeting")
```

Machine-translated XLIFF marker (understand when reviewing agent output):

```xml
<trans-unit id="Grand Canyon" xml:space="preserve">
  <source>Grand Canyon</source>
  <target state="translated" state-qualifier="leveraged-mt">Grand Canyon</target>
  <note>Name of the 'Grand Canyon' landmark.</note>
</trans-unit>
```

## Takeaways

- A single agent prompt can handle the entire localization pipeline — project setup, string discovery, and multi-language translation — eliminating hours of manual workflow.
- `comment:` arguments on every localizable string are the single highest-leverage investment for improving translation quality; without context, agents (and human translators) produce generic results.
- The `state-qualifier="leveraged-mt"` XLIFF marker is critical when handing off agent-generated translations to professional translators so they know which strings need human review.
- Agents can re-translate individual strings with targeted constraints, making iterative quality improvement practical rather than requiring full re-runs.

---
_Source: WWDC26 Session 213 page (abstract, chapter summaries, code samples, and resource links)._
