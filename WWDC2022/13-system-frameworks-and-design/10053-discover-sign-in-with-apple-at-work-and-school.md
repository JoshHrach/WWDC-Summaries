# Discover Sign in with Apple at Work & School
**WWDC22 · Session 10053** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10053/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, tvOS 16

## Overview
This session introduces two new enterprise and education features built on top of Sign in with Apple: support for Managed Apple IDs ("Sign in with Apple at Work & School"), and the Roster API for programmatic access to Apple School Manager user and class data. Together, these features allow enterprise and education apps to provide the same fast, password-free sign-in experience of Sign in with Apple while giving IT administrators centralized management and control.

With Sign in with Apple at Work & School, name and email are always shared (unlike personal Apple IDs where users can hide email). The Roster API uses OAuth 2.0 to let IT admins authorize apps to access student, teacher, and class data from Apple School Manager — eliminating manual data entry at the start of a school year.

## Key Topics

### Sign in with Apple at Work & School
- Sign in with Apple now extended to support **Managed Apple IDs** **[NEW]**.
- Managed Apple IDs are owned by an organization and administered through Apple School Manager (ASM) or Apple Business Manager (ABM).
- UI difference from personal Apple IDs: name and email are **always provided** — users cannot hide their email. This is intentional so apps can provide appropriate access control (e.g., which Slack channels to show).
- Uses the same `ASAuthorizationAppleIDCredential` API as personal Sign in with Apple — no code changes required if already implemented.
- Signed in using the primary Managed Apple ID on the device.
- Supports both native app flow and web flow.
- Accounts may have no email address (e.g., younger students); apps should handle this case.
- If Sign in with Apple is already implemented, Managed Apple ID support is automatic.

### Roster API
- New REST API **[NEW]** for programmatic access to Apple School Manager resources: users and classes.
- Accessed via OAuth 2.0 authorization flow ("Organizational Data Sharing") **[NEW]**.
- Only Administrator and Site Manager accounts in ASM can grant access.
- Endpoints available:
  - `GET /users` — list users (students, instructors, staff); supports `role`, `pageToken`, `limit` params.
  - `GET /users/{userId}` — specific user.
  - `GET /classes` — list classes; supports `pageToken`, `limit`.
  - `GET /classes/{classId}` — specific class.
  - `GET /classes/{classId}/users` — users in a class.
- User records include: `givenName`, `familyName`, stable unique identifier (same as Sign in with Apple identifier), email, roles.
- Class records include: `name`, class identifier, list of `instructorIds`, list of `studentIds`.
- Pagination via `nextPageToken` / `moreToFollow` in responses.

### OAuth 2.0 Authorization Flow (Organizational Data Sharing)
1. Register in the Developer portal — configure scopes (user access, class access) and return URLs under "Account & Organizational Data Sharing."
2. IT admin initiates authorization in your app → `GET /authorize` request with `client_id`, `redirect_uri`, `state`, `response_type`, `scope`.
3. IT admin sees consent screen (showing app name and requested scopes), clicks Allow → authorization code delivered to your return URL.
4. App server exchanges authorization code for access token via `POST /token`.
5. Refresh tokens used to obtain new access tokens when they expire.
6. Access token used as Bearer token in `Authorization` header for all Roster API calls.

### Access Management in ASM / ABM
- IT admins configure Sign in with Apple at Work & School and Organizational Data Sharing in ABM/ASM under Access Management.
- Two modes for Sign in with Apple at Work & School:
  - **Allow all apps** — all users can sign in to any Sign in with Apple app.
  - **Allow only certain apps** — IT admin curates a list of approved apps.
- Same two modes for Organizational Data Sharing — IT admin controls which apps can access ASM data via Roster API.
- Available in Apple School Manager, Apple Business Manager, and Business Essentials.

## APIs & Frameworks

### Authentication Services (existing, extended)
- `ASAuthorizationAppleIDCredential.fullName` — always populated for Managed Apple IDs **[NEW behavior]**
- `ASAuthorizationAppleIDCredential.email` — always populated (may be nil if no email assigned)
- `ASAuthorizationController` — unchanged; Managed Apple ID support is automatic

### Roster API (REST) **[NEW]**
- Base URL: Apple's Roster API endpoint
- `GET /v1/users` — list users with `role`, `pageToken`, `limit` query params
- `GET /v1/users/{id}` — single user
- `GET /v1/classes` — list classes with `pageToken`, `limit`
- `GET /v1/classes/{id}` — single class
- `GET /v1/classes/{id}/users` — users in class
- Response fields: `users[]`, `classes[]`, `nextPageToken`, `moreToFollow`
- User fields: `givenName`, `familyName`, unique `identifier` (matches Sign in with Apple ID), `email`, `roles`
- Class fields: `name`, `identifier`, `instructorIds[]`, `studentIds[]`

### OAuth 2.0 Endpoints **[NEW]**
- `GET /auth/authorize` — initiate authorization
- `POST /auth/token` — exchange code for access token / refresh token
- Scopes: `user.read`, `class.read`

### Developer Portal Configuration
- Certificates, Identifiers & Profiles → Account & Organizational Data Sharing
- Configure scopes and return URLs per app ID

## Code Highlights

```swift
// Native app: request full name and email (unchanged from existing Sign in with Apple)
let request = ASAuthorizationAppleIDProvider().createRequest()
request.requestedScopes = [.fullName, .email]

// On success, name and email always provided for Managed Apple IDs
func authorizationController(..., didCompleteWithAuthorization authorization: ASAuthorization) {
    if let credential = authorization.credential as? ASAuthorizationAppleIDCredential {
        let fullName = credential.fullName  // Always non-nil for Managed Apple IDs
        let email = credential.email        // May be nil if no email assigned
    }
}
```

```
// Roster API: GET /v1/users (students)
GET /v1/users?role=student&limit=10
Authorization: Bearer <access_token>

// Response
{
  "users": [{ "givenName": "...", "familyName": "...", "identifier": "...", "email": "...", "roles": [...] }],
  "nextPageToken": "abc123",
  "moreToFollow": true
}
```

## Takeaways
- Sign in with Apple at Work & School requires zero code changes if you already support Sign in with Apple — just ensure your app handles always-present name/email and the case where email may be nil.
- The Roster API dramatically simplifies school district onboarding by replacing manual data entry with a standards-based OAuth 2.0 integration to Apple School Manager.
- IT administrators retain full control through ABM/ASM Access Management — apps must be approved before they can use these features.
- Register in the Developer portal to configure Roster API scopes and return URLs before implementing the OAuth flow.

---
_Source: WWDC22 Session 10053 page (abstract, chapter summaries, code samples, and resource links)._
