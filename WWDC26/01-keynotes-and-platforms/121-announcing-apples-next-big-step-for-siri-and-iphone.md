# Announcing Apple's Next Big Step for Siri and iPhone
**WWDC26 · Session 121** · [Watch](https://developer.apple.com/videos/play/wwdc2026/121/)

_Platforms:_ iOS 27, iPadOS 27

## Overview
This session announces the major leap forward for Siri and iPhone in iOS 27. It introduces Siri AI — a dramatically more capable version of Siri with enhanced natural language abilities, deeper system integration, and new AI-powered features across the iPhone experience. This is positioned as the next big step in Apple's multi-year journey with Apple Intelligence.

The session covers three major new capabilities: next-generation image transformation and photorealistic image creation via Image Playground, a more conversational and capable Siri AI that can write and edit emails, texts, and documents, and enhanced privacy-first personal information safety built into the Siri experience.

This announcement sets the stage for a broad set of related developer sessions at WWDC26 covering the APIs that power these features.

## Key Topics

**Siri AI in iOS 27**
The next generation of Siri brings significantly more powerful natural language understanding and generation. Siri can now engage in more conversational interactions, understand context across a dialogue, and take more complex actions on behalf of users — including editing and composing emails, text messages, and documents.

**Image Playground — Photorealistic Image Generation**
Image Playground gains next-generation AI capabilities for transforming and creating images. It can now generate photorealistic imagery used for wallpapers, Contact Posters, and system backgrounds across iOS 27. This represents a significant leap from the stylized image generation available in earlier versions.

**Conversational Writing and Editing**
Siri AI can now assist users in writing and editing content across the system — emails, texts, and documents — using natural language conversation. Users can describe what they want, have Siri draft or revise content, and refine results through dialogue.

**Privacy and Password Safety**
Siri AI gains new capabilities to protect users' personal information and proactively identify compromised passwords. Compromised passwords can be updated with just a tap — a significant UX improvement for account security that builds on existing Password Manager capabilities.

## APIs & Frameworks
_This is a feature announcement session. The following products and technologies are named in the abstract:_

- **Siri AI** **[NEW]** — next-generation Siri with enhanced natural language abilities in iOS 27
- **Apple Intelligence** — the underlying AI platform powering all Siri AI capabilities
- **Image Playground** **[UPDATED]** — photorealistic image generation for wallpapers, Contact Posters, and system backgrounds
- **iOS 27** — the platform release containing all these capabilities
- **iPadOS 27** — also includes Siri AI capabilities
- **App Intents** — the framework enabling third-party app integration with Siri AI actions
- **Password Manager / Passwords app** — compromised password detection and one-tap remediation
- **Contact Posters** — personalized contact appearance, now with AI-generated photorealistic imagery

## Code Highlights
No code samples in this session (announcement/overview format).

## Takeaways
- Siri AI in iOS 27 is a platform-level shift — developers should review how their apps integrate with App Intents to take advantage of the more capable Siri.
- Image Playground's photorealistic generation is now available system-wide — consider how your app could expose image creation or transformation to users.
- The conversational writing assistant capabilities create new user expectations for AI-powered editing features in apps — consider where this pattern fits your app.
- Password safety improvements mean users will be more likely to update compromised credentials; apps relying on older auth flows should ensure compatibility with updated credentials.

---
_Source: WWDC26 Session 121 page (abstract only). No chapters or code samples were listed on the session page at time of retrieval._
