# Advances in macOS Security
**WWDC19 · Session 701** · [Watch](https://developer.apple.com/videos/play/wwdc2019/701/)

_Platforms:_ macOS Catalina 10.15

## Overview
macOS Catalina introduces significant expansions to two complementary security layers: Gatekeeper (preventing malicious software from running) and user privacy protections (preventing even admitted software from accessing sensitive data without consent). The session walks through the defense-in-depth philosophy that underlies macOS security design, then details every change in each area that developers need to understand and adapt to.

On the Gatekeeper side, all new software signed after June 1, 2019 must be notarized; Gatekeeper enforcement is extended beyond LaunchServices to all quarantined software regardless of load path; and a new universal malicious content scan applies even to non-quarantined code. On the privacy side, screen recording, keyboard monitoring, and a new set of document locations (Desktop, Documents, Downloads, iCloud Drive, removable/network volumes, Trash) all require explicit user consent. Open/Save panels are now out-of-process for security, which carries API impact.

## Key Topics

### Defense in Depth
- macOS security is layered; no single technology provides complete protection.
- Different layers serve different roles: prevent malware entry (Gatekeeper), reduce attack surface (sandboxing), protect data (privacy TCC), detect tampering (code signing).
- The session focuses on two: Gatekeeper (outermost) and user privacy protection (inner).

### Gatekeeper Changes **[NEW in macOS Catalina]**

**Previous model (macOS Mojave):**
- Ran on first launch of _quarantined_ software _via LaunchServices_.
- Checked: malicious content scan, signature validity, identity policy (Developer ID or App Store), first-launch user prompt.

**macOS Catalina changes:**
1. **Notarization required for all new software** — software signed after June 1, 2019 must be notarized to pass Gatekeeper. Existing software with only a Developer ID certificate continues to pass unchanged.
2. **Gatekeeper enforced on all quarantined software, regardless of load path** — previously only LaunchServices-launched executables; now also covers `NSTask`, `exec`, `posix_spawn`, `dlopen`, `NSBundle` loads, etc., if the file is quarantined. First-launch prompt applies to bundled software; standalone executables and libraries skip the prompt but still pass through all other checks.
3. **Universal malicious content scan** — regardless of quarantine status and regardless of how code is loaded, if known malicious content is detected, execution is blocked and the user is alerted.

**Future direction (announced):**
- In a future macOS, unsigned code will not run by default.
- Developer action: sign and notarize all distributed software; never break bundle signatures at runtime; handle load failures gracefully.

**Quarantine:** opt-in model; web browsers, Messages, AirDrop add the quarantine xattr. App-sandboxed apps quarantine downloaded files by default. Background self-update downloads typically are not quarantined (unless sandboxed).

### User Privacy Protections **[NEW in macOS Catalina]**

**Screen Recording (new in Catalina):**
- `CGDisplayStream` creation returns `nil` and triggers a system dialog directing the user to Security & Privacy → Screen Recording on the first attempt without approval.
- `CGWindowListCreateImage` returns `nil` for windows not owned by the calling app (except desktop background image and menu bar).
- `CGWindowListCopyWindowInfo` — window name and sharing state omitted from results unless app is pre-approved for screen recording; never triggers a prompt (silently filters).
- **Safe pattern**: identify desktop background windows by `kCGWindowLayer` (window level), not by `kCGWindowName` — avoids requiring screen recording approval.

**Keyboard Monitoring (new in Catalina):**
- `CGEventTapCreate` with listen-only tap requires user approval for **Input Monitoring**.
- `CGEventTapCreate` with modifying tap requires user approval for **Accessibility** (stronger).
- Apps can monitor their own keyboard events via `NSEvent.addLocalMonitorForEvents` (no consent needed) or `NSApplication.sendEvent` subclass.
- `IOHIDCheckAccess(kIOHIDRequestTypeListenEvent)` — check authorization status without prompting.
- `IOHIDRequestAccess(kIOHIDRequestTypeListenEvent)` — trigger approval prompt without creating an event tap.
- `IOHIDCheckAccess(kIOHIDRequestTypePostEvent)` — check authorization for synthesizing events (Accessibility).

**Document Location Protections (new in Catalina):**
New protected locations requiring user consent to read existing files:
- Desktop, Documents, Downloads folders
- iCloud Drive and third-party cloud storage
- Removable volumes (USB drives, external disks)
- Network-attached storage volumes
- Trash **[NEW]**

Key rules:
- Creating new files in protected locations does not require consent.
- **User intent inference**: double-clicking in Finder, drag-and-drop, and Open/Save panel selections grant per-file access proactively — no consent prompt needed for those specific files.
- Sidecar files: use `NSFileCoordinator` with `NSFilePresenter` (`primaryPresentedItemURL` = user-selected file, `presentedItemURL` = sidecar); declare sidecar extension in `CFBundleDocumentTypes` with `NSIsRelatedItemType = YES`.

**Open/Save Panel Changes:**
- `NSOpenPanel` and `NSSavePanel` are now always **out-of-process** for security — class hierarchy and view hierarchies changed.
- Apps can no longer programmatically dismiss panels by calling `-ok:`.
- `NSOpenSavePanelDelegate` methods: apps can no longer rewrite the user selection; accessing provided URLs inside delegate methods may trigger consent prompts (user has not yet confirmed selection).
- Test file access without prompting: `FileManager.default.isReadableFile(atPath:)` / `isWritableFile(atPath:)`.

**Full Disk Access:**
- Includes same Mojave-protected categories (Mail, Messages, Safari history, etc.) plus Trash (new in Catalina).
- Helpers denied access are now **pre-populated (unchecked)** in the Full Disk Access pane automatically — no longer requires users to use the + button to find them.
- `FileManager.trashItem(at:resultingItemURL:)` — no Full Disk Access required to move a file to Trash or to access a file the app itself moved there.

**Automation (Mojave-introduced, recap):**
- Synthetic input events (AppleScript, CGEvent posting) require **Accessibility** TCC approval.
- Apple Events to other apps require **Automation** TCC consent per target app.
- `AEDeterminePermissionToAutomate(_:checkDestinationService:askUserIfNeeded:)` — check/request Automation authorization; call on background thread (blocks waiting for user).

**MDM / Reset:**
- `Privacy Preferences Policy Control` MDM payload extended with new service names for Catalina protected resources.
- `tccutil reset <ServiceName>` — reset consent status during development.

## APIs & Frameworks

**Core Graphics**
- `CGDisplayStream` — returns nil without screen recording approval
- `CGWindowListCreateImage(_:_:_:_:)` — returns nil for other apps' windows without approval
- `CGWindowListCopyWindowInfo(_:_:)` — filters `kCGWindowName` / `kCGWindowSharingState` without approval; no prompt triggered
- `CGEventTapCreate` — requires Input Monitoring (listen) or Accessibility (modify) approval

**IOKit / HID**
- `IOHIDCheckAccess(kIOHIDRequestTypeListenEvent)` — check keyboard monitoring authorization
- `IOHIDRequestAccess(kIOHIDRequestTypeListenEvent)` — request keyboard monitoring approval dialog
- `IOHIDCheckAccess(kIOHIDRequestTypePostEvent)` — check Accessibility (event synthesis) authorization
- `IOHIDRequestAccess(kIOHIDRequestTypePostEvent)` — request Accessibility approval dialog

**NSEvent**
- `NSEvent.addLocalMonitorForEvents(matching:handler:)` — monitor own app's events (no approval needed)

**AppKit**
- `NSOpenPanel` / `NSSavePanel` — now out-of-process; `-ok:` programmatic dismiss removed **[CHANGED]**
- `NSOpenSavePanelDelegate` — `panel(_:shouldEnable:)`, `panel(_:validate:error:)` changed behavior **[CHANGED]**
- `NSFileCoordinator` with `NSFilePresenter` — sidecar file access via `primaryPresentedItemURL` / `presentedItemURL`
- `NSFilePresenter.primaryPresentedItemURL: URL?` — primary (user-selected) file URL

**Foundation**
- `FileManager.trashItem(at:resultingItemURL:)` — move to Trash; no Full Disk Access needed
- `FileManager.isReadableFile(atPath:)` / `isWritableFile(atPath:)` — test access without triggering prompt

**Apple Events**
- `AEDeterminePermissionToAutomate(_:checkDestinationService:askUserIfNeeded:)` — returns `OSStatus` (`.permitted`, `.userCanceled`, `.notInstalled`, etc.)

**Info.plist keys**
- `CFBundleDocumentTypes` → `NSIsRelatedItemType = YES` — declare sidecar file extension for related-item access
- Purpose strings for protected locations (optional but recommended)

## Code Highlights

```swift
// Safe screen recording check — no approval needed for desktop background
func desktopBackgroundWindowIDs() -> [CGWindowID] {
    let windowList = CGWindowListCopyWindowInfo(.optionOnScreenOnly, kCGNullWindowID) as? [[String: Any]] ?? []
    let desktopLevel = CGWindowLevelForKey(.desktopWindow)
    return windowList.compactMap { info -> CGWindowID? in
        guard let level = info[kCGWindowLayer as String] as? Int,
              level == desktopLevel,
              let wid = info[kCGWindowNumber as String] as? CGWindowID else { return nil }
        return wid
        // Note: NOT filtering by kCGWindowName — that would require Screen Recording approval
    }
}

// Check keyboard monitoring authorization without prompting
let status = IOHIDCheckAccess(kIOHIDRequestTypeListenEvent)
// status: .granted, .denied, .notDetermined

// NSFileCoordinator sidecar access (Info.plist: NSIsRelatedItemType=YES for .srt)
class MoviePresenter: NSObject, NSFilePresenter {
    var primaryPresentedItemURL: URL?     // the .mp4 the user selected
    var presentedItemURL: URL?            // the .srt sidecar
    var presentedItemOperationQueue = OperationQueue.main
}

let presenter = MoviePresenter()
presenter.primaryPresentedItemURL = movieURL
presenter.presentedItemURL = subtitleURL
let coordinator = NSFileCoordinator(filePresenter: presenter)
coordinator.coordinate(readingItemAt: subtitleURL, options: [], error: nil) { url in
    // access subtitle file here — no consent prompt needed
}

// Check Apple Events authorization
let status = AEDeterminePermissionToAutomate(keynoteTarget, typeWildCard, keyWildCard, false)
```

## Takeaways
- **All new software must be notarized** (signed after June 1, 2019) — Developer ID alone is no longer sufficient for macOS Catalina.
- Never break a bundle signature at runtime; if an app updates itself, the result must be a properly signed and notarized bundle.
- Screen recording, keyboard monitoring, Desktop/Documents/Downloads/iCloud Drive/Trash access, and automation of other apps all require explicit TCC consent in Catalina — test each one and provide purpose strings in Info.plist.
- Prefer standard Finder/Open Panel interactions to infer user intent rather than triggering blanket consent prompts; use `NSFileCoordinator` for sidecar files.
- Open/Save panels are now out-of-process — remove any subclass or programmatic dismissal code.

---
_Source: WWDC19 Session 701 page (abstract, chapter summaries, code samples, and resource links)._
