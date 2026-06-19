# Design for Collaboration with Messages
**WWDC22 · Session 10015** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10015/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13

## Overview
This session covers the design principles and UI patterns behind Apple's new Messages-centered collaboration model introduced in iOS 16 and macOS Ventura. Rather than relying solely on email (an asynchronous tool), the new architecture weaves Messages directly into collaborative app workflows so people can initiate, manage, and communicate about shared documents without leaving their apps.

The session demonstrates an end-to-end collaboration flow using Pages as a reference app, from tapping the share button and sending an invitation in an existing Messages conversation through real-time co-editing, the collaboration popover, and activity notifications back in Messages. It provides actionable design guidance for placing share buttons, customizing the system share sheet, designing the collaboration popover, and configuring Messages banners.

## Key Topics

### Why Messages for Collaboration
- Email is asynchronous and slow; Messages provides responsive, real-time communication.
- Messages conversations can escalate to FaceTime audio/video calls with screen sharing.
- Messages surfaces collaboration activity notifications (edit banners) back in conversations.
- Conversation suggestions in the share sheet help users quickly find their collaborators.

### System Share Sheet / Share Popover
- The **system share sheet** (iOS/iPadOS) and redesigned **share popover** (macOS Ventura) are the entry points for collaboration **[NEW]**.
- A popup button in the header lets users choose between "Collaborate" and "Send a Copy."
- Apps that only support collaboration (no copy sending) should hide the popup button.
- Permission summary string (e.g., "Everyone can make changes") opens the permissions settings screen — keep it concise to avoid truncation.
- Permission settings screen should use a simple, skimmable structure for quick decision-making.
- macOS share popover mirrors the iOS design with platform-appropriate components.

### Collaboration Button
- Once collaboration begins, a **collaboration button** appears in the app toolbar **[NEW]**.
- Place it adjacent to the share button in a prominent, visible location.
- Appearance varies: one-on-one chat shows the recipient's profile picture; group with a photo shows group photo; group without a photo uses a system-provided symbol.

### Collaboration Popover
- Three-section structure **[NEW]**:
  1. **Top section** (system-provided): participant avatars from Contacts; Messages and FaceTime buttons.
  2. **Middle section** (fully customizable): show relevant app info (active participants, recent activity, action buttons). Keep it glanceable — avoid overloading.
  3. **Bottom section** (system-provided): "Manage Shared File" button (label customizable) opens participant management screen.
- If the app has nothing to add, the middle section can be left empty.
- Use macOS UI components/patterns when adapting the popover for macOS.
- If using CloudKit sharing, the management screen is provided by the system; otherwise provide your own.

### Messages Collaboration Banners
- Apps can send activity banners into Messages conversations **[NEW]**.
- Banner templates: "Edits made," "Comments added," "You were mentioned," "Files modified."
- Messages consolidates multiple banners from multiple documents into a single grouped banner to reduce clutter.
- Users tap "View" to see all updates and choose which document to open.

### Drag and Drop
- Users can drag a file directly into a Messages conversation to initiate collaboration without the share sheet.
- Apps should allow permission configuration from the Messages input field in this flow.

## APIs & Frameworks
- **UIActivityViewController** / **NSSharingServicePicker** — system share sheet / popover **[NEW collaboration UI]**
- **UICollaborationButton** — collaboration button in toolbar **[NEW]**
- **UIActivityItemSource** — customize share sheet content and metadata
- **CloudKit sharing** — system-provided participant management screen when used
- **Messages collaboration integration** — link conversations to shared documents **[NEW]**
- **FaceTime** — audio/video call escalation from collaboration popover
- **CollaborationPopover** — three-section customizable popover **[NEW]**
- **NSItemProvider** — used for drag-and-drop collaboration initiation
- **SharedWithYou** framework — surfaces collaboration content in Messages **[NEW]**

## Code Highlights
This is a design-focused session; no direct code samples were presented. The companion development session covers the implementation APIs. Key design decisions:

```
Share button placement: toolbar, adjacent to collaboration button
Permission summary string: concise, avoid truncation
Collaboration popover middle section: glanceable, consistent styling
Banner type selection: choose the most relevant template per event type
```

## Takeaways
- Put Messages at the center of collaboration: use the system share sheet so users share via existing conversations rather than exchanging email addresses.
- The collaboration button is the most critical new UI element — place it prominently in the toolbar next to the share button so users can always find their way back to the conversation.
- Customize the collaboration popover's middle section with the most relevant, glanceable information for your app; don't overload it.
- Use Messages activity banners to keep collaborators informed of changes without requiring them to open the document, and let Messages handle consolidation of multiple notifications.

---
_Source: WWDC22 Session 10015 page (abstract, chapter summaries, code samples, and resource links)._
