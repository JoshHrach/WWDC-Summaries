# Build an Endpoint Security App
**WWDC20 · Session 10159** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10159/)

_Platforms:_ macOS Big Sur 11

## Overview
The Endpoint Security framework, introduced in macOS Catalina and significantly enhanced in macOS Big Sur, is the modern replacement for Kernel Authorization (Kauth) KPIs, the unsupported Mac kernel framework, and the OpenBSM audit trail. This session covers building endpoint security products as System Extensions — providing protection, stability, and access to early-boot capabilities that kernel extensions (KEXTs) cannot offer.

The framework provides a C library callable from Swift, Objective-C, and Rust. Clients subscribe to roughly 100 event types split into NOTIFY events (informational, asynchronous) and AUTH events (synchronous, requiring a response before a per-message deadline). The session walks through two live demos: building a notification-only client and an authorization client that blocks process execution and file write operations.

macOS Big Sur brings substantial performance improvements through rewritten data structures, reduced memory allocations, and improved cache invalidation, resulting in better throughput and fewer dropped events.

## Key Topics

**Client Setup and Architecture**
Each call to `es_new_client` creates an independent ES client with its own event handler block and subscription set. Multiple clients can exist in one process. Events are now delivered to all subscribed clients simultaneously (changed from serial delivery in Catalina). The `es_message_t` struct carries metadata, instigating process info, and event-specific data.

**NOTIFY vs AUTH Events**
NOTIFY events are asynchronous — the operation continues before the ES client can act. AUTH events are synchronous — the kernel holds the operation until `es_respond_auth_result` or `es_respond_flags_result` is called, or the per-message deadline expires (missed deadline = implicit ALLOW, no cache). The `es_respond_flags_result` API is currently used only for `AUTH_OPEN`; the flags response must include all flags the client will ever permit (not just those requested), due to caching semantics.

**Muting**
`es_mute_process` (by audit token, auto-cleaned on process exit), `es_mute_path_literal`, and `es_mute_path_prefix` prevent message delivery from specified processes. Muting by audit token is preferred for performance.

**Caching**
A single global cache shared across all clients stores combined AUTH responses. Cache entries may expire at any time; caching should be used for performance only, never for policy. `es_clear_cache` clears the entire cache. Setting `cache: false` in a response guarantees that result is not cached.

**Early Boot**
System-extension-only feature (opt-in via `NSEndpointSecurityEarlyBoot` in Info.plist). Prevents third-party process execution until the extension calls `es_subscribe` at least once. All subscriptions should be made in a single `es_subscribe` call to avoid missing events.

**Message Lifetime and Async Processing**
Messages are only valid for the duration of the handler block invocation. Use `es_copy_message` to extend a message's lifetime for asynchronous processing; call `es_free_message` when done. AUTH events can be responded to after the handler returns.

**New in macOS Big Sur**
- `ES_EVENT_TYPE_NOTIFY_TRACE` — process debugging notification **[NEW]**
- `ES_EVENT_TYPE_NOTIFY_CS_INVALIDATED` — code-signing invalidation event **[NEW]**
- `AUTH_EXEC` now provides file descriptor list, types, and pipe IDs for the new process **[NEW]**
- OpenBSM audit subsystem deprecated (audit trail/pipe events); migrate to Endpoint Security

## APIs & Frameworks

### Endpoint Security (C library)
- `es_new_client(_:_:)` **[NEW in Catalina]** — creates a new ES client and event handler block
- `es_delete_client(_:)` — releases client resources
- `es_subscribe(_:_:_:)` — subscribes the client to a list of event types
- `es_message_t` — event envelope struct: `event_type`, `time`, `version`, `process`, `event` union
- `es_process_t` — instigating process info: executable path, stat, code signing info, audit token, PID, UID
- `es_respond_auth_result(_:_:_:_:)` — responds to AUTH events with ALLOW or DENY
- `es_respond_flags_result(_:_:_:_:)` — responds to AUTH_OPEN with a flags bitmask
- `ES_AUTH_RESULT_ALLOW`, `ES_AUTH_RESULT_DENY` — response constants
- `es_mute_process(_:_:)` — mutes a process by audit token
- `es_unmute_process(_:_:)` — re-enables messages from a muted process
- `es_mute_path_literal(_:_:)` — mutes by exact path
- `es_mute_path_prefix(_:_:)` — mutes by path prefix
- `es_copy_message(_:)` — extends message lifetime beyond handler block
- `es_free_message(_:)` — releases a copied message
- `es_clear_cache(_:)` — clears the global AUTH response cache
- `es_event_exec_t` — EXEC event data: target process, arguments, file descriptors **[NEW fields in Big Sur]**
- `es_event_open_t` — OPEN event data: file, open flags (kernel flags, not oflags)
- `es_event_exit_t` — EXIT event data: `stat` exit status
- `es_event_signal_t`, `es_event_fork_t`, `es_event_create_t` — other event types
- `ES_EVENT_TYPE_NOTIFY_EXEC`, `ES_EVENT_TYPE_NOTIFY_FORK`, `ES_EVENT_TYPE_NOTIFY_EXIT` — notify event type constants
- `ES_EVENT_TYPE_AUTH_EXEC`, `ES_EVENT_TYPE_AUTH_OPEN` — auth event type constants
- `ES_EVENT_TYPE_NOTIFY_TRACE` **[NEW]** — process trace/debug notification
- `ES_EVENT_TYPE_NOTIFY_CS_INVALIDATED` **[NEW]** — code signing invalidation event
- `message.version` — integer version field for compatibility checks
- `message.is_es_client` — Boolean indicating if the instigating process is an ES client
- `message.seq_num` — per-client, per-event-type sequence number to detect dropped messages

### System Extensions
- `NSEndpointSecurityEarlyBoot` — Info.plist key enabling early-boot mode **[NEW]**
- Endpoint Security entitlement (restricted, requires provisioning profile)
- System Extension entitlement for containing app
- Full Disk Access user consent (pre-populated in System Preferences for extensions)
- MDM payloads: allowed extensions list, Privacy Preferences Policy Control

### libBSM
- `audit_token_to_pid(_:)` — extracts PID from an audit token

## Code Highlights

Initializing a client and subscribing to events:
```c
es_client_t *client = NULL;
es_new_client_result_t result = es_new_client(&client, ^(es_client_t *c, const es_message_t *msg) {
    handle_event(c, msg);
});
es_event_type_t events[] = { ES_EVENT_TYPE_NOTIFY_EXEC, ES_EVENT_TYPE_NOTIFY_FORK };
es_subscribe(client, events, 2);
dispatch_main();
```

Responding to AUTH_EXEC with a deny-by-signing-ID policy:
```c
if (strcmp(msg->event.exec.target->signing_id.data, signing_id_to_block) == 0) {
    es_respond_auth_result(client, msg, ES_AUTH_RESULT_DENY, true);
} else {
    es_respond_auth_result(client, msg, ES_AUTH_RESULT_ALLOW, true);
}
```

Asynchronous AUTH_OPEN processing:
```c
es_message_t *copied = es_copy_message(msg);
dispatch_async(queue, ^{
    handle_open_worker(client, copied);
    es_free_message(copied);
});
```

## Takeaways
- Endpoint Security replaces Kauth KEXTs and OpenBSM with a stable, user-space C API that supports ~100 event types; the OpenBSM audit subsystem is deprecated in macOS Big Sur.
- AUTH events must be responded to before their per-message deadline or the process is terminated; use `es_copy_message` for asynchronous processing and minimize work inside the handler block.
- The `es_respond_flags_result` response must include all flags the client will ever permit — not just those requested — because cached results are compared against future requests without re-querying clients.
- Deploying as a System Extension (rather than a standalone app) enables early-boot interception, SIP protection, launchd auto-restart, and pre-populated Full Disk Access in System Preferences.

---
_Source: WWDC20 Session 10159 page (abstract, chapter summaries, code samples, and resource links)._
