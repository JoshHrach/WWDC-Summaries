# Get to know the ManagedApp framework

**Session ID:** 203  
**WWDC Year:** 2025  
**Folder:** `13-system-frameworks-and-design`  
**URL:** https://developer.apple.com/videos/play/wwdc2025/203/

---

## Overview

This session introduces the new ManagedApp framework (iOS 26, iPadOS 26, macOS 26), which gives enterprise and education apps a Swift-native way to read MDM (Mobile Device Management) configuration, respond to MDM commands, and report status back to the MDM server — all without leaving the app process to use legacy Managed App Configuration plist APIs. The session walks through the full lifecycle: reading managed configuration keys pushed by an MDM profile, writing managed feedback/status back to MDM, implementing graceful configuration updates at runtime, and gating features behind MDM-controlled flags.

---

## Key Topics

- What ManagedApp replaces: legacy `NSUserDefaults(suiteName: "com.apple.configuration.managed")` plist approach
- Reading MDM-pushed configuration with `ManagedAppConfiguration`
- Writing feedback/status to the MDM server with `ManagedAppFeedback`
- Responding to live configuration changes via `ManagedAppConfiguration.updates` async sequence
- Restricting app features based on MDM-controlled boolean keys
- Declarative device management (DDM) integration
- Testing managed configuration in Xcode Simulator and with Apple Configurator

---

## APIs & Frameworks

- **ManagedApp** framework (`import ManagedApp`) – **[NEW]** (iOS 26, iPadOS 26, macOS 26) Swift-native framework for MDM-managed app configuration and feedback.
- **`ManagedAppConfiguration`** – **[NEW]** Observable class providing typed access to MDM-pushed configuration key-value pairs.
- **`ManagedAppConfiguration.current`** – **[NEW]** Singleton property returning the current configuration snapshot.
- **`ManagedAppConfiguration.current.value(forKey:)`** – **[NEW]** Generic method returning a typed value for a given MDM configuration key; returns `nil` if the key is absent or the type mismatches.
- **`ManagedAppConfiguration.updates`** – **[NEW]** `AsyncSequence` of `ManagedAppConfiguration` values; iterate with `for await` to respond to live MDM pushes without relaunching the app.
- **`ManagedAppFeedback`** – **[NEW]** Class for writing key-value status pairs back to the MDM server (surfaced in the MDM console as managed app feedback).
- **`ManagedAppFeedback.current.setValue(_:forKey:)`** – **[NEW]** Sets a typed feedback value; changes are batched and uploaded to MDM on a system-determined schedule.
- **`ManagedAppConfiguration.isManaged`** – **[NEW]** Bool property; `true` when the app is running under MDM management; use to conditionally show managed-mode UI.
- **`@ManagedConfiguration`** property wrapper – **[NEW]** SwiftUI-compatible property wrapper that binds a SwiftUI view property to an MDM configuration key, updating automatically on MDM push.
- **Declarative Device Management (DDM)** – Supported via the same `ManagedAppConfiguration` API; DDM-pushed declarations appear as configuration keys with no additional app changes required.
- **`com.apple.configuration.managed` UserDefaults** – Legacy API; still functional but not recommended for new development.

---

## Code Highlights

Reading a typed configuration value:
```swift
import ManagedApp

let config = ManagedAppConfiguration.current
let serverURL: URL? = config.value(forKey: "serverURL")
let maxUploadSizeMB: Int = config.value(forKey: "maxUploadSizeMB") ?? 50
let featuresEnabled: Bool = config.value(forKey: "advancedFeaturesEnabled") ?? false
```

Responding to live configuration changes:
```swift
Task {
    for await updatedConfig in ManagedAppConfiguration.updates {
        let newURL: URL? = updatedConfig.value(forKey: "serverURL")
        await reconfigureNetworkStack(url: newURL)
    }
}
```

Writing feedback back to MDM:
```swift
ManagedAppFeedback.current.setValue("2025-06-15T10:30:00Z", forKey: "lastSyncTimestamp")
ManagedAppFeedback.current.setValue(true, forKey: "onboardingComplete")
```

SwiftUI property wrapper usage:
```swift
struct SettingsView: View {
    @ManagedConfiguration("serverURL") var serverURL: URL?
    @ManagedConfiguration("theme") var theme: String = "default"

    var body: some View {
        Text("Server: \(serverURL?.absoluteString ?? "Not configured")")
    }
}
```

---

## Takeaways

- The ManagedApp framework replaces the clunky `NSUserDefaults(suiteName: "com.apple.configuration.managed")` pattern with a proper Swift-native, type-safe, async-aware API.
- `ManagedAppConfiguration.updates` enables live configuration changes without requiring app restart — critical for enterprise apps where MDM admins push policy changes during the workday.
- `ManagedAppFeedback` closes the loop: apps can report status and compliance data back to the MDM console, replacing custom back-channel reporting.
- The `@ManagedConfiguration` property wrapper makes MDM-driven feature flags trivially easy to integrate into SwiftUI views.
- Test managed configuration in Simulator by creating a `.mobileconfig` profile with `com.apple.managedapp.configuration` payload and installing it via Apple Configurator.
- Check `ManagedAppConfiguration.isManaged` before rendering any MDM-specific UI to avoid a jarring experience for App Store users who are not MDM-managed.
