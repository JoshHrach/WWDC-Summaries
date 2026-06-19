# What's New in Endpoint Security
**WWDC22 · Session 110345** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110345/)

_Platforms:_ macOS Ventura 13

## Overview
Endpoint Security is Apple's C API for building macOS security products such as anti-virus, endpoint detection and response (EDR), and data-leakage prevention solutions. Introduced in macOS Catalina as a replacement for deprecated kernel-level APIs (KAuth KPI, MAC Kernel Framework, OpenBSM audit trail), it now supports over 100 event types on macOS Monterey.

macOS Ventura brings three major additions: new user-space security events covering authentication, login/logout, and Gatekeeper/XProtect activity; advanced muting capabilities including target-path muting and muting inversion for precise event filtering; and a new `eslogger` command-line utility that exposes the Endpoint Security event stream as JSON without requiring a native client.

These improvements close the remaining gap with the deprecated OpenBSM audit trail and provide new visibility into user-space security events that were previously unobservable in a structured way.

## Key Topics

### New Events in macOS Ventura
Three new categories of events expand Endpoint Security beyond kernel-level events:
- **Authentication events**: User authentication to the OS, admin authorization, and Apple Watch Auto Unlock (richer than OpenBSM equivalents)
- **Login/logout events**: Local console and remote logins (SSH, etc.), enabling visibility into lateral movement across fleets
- **XProtect/Gatekeeper events**: Detection and remediation actions by macOS's built-in malware scanner — previously unavailable in structured form

### Advanced Muting
New muting capabilities in macOS Ventura:
- **Target path muting**: Mute events based on target file path or path prefix using `es_mute_path` with `ES_MUTE_PATH_TYPE_TARGET_PREFIX` or `ES_MUTE_PATH_TYPE_TARGET_LITERAL`
- **Per-event-type path muting**: Mute specific event types on specific paths with `es_mute_path_events`
- **Muting inversion**: Invert any mute type (process, executable path, or target path) to select only matching events using `es_invert_muting` and `es_unmute_all_target_paths`

### eslogger Utility (New)
A new command-line tool (`eslogger`) ships with macOS Ventura and is pre-entitled for Endpoint Security. It taps into the Endpoint Security event stream and emits JSON-formatted event data to stdout or unified logging. Supports all 80 NOTIFY events. Requires superuser and Full Disk Access for the calling process (e.g., Terminal). Intended for security analysts and prototyping, not production applications.

## APIs & Frameworks

**Endpoint Security (C API)**
- `es_mute_path()` — mute events by process/executable/target path **[enhanced with TARGET types]**
- `ES_MUTE_PATH_TYPE_TARGET_PREFIX` **[NEW]** — mute all events with matching target path prefix
- `ES_MUTE_PATH_TYPE_TARGET_LITERAL` **[NEW]** — mute events with exact target path match
- `es_mute_path_events()` **[NEW]** — mute specific event types on specific target paths
- `es_invert_muting()` **[NEW]** — invert muting logic for process, executable, or target path
- `ES_MUTE_INVERSION_TYPE_TARGET_PATH` **[NEW]** — inversion type for target path muting
- `es_unmute_all_target_paths()` **[NEW]** — remove all target path mutes
- `ES_EVENT_TYPE_NOTIFY_AUTHENTICATION` **[NEW]** — user authentication events
- `ES_EVENT_TYPE_NOTIFY_OPENSSH_LOGIN` / `ES_EVENT_TYPE_NOTIFY_OPENSSH_LOGOUT` **[NEW]** — SSH login/logout events
- `ES_EVENT_TYPE_NOTIFY_LOGIN_LOGIN` / `ES_EVENT_TYPE_NOTIFY_LOGIN_LOGOUT` **[NEW]** — console/remote login/logout
- XProtect detection and remediation event types **[NEW]**
- `eslogger` CLI utility **[NEW]** — JSON Endpoint Security event stream from command line

## Code Highlights

```c
// Mute events operating on /var/log
es_mute_path(client, "/private/var/log", ES_MUTE_PATH_TYPE_TARGET_PREFIX);

// Mute only write events to /dev/null
var events = [ ES_EVENT_TYPE_NOTIFY_WRITE ];
es_mute_path_events(client, "/dev/null", ES_MUTE_PATH_TYPE_TARGET_LITERAL,
                    &events, events.count);

// Invert muting to select ONLY events pertaining to /Library/LaunchDaemons
es_invert_muting(client, ES_MUTE_INVERSION_TYPE_TARGET_PATH);
es_unmute_all_target_paths(client);
es_mute_path(client, "/Library/LaunchDaemons", ES_MUTE_PATH_TYPE_TARGET_PREFIX);
```

```bash
# Use eslogger to observe SSH login/logout events
sudo eslogger openssh_login openssh_logout > out.jsonl
```

## Takeaways

- New user-space events (authentication, login/logout, XProtect) mean most Endpoint Security clients no longer need the deprecated OpenBSM audit trail, which will be removed in a future macOS version.
- Target path muting and mute inversion enable surgical precision in event filtering, reducing performance overhead and improving client stability.
- `eslogger` provides no-code access to the Endpoint Security event stream as JSON — ideal for prototyping detections and security analysis workflows.
- The deprecated OpenBSM audit trail has been removed in macOS Big Sur and will be fully removed in a future version; migrate to Endpoint Security now.

---
_Source: WWDC22 Session 110345 page (abstract, chapter summaries, code samples, and resource links)._
