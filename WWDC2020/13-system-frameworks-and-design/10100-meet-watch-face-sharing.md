# Meet Watch Face Sharing
**WWDC20 · Session 10100** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10100/)

_Platforms:_ watchOS 7, iOS 14

## Overview
Watch Face Sharing is a new watchOS 7 / iOS 14 feature that lets users share fully configured Apple Watch faces — including colors, styles, photos, and complications — with friends, from apps, or through websites. When a shared face includes an app's complication and the recipient does not have that app installed, they are prompted to install it from the App Store; complication sample data is embedded in the face file for preview even without the app.

The developer-facing addition is `CLKWatchFaceLibrary.addWatchFace(at:completionHandler:)` **[NEW]**, which can be called from both watchOS and iOS apps to programmatically prompt the user to add a pre-configured `.watchface` file to their Apple Watch. The session also covers website distribution, design resources, and fallback face handling for Series 3 compatibility.

## Key Topics

### How Watch Face Sharing Works
- Face files (`.watchface`) contain the face's entire configuration: style, color, photos, complications, and complication user info dictionaries
- Users can share directly from the watch (long-press face → share button) or from the iOS Watch app (share icon)
- Recipients see an inline preview and tap to add the face; if the sending app is not installed, they are prompted to get it from the App Store
- Complication user info dictionaries and user activities from the sender are embedded; these can specify initial complication content (e.g., which city's data to show)
- Before sharing, use the "Include without data" option to strip user-specific data from complications while keeping the complication slot

### In-App Face Sharing (CLKWatchFaceLibrary)
- Bundle the `.watchface` file in the app target (iOS or watchOS)
- Obtain a preview image by sharing the face via email from the iOS Watch app; embed the preview image in asset catalog for display in UI
- Call `CLKWatchFaceLibrary().addWatchFace(at:completionHandler:)` with the bundle URL to the face file
- Check the error for `CLKWatchFaceLibrary.ErrorCode.faceNotAvailable` — this occurs when the face style is incompatible with the user's watch (e.g., a newer face on Series 3)
- Provide a fallback `.watchface` file using a face compatible with Series 3 (e.g., Modular); retry in the error handler
- Use `WCSession.isPaired` (WatchConnectivity) to detect whether the user has a paired watch before showing the face-sharing UI in an iOS app
- The app **must be published on the App Store** before sharing a face that includes its complication — App Store Connect ID must be embedded in the face file at share time

### Website Distribution
- Serve `.watchface` files with MIME type `application/vnd.apple.watchface`
- Safari on iOS presents an "Add to Apple Watch" prompt when the user downloads the file
- Include a watch face preview image and the official "Add Apple Watch Face" button from Apple Design Resources
- Provide separate preview + button for each face when offering a Series 3-compatible fallback
- Face files can also be linked from QR codes and NFC tags

### Complication User Info
- `CLKComplication.userInfo` and `CLKComplication.userActivity` are embedded in the `.watchface` file
- When the face is added and the complication's app runs its first timeline request, it can read this info to configure initial content
- Supports multiple complications from the same app on a single face (new in watchOS 7)

### Best Practices
- Include sample complication data so the face preview is never blank for recipients
- Exclude private/personal user data before sharing by using "Include without data" for your complication slot
- Use WatchConnectivity's `isPaired` check in iOS before presenting face-sharing UI
- Obtain the preview image by emailing the face from the iOS Watch app; do not generate it programmatically
- Always provide a Series 3-compatible fallback face when the primary face requires a newer watch
- Nike and Hermès faces can only be generated and added on respective hardware

## APIs & Frameworks

- **ClockKit**
  - `CLKWatchFaceLibrary` **[NEW]** — manages watch face files
  - `CLKWatchFaceLibrary.addWatchFace(at:completionHandler:)` **[NEW]** — prompts the user to add a `.watchface` file; available on both watchOS and iOS (iOS 14+)
  - `CLKWatchFaceLibrary.ErrorCode` **[NEW]** — error codes returned in the completion handler
  - `CLKWatchFaceLibrary.ErrorCode.faceNotAvailable` **[NEW]** — face is not compatible with the user's current device (e.g., newer face on Series 3)
  - `CLKComplication.userInfo: [AnyHashable: Any]?` — data embedded in the watch face file and passed to the complication's timeline data source
- **WatchConnectivity**
  - `WCSession.isSupported()` — checks if the device can support a Watch session (returns `false` on iPad)
  - `WCSession.default.isPaired` — `Bool`; whether an Apple Watch is paired to the iPhone
  - `WCSession.default.activate()` — activates the session before reading properties
  - `WCSessionDelegate` — protocol; implement to receive session state updates

## Code Highlights

Detecting a paired watch in an iOS app:
```swift
var isPaired: Bool {
    if WCSession.isSupported() {
        let session = WCSession.default
        session.delegate = self
        session.activate()
        return session.isPaired
    }
    return false
}
```

Adding a watch face from the app bundle:
```swift
private func addFaceWrapper(withName: String) {
    guard let watchfaceURL = Bundle.main.url(forResource: withName, withExtension: "watchface") else { return }
    CLKWatchFaceLibrary().addWatchFace(at: watchfaceURL) { error in
        if let nsError = error as NSError?,
           nsError.code == CLKWatchFaceLibrary.ErrorCode.faceNotAvailable.rawValue {
            print("Face not available on this device: \(nsError)")
        }
        isLoading = false
    }
}
```

Providing a Series 3 fallback face:
```swift
private func addFaceWrapper(withName: String, fallbackName: String?) {
    guard let url = Bundle.main.url(forResource: withName, withExtension: "watchface") else { return }
    CLKWatchFaceLibrary().addWatchFace(at: url) { error in
        if let nsError = error as NSError?,
           nsError.code == CLKWatchFaceLibrary.ErrorCode.faceNotAvailable.rawValue,
           let name = fallbackName {
            // Primary face incompatible; try the fallback
            addFaceWrapper(withName: name, fallbackName: nil)
        }
        isLoading = false
    }
}

// Call with primary face and Series 3-compatible fallback
addFaceWrapper(withName: "ModularCompact-Coffee", fallbackName: "Modular-Coffee")
```

## Takeaways
- `CLKWatchFaceLibrary.addWatchFace(at:completionHandler:)` is the single new API for in-app face sharing; it works from both iOS and watchOS targets.
- Always bundle a fallback `.watchface` file compatible with Series 3 and handle `CLKWatchFaceLibrary.ErrorCode.faceNotAvailable` in the completion handler.
- Your app must already be published on the App Store before you can share a watch face containing your complication — the App Store Connect ID is embedded at share time.
- Use complication user info dictionaries to persist initial configuration (e.g., city, preferences) in the shared face so recipients get a meaningful first experience.

---
_Source: WWDC20 Session 10100 page (abstract, transcript, and code samples)._
