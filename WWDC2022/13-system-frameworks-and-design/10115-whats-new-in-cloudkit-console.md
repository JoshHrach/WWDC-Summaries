# What's New in CloudKit Console
**WWDC22 · Session 10115** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10115/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, watchOS 9, tvOS 16

## Overview
CloudKit Console is a web-based tool for exploring and debugging CloudKit containers. The 2022 update introduces three major new capabilities: hidden containers for cleaner workspace organization, "Act As iCloud" for viewing data from a specific user account perspective, and zone sharing support for collaborative record management.

These improvements help developers understand how their data behaves in production, debug user-specific issues more effectively, and keep development workspaces tidy. Hidden containers apply at the team level, reducing clutter for all teammates simultaneously.

The Act As iCloud feature allows debugging private database issues by viewing data from a specific iCloud account perspective, though encrypted fields remain unreadable to maintain user privacy and security.

## Key Topics

### Hidden Containers
Developers can now toggle visibility of containers in CloudKit Console via a "Manage Containers" menu. Hidden containers disappear from both the Console and Xcode container selectors, and the setting applies team-wide. Useful for hiding prototype or deprecated containers from all teammates.

### Act As iCloud
A new "Act As iCloud" feature lets developers sign in as a separate iCloud account within the Console to view private database records from that account's perspective. This enables debugging production data issues on behalf of users. Restrictions: only applies to data (not schema), encrypted fields remain unreadable, and switching containers/environments ends the session.

### Zone Sharing
CloudKit Console now supports creating and managing zone-wide shares. Shared zones apply a single `CKShare` record to all records in a zone. Supported share types:
- **Public** shared zones: visible to anyone with the share code (read-only or read/write)
- **Private** shared zones: participant-list controlled, with per-participant permissions

New "Accept Shared Record" option in the Records page allows joining shared zones in the Console.

## APIs & Frameworks

**CloudKit**
- `CKContainer` — CloudKit container management
- `CKShare` — share record applied to zones **[NEW zone sharing in Console]**
- `CKRecordZone` — zone-level operations
- CloudKit Console "Hidden Containers" **[NEW]** — team-level container visibility toggle
- CloudKit Console "Act As iCloud" **[NEW]** — view private data as a specific iCloud account
- CloudKit Console "Zone Sharing" **[NEW]** — create and manage zone-wide CKShare records
- CloudKit Console "Configure zone wide sharing" button **[NEW]**
- CloudKit Console "Accept Shared Record" **[NEW]** — join shared zones in Console

## Code Highlights

No code samples shown; this session focuses on CloudKit Console web UI workflows.

## Takeaways

- Hide unused or prototype containers from the Console and Xcode team-wide using the new Manage Containers feature.
- Use "Act As iCloud" to view private database records from a user's perspective for production debugging — encrypted fields remain protected.
- Zone sharing in CloudKit Console lets you create public or private zone-wide shares; public shares allow anyone with a code to join, private shares require explicit participant management.
- Switching containers or environments automatically ends an "Act As iCloud" session.

---
_Source: WWDC22 Session 10115 page (abstract, chapter summaries, code samples, and resource links)._
