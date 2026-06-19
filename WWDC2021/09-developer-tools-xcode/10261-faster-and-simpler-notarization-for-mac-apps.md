# Faster and Simpler Notarization for Mac Apps
**WWDC21 · Session 10261** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10261/)

_Platforms:_ macOS Monterey 12

## Overview
This session introduces `notarytool`, a new command-line tool shipped with Xcode 13 that replaces `altool` for Mac app notarization. `notarytool` is focused exclusively on the Notary service (unlike `altool`, which also handles App Store operations), enabling a dramatically simpler one-command submit-and-wait workflow, up to 4x faster uploads, and a new webhook notification feature for CI integration.

Apple also introduced a new dedicated backend service for Notary, designed for improved reliability and end-to-end processing speed. `altool` remains available but is now deprecated for notarization use.

## Key Topics

**What Is Notarization**
Notarization is a pre-distribution step for Mac software distributed outside the App Store. Developers submit a signed `.app`, `.dmg`, `.pkg`, or `.zip` to Apple's Notary service, which scans for malicious content and code signing issues. Apple publishes a ticket; macOS verifies the ticket when users launch the software. Apple targets 98% of submissions completing within 15 minutes (most under 5 minutes).

**notarytool — The New Tool**
`notarytool` ships with Xcode 13 and is the new primary CLI for notarization. Key improvements over `altool`:
- Single command with `--wait` flag: submits the file and blocks until the result is ready, eliminating the manual polling loop required with `altool`
- Built-in log retrieval: view the Notary log directly from the tool
- Up to 4x faster upload speeds
- New webhook support for CI/CD pipelines

**Webhook Notifications**
When submitting, developers can provide a server URL via a webhook parameter. The Notary service calls back to that URL as soon as processing completes, allowing CI systems to automatically receive the result and log without polling.

**New Dedicated Backend**
The Notary service has a new dedicated backend infrastructure focused on reliability and throughput, separate from altool's App Store Connect infrastructure.

**Migration from altool**
`altool` is deprecated for notarization but not removed. Teams using `altool` polling loops should migrate to `notarytool submit --wait` for immediate simplification. The `--key`, `--key-id`, and `--issuer` flags replace the `altool` API key/issuer arguments.

## APIs & Frameworks

- `notarytool` **[NEW]** — command-line tool, distributed with Xcode 13
  - `notarytool submit <path> --wait` **[NEW]** — submit and block until result
    - `--key <path>` — path to App Store Connect API key (.p8)
    - `--key-id <id>` — API key ID
    - `--issuer <uuid>` — API issuer UUID
  - `notarytool log <submission-id>` **[NEW]** — retrieve Notary log for a submission
  - `notarytool submit <path> --webhook <url>` **[NEW]** — submit with server callback
  - `notarytool history` — list recent submissions
  - `notarytool info <submission-id>` — get status of a specific submission
- `xcrun altool --notarize-app` — deprecated for notarization
- `xcrun altool --notarization-info` — deprecated for notarization
- macOS Gatekeeper — verifies Notary ticket at launch time (unchanged)
- App Store Connect API keys — authentication method for `notarytool`

## Code Highlights

Old workflow with `altool` (deprecated pattern — requires a polling loop):
```bash
xcrun altool --notarize-app -f path/to/submission.zip \
    --primary-bundle-id "$BUNDLE_ID" \
    --apiKey "$KEY_ID" --apiIssuer "$ISSUER"
while true; do
  STATUS=$(xcrun altool --notarization-info "$SUBMISSION_ID" \
      --apiKey "$KEY_ID" --apiIssuer "$ISSUER" | grep "Status:" | ...)
  if [[ "$STATUS" != "in progress" ]]; then break; fi
  sleep 30
done
```

New workflow with `notarytool` (single command):
```bash
notarytool submit path/to/submission.zip --wait \
    --key "$KEY_PATH" --key-id "$KEY_ID" --issuer "$ISSUER"
```

## Takeaways

- `notarytool` replaces `altool` for notarization with a single `submit --wait` command, eliminating the polling loop entirely.
- Upload speeds are up to 4x faster due to both client-side improvements and a new dedicated Notary backend.
- Webhook support enables fully automated CI/CD notarization pipelines without any polling or scheduled checks.
- `altool` is deprecated for notarization but not removed; migrate to `notarytool` as soon as possible to benefit from speed and reliability improvements.

---
_Source: WWDC21 Session 10261 page (abstract, chapter summaries, code samples, and resource links)._
