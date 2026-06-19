# Adopt Declarative Device Management
**WWDC22 · Session 10046** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10046/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, tvOS 16

## Overview
Declarative device management — introduced in WWDC21 for iOS user enrollments only — is now expanded to every MDM enrollment type and every platform Apple supports: iOS 16, macOS Ventura, tvOS 16, Shared iPad, and all enrollment channels (automatic device enrollment, profile-based, and user enrollment). Declarative management enables autonomous, proactive device behavior: the device applies management logic itself based on its own state, without server polling.

This session covers three key enhancements: expanded platform and enrollment scope, new status items (passcode compliance, installed accounts, MDM app state), and enhanced predicates including the new `management properties` declaration. Together, these changes allow MDM servers to move complex business logic onto devices, reducing server round-trips, polling overhead, and network bandwidth.

The `management properties` declaration is a particularly powerful addition — servers can preload complete configuration sets onto devices and then trigger or switch configurations with a single declaration update containing arbitrary JSON properties, enabling just-in-time device assignment scenarios.

## Key Topics

### Expanded Scope
- Full declarative management on iOS 16, macOS Ventura 13, tvOS 16
- All MDM enrollment types: automated device enrollment (supervised), profile-based, user enrollment, account-based enrollment
- Shared iPad support
- Separate enablement per channel (device channel and user channel each require a `DeclarativeManagement` command)
- Configurations visible in Settings app MDM profile detail view on all platforms

### New Status Items
- **Passcode**: `passcode.is-compliant`, `passcode.is-present` — Boolean values; eliminates polling for passcode compliance
- **Account lists** (accounts installed by configurations only): `account.list.caldav`, `account.list.carddav`, `account.list.exchange`, `account.list.google`, `account.list.ldap`, `account.list.mail.incoming`, `account.list.mail.outgoing`, `account.list.subscribed-calendar`
- **MDM App**: `mdm.app` — array of objects with `identifier`, `name`, `version`, `short-version`, `external-version-id`, `state` (`prompting`, `installing`, `managed`, `managed-but-uninstalled`)
- Array-valued status items reported incrementally: new objects added, changed objects replaced, removed objects indicated with `"removed": true`

### Enhanced Predicates
- New `@status(item.name)` syntax replaces the deprecated dot-path syntax
- `@property(key)` syntax for referencing management properties in predicates
- `SUBQUERY` support for array-valued status items (e.g., testing `mdm.app` array for a specific bundle ID and state)
- `@key(keyname)` extension term for safe key path access within array item objects

### Management Properties Declaration (`com.apple.management.properties`)
- New declaration type allowing servers to set arbitrary JSON properties on the device
- Values can be any JSON type: string, number, boolean, array, or object
- Referenced in activation predicates via `@property(keyName)`
- Enables preloading full configuration sets with predicate-gated activations; triggering a role change requires only sending/updating a single management properties declaration
- Multiple declarations allowed; keys must be unique across all to avoid unpredictable behavior

## APIs & Frameworks

**Device Management Protocol (declarative layer)**
- `DeclarativeManagement` MDM command — enables declarative management per channel **[updated]**
- Declaration types:
  - `com.apple.activation.simple` — activation with optional `Predicate` string
  - `com.apple.management.properties` **[NEW]** — arbitrary JSON payload for predicate use
  - `com.apple.configuration.*` — configuration types (Wi-Fi, mail, calendar, etc.)
  - `com.apple.asset.*` — asset types
- Status items **[NEW]**:
  - `passcode.is-compliant` — Boolean **[NEW]**
  - `passcode.is-present` — Boolean **[NEW]**
  - `account.list.mail.incoming` — array **[NEW]**
  - `account.list.mail.outgoing` — array **[NEW]**
  - `account.list.exchange` — array **[NEW]**
  - `account.list.google` — array **[NEW]**
  - `account.list.caldav` — array **[NEW]**
  - `account.list.carddav` — array **[NEW]**
  - `account.list.ldap` — array **[NEW]**
  - `account.list.subscribed-calendar` — array **[NEW]**
  - `mdm.app` — array **[NEW]**
- Array status item object keys:
  - `identifier` — primary key for the object within the array
  - `declaration-identifier` — cross-references the configuration that created the item
  - `removed` — boolean, `true` signals removal
- Status report rate limiting: device aggregates changes over up to 1 minute before sending
- Predicate syntax extensions **[NEW]**:
  - `@status(item.name)` — reference status item in predicate
  - `@property(keyName)` — reference management property in predicate
  - `@key(keyName)` — safe key path access for array item objects
  - `SUBQUERY(collection, $var, predicate)` — filter array status items

## Code Highlights

Passcode status subscription:
```json
{ "status-items": ["passcode.is-compliant", "passcode.is-present"] }
```

MDM app status report entry:
```json
{
  "identifier": "com.apple.Pages",
  "name": "Pages",
  "version": "7358.0.134",
  "short-version": "12.0",
  "state": "managed"
}
```

Predicate checking for a specific managed app:
```
SUBQUERY(@status(mdm.app), $app,
  ($app.@key(identifier) == "com.example.app") AND ($app.@key(state) == "managed")
).@count == 1
```

Management properties declaration:
```json
{
  "Type": "com.apple.management.properties",
  "Identifier": "AAE09D73-...",
  "Payload": { "name": "Student One", "age": 7, "roles": ["grade1", "spanish"] }
}
```

Activation predicate using management properties:
```json
"Predicate": "(@property(age) >= 18) AND (\"Grade12\" IN @property(roles))"
```

## Takeaways
- Declarative management now covers every Apple platform and enrollment type — there is no reason to remain on purely imperative MDM for new features.
- The `mdm.app` status item eliminates one of MDM's most common polling bottlenecks; servers receive real-time app installation state without polling.
- The `management properties` declaration enables just-in-time device assignment: preload all configurations with predicated activations, then trigger role assignment with a single declaration update.
- Status reports are rate-limited to aggregate changes over up to one minute — design server workflows to accept asynchronous, near-real-time updates rather than immediate responses.

---
_Source: WWDC22 Session 10046 page (abstract, chapter summaries, code samples, and resource links)._
