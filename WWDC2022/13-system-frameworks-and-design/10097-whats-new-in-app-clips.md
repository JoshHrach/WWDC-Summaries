# What's New in App Clips
**WWDC22 · Session 10097** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10097/)

_Platforms:_ iOS 16, iPadOS 16

## Overview
This session covers four improvements to App Clips in iOS 16: a larger size limit (15 MB, up from 10 MB), a built-in on-device diagnostics tool for validating App Clip and universal link configuration, CloudKit public database read access for App Clips, and direct keychain-to-keychain transfer when a user installs the full app. It also introduces the App Store Connect App Clip experiences REST API for automating advanced App Clip experience management at scale.

## Key Topics

### Increased Size Limit (15 MB)
In iOS 16, the App Clip binary size limit increases from 10 MB to 15 MB. To use the new limit, set the App Clip's minimum deployment target to iOS 16. Apps targeting iOS 15 and earlier must remain under 10 MB. On-demand resources can be downloaded after launch regardless of size.

### App Clip Diagnostics Tool (New in iOS 16)
A new on-device validation tool is available in **Settings → Developer → App Clips Testing → Diagnostics**. Enter any URL to check:
- App Clip experience configuration (custom codes, Safari banner, iMessage)
- Universal Links associated domains configuration

Each diagnostic item shows a green checkmark on success or a detailed error with a link to documentation on failure. Enabling Developer Settings requires connecting the device to Xcode.

### CloudKit Public Database Access (New in iOS 16)
App Clips can now read from a CloudKit public database — the same container used by the full app. This allows shared data access across App Clip and app with no duplication. Restrictions to maintain the data-deletion promise:
- Read-only access to public database only
- No write access to CloudKit
- No cloud documents or key-value store

Enable in Xcode: App Clip target → Signing & Capabilities → add iCloud capability → select CloudKit container.

### Keychain Migration (New in iOS 16)
When a user installs the full app after using the App Clip, keychain items stored by the App Clip are automatically transferred to the full app's keychain. Previously, developers had to use an app group container as an intermediate store. Restrictions:
- Shared keychain groups not supported
- iCloud Keychain not supported (items are App Clip-local only)

The API for reading and writing keychain items is identical in both the App Clip and the full app. Use `kSecAttrLabel` to tag items so the full app can identify their origin.

### App Store Connect App Clip Experiences API
A REST API to automate creating and managing advanced App Clip experiences. Three-step workflow:
1. `GET /v1/apps/{appId}/appClips` — look up the App Clip resource ID by bundle ID
2. `POST /v1/appClipAdvancedExperienceImages` — upload a header image; receive its resource ID
3. `POST /v1/appClipAdvancedExperiences` — create the experience with attributes (action, business category, language, link, place/map coordinates), relationships (App Clip ID, header image ID), and localizations (title, subtitle)

## APIs & Frameworks

**App Clips (iOS 16)**
- Size limit: 15 MB **[NEW]** (was 10 MB); requires iOS 16 minimum deployment target
- App Clip Diagnostics — Developer Settings → App Clips Testing → Diagnostics **[NEW]**

**CloudKit (iOS 16 for App Clips)**
- `CKContainer.default()` / `CKContainer(identifier:)` — access the CloudKit container
- `CKContainer.publicCloudDatabase` — read-only public database **[NEW for App Clips]**
- `CKDatabase.record(for: CKRecord.ID)` — async fetch of a public record

**Security / Keychain (iOS 16)**
- `SecItemAdd(_:_:)` — add a keychain item in App Clip; automatically migrated to full app on install **[NEW behavior]**
- `SecItemCopyMatching(_:_:)` — fetch keychain items; same code works in App Clip and full app
- `kSecAttrLabel` — recommended tag to identify App Clip-origin items

**App Store Connect REST API**
- `GET /v1/apps/{id}/appClips` — list App Clips and retrieve resource IDs
- `POST /v1/appClipAdvancedExperienceImages` — upload header image
- `POST /v1/appClipAdvancedExperiences` — create advanced App Clip experience with place, map action, localizations

## Code Highlights

Reading from CloudKit public database in an App Clip:
```swift
let container = CKContainer.default()
let publicDatabase = container.publicCloudDatabase
let record = try await publicDatabase.record(for:
    CKRecord.ID(recordName: "A928D582-9BB6-E9C5-7881-E4EAF615E0CD"))

if let title = record["Title"] as? String,
   let description = record["Description"] as? String {
    print("Fetched: \(title) – \(description)")
}
```

Writing a keychain item in an App Clip (migrated to full app on install):
```swift
let addQuery: [String: Any] = [
    kSecClass as String: kSecClassGenericPassword,
    kSecValueData as String: "auth-token".data(using: .utf8)!,
    kSecAttrLabel as String: "myapp-appclip"   // tag for full app to identify origin
]
SecItemAdd(addQuery as CFDictionary, nil)

// Reading in full app (same code):
var readQuery: [String: Any] = [
    kSecClass as String: kSecClassGenericPassword,
    kSecReturnAttributes as String: true,
    kSecAttrLabel as String: "myapp-appclip",
    kSecReturnData as String: true
]
var result: AnyObject?
SecItemCopyMatching(readQuery as CFDictionary, &result)
```

App Store Connect REST API — create advanced App Clip experience (abbreviated):
```json
POST /v1/appClipAdvancedExperiences
{
  "data": {
    "type": "appClipAdvancedExperiences",
    "attributes": {
      "action": "OPEN",
      "businessCategory": "FOOD_AND_DRINK",
      "defaultLanguage": "EN",
      "link": "https://example.com/restaurant/simply_salad",
      "place": {
        "names": ["Caffe Macs"],
        "mapAction": "RESTAURANT_ORDER_FOOD",
        "displayPoint": {
          "coordinates": { "latitude": 37.33611, "longitude": -122.00731 },
          "source": "CALCULATED"
        }
      }
    },
    "relationships": {
      "appClip": { "data": { "type": "appClip", "id": "<appClipResourceId>" } },
      "headerImage": { "data": { "type": "appClipAdvancedExperienceImages", "id": "<imageId>" } }
    }
  }
}
```

## Takeaways
- The 15 MB size limit (iOS 16+) gives more headroom for App Clips without sacrificing instant-launch characteristics; use on-demand resources for anything beyond the limit.
- The new on-device diagnostics tool in Developer Settings eliminates guesswork about App Clip and universal link configuration, showing exactly which step in the validation chain fails.
- App Clips can now read from the same CloudKit public container as the full app — use the identical `CKContainer` / `CKDatabase` API with no code changes needed.
- Keychain items written by an App Clip are automatically migrated to the full app on install in iOS 16; tag items with `kSecAttrLabel` so the app can distinguish them.

---
_Source: WWDC22 Session 10097 page (abstract, transcript, and code samples)._
