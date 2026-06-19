# Broaden Your Reach with Siri Event Suggestions
**WWDC20 · Session 10197** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10197/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11

## Overview
Siri Event Suggestions (introduced in iOS 13) automatically adds reservations to Calendar using on-device intelligence — enabling lock screen departure reminders, Maps quick-directions, Do Not Disturb suggestions, and check-in prompts. iOS 14 and macOS Big Sur bring three major expansions: new reservation categories (bus and boat), cross-platform support (macOS and Mac Catalyst), and a new web/email markup path using schema.org that works without an app.

The session covers the full integration surface: the Intents framework API for app donations, schema.org JSON-LD/Microdata markup for Safari and Mail, correct use of `reservationId`/`containerReference`/`itemReference`, a new `url` property on `INReservation` for the "Show in Safari" fallback on devices without the app, and developer tooling (Simulator support, Console app logging via the `siri-event-suggestions` category).

## Key Topics
- **New reservation categories (iOS 14)** **[NEW]** — `INBusReservation` and `INBoatReservation`; for scheduled, booked trips (not all-day passes).
- **macOS support (iOS 14 / macOS Big Sur)** **[NEW]** — Donate reservations via Intents framework from Mac Catalyst apps and native macOS apps; the system syncs reservations across devices via end-to-end encrypted iCloud when donation data is consistent.
- **Simulator support (iOS 14)** **[NEW]** — Siri Event Suggestions API now works in the iOS Simulator for development and testing.
- **Schema.org markup (iOS 14 / macOS Big Sur)** **[NEW]** — Embed JSON-LD or Microdata in website HTML or email; Safari and Mail parse it and donate to the system automatically; no app required. Requires domain registration at developer.apple.com and HTTPS (web) / DKIM (email).
- **`reservationStatus`** — Use `ReservationConfirmed` only when payment is processed; `ReservationCancelled` to automatically remove the Calendar event. Setting the same `reservationId` across modifications/cancellations lets Siri update the existing event.
- **`containerReference` vs. `itemReference`** — `containerReference` identifies all parts of a reservation (use the reservation number); `itemReference` must be unique per part (use ticket number or leg identifier for multi-leg trips). The system may launch the app with just one `itemReference` to show a single leg.
- **`INReservation.url`** **[NEW]** — `URL?` property on `INReservation`; when a Calendar event syncs to a device without the app, Calendar shows a "Show in Safari" button opening this URL.
- **Donation timing** — Donate when: displaying a list of upcoming reservations, viewing a single reservation detail, viewing a part of a reservation, or in response to a background update (Background App Refresh / silent push notifications).
- **Debugging** — Console app: filter by category `siri-event-suggestions`; new in iOS 14 / macOS Big Sur, the system emits structured log messages identifying common errors (duplicate `itemReference`, user hasn't acknowledged Calendar onboarding, past event end time).
- **Developer settings for markup testing** — `SuggestionsAllowAnyDomainForMarkup` (bypass domain registration) and `SuggestionsAllowUnverifiedSourceForMarkup` (bypass HTTPS/DKIM) in iOS Developer Settings or via `defaults write` on Mac.

## APIs & Frameworks

### SiriKit / Intents (iOS 14 / macOS Big Sur)
- **`INBusReservation`** **[NEW]** — `class INBusReservation: INReservation`; booked bus trip with specific departure time
- **`INBoatReservation`** **[NEW]** — `class INBoatReservation: INReservation`; booked boat/ferry trip with specific departure time
- **`INReservation.url`** **[NEW]** — `var url: URL?`; shown as "Show in Safari" in Calendar on devices without the app
- **`INReservation.reservationNumber`** — Used by Siri to match and update existing Calendar events across API donations and schema.org markup; must be consistent across all sources
- **`INReservation.itemReference`** — `INSpeakableString`; unique identifier for a single part within a reservation (e.g., one flight leg); used to launch the app to a specific part
- **`INReservation.containerReference`** — `INSpeakableString`; identifies the reservation as a whole (all parts); used alongside `itemReference` for app launch
- **`INGetReservationDetailsIntent`** — Delivered to the app when user taps "Show in App" in Calendar; contains `containerReference` and `itemReference`
- **`INInteraction`** — Wraps intent + response for donation; `INInteraction(intent:response:).donate(completion:)`
- **Existing reservation types** — `INRestaurantReservation`, `INFlightReservation`, `INTrainReservation`, `INLodgingReservation`, `INEventReservation`, `INRentalCarReservation`, `INTicketedEventReservation`

### Schema.org Markup (web / email)
- **`@type`** — e.g., `FoodEstablishmentReservation`, `FlightReservation`, `TrainReservation`, `LodgingReservation`, `BusReservation`, `BoatReservation`
- **`reservationStatus`** — `http://schema.org/ReservationConfirmed` or `http://schema.org/ReservationCancelled`
- **`reservationId`** — String; same as `INReservation.reservationNumber`; used to update/cancel the Calendar event
- **`startDate`** — ISO 8601 with time zone offset (e.g., `2020-06-26T19:30:00-07:00`)
- **`url`** — URL to reservation page; shown as "Show in Safari" link in Calendar

## Code Highlights

Donate a bus reservation (new in iOS 14):
```swift
let busTrip = INBusTrip()
// set departure/arrival time, bus name, etc.

let reservation = INBusReservation(
    itemReference: INSpeakableString(spokenPhrase: "Bus ABC-001"),
    reservationNumber: "ABC-001",
    bookingTime: Date(),
    reservationStatus: .confirmed,
    reservationHolderName: "John Appleseed",
    actions: nil,
    url: URL(string: "https://example.com/reservations/ABC-001"),
    busTrip: busTrip
)

let intent = INGetReservationDetailsIntent(
    containerReference: INSpeakableString(spokenPhrase: "My Bus Trips"),
    itemReferences: nil
)
let response = INGetReservationDetailsIntentResponse(code: .success, userActivity: nil)
response.reservations = [reservation]

let interaction = INInteraction(intent: intent, response: response)
interaction.donate { error in
    if let error = error { print("Donation failed: \(error)") }
}
```

Schema.org JSON-LD for a confirmed restaurant reservation:
```html
<script type="application/ld+json">
{
  "@context": "http://schema.org",
  "@type": "FoodEstablishmentReservation",
  "reservationStatus": "http://schema.org/ReservationConfirmed",
  "reservationId": "IWDSCA",
  "partySize": "2",
  "url": "https://example.com/reservations/IWDSCA",
  "reservationFor": {
    "@type": "FoodEstablishment",
    "name": "EPIC Steak",
    "startDate": "2020-06-26T19:30:00-07:00",
    "address": {
      "@type": "PostalAddress",
      "streetAddress": "369 The Embarcadero",
      "addressLocality": "San Francisco",
      "addressRegion": "CA",
      "postalCode": "95105"
    }
  }
}
</script>
```

Cancel the same reservation (same `reservationId`):
```json
{
  "reservationStatus": "http://schema.org/ReservationCancelled",
  "reservationId": "IWDSCA"
}
```

Enable markup testing without domain registration (macOS):
```shell
defaults write com.apple.suggestions SuggestionsAllowAnyDomainForMarkup -bool true
defaults write com.apple.suggestions SuggestionsAllowUnverifiedSourceForMarkup -bool true
```

Debug donations in Console (filter category):
```
Category filter: siri-event-suggestions
```

## Takeaways
- The new schema.org markup integration lets Safari and Mail donate reservations to Calendar with no app needed — embed JSON-LD or Microdata in your website's HTML or confirmation emails and register your domain at developer.apple.com.
- Always use the same `reservationId` (or `reservationNumber`) across all sources (app donation, website markup, email markup) — Siri uses it to update and cancel the existing Calendar event rather than creating duplicates.
- Set `itemReference` to a unique per-part identifier for multi-part reservations (multi-leg flights, round-trip trains) so the system can launch your app to the specific part the user wants to see.
- Adopt the new `INReservation.url` property so users on devices without your app get a "Show in Safari" link in Calendar — reservations sync via iCloud to all devices.
- Debug donation failures with the Console app filtered to category `siri-event-suggestions`; common errors (duplicate itemReference, unacknowledged Calendar onboarding, past event time) are now logged with clear messages.

---
_Source: WWDC20 Session 10197 page (abstract, transcript, and code samples)._
