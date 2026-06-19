# Meet In-App Events on the App Store
**WWDC21 · Session 10171** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10171/)

_Platforms:_ iOS 15, iPadOS 15

## Overview
In-App Events is a new App Store feature launching fall 2021 that lets developers promote timely, limited-duration activities happening inside their apps — game competitions, fitness challenges, movie premieres, live events, and more — directly on the App Store. Events appear as visual event cards across multiple App Store surfaces, giving them organic discoverability beyond the app's own product page.

Events are created and managed entirely through App Store Connect (no new app binary required), reviewed by Apple, and automatically published and unpublished according to scheduled start and end dates. All In-App Event metadata and analytics are also accessible through the App Store Connect API. Event performance is tracked in App Analytics with dedicated metrics including impressions, product page views, app downloads/redownloads, and notification opt-ins.

## Key Topics

**Event Card Design**
Each event is represented by a visual event card — an image or looping video (up to 30 seconds), event name (30 chars), short description (50 chars), and an automatic time indicator showing countdown until start or elapsed time since start. An "Open" button for users who already have the app jumps directly into the event via deep link.

**Event Discovery Surfaces**
- App's product page — above screenshots for existing users, below for new users
- App Store search results — next to the app for both existing and new users
- Search when users look up the event by name
- Games/Apps tabs — personalized event recommendations
- Editorial curation — Today, Games, and Apps tab features

**Event Details Page**
A dedicated page with a longer description (120 chars), larger media asset, and a notification request button. Linkable via `https://apps.apple.com/app/id<AppID>?event=<EventID>` format.

**Event Scheduling**
- Start date/time — when the event begins inside the app
- End date date/time — when the event ends; max duration is 31 days, minimum is 15 minutes
- Publish date — up to 14 days before start; when the event becomes discoverable on the App Store
- Post-end visibility — event details page remains link-accessible for 30 days after end, then archived
- Custom scheduling per Country/Region — within ±48 hours of default dates

**Event State Lifecycle**
Draft → Approved → Set to Publish → Published → Past → Archived

**Limits**
- 10 events maximum in the Approved state at one time
- 5 events maximum Published to the App Store simultaneously

**Analytics**
App Analytics gains new event-level metrics: impressions (with source breakdown), product page opens, event details page opens, app downloads, app redownloads, and notification opt-in counts.

## APIs & Frameworks

### App Store Connect (Web UI) **[NEW]**
- In-App Events section under app Features
- Create/edit event with metadata: reference name, event name, short description, long description, badge type, event card media, event details media
- Availability configuration per Country/Region
- Scheduling: start, end, publish dates; custom dates per region
- Additional information: deep link URL, event purpose, priority level, In-App Purchase requirement indicator
- Event state management: draft, submit for review, approve, archive
- Direct link to event in App Analytics

### App Store Connect API **[NEW — spec releasing later fall 2021]**
- Full programmatic access to create, edit, schedule, and manage In-App Events
- Referenced at `developer.apple.com/documentation/AppStoreConnectAPI`

### App Store Features
- Event card — visual unit shown on product pages and search results
- Event details page — `https://apps.apple.com/app/id<AppID>?event=<EventID>` URL format **[NEW]**
- Notification system — system-sent push notifications from App Store when event starts (no developer code required)
- Event deep link — universal link or custom URL scheme directing users into the in-app event

### App Analytics (New Metrics) **[NEW]**
- Impressions (total and by source: product page, search, editorial, etc.)
- Event details page views
- App opens from event
- App downloads and redownloads attributed to event
- Notification opt-in count

### Event Badge Types
- Challenge
- Competition
- Live Event
- Major Update
- New Season
- Premiere
- Special Event

### Event Purpose (for personalization algorithm)
- Attract new users
- Keep active users informed
- Bring lapsed users back
- All users (default)

## Code Highlights

No client-side API code. Integration on the app side requires a working deep link (universal link or custom URL scheme) that opens directly to the in-app event, for example:

```
// Universal link (preferred)
https://myapp.com/events/spring-competition

// Custom URL scheme
myapp://events/spring-competition
```

The deep link should not use URL shorteners or redirect services.

## Takeaways
- In-App Events require no new app binary or Xcode changes — they are pure App Store Connect metadata with a deep link pointing into the existing app.
- Events are discoverable organically across the App Store (product page, search, editorial, personalized recommendations) without any additional marketing spend.
- The full event creation, scheduling, and management workflow will be automatable via the App Store Connect API.
- App Analytics provides granular event-level performance data, enabling developers to measure ROI on each event (impressions, installs, notification opt-ins).

---
_Source: WWDC21 Session 10171 page (abstract, chapter summaries, code samples, and resource links)._
