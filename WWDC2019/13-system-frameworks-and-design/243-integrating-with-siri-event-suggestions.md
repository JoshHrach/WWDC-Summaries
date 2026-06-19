# Integrating with Siri Event Suggestions
**WWDC19 · Session 243** · [Watch](https://developer.apple.com/videos/play/wwdc2019/243/)

_Platforms:_ iOS 13, iPadOS 13 (SiriKit / Intents framework)

## Overview
iOS 13 introduces new SiriKit APIs that allow apps to donate reservation details directly to the system, enabling Siri to surface proactive suggestions throughout the OS: time-to-leave notifications on the lock screen, Siri suggestions in Maps with indoor directions to gates, automatic Calendar event creation, and targeted check-in shortcuts. Previously this intelligence was limited to reservations found in Mail and Messages; with Session 243's APIs, any app handling bookings — flights, hotels, restaurants, car rentals, ticketed events, train trips, and more — can participate.

The architecture builds on the existing `INInteraction` donation pattern used by Siri shortcuts. Apps create `INReservation` subclass objects, attach them to `INGetReservationDetailsIntentResponse`, wrap that in an `INInteraction`, and donate it. Siri processes the information on-device using end-to-end encryption and synchronizes it across the user's devices.

## Key Topics

**Donation Flow**
1. Create an `INReservation` subclass object (e.g., `INFlightReservation`) with all available details
2. Create `INGetReservationDetailsIntent` specifying the `containerReference` (the full reservation) and `reservationItemReferences` (set to `nil` at donation time)
3. Create `INGetReservationDetailsIntentResponse` with `code: .success` and populate the `reservations` array
4. Wrap intent + response in `INInteraction` and call `donate(completion:)`
5. Register `INGetReservationDetailsIntent` and custom activity types in Info.plist

**When to Donate**
- Donate when the user views reservation details in the app — Siri may show a notification at this moment; the context is appropriate
- Donate from the background when reservation data is updated (e.g., gate change, seat selection) — no notification is shown in this case
- Do not donate when showing an undifferentiated list of reservations
- Do not add explicit "Add to Siri" UI elements; donation should be automatic

**Reservation Types (New INReservation Subclasses)**
- `INFlightReservation` — contains `INFlight` (flight number, airline, departure/arrival including gate, terminal, time zone)
- `INRestaurantReservation` — restaurant reservations
- `INLodgingReservation` — hotel/lodging stays
- `INCarRentalReservation` — car rentals
- `INTrainReservation` — train trips
- `INBoatTripReservation` — boat/ferry trips
- `INBusReservation` — bus reservations
- `INTicketedEventReservation` — concerts, sporting events, theatre
- `INRentalCarReservation` — (car rental variant)
- All inherit from `INReservation` base class

**Key Reservation Properties**
- `itemReference: INSpeakableString` **[NEW]** — unique identifier for the reservation item across its full lifecycle; must never change; used to link app launch back to the correct item
- `reservationNumber: String` — the booking reference
- `reservationHolderName: INPerson` — customer name
- `reservationStatus: INReservationStatus` — `.confirmed`, `.cancelled`, `.pending`, `.hold`
- `bookingTime: DateComponents`
- Location: `CLPlacemark` — include both `location` (coordinates) and `postalAddress`; if coordinates unknown, set to `(0, 0)` so Siri uses only the postal address
- Start/end time: `INDateComponentsRange` with local time zone set explicitly; use `nil` for end time if reservation type has no natural end (Siri uses type-based defaults)

**Check-in Shortcut (`INReservationAction`)**
- `INReservationAction` **[NEW]** — attach to a reservation to advertise a check-in capability
- `actionType: INReservationActionType` — `.checkIn`
- `validDuration: INDateComponentsRange` — the window when check-in is available (e.g., 24h to 1h before departure)
- `userActivity: NSUserActivity` — the activity launched when the user taps the shortcut; set `activityType`, `title`, `userInfo`, `requiredUserInfoKeys`, and `webpageURL` (used on devices without the app via Safari)
- Siri displays the shortcut on lock screen and in Search at the right time; `title` on `NSUserActivity` is shown verbatim — make it short and descriptive

**App Launch Handling**
- Register `INGetReservationDetailsIntent` in Info.plist `NSUserActivityTypes`
- Implement `application(_:continue:restorationHandler:)` in `AppDelegate`
- Activity type `INGetReservationDetailsIntent` → extract `INInteraction` → cast intent → read `reservationItemReferences`:
  - `nil` → show the full reservation (all items for the container)
  - Non-nil array → show only the specific item identified by `itemReference`
- Custom activity type (from check-in `NSUserActivity`) → start check-in flow directly

**Reservation Updates and Cancellations**
- Re-donate with the same `itemReference` when details change (seat selection, gate update, etc.)
- Donate `reservationStatus: .cancelled` to notify Siri of a cancellation
- `itemReference` must remain identical throughout the reservation lifecycle

**Cross-Device Sync**
- Siri syncs donated reservation details across user's devices using end-to-end encryption
- Check-in shortcut may appear on a device that does not have the app installed; `webpageURL` on `NSUserActivity` provides Safari fallback

## APIs & Frameworks

**Intents Framework** (iOS 13) **[NEW APIs]**
- `INReservation` **[NEW]** — base class for all reservation types
- `INFlightReservation` **[NEW]** — `flight: INFlight`, `seats: [INSeat]`, `reservationActions: [INReservationAction]`
- `INFlight` **[NEW]** — `flightNumber`, `airline: INAirline`, `boardingTime`, `flightDuration`, `departureAirportGate: INAirportGate`, `arrivalAirportGate: INAirportGate`
- `INAirline` **[NEW]** — `name`, `iataCode`, `icaoCode`
- `INAirport` **[NEW]** — `name`, `iataCode`, `icaoCode`
- `INAirportGate` **[NEW]** — `airport: INAirport`, `gate`, `terminal`
- `INRestaurantReservation`, `INLodgingReservation`, `INCarRentalReservation`, `INTrainReservation`, `INBoatTripReservation`, `INBusReservation`, `INTicketedEventReservation` **[NEW]**
- `INReservationAction` **[NEW]** — `actionType: INReservationActionType`, `validDuration: INDateComponentsRange`, `userActivity: NSUserActivity`
- `INReservationActionType` **[NEW]** — `.checkIn`
- `INGetReservationDetailsIntent` **[NEW]** — `init(containerReference:reservationItemReferences:)`
- `INGetReservationDetailsIntentResponse` **[NEW]** — `init(code:userActivity:)`; `.reservations: [INReservation]`
- `INSpeakableString` — existing class used for `itemReference`; takes `vocabularyIdentifier` (machine ID) + `spokenPhrase` (displayed to user)
- `INDateComponentsRange` — existing class; used for `validDuration` and flight departure/arrival times; set `startDateComponents` and `endDateComponents` with explicit time zone
- `INInteraction` — existing class; `init(intent:response:)`; `donate(completion:)`
- `INPerson` — existing class; represents `reservationHolderName`

**Foundation**
- `NSUserActivity` — used for check-in action; `activityType`, `title`, `userInfo`, `requiredUserInfoKeys`, `webpageURL`

**Core Location**
- `CLPlacemark` — used for reservation location; include `location` (CLLocation) and `postalAddress` (CNPostalAddress)

## Code Highlights

Creating and donating a flight reservation with a check-in action:
```swift
import Intents

func donateFlightReservation(_ reservation: FlightReservation) {
    // Build the item reference
    let itemRef = INSpeakableString(
        vocabularyIdentifier: reservation.reservationNumber,
        spokenPhrase: "Flight \(reservation.flightNumber)",
        pronunciationHint: nil)

    // Build the check-in NSUserActivity
    let checkInActivity = NSUserActivity(activityType: "com.example.flights.checkin")
    checkInActivity.title = "Check-In for Flight \(reservation.flightNumber)"
    checkInActivity.userInfo = ["reservationNumber": reservation.reservationNumber]
    checkInActivity.requiredUserInfoKeys = ["reservationNumber"]
    checkInActivity.webpageURL = reservation.checkInURL

    // Build the check-in action
    let checkInStart = DateComponents(/* 24h before departure */)
    let checkInEnd = DateComponents(/* 1h before departure */)
    let checkInAction = INReservationAction(
        type: .checkIn,
        validDuration: INDateComponentsRange(start: checkInStart, end: checkInEnd),
        userActivity: checkInActivity)

    // Build INFlightReservation
    let flightReservation = INFlightReservation()
    flightReservation.itemReference = itemRef
    flightReservation.reservationNumber = reservation.reservationNumber
    flightReservation.reservationStatus = .confirmed
    flightReservation.reservationActions = [checkInAction]
    // ... set flight, seats, etc.

    // Donate via INInteraction
    let intent = INGetReservationDetailsIntent(
        containerReference: itemRef,
        reservationItemReferences: nil)
    let response = INGetReservationDetailsIntentResponse(code: .success, userActivity: nil)
    response.reservations = [flightReservation]
    let interaction = INInteraction(intent: intent, response: response)
    interaction.donate { error in
        if let error = error { print("Donation failed: \(error)") }
    }
}
```

Handling app launch from Siri:
```swift
func application(_ application: UIApplication,
                 continue userActivity: NSUserActivity,
                 restorationHandler: @escaping ([UIUserActivityRestoring]?) -> Void) -> Bool {
    switch userActivity.activityType {
    case String(describing: INGetReservationDetailsIntent.self):
        if let interaction = userActivity.interaction,
           let intent = interaction.intent as? INGetReservationDetailsIntent {
            if let refs = intent.reservationItemReferences, let ref = refs.first {
                showReservation(byItemReference: ref.vocabularyIdentifier)
            } else {
                showReservation(byContainerReference: intent.containerReference?.vocabularyIdentifier)
            }
        }
        return true
    case "com.example.flights.checkin":
        if let number = userActivity.userInfo?["reservationNumber"] as? String {
            startCheckIn(reservationNumber: number)
        }
        return true
    default:
        return false
    }
}
```

## Takeaways
- Set `itemReference` on every `INReservation` to a stable, unique identifier — it is the key Siri uses to link a Calendar event back to your app and to deliver updates.
- Always set the local time zone on `INDateComponentsRange` departure/arrival times; without it Siri may display the wrong time in the user's current locale.
- Provide `webpageURL` on check-in `NSUserActivity` so the shortcut works on devices without the app; Siri syncs donations across all the user's devices via end-to-end encrypted iCloud.
- Donate on reservation view (for user-visible notifications) and from the background on data updates (for silent Calendar updates); do not donate from list screens.

---
_Source: WWDC19 Session 243 page (abstract, chapter summaries, code samples, and resource links)._
