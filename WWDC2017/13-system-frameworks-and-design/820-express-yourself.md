# Express Yourself!
**WWDC17 · Session 820** · [Watch](https://developer.apple.com/videos/play/wwdc2017/820/)

_Platforms:_ iOS 11, Messages (iMessage App Store)

## Overview
This design session focuses on the principles behind successful iMessage apps and sticker packs. Presented one year after the iMessage App Store launched, it distills what differentiates breakout iMessage experiences from ones that fail to gain traction. The core thesis is that iMessage apps must embody three qualities: they must be **convenient** (fast, lightweight, low friction), **personal** (enabling or showcasing individual expression), and **fun** (enjoyable to send, receive, and interact with). These qualities apply equally to sticker packs, expressive tools, sharing utilities, and collaborative apps like games and polls.

The session uses three real-world examples — Tumblr (GIF generator), Dunkin' Donuts (gift cards), and Tinder (image polls) — to show how apps with very different iOS experiences each carved out a focused, high-value iMessage-specific feature. None of them attempted to replicate their full app inside Messages; instead, each found a single communication-oriented capability that made sense in a conversational context.

## Key Topics

- **Stickers as the baseline** — the sticker experience defines the foundational UX contract for all iMessage apps: peel-and-place on any bubble (including older bubbles), overlay on top of images, scale and rotate freely. Full-screen message effects (e.g., Echo) add expressiveness. The half/full-screen drawer design keeps the conversation visible and the experience lightweight.
- **Three categories of iMessage apps** — (1) expressive apps (GIF generators, drawing tools, creative customization); (2) sharing apps (gift cards, tickets, commerce); (3) collaborative apps (games, polls, shared decisions).
- **Expressive apps (Tumblr)** — the Tumblr iMessage app is a GIF editor, not a social media viewer. Users trim video, set playback speed and loop behavior, add customizable text (font style, color, scale, rotation), and send directly from Messages. This is a communication-first feature built from scratch for the conversational context, not a port of the iOS app.
- **Sharing apps (Dunkin' Donuts)** — gift card sending is the one feature from the iOS app that makes sense in Messages. Apple Pay integration completes the transaction without leaving the conversation. The design insight: identify the one feature of your app that maps to a "giving" or "sharing" human gesture.
- **Collaborative apps (Tinder)** — Tinder's Stacks feature lets users create an image poll and send it into a conversation; recipients vote by tapping rather than by typing. Polling becomes an evolving event for both sender and receiver. The design insight: collaborative features require both parties to participate, so the interaction must be low-friction on both ends.
- **Design tip 1: focus on communication** — do not try to put your entire app in Messages. Select only the features that are inherently communicative — features where the value depends on sharing with another person.
- **Design tip 2: enable personalization** — handcrafted messages carry more emotional weight than generic ones. Give users controls to customize: text, color, sticker placement, image selection. Personalization creates delight on both sides of the conversation.
- **Design tip 3: leverage platform tools** — iMessage provides peel-and-place stickers, Apple Pay, microphone access, camera and photo library access, and full-screen effects. Choose the platform affordances that best serve your feature, rather than re-implementing primitives from scratch.
- **Convenience as a hard constraint** — every interaction in Messages competes with the conversation itself for attention. If an action requires too many taps, users switch back to the conversation and never return. Every iMessage feature must be achievable in seconds.

## APIs & Frameworks

**Messages framework**
- `MSMessagesAppViewController` — root view controller for iMessage app extensions; handles transitions between compact and expanded presentation modes
- `MSConversation` — represents the active conversation; used to insert messages and stickers
- `MSMessage` — a rich message object with URL, layout, and session support for interactive/collaborative messages
- `MSMessageLayout` / `MSMessageTemplateLayout` — defines how a sent message bubble appears in the conversation transcript
- `MSStickerBrowserViewController` — built-in browser for sticker pack apps
- `MSSticker` — individual sticker object created from a file URL
- `MSStickerView` — renders a sticker and supports the peel-and-place drag interaction natively

**PassKit / Apple Pay**
- `PKPaymentAuthorizationViewController` — present Apple Pay sheet inside the iMessage extension for commerce/gifting flows

**Photo and Camera access**
- `UIImagePickerController` — select photos/videos from library within the extension
- `PHPhotoLibrary` — direct access for more complex media selection workflows

**Info.plist Keys**
- `NSExtension` → `NSExtensionPrincipalClass` — entry point for the extension
- `NSExtension` → `MSMessagesAppPresentationContextMessages` — declares the extension works in the Messages context

## Code Highlights

No code samples were presented. The session is a design and strategy talk.

Key implementation note: interactive/collaborative messages use `MSMessage` with a session (`MSSession`) so that when a recipient taps the bubble and interacts, the original sender sees the updated state in their conversation without a new message being inserted.

## Takeaways

- Choose a single communication-oriented feature from your iOS app and build the iMessage extension around it exclusively — do not port the entire app.
- Measure every interaction against the convenience threshold: if it takes more than a few taps to produce and send the content, it will not be used.
- Build in personalization controls (text, color, position, scale) even for simple sharing features — the act of customizing is itself a form of expression.
- Use `MSMessage` with `MSSession` for any collaborative or game feature so both participants see the evolving state without generating a flood of new message bubbles.

---
_Source: WWDC17 Session 820 page (abstract, transcript, and resource links)._
