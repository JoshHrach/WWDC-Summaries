# Sync Files to the Cloud with FileProvider on macOS
**WWDC21 · Session 10182** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10182/)

_Platforms:_ macOS Monterey 12

## Overview
This session introduces the macOS FileProvider framework, which allows cloud storage vendors to integrate their services deeply into the macOS file system using an app extension rather than a kernel extension (KEXT/FUSE). Built on new APFS "dataless" file primitives, the framework lets apps provide files on demand: files and folders appear in Finder immediately but their actual content is downloaded only when a user or process accesses them.

The session walks through four core sync flows (content fetch, directory enumeration, remote change propagation, and local change upload), demonstrates a sample app called FruitBasket, and covers the implementation order and optional integration points including icon decorations, contextual menu actions, and pre-flight alerts.

## Key Topics

### Domains and Finder Integration
A `NSFileProviderDomain` represents a logged-in cloud account. Adding a domain via `NSFileProviderManager` makes it appear in the Finder sidebar with a root directory. The domain's lifecycle is driven by user actions; the extension is launched on demand.

### Dataless Files (APFS)
APFS dataless objects are transparent to processes: a read access on a dataless file is paused by the kernel while the extension fetches its content, then resumed once the fill is complete. Dataless directories are enumerated the same way. Subsequent accesses to already-downloaded files go directly to disk without involving the extension.

### Four Core Sync Flows
1. **Content fetch** — `fetchContents(for:version:request:completionHandler:)` is called when a dataless file is read; the extension downloads to a local URL and calls the completion handler.
2. **Directory enumeration** — `NSFileProviderEnumerator` returns paginated `NSFileProviderItem` pages; the system caches the result.
3. **Remote change propagation** — the extension signals the system via `NSFileProviderManager.signalEnumerator(for:completionHandler:)` for `.workingSet`; the system calls `enumerateChanges(from:for:)` with the last `NSFileProviderSyncAnchor`.
4. **Local change upload** — `createItem(basedOn:fields:contents:options:request:completionHandler:)`, `modifyItem(_:baseVersion:changedFields:contents:options:request:completionHandler:)`, and `deleteItem(identifier:baseVersion:options:request:completionHandler:)` receive local changes aggregated by the system.

### Eviction and Disk Pressure
The system automatically evicts least-recently-used uploaded files when disk space is critically low without calling the extension. Only items reported as uploaded (via the completion handler) are eligible for eviction. Extensions can also trigger or prevent eviction programmatically.

### Safe Save and Package Files
The system detects safe-save patterns (atomic renames) and remaps item identifiers to new file IDs transparently. It can zip package bundles and present consistent package-level changes to the extension.

### Optional Integration Points
- **Icon decorations** — badge or emboss file/folder icons in Finder using custom `UTType` artwork via `NSFileProviderItemDecorating`.
- **Contextual menu actions** — UI and non-UI custom actions defined via `NSPredicates` in the extension's `Info.plist`.
- **Pre-flight alerts** — warn users before potentially destructive actions; criteria configured in `Info.plist`.

## APIs & Frameworks

- `FileProvider` framework **[NEW on macOS]** — user-space cloud sync extension framework
- `NSFileProviderExtension` — base class for the file provider extension (non-replicated API)
- `NSFileProviderReplicatedExtension` **[NEW]** — protocol for the new replicated (macOS 12) file provider model
- `NSFileProviderDomain` — represents a cloud storage account/domain
- `NSFileProviderManager` — manages domains; `add(_:completionHandler:)`, `remove(_:completionHandler:)`
- `NSFileProviderManager.signalEnumerator(for:completionHandler:)` — signals remote changes for `.workingSet`
- `NSFileProviderEnumerator` protocol — `enumerateItems(for:startingAt:)`, `enumerateChanges(from:for:)`
- `NSFileProviderSyncAnchor` — opaque token tracking the last enumerated change point
- `NSFileProviderItem` protocol — represents a file/folder item with metadata
- `NSFileProviderItemFields` — bitmask describing which fields changed in a modify call
- `fetchContents(for:version:request:completionHandler:)` — content fetch callback
- `createItem(basedOn:fields:contents:options:request:completionHandler:)` — local create callback
- `modifyItem(_:baseVersion:changedFields:contents:options:request:completionHandler:)` — local modify callback
- `deleteItem(identifier:baseVersion:options:request:completionHandler:)` — local delete callback
- `NSFileProviderItemDecorating` — protocol for providing Finder icon decorations
- APFS dataless file objects — new APFS primitive; reads are transparently blocked until fetched
- `NSFileProviderItemIdentifier.workingSet` — special enumerator identifier for remote changes
- `NSFileProviderRequest` — contextual info about who triggered the operation

## Code Highlights

Creating and registering a domain:
```swift
let domain = NSFileProviderDomain(
    identifier: NSFileProviderDomainIdentifier("com.example.mycloud"),
    displayName: "My Cloud"
)
NSFileProviderManager.add(domain) { error in
    // Domain now visible in Finder sidebar
}
```

Implementing content fetch:
```swift
func fetchContents(for itemIdentifier: NSFileProviderItemIdentifier,
                   version requestedVersion: NSFileProviderItemVersion?,
                   request: NSFileProviderRequest,
                   completionHandler: @escaping (URL?, NSFileProviderItem?, Error?) -> Void) {
    downloadFromServer(itemIdentifier) { localURL in
        completionHandler(localURL, updatedItem, nil)
    }
}
```

Signaling remote changes:
```swift
NSFileProviderManager(for: domain)?.signalEnumerator(
    for: .workingSet
) { _ in }
```

## Takeaways

- FileProvider replaces KEXT/FUSE on macOS and is the only supported path for deep cloud storage integration going forward as kernel extensions are deprecated.
- The APFS dataless primitive makes on-demand download completely transparent to existing apps — they block on reads automatically, no special handling required.
- The replicated extension model delegates all local-change aggregation, safe-save detection, and package zipping to the system, dramatically reducing boilerplate.
- The sample project (FruitBasket) published with this session provides a comprehensive, runnable reference implementation.

---
_Source: WWDC21 Session 10182 page (abstract, chapter summaries, code samples, and resource links)._
