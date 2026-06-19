# Get Started with Privacy Manifests
**WWDC23 · Session 10060** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10060/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17, watchOS 10

## Overview
Privacy manifests are a new mechanism introduced in 2023 that allows third-party SDK developers to formally declare the privacy practices of their frameworks. This gives app developers a reliable, structured way to understand what data their dependencies collect, how it is used, and whether it is linked to users or used for tracking — directly addressing the difficulty of accurately filling out App Store Privacy Nutrition Labels when apps include multiple third-party SDKs.

In iOS 17, the system automatically enforces tracking domain declarations from privacy manifests: if a user has not granted App Tracking Transparency permission, connections to declared tracking domains are blocked by the OS, eliminating accidental tracking edge cases even when SDK defaults haven't been properly configured.

A new category called Required Reason APIs addresses the fingerprinting risk from certain system APIs (such as `NSFileSystemFreeSize`). Apps and SDKs must declare approved reasons for using these APIs in their privacy manifests; this becomes enforceable in App Review starting Spring 2024.

## Key Topics

### Privacy Manifests
- A new property list file named `PrivacyInfo.xcprivacy` added to an app target or SDK bundle.
- Declares: data types collected, how each type is used, whether data is linked to the user, and whether it is used for tracking (per ATT policy).
- Created directly from the Xcode navigator as a new file type.
- Data type and usage definitions align exactly with Privacy Nutrition Label categories.
- Third-party SDK developers include manifests in their SDK distributions; app developers aggregate them.

### Privacy Report
- Xcode 15 aggregates all `PrivacyInfo.xcprivacy` manifests in an app's project (including from dependencies) and generates a Privacy Report PDF.
- Accessed via Xcode Organizer: right-click an archive → "Generate Privacy Report."
- Organized similarly to Privacy Nutrition Labels; used to populate App Store Connect privacy details accurately.

### Tracking Domains
- Privacy manifests that declare tracking must also list tracking domains.
- iOS 17: automatically blocks connections to declared tracking domains when the user has not granted ATT permission — prevents accidental tracking.
- App developers can also create their own app-level `PrivacyInfo.xcprivacy` with tracking domains.
- Best practice: separate tracking and non-tracking functionality onto different hostnames (e.g., `tracking.example.com` vs. `non-tracking.example.com`).
- Xcode 15 Points of Interest instrument now flags connections to domains that may track users across apps and websites.

### Required Reason APIs
- A new category of APIs that have fingerprinting potential but provide important user-facing functionality.
- Apps and SDKs may only use these APIs for approved reasons listed in Apple documentation.
- Approved reasons are declared per API category in the `PrivacyInfo.xcprivacy` manifest.
- Example: `NSFileSystemFreeSize` — approved reason: checking available disk space before writing files.
- Fingerprinting (using device signals to identify a device/user) is never allowed regardless of ATT permission status.
- A feedback form is provided for use cases not covered by existing approved reasons.

### Privacy-Impacting SDKs
- Apple has identified a list of high-impact third-party SDKs ("privacy-impacting SDKs") published in developer documentation.
- Apps including these SDKs are required to use a version with a privacy manifest AND a digital signature.
- Timeline: Fall 2023 — informational emails for non-compliant apps; Spring 2024 — required for App Review.

## APIs & Frameworks
- `PrivacyInfo.xcprivacy` **[NEW]** — privacy manifest property list file for apps and SDKs
- Privacy manifest `NSPrivacyCollectedDataTypes` key **[NEW]** — array of collected data type declarations
- Privacy manifest `NSPrivacyTracking` key **[NEW]** — boolean declaring whether SDK/app tracks users
- Privacy manifest `NSPrivacyTrackingDomains` key **[NEW]** — list of tracking hostnames for automatic OS blocking
- Privacy manifest `NSPrivacyAccessedAPITypes` key **[NEW]** — Required Reason API declarations with approved reason codes
- Xcode 15 Privacy Report **[NEW]** — PDF aggregating all manifest declarations; generated from Organizer
- Points of Interest instrument (Xcode 15 update) **[NEW]** — now highlights connections to potential tracking domains during testing
- `NSFileSystemFreeSize` — example Required Reason API (file system free space)
- App Tracking Transparency (ATT) framework — permission framework for tracking; tracking domain blocking integrates with ATT status **[NEW iOS 17 enforcement]**
- SDK digital signatures (Xcode 15) **[NEW]** — required for privacy-impacting SDKs; covered in companion session "Verify app dependencies with digital signatures"

## Code Highlights
Privacy manifests are property list files, not Swift/Obj-C code. A minimal `PrivacyInfo.xcprivacy` example:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" ...>
<plist version="1.0">
<dict>
    <key>NSPrivacyTracking</key>
    <false/>
    <key>NSPrivacyCollectedDataTypes</key>
    <array>
        <dict>
            <key>NSPrivacyCollectedDataType</key>
            <string>NSPrivacyCollectedDataTypeName</string>
            <key>NSPrivacyCollectedDataTypeLinked</key>
            <true/>
            <key>NSPrivacyCollectedDataTypeTracking</key>
            <false/>
            <key>NSPrivacyCollectedDataTypePurposes</key>
            <array>
                <string>NSPrivacyCollectedDataTypePurposeAppFunctionality</string>
            </array>
        </dict>
    </array>
    <key>NSPrivacyAccessedAPITypes</key>
    <array>
        <dict>
            <key>NSPrivacyAccessedAPIType</key>
            <string>NSPrivacyAccessedAPICategoryFileTimestamp</string>
            <key>NSPrivacyAccessedAPITypeReasons</key>
            <array>
                <string>C617.1</string>
            </array>
        </dict>
    </array>
</dict>
</plist>
```

## Takeaways
- Privacy manifests shift the burden of privacy declaration from app developers onto SDK authors, making accurate Nutrition Labels achievable via the Xcode Privacy Report.
- iOS 17 automatically blocks tracking domain connections when ATT permission is not granted — a system-enforced safety net against accidental tracking.
- Required Reason APIs codify approved uses for fingerprinting-risk APIs; declaring reasons in `PrivacyInfo.xcprivacy` is required for App Review starting Spring 2024.
- App developers should immediately request `PrivacyInfo.xcprivacy` files from all third-party SDK vendors, particularly those on Apple's privacy-impacting SDK list.

---
_Source: WWDC23 Session 10060 page (abstract, chapter summaries, code samples, and resource links)._
