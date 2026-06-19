# What's new in assessment on macOS
**WWDC26 · Session 230** · [Watch](https://developer.apple.com/videos/play/wwdc2026/230/)

_Platforms:_ macOS 27

## Overview
The Automatic Assessment Configuration (AAC) framework on macOS 27 gains a substantial set of new APIs that allow education apps to create a granularly controlled, locked-down testing environment. Rather than building device-management workarounds, apps can now rely entirely on AAC to enforce a secure exam experience — from pre-flight device checks through post-exam cleanup.

Five capability areas are covered: precondition checks (validating the device meets integrity requirements before an exam starts), accessibility restrictions (selectively allowing accommodations while blocking features that could be misused), system experience customization (controlling the Menu Bar, Dock, input methods, and Finder access), application launch restrictions (limiting which processes run), and best practices for adopting the framework responsibly.

A key theme is minimal restriction: the framework's APIs exist to enable equitable, reliable testing, not to over-lock devices. Apple recommends allowing all needed accessibility features and blocking only what is genuinely necessary for exam integrity.

## Key Topics

### Precondition Checks
New Boolean properties on `AEAssessmentConfiguration` allow apps to require that a Mac has System Integrity Protection enabled, is MDM-enrolled, has only a single signed-in user, uses a standard (not admin) account, and has disabled features like Lockdown Mode and iCloud Private Relay. These checks fail gracefully before the session starts rather than silently permitting a non-hardened device.

### Accessibility Restrictions
Per-feature toggles let examiners allow approved accommodations (VoiceOver, Hover Text, Spoken Content, Voice Control, Live Speech, Background Sounds, Zoom, Alternative Input Methods) while blocking features that accept user-generated content (Switch Control in specific modes). The principle is to allow by default and restrict only where needed.

### System Experience Customization
Fine-grained control over the Menu Bar (show/hide, allowed items from `AEAssessmentMenuBarItem`), the Apple menu (allowed items from `AEAssessmentAppleMenuItem`), the Dock, input technologies (Dictation, AutoFill, structural input, emoji keyboard), and Finder directory access via `allowedDirectoriesAndFiles`.

### Application Launch Restrictions
`allowOnlyParticipantsToRun = true` restricts process execution to the exam app and explicitly allowlisted participant apps. `allowsUserScriptExecution = false` blocks Shortcuts automations and Automator scripts from running during the session.

### Best Practices
- Rely on AAC APIs; do not build redundant restrictions outside the framework
- Restrict the minimum necessary — over-restriction harms accessibility and user experience
- Treat accessibility support as a requirement, not an optional
- Register for session transition callbacks to handle graceful start/end flows
- Test with realistic exam workflows, not just isolated API calls

## APIs & Frameworks

### Automatic Assessment Configuration
- `AEAssessmentConfiguration` — primary configuration object; all new properties listed below
- **Precondition checks (NEW)**
  - `.requiresSIP: Bool` — require System Integrity Protection to be enabled
  - `.requiresManagedDevice: Bool` — require MDM enrollment
  - `.requiresSingleUser: Bool` — require exactly one signed-in account
  - `.requiresUserAccountType: AEAssessmentUserAccountType` — e.g., `.standard` to block admin accounts
  - `.allowLockdownMode: Bool` — set `false` to block Lockdown Mode devices
  - `.allowPrivateRelay: Bool` — set `false` to block iCloud Private Relay
- **Accessibility restrictions (NEW)**
  - `.allowsAccessibilityVoiceOver: Bool`
  - `.allowsAccessibilitySwitchControl: Bool`
  - `.allowsAccessibilityAlternativeInputMethods: Bool`
  - `.allowsAccessibilityBackgroundSounds: Bool`
  - `.allowsAccessibilityHoverText: Bool`
  - `.allowsAccessibilityLiveSpeech: Bool`
  - `.allowsAccessibilitySpokenContent: Bool`
  - `.allowsAccessibilityVoiceControl: Bool`
  - `.allowsAccessibilityZoom: Bool`
- **System experience customization (NEW)**
  - `.allowsMenuBar: Bool` — show or hide the Menu Bar
  - `.allowedMenuBarItems: [AEAssessmentMenuBarItem]` — e.g., `.battery`, `.clock`, `.volume`
  - `.allowedAppleMenuItems: [AEAssessmentAppleMenuItem]` — e.g., `.sleep`
  - `.allowsDock: Bool` — show or hide the Dock
  - `.allowsDictation: Bool`
  - `.allowsAutoFill: Bool`
  - `.allowsStructuralInput: Bool` — structural/predictive input methods
  - `.allowsEmojiKeyboard: Bool`
  - `.allowedDirectoriesAndFiles: [URL]` — Finder access allowlist
- **Application launch restrictions (NEW)**
  - `.allowOnlyParticipantsToRun: Bool` — restrict execution to allowlisted apps
  - `.allowsUserScriptExecution: Bool` — block Shortcuts/Automator scripts
- `AEAssessmentMenuBarItem` **[NEW]** — enum of allowed Menu Bar status items
- `AEAssessmentAppleMenuItem` **[NEW]** — enum of allowed Apple menu items
- `AEAssessmentUserAccountType` **[NEW]** — enum; `.standard`, `.administrator`
- `AEAssessmentSession` — existing session type; start/stop the locked environment
- Session transition callbacks (existing) — register for begin/end notifications

## Code Highlights

Device integrity prechecks:
```swift
let config = AEAssessmentConfiguration()
config.requiresSIP = true
config.requiresManagedDevice = true
config.requiresSingleUser = true
config.requiresUserAccountType = .standard
config.allowLockdownMode = false
config.allowPrivateRelay = false
```

Accessibility configuration:
```swift
config.allowsAccessibilityVoiceOver = true
config.allowsAccessibilitySwitchControl = false
config.allowsAccessibilityZoom = true
// ... set other per-feature toggles
```

Menu Bar and input customization:
```swift
config.allowsMenuBar = true
config.allowedMenuBarItems = [.battery, .clock, .volume]
config.allowedAppleMenuItems = [.sleep]
config.allowsDictation = false
config.allowsAutoFill = false
config.allowedDirectoriesAndFiles = [URL(fileURLWithPath: "~/Documents/")]
```

## Takeaways
- macOS 27 AAC gains precondition device checks, per-feature accessibility toggles, Menu Bar/Dock customization, Finder access control, and process allowlisting — covering nearly every exam integrity requirement via framework API.
- Restrict only what is necessary; all accessibility features default to allowed and should be unblocked where possible for equitable testing.
- `requiresSIP`, `requiresManagedDevice`, and `requiresUserAccountType = .standard` are the recommended baseline prechecks.
- `allowOnlyParticipantsToRun = true` combined with `allowsUserScriptExecution = false` provides strong process isolation without custom kernel extensions.

---
_Source: WWDC26 Session 230 page (abstract, chapter summaries, code samples, and resource links)._
