# Automate your development process with the App Store Connect API
**WWDC25 · Session 324** · [Watch](https://developer.apple.com/videos/play/wwdc2025/324/)

_Platforms:_ iOS 26, iPadOS 26, macOS Tahoe 26, tvOS 26, visionOS 26

## Overview
App Store Connect has significantly expanded its API suite with three major new capabilities: a Webhooks API for real-time event notifications, a BuildUpload API for programmatic build submission, and a Feedback API for retrieving TestFlight tester submissions. Together these additions close the loop on the full development cycle — from build upload through beta testing to feedback collection — enabling fully automated, event-driven workflows.

The session is framed around the iterative development loop: upload a build, distribute to beta testers, collect feedback, and repeat. The new APIs let teams instrument every stage of this loop with automation, replacing polling with push notifications and manual CLI steps with standard REST calls. This matters especially for large teams managing many apps or building CI/CD pipelines.

All new APIs follow the existing App Store Connect API conventions and can be called from any language or platform, with well-formatted error messages for automated error handling.

## Key Topics

### Webhook Notifications
Webhooks replace constant polling with server-to-server HTTP callbacks. Developers register a listener URL, a secret key (used for HMAC-SHA256 signature verification), and a set of event subscriptions via the App Store Connect website or API. Supported event types include: Build Upload state changes, Build Beta state changes, TestFlight Feedback submissions (screenshot and crash), App Version state changes, and Apple-Hosted Background Asset state changes. Payloads are signed with the `X-Apple-SIGNATURE` header and verified using the registered secret.

### Build Upload API
A new set of standardized REST endpoints enables programmatic build uploads without Xcode or `altool`. The flow: POST to create a `BuildUpload` resource (specifying bundle version and platform), POST a `BuildUploadFile` to declare the file (name, size, asset type), receive upload instructions (URL, method, headers, chunk splits for large files), PUT the binary, and PATCH to mark it as uploaded. App Store Connect then processes the build and fires a webhook when complete.

### Beta Testing & TestFlight APIs
Existing TestFlight APIs (assigning builds to groups, submitting for Beta App Review, notifying testers) are now complemented by a new `build beta state` webhook event that fires the moment external Beta App Review completes.

### Feedback API
The new Feedback API allows programmatic retrieval of screenshot and crash feedback. Webhooks deliver minimal notifications (feedback type + ID) so apps can query full detail — including device info, screenshot URLs, and crash log download links — on demand.

### Additional APIs
New Apple-Hosted Background Assets APIs automate asset management; App Version state webhooks complete the release pipeline.

## APIs & Frameworks

**App Store Connect API (REST)**
- **[NEW]** `POST /webhooks` — register a webhook listener
- **[NEW]** Webhook event types: `BUILD_UPLOAD_STATE`, `BUILD_BETA_STATE`, `FEEDBACK_SCREENSHOT`, `FEEDBACK_CRASH`, `APP_VERSION_STATE`, `APPLE_HOSTED_BACKGROUND_ASSET_STATE`
- **[NEW]** `POST /buildUploads` — create a build upload session
- **[NEW]** `POST /buildUploadFiles` — declare build file details, receive upload instructions
- **[NEW]** `PATCH /buildUploads/{id}` — mark upload as complete (`uploaded: true`)
- **[NEW]** `GET /betaFeedbackScreenshotSubmissions/{id}` — retrieve screenshot feedback detail
- **[NEW]** `GET /betaFeedbackCrashSubmissions/{id}` — retrieve crash feedback detail
- **[NEW]** Crash log download endpoint
- Existing: prerelease versions, beta tester groups, Beta App Review submissions
- `X-Apple-SIGNATURE` header — HMAC-SHA256 webhook payload signature

**Documentation Links**
- [Webhook notifications](https://developer.apple.com/documentation/AppStoreConnectAPI/webhook-notifications)
- [Beta feedback screenshot submissions](https://developer.apple.com/documentation/AppStoreConnectAPI/beta-feedback-screenshot-submissions)
- [Beta feedback crash submissions](https://developer.apple.com/documentation/AppStoreConnectAPI/beta-feedback-crash-submissions)
- [Managing Apple-Hosted Background Assets](https://developer.apple.com/documentation/AppStoreConnectAPI/managing-apple-hosted-background-assets)

## Code Highlights
Webhook registration via API (key fields):
```json
POST /webhooks
{
  "attributes": {
    "eventTypes": ["BUILD_UPLOAD_STATE", "BUILD_BETA_STATE", "FEEDBACK_SCREENSHOT"],
    "secret": "<your-secret>",
    "url": "https://yourserver.example/webhook"
  }
}
```

Build upload completion webhook payload (state transition):
```json
{ "state": "COMPLETE", "buildId": "..." }
```

## Takeaways
- Register webhook listeners to replace polling; use the `X-Apple-SIGNATURE` header to verify payload authenticity.
- Adopt the BuildUpload API to integrate build submission directly into language-agnostic CI/CD pipelines.
- Use the Feedback API with webhook triggers to immediately track and act on TestFlight feedback in your issue tracker.
- Explore the Apple-Hosted Background Assets APIs and App Version state webhooks to automate the full release pipeline end-to-end.

---
_Source: WWDC25 Session 324 page (abstract, chapter summaries, code samples, and resource links)._
