# Enabling Your App for CarPlay
**WWDC17 · Session 719** · [Watch](https://developer.apple.com/videos/play/wwdc2017/719/)

_Platforms:_ iOS 11, CarPlay

## Overview
This session covers how to integrate iOS apps into CarPlay across the three supported categories: audio apps, messaging/VoIP calling apps, and automaker apps. Each category uses a distinct approach — audio apps provide a data source and navigation hierarchy rendered by the CarPlay system, messaging/VoIP apps operate exclusively through SiriKit and CallKit with CarPlay-specific entitlements, and automaker apps are the only category permitted to display a fully custom UIKit interface on the CarPlay screen.

The session walks through the entitlement request process, required frameworks and delegate implementations, content limits for vehicles in motion, the new playback rate display in iOS 11, CarPlay notification setup for messaging apps, and the protocol-matching mechanism that determines which automaker apps appear on which vehicles. It also covers data protection caveats (apps run while the phone is locked), audio session best practices, and debugging via Xcode 9 wireless debugging.

## Key Topics

- **App categories** — three CarPlay-integrated categories: audio, messaging/VoIP, automaker; all require a CarPlay-specific entitlement requested via the CarPlay developer portal.
- **All apps in CarPlay** — even non-integrated apps: don't play audio unless explicitly requested; use correct AVAudioSession modes (spoken audio protocol, ducking behavior for navigation guidance).
- **Audio apps** — implement `MPPlayableContentDataSource` and `MPPlayableContentDelegate`; respond to `MPRemoteCommandCenter` commands; update `MPNowPlayingInfoCenter`; content hierarchy uses `NSIndexPath` (not the `UITableView` meaning).
- **Content limits** — `MPPlayableContentManager.contentLimitsEnforced` and context properties (`enforcedContentItemsCount`, `enforcedContentTreeDepth`) for vehicles restricting content while in motion.
- **Tabs for audio apps** — add `UIBrowsableContentSupportSectionBrowsing` to Info.plist; maximum four tabs with short titles; tab images rendered as template images.
- **Playback rate in Now Playing (new iOS 11)** — add `MPNowPlayingInfoPropertyDefaultPlaybackRate` to `MPNowPlayingInfoCenter`; respond to `MPRemoteCommandCenter.changePlaybackRateCommand` with supported rate array.
- **Messaging apps** — implement `INSendMessageIntent`, `INSearchForMessagesIntent`, `INSentMessageAttributeIntent`; sign with messaging CarPlay entitlement.
- **VoIP calling apps** — implement `INStartAudioCallIntent`, `INSearchCallHistoryIntent`; implement CallKit (`CXProvider`, `CXCallController`); sign with VoIP CarPlay entitlement.
- **CarPlay notifications** — include `.carPlay` option in `UNUserNotificationCenter.requestAuthorization`; create a dedicated `UNNotificationCategory` with `.allowInCarPlay` option and a `INSearchForMessagesIntentIdentifier` intent; this category must be used only for messages, not other features.
- **Automaker apps** — custom UIKit UI on CarPlay screen; signed with CarPlay protocols entitlement matching reverse-DNS protocol names declared by the vehicle; communicate via `ExternalAccessory` framework; appear only on vehicles whose declared protocols overlap with the app's entitlement.
- **Automaker UIScreen setup** — observe `UIScreenDidConnect`/`UIScreenDidDisconnect`; filter for `.carPlay` `UIUserInterfaceIdiom`; create `UIWindow` for CarPlay screen; set `rootViewController` to custom `UIViewController`.
- **SiriKit intents for automaker** — commands intents (lock vehicle, check fuel level, sound horn) available to all apps; CarPlay-only intents (climate, defroster, seat heater, radio tuning, audio source) require a CarPlay connection and automaker entitlement.
- **Data protection warning** — apps run while iPhone is passcode-locked in CarPlay; files/keychains using `NSFileProtectionComplete` or `NSFileProtectionCompleteUnlessOpen` may be unavailable; use `NSFileProtectionCompleteUntilFirstUserAuthentication` instead.

## APIs & Frameworks

**MediaPlayer**
- `MPPlayableContentDataSource` — provides content items at `NSIndexPath`
- `MPPlayableContentDelegate` — initializes playback for selected items
- `MPPlayableContentManager` — manages content limits (`contentLimitsEnforced`, context properties)
- `MPRemoteCommandCenter` — handle play, pause, skip, shuffle, repeat, `changePlaybackRateCommand` **[NEW in iOS 11]**
- `MPNowPlayingInfoCenter` — set metadata dictionary; `MPNowPlayingInfoPropertyDefaultPlaybackRate` **[NEW]**
- `MPNowPlayingInfoPropertyPlaybackRate` **[NEW]** — display current rate in Now Playing

**SiriKit / Intents**
- `INSendMessageIntent` — required for messaging CarPlay
- `INSearchForMessagesIntent` — required for messaging CarPlay
- `INSentMessageAttributeIntent` — mark messages read
- `INStartAudioCallIntent` — required for VoIP CarPlay
- `INSearchCallHistoryIntent` — required for VoIP CarPlay
- `INLockDoorIntent`, `INGetCarFuelLevelIntent`, `INActivateCarSignalIntent` — automaker commands intents
- `INSetClimateSettingsInCarIntent`, `INSetDefrosterSettingsInCarIntent`, `INSetSeatSettingsInCarIntent` — automaker CarPlay-only climate intents
- `INSetRadioStationIntent`, `INSetAudioSourceInCarIntent` — automaker CarPlay-only radio intents

**CallKit**
- `CXProvider` — report incoming calls, handle answer/end/mute/hold actions
- `CXCallController` — request actions programmatically

**UserNotifications**
- `UNUserNotificationCenter.requestAuthorization(options:)` — include `.carPlay`
- `UNNotificationCategory` — create with `.allowInCarPlay` option; set `intentIdentifiers` to `[INSearchForMessagesIntentIdentifier]`

**ExternalAccessory**
- `EAAccessory`, `EASession` — communicate with vehicle hardware from automaker apps
- `EAAccessoryManager` — observe accessory connect/disconnect events

**UIKit (automaker)**
- `UIScreen` — detect `.carPlay` idiom screen
- `UIWindow(screen:)` — create window on CarPlay screen
- `UIButtonTypeSystem` — recommended button style for CarPlay consistency
- `UITableViewController` — used in automaker apps; may limit row count on motion
- `UIFocusEnvironment` — handle hardware navigation device focus for knob-based head units

**Info.plist Key**
- `UIBrowsableContentSupportSectionBrowsing` — enables tab bar UI in CarPlay audio apps

## Code Highlights

Playback rate support (iOS 11 new):
```swift
MPNowPlayingInfoCenter.default().nowPlayingInfo = [
    MPNowPlayingInfoPropertyDefaultPlaybackRate: 1.0
]
MPRemoteCommandCenter.shared().changePlaybackRateCommand.supportedPlaybackRates = [0.5, 1.0, 1.5, 2.0]
MPRemoteCommandCenter.shared().changePlaybackRateCommand.addTarget { event in
    let rates: [Double] = [0.5, 1.0, 1.5, 2.0]
    let currentIndex = rates.firstIndex(of: player.rate) ?? 1
    player.rate = Float(rates[(currentIndex + 1) % rates.count])
    return .success
}
```

Automaker CarPlay screen setup:
```swift
var carWindow: UIWindow?

func updateCarWindow() {
    guard let carScreen = UIScreen.screens.first(where: {
        $0.traitCollection.userInterfaceIdiom == .carPlay
    }) else {
        carWindow = nil
        return
    }
    carWindow = UIWindow(frame: carScreen.bounds)
    carWindow?.screen = carScreen
    carWindow?.rootViewController = CarViewController()
    carWindow?.makeKeyAndVisible()
}
// Call in applicationDidFinishLaunching, and on UIScreenDidConnect/DidDisconnect
```

## Takeaways

- Request the appropriate CarPlay entitlement early via the CarPlay developer portal — without it, the app icon won't appear on the CarPlay Home screen.
- Audio apps must handle `MPPlayableContentManager` content limits to avoid showing deep hierarchies or long lists when the vehicle enforces driving restrictions.
- Messaging/VoIP apps need both the CarPlay entitlement and a dedicated `UNNotificationCategory` with `.allowInCarPlay` set to show notifications in CarPlay; message content must never appear in the notification itself.
- Automaker apps are protocol-matched to vehicles — declare only the protocols the vehicle actually supports in both the app entitlement and vehicle firmware, and query supported protocols at launch to show only the appropriate UI.

---
_Source: WWDC17 Session 719 page (abstract, transcript, and resource links)._
