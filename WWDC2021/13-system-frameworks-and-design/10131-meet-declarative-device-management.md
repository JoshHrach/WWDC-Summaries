# Meet Declarative Device Management
**WWDC21 · Session 10131** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10131/)

_Platforms:_ iOS 15, iPadOS 15

## Overview
Declarative device management is a transformative update to Apple's MDM protocol that shifts policy management logic from the server to the device. Where the existing MDM protocol is imperative and reactive (requiring multiple server round-trips per management action), declarative management makes devices autonomous (able to apply policy logic without server prompting) and proactive (able to report state changes asynchronously via a status channel).

The declarative model is built on top of the existing MDM protocol — not a replacement — enabling MDM vendors to adopt it gradually. It leverages existing MDM enrollment, HTTP transport, and authentication infrastructure. In iOS 15, it is available on User Enrollment devices and covers account/passcode configurations, status subscriptions, profile installations, user identity/credential assets, organization details, and server capability declarations.

The core design philosophy: the server defines "what" should be true on the device; the device autonomously figures out "how" and "when" to make it true, and proactively reports its state back.

## Key Topics

**Three Pillars of Declarative Management**
1. **Declarations** — JSON payloads sent by the server describing desired policy state
2. **Status Channel** — a new asynchronous reporting channel where the device proactively sends state updates
3. **Extensibility** — mutual capability advertisement between device and server, enabling smooth adoption of new features without hardcoded version checks

**Declaration Types**
- **Configuration** — policy to apply (accounts, passcode, restrictions); similar to MDM profile payloads but in JSON
- **Asset** — references to ancillary data (user identity, credentials, large binary assets via URL); one asset can be shared by multiple configurations
- **Activation** — defines a set of configurations to atomically apply; optional predicate evaluated on-device determines when active
- **Management** — conveys organizational metadata and server capabilities

**Activation Predicates**
Predicates on activations let the server send all declarations for all device states; the device evaluates predicates autonomously and applies only the relevant policy. As device state changes (OS version, device type), predicates are re-evaluated without server intervention.

**Status Channel**
Server subscribes to specific status items via a StatusSubscription configuration. The device sends initial reports on subscription and incremental reports on state change. Declaration application status is always reported without explicit subscription. Status item key-paths are period-separated strings (e.g., `device.operating-system.version`).

**MDM Integration**
- `DeclarativeManagement` MDM command activates declarative management (one-way, cannot be disabled, but all declarations can be removed)
- New `DeclarativeManagement` CheckIn request type handles synchronization and status reports
- Synchronization flow: device fetches manifest (declaration metadata), compares to local state, fetches only new/changed declarations, applies policy, reports status

**iOS 15 Initial Support**
- Configurations: account, passcode, profile (installs full MDM profile from URL)
- Activations: simple activation with optional predicate
- Assets: user identity, user credential
- Management: organization details, server capabilities
- Status items: per-declaration status, device hardware model, OS version/type
- Available on User Enrollment (iOS 13 flow and new iOS 15 onboarding flow)

## APIs & Frameworks

### Device Management Protocol (MDM) **[NEW in iOS 15]**
All declarative management items are JSON-serialized and exchanged over the existing MDM CheckIn HTTP transport.

**Declaration JSON Structure (all types)**
- `Type` (String) — declaration type identifier
- `Identifier` (String, UUID) — unique ID within the set of declarations
- `ServerToken` (String) — unique revision token (counter or UUID); changes on each update
- `Payload` (Object) — type-specific configuration data

**Declaration Types**
- `com.apple.configuration.passcode.settings` — passcode policy configuration **[NEW]**
- `com.apple.configuration.account.*` — account configurations **[NEW]**
- `com.apple.configuration.management.status-subscriptions` — status subscription configuration **[NEW]**
- `com.apple.configuration.management.profile` — installs a traditional MDM profile via URL **[NEW]**
- `com.apple.asset.useridentity` — user identity asset (name, email) **[NEW]**
- `com.apple.asset.credential.userpassword` — user credential asset (username, password) **[NEW]**
- `com.apple.management.organization` — organization details management declaration **[NEW]**
- `com.apple.management.server-capabilities` — server capability advertisement **[NEW]**

**Activation**
- `com.apple.activation.simple` — list of configuration identifiers + optional predicate **[NEW]**
- Activation `Predicate` — logical expression referencing status item key-paths

**MDM Protocol Additions**
- `DeclarativeManagement` MDM command — activates declarative management on device **[NEW]**
- `DeclarativeManagement` CheckIn `MessageType` — new request type for sync and status **[NEW]**
- CheckIn `Endpoint` values:
  - `declaration-items` — fetch manifest of all declarations
  - `declaration/<type>/<identifier>` — fetch a specific declaration
  - `status` — send a status report

**Status Items (initial set)**
- `device.operating-system.version`
- `device.operating-system.family`
- `device.identifier.serial-number`
- `device.identifier.udid`
- Per-declaration status (always reported, no subscription needed)
- `management.client-capabilities`

## Code Highlights

Example Configuration Declaration (passcode policy, JSON):
```json
{
    "Type": "com.apple.configuration.passcode.settings",
    "Identifier": "F4EA2C62-ECA7-4B51-A0A1-C8F23E80C8EC",
    "ServerToken": "1",
    "Payload": {
        "RequireAlphanumeric": true,
        "MaximumPasscodeAgeDays": 90
    }
}
```

Example Activation with predicate:
```json
{
    "Type": "com.apple.activation.simple",
    "Identifier": "A9021DB6-72BE-4B57-988A-F7E38B7CBDE0",
    "ServerToken": "1",
    "Payload": {
        "StandardConfigurations": [
            "F4EA2C62-ECA7-4B51-A0A1-C8F23E80C8EC"
        ],
        "Predicate": "device.model.family == 'iPad'"
    }
}
```

## Takeaways
- Declarative management shifts from an imperative server-driven model to a device-driven model where the device autonomously applies policy and proactively reports state — dramatically reducing round-trips and improving scalability.
- It is additive to the existing MDM protocol; vendors can adopt individual features (e.g., status subscriptions only) without rearchitecting their entire MDM infrastructure.
- Activation predicates allow the server to send a complete policy set; each device evaluates which portions apply to its own current state, re-evaluating autonomously as state changes.
- iOS 15 support is scoped to User Enrollment only in the initial release, with account/passcode/profile configurations and a rich initial set of status items.

---
_Source: WWDC21 Session 10131 page (abstract, chapter summaries, code samples, and resource links)._
