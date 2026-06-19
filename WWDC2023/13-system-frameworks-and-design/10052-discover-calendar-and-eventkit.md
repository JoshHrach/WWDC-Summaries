# Discover Calendar and EventKit
**WWDC23 · Session 10052** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10052/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14

## Overview
This session by Apple engineer Adam covers the full range of ways an app can integrate with the system Calendar using EventKit and EventKitUI. It introduces a new privacy-preserving access level — write-only — and clarifies the updated access model in iOS 17 and macOS Sonoma. The session walks through four concrete use cases: adding events via EventKitUI, creating Siri Event Suggestions, writing events directly with write-only access, and fetching events with full access. It also covers implementing a Virtual Conference extension.

Calendar integration is a high-privacy area. The session emphasizes the principle of least privilege: use no-access APIs where possible, fall back to write-only when UI-based adding is insufficient, and request full access only when the core app feature genuinely requires reading existing events.

## Key Topics

### Access Levels (Updated in iOS 17) **[NEW]**
- **No access**: Apps can add events using EventKitUI (runs in a separate process in iOS 17 — no permission prompt needed) or Siri Event Suggestions (Intents framework, no UI, no prompt).
- **Write-only** **[NEW in iOS 17]**: App can save events directly via EventKit but cannot read any events (including its own). Requires `NSCalendarsWriteOnlyAccessUsageDescription` in Info.plist.
- **Full access**: App can fetch, modify, and delete existing events and manage calendars. Requires `NSCalendarsFullAccessUsageDescription`. Only appropriate when reading events is a core feature.

### Adding Events via EventKitUI (No Permission Required)
In iOS 17, `EKEventEditViewController` runs in a separate process — no calendar access required. Create an `EKEvent`, populate it, attach it to an `EKEventEditViewController`, present it. The user reviews details and taps Add.

### Siri Event Suggestions (No Permission Required)
Part of the Intents framework. Create an `INReservation` subclass (restaurant, hotel, flight, car rental, ticketed event), wrap it in an intent + response, create an `INInteraction`, and call `donate()`. Events appear in the Calendar inbox as suggestions.

### Write-Only Access **[NEW]**
`EKEventStore.requestWriteOnlyAccessToEvents()` — new async method. If granted, app can call `eventStore.save(_:span:)` directly. Must set `calendar`, `title`, `startDate`, and `endDate` explicitly (no defaults). Use `store.defaultCalendarForNewEvents` for the calendar property.

### Full Access
`EKEventStore.requestFullAccessToEvents()` — async method. After access, create a predicate with `predicateForEvents(withStart:end:calendars:)` and call `events(matching:)` to fetch. Use the shortest date range possible. Results are unordered; sort if needed.

### Virtual Conference Extension
A new Xcode extension target type. Subclass `EKVirtualConferenceProvider` and implement:
- `fetchAvailableRoomTypes()` — returns `[EKVirtualConferenceRoomTypeDescriptor]` with title and unique identifier for each room type. Appears in Calendar's location picker.
- `fetchVirtualConference(identifier:)` — returns `EKVirtualConferenceDescriptor` with one or more `EKVirtualConferenceURLDescriptor` objects (use Universal Links) and an optional details string.

## APIs & Frameworks

### EventKit
- `EKEventStore` — main access point; one per app **[NEW async request methods]**
- `EKEventStore.requestWriteOnlyAccessToEvents()` — new async method **[NEW]**
- `EKEventStore.requestFullAccessToEvents()` — new async method **[NEW]**
- `EKEventStore.requestAccess(to:completion:)` — legacy method (pre-iOS 17)
- `EKEventStore.defaultCalendarForNewEvents` — returns the system-default new-event calendar
- `EKEventStore.save(_:span:)` / `EKEventStore.save(_:span:commit:)` — saves an event
- `EKEventStore.predicateForEvents(withStart:end:calendars:)` — creates a fetch predicate
- `EKEventStore.events(matching:)` — fetches events matching a predicate
- `EKEvent` — event object with `title`, `startDate`, `endDate`, `timeZone`, `location`, `notes`, `calendar`
- `EKCalendar` — calendar with `title`, `cgColor`
- `EKSource` — calendar account grouping
- `EKVirtualConferenceProvider` — base class for virtual conference extensions **[NEW]**
- `EKVirtualConferenceRoomTypeDescriptor` — describes a room type in the location picker **[NEW]**
- `EKVirtualConferenceURLDescriptor` — wraps a join URL with optional title **[NEW]**
- `EKVirtualConferenceDescriptor` — returned from `fetchVirtualConference`; has URL descriptors and details string **[NEW]**

### EventKitUI
- `EKEventEditViewController` — event editor UI (runs out-of-process in iOS 17; no permission needed) **[UPDATED]**
- `EKEventEditViewDelegate` — protocol for add/cancel callbacks
- `EKEventViewController` — event detail UI
- `EKCalendarChooser` — calendar picker UI

### Intents (Siri Event Suggestions)
- `INReservation` and subclasses: `INRestaurantReservation`, `INHotelReservation`, `INFlightReservation`, `INRentalCarReservation`, `INTicketedEventReservation`
- `INSpeakableString` — reservation reference with spoken phrase
- `INDateComponentsRange` — start/end time range
- `INGetReservationDetailsIntent` / `INGetReservationDetailsIntentResponse`
- `INInteraction` — wraps intent + response
- `INInteraction.donate()` — submits suggestion to system

### Info.plist Keys (Updated)
- `NSCalendarsWriteOnlyAccessUsageDescription` **[NEW]**
- `NSCalendarsFullAccessUsageDescription` **[NEW]**
- `NSCalendarsUsageDescription` — legacy (pre-iOS 17)
- `NSContactsUsageDescription` — required for EventKitUI apps on pre-iOS 17

## Code Highlights

```swift
// Write-only: add event directly
let store = EKEventStore()
guard try await store.requestWriteOnlyAccessToEvents() else { return }
let event = EKEvent(eventStore: store)
event.calendar = store.defaultCalendarForNewEvents
event.title = "WWDC23 Keynote"
event.startDate = myEventStartDate
event.endDate = myEventEndDate
try eventStore.save(event, span: .thisEvent)
```

```swift
// Full access: fetch current month's events
guard try await store.requestFullAccessToEvents() else { return }
let interval = Calendar.current.dateInterval(of: .month, for: Date())!
let predicate = store.predicateForEvents(withStart: interval.start,
                                         end: interval.end, calendars: nil)
let events = store.events(matching: predicate)
    .sorted { $0.compareStartDate(with: $1) == .orderedAscending }
```

```swift
// Virtual conference extension
override func fetchAvailableRoomTypes() async throws -> [EKVirtualConferenceRoomTypeDescriptor] {
    [EKVirtualConferenceRoomTypeDescriptor(title: "My Room", identifier: "my_room")]
}

override func fetchVirtualConference(identifier: EKVirtualConferenceRoomTypeIdentifier)
    async throws -> EKVirtualConferenceDescriptor {
    EKVirtualConferenceDescriptor(title: nil,
                                  urlDescriptors: [EKVirtualConferenceURLDescriptor(title: nil, url: myURL)],
                                  conferenceDetails: "Enter code 12345 to join.")
}
```

## Takeaways
- In iOS 17, `EKEventEditViewController` runs out-of-process — use it to add single events with zero permission prompts.
- Write-only access (new in iOS 17) is ideal for apps that need to programmatically save events without reading any calendar data.
- Request full access only when reading existing events is genuinely a core feature; users grant it rarely.
- Implement a Virtual Conference extension to appear in Calendar's location picker and provide one-tap join URLs using Universal Links.

---
_Source: WWDC23 Session 10052 page (abstract, chapter summaries, code samples, and resource links)._
