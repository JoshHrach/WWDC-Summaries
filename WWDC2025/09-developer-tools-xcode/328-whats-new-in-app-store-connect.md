# What's New in App Store Connect
**WWDC25 · Session 328** · [Watch](https://developer.apple.com/videos/play/wwdc2025/328/)

_Platforms:_ iOS, iPadOS, macOS, tvOS, visionOS, watchOS (App Store Connect web, iOS/iPadOS app, App Store Connect API)

## Overview
App Store Connect receives a sweeping set of improvements across build management, TestFlight, app discovery, user trust, and the App Store Connect API. Key highlights include a new build-upload progress view (with error retention), App Store Tags powered by large language models, keywords for custom product pages, offer codes for non-subscription In-App Purchases, review summaries, a revamped five-tier age rating system, and Accessibility Nutrition Labels. The session also previews Apple-Hosted Background Assets (up to 200 GB per app) and improvements to App Analytics, Game Center, and App Review.

## Key Topics

### Build Management
- New **Build Uploads** section in App Store Connect (TestFlight tab) shows real-time status as a build moves from delivery through processing; failed deliveries are now retained so teams can see error details
- Failed builds no longer block build number reuse
- **App Store Connect API** can now be used to upload builds directly (launching later in 2025), enabling full CI/CD integration
- **Webhooks** provide real-time notifications for build status changes and TestFlight screenshot/crash feedback

### TestFlight Feedback in App Store Connect App
- Screenshot feedback and crash logs are now surfaced in the App Store Connect app on iPhone/iPad with push notifications
- Digest mode scales notifications to avoid overload
- Feedback can be shared with team members directly from the app
- New TestFlight Feedback APIs allow programmatic access to all screenshot and crash feedback

### Apple-Hosted Background Assets
- New distribution mechanism: up to 200 GB of assets hosted by Apple, updated independently of the app binary
- Available for Mac, iPhone/iPad, Apple TV, Apple Vision Pro

### App Discovery Improvements
- **App Store Tags** — AI-generated (large language model) tags, human-reviewed, highlight specific features and functionality; appear alongside categories on the search page and in search results; controllable per-app in App Store Connect
- **Keywords for Custom Product Pages** — custom product pages can now be assigned keywords from the app's keyword list; targeted pages appear in search results for those keywords without a review submission; searchability per custom page visible in App Analytics
- **Offer Codes for IAP** — offer codes (free and discount) now extend beyond subscriptions to consumables, non-consumables, and non-renewing subscriptions; up to 10 active offers per IAP; 1 million codes per app per quarter; redemption eligibility criteria (never spent / spent not recently / spent recently); sandbox testing with up to 10,000 codes per app per quarter; StoreKit handles in-app redemption

### App Trust and Transparency
- **Review Summaries** — LLM-generated concise summaries of customer reviews displayed on the product page; refreshed regularly; reportable via App Store Connect
- **Expanded Age Ratings** — five thresholds (3 new); regional variants; new declaration fields for messaging/chat, user-generated content, advertising, and in-app controls/parental controls; age-rating override for policy-based needs; updated questionnaire UX with learn-more modals
- **Accessibility Nutrition Labels** — new section on product pages declaring supported accessibility features (e.g., Larger Text, VoiceOver) per device; declared independently per device; publishable as drafts; optional accessibility support URL

### Additional Updates
- App Analytics: 100+ new metrics, new subscriptions/monetization data, redesigned navigation
- Game Center: new Apple Games app, new activities and challenges, large-content install improvements for Mac
- App Review: new review submission types (Apple-Hosted Background Assets, Game Center items), grouped draft submissions, updated submission UI

## APIs & Frameworks

**App Store Connect API**
- Build upload via API **[NEW]** — replaces Xcode/Transporter as the only upload paths
- Webhooks for build status and TestFlight feedback **[NEW]**
- TestFlight Feedback APIs **[NEW]** — programmatic access to screenshot and crash feedback

**App Store Connect (Web / iOS app)**
- Build Uploads section **[NEW]** — real-time delivery progress and error retention
- App Store Tags management **[NEW]** — view and toggle AI-generated tags per app
- Keywords for Custom Product Pages **[NEW]** — assign existing keywords to custom product pages
- Offer Codes for In-App Purchases (consumables/non-consumables/non-renewing subscriptions) **[NEW]**
- Review Summaries **[NEW]** — LLM-generated; reportable in App Store Connect
- Redesigned Age Rating questionnaire with 5 tiers and new capability declarations **[NEW]**
- Accessibility Nutrition Labels **[NEW]** — per-device declaration, draft and publish workflow
- Apple-Hosted Background Assets **[NEW]** — up to 200 GB, updatable independently

**StoreKit**
- In-app redemption of IAP offer codes — existing redemption flow extended to support the new IAP types

## Code Highlights
No new code APIs in this session — all changes are to the App Store Connect web UI, iOS/iPadOS app, and REST API. Refer to the App Store Connect Help guide and the "Automate your development process with the App Store Connect API" session (WWDC25) for API details.

## Takeaways
- Add keywords to your custom product pages to surface the most relevant experience in search results without additional review cycles.
- Create offer codes for IAP (not just subscriptions) to drive re-engagement campaigns for consumables and non-renewables.
- Review your automatically recalculated age ratings under the new five-tier system and answer the new capability declaration questions.
- Declare Accessibility Nutrition Labels to build user trust and help people find apps that meet their accessibility needs.
- Register your business and email addresses in Apple Business Connect to unify branding in Review Summaries, order tracking, and preauthorized payments.

---
_Source: WWDC25 Session 328 page (abstract, chapter summaries, code samples, and resource links)._
