# Discover account-driven User Enrollment
**WWDC21 · Session 10136** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10136/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12

## Overview
This session introduces a redesigned onboarding flow for Mobile Device Management (MDM) User Enrollment, aimed at Bring Your Own Device (BYOD) enterprise deployments. The previous profile-based enrollment is supplemented by a new **account-driven** flow in iOS 15 that uses the employee's Managed Apple ID as the enrollment entry point—eliminating the need for admins to create and distribute per-user enrollment profiles.

The session covers three pillars of User Enrollment (Managed Apple IDs, data separation, and limited management capabilities), iOS 15 and macOS Monterey enhancements to Managed Apple ID features, the new account-based onboarding protocol flow, and a new ongoing user authentication mechanism that lets MDM servers re-challenge users at any time.

## Key Topics

### User Enrollment Core Concepts
- Designed for BYOD: the user owns the device; management capabilities are intentionally limited to protect personal data.
- Device is **not supervised** under User Enrollment; the MDM server cannot remotely wipe the device or access personal accounts or unique device identifiers.
- **Managed Apple ID**: owned and managed by the organization via Apple Business Manager or Apple School Manager; supports federation with Azure Active Directory.
- **Data separation**: enrollment creates a separate APFS volume with distinct cryptographic keys for managed app data; that volume is erased on unenrollment.
- **Limited management capabilities**: MDM server controls only organizational content.

### New in iOS 15 / macOS Monterey — Managed Apple ID Enhancements
- Managed Apple ID now shown at the top level of the Settings app on enrolled devices, making the managed account clearly visible.
- **iCloud Drive for Managed Apple IDs** **[NEW]**: organizations can now provide built-in cloud storage to user-enrolled devices.
  - Shows as a new location in the Files app (iOS/iPadOS) and Finder (macOS).
  - Document browser-based apps can access the Managed iCloud Drive location.
  - Respects Managed Open-In restrictions for managed apps.

### Managed Apps on macOS in User Enrollment (NEW)
- macOS Big Sur introduced managed apps for supervised Mac deployments; macOS Monterey extends this to User Enrollment.
- App data stored on the separate managed APFS volume; removed on MDM command or unenrollment.
- Managed apps using CloudKit automatically use the Managed Apple ID associated with the MDM profile.
- Requires `InstallAsManaged` key in the `InstallApplication` MDM command.
- Apps must be installed to `/Applications` with a single app bundle; adopt Data Protection Keychain and app sandboxing.

### Managed Open-In Enhancements **[NEW]**
- Copy & Paste restrictions added to Managed Open-In: organizations can now prevent data from being pasted across the managed/personal boundary in both directions.
- Required app installation: an app can be pre-approved for install at enrollment time, skipping additional user confirmation prompts.

### Account-Driven Enrollment Onboarding Flow **[NEW in iOS 15]**
The new flow uses the user's organization identity (e.g., `user@company.com`) as the entry point:

1. **Service discovery**: device extracts the domain from the organization ID and sends a `GET` to `https://<domain>/.well-known/<mdm-discovery-resource>`, receiving a JSON document with `version` and `BaseURL` keys pointing to the MDM enrollment endpoint.
2. **User authentication**: device POSTs device attributes to the enrollment endpoint; server responds with HTTP 401 and a `WWW-Authenticate: Bearer` header containing a `method` parameter and a `url` parameter pointing to an authentication endpoint.
3. **AuthenticationServices web sign-in**: device displays a web view to the authentication URL; the server can show a login form, redirect to a third-party identity provider, or perform MFA. Flow completes when the server redirects to a URL with a custom scheme containing an `access-token`.
4. **Enrollment**: device re-POSTs with `Authorization: Bearer <token>`; server returns the MDM enrollment profile containing two new required keys:
   - `AssignedManagedAppleID` — the Managed Apple ID to activate on the device **[NEW]**
   - `EnrollmentMode` — must be present and set to BYOD type **[NEW]**

### Ongoing User Authentication **[NEW in iOS 15]**
- After enrollment, the session token is included in every MDM HTTP request via the `Authorization` header.
- MDM server can invalidate the token at any time, triggering a re-authentication notification to the user via Notification Center.
- On tap, the user goes through the same AuthenticationServices web sign-in flow to obtain a new token.
- If authentication fails, the server can remove sensitive payloads or fully unenroll the device.
- **Key difference from profile-based enrollment**: with account-driven User Enrollment, HTTP 401 triggers re-authentication (not unenrollment). Unenrollment still requires sending the `RemoveProfile` MDM command.

## APIs & Frameworks

**MDM Protocol**
- `AssignedManagedAppleID` key in MDM payload **[NEW]**
- `EnrollmentMode` key in MDM payload **[NEW]**
- `InstallAsManaged` key in `InstallApplication` command **[NEW for User Enrollment on macOS]**
- HTTP 401 `WWW-Authenticate: Bearer` with `method` and `url` parameters — account-driven enrollment challenge **[NEW]**
- Service discovery JSON: `version` and `BaseURL` keys **[NEW]**

**AuthenticationServices**
- Web sign-in flow used for MDM enrollment and ongoing re-authentication **[existing framework, new MDM integration]**

**Apple Business Manager / Apple School Manager**
- Source for Managed Apple IDs; supports Azure Active Directory federation **[existing]**

**APFS**
- Separate volume with distinct cryptographic keys for managed data — created at enrollment, erased at unenrollment **[existing mechanism]**

**CloudKit**
- Managed apps using CloudKit now automatically use the Managed Apple ID **[NEW behavior in User Enrollment]**

## Code Highlights

Service discovery document the MDM server must host at the well-known URL:
```json
{
  "version": "mdm-byod",
  "BaseURL": "https://mdm.company.com/enroll"
}
```

HTTP 401 challenge the server issues to trigger user authentication:
```
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Bearer method="apple-as-web",
                         url="https://mdm.company.com/auth"
```

Redirect response completing the AuthenticationServices web flow:
```
HTTP/1.1 302 Found
Location: apple-mdm-enrollment://enroll?access-token=<opaque-token>
```

Required new keys in the MDM enrollment payload:
```xml
<key>AssignedManagedAppleID</key>
<string>user@managed.company.com</string>
<key>EnrollmentMode</key>
<string>BYOD</string>
```

## Takeaways
- The new account-driven flow removes the admin burden of generating and distributing per-user enrollment profiles; users self-enroll using their organization ID.
- A server-side authentication challenge (HTTP 401 with `WWW-Authenticate: Bearer`) is now **required** for enrollment to succeed—this guarantees the user's identity is verified before any organizational data is sent to the device.
- The session token sent with every MDM request enables ongoing authentication: admins can re-challenge users at any time for added security without disrupting device management.
- iCloud Drive support for Managed Apple IDs and managed app support in User Enrollment on macOS Monterey close major feature gaps compared to supervised deployment.
- Update MDM payloads to include both `AssignedManagedAppleID` and `EnrollmentMode`; enrollment fails if either key is missing.

---
_Source: WWDC21 Session 10136 page (abstract, full transcript, and resource links)._
