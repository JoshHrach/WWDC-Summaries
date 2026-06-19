# Designing Sound
**WWDC17 · Session 803** · [Watch](https://developer.apple.com/videos/play/wwdc2017/803/)

_Platforms:_ iOS 11, macOS High Sierra 10.13, watchOS 4, tvOS 11

## Overview
This session, presented by Apple's Human Interface team sound designer, makes the case that sound is an equal design discipline alongside visual design. Sound shapes product experience, communicates meaning without requiring the user to look at the screen, and gives apps a distinctive voice and brand identity. The session covers the purpose of sound in UI, how to design effective notification sounds, how Apple approaches UI element sounds (keyboard clicks, button presses, transitions), and practical tips for getting started with sound design — including using tools developers and designers already have.

The session reveals real Apple design stories: how the Messages notification was recorded on a kalimba, how the iPhone 7 Home button's haptic click is accompanied by a synthetic sound that was carefully synchronized (changing the sound by 10 ms changes the user's perception of the haptic feel itself), how Apple TV transition sounds convey direction and depth, and how the Apple Watch sounds were derived from recordings of the actual watch housing materials.

The overall message is that every developer, regardless of audio background, should at minimum consider creating a custom notification sound rather than defaulting to the system beep — doing so is one of the most effective and underused branding opportunities on the platform.

## Key Topics

- **Why sound matters** — sound communicates context (church bells), provides early warnings (screeching brakes), gives instant confirmation (seatbelt click), and conveys emotional state; it shapes how users feel about a product.
- **Notification sound design** — default system sounds convey no app identity; a custom notification sound can be brand-defining; the Dark Sky rain notification is analyzed as a best practice (recognizable, conveys meaning, friendly, simple, clean).
- **Sound design process** — start with exploratory recording of real-world sources related to the app concept; prototype on device; iterate; most early ideas are discarded; the goal of exploration is to discover what you don't know.
- **Good notification sound characteristics** — distinguishable (identifies the app), matches the app's aesthetic, unobtrusive and repeatable (not annoying after daily exposure), clear and clean, cuts through background noise without being abrasive.
- **UI sound design** — keyboard sounds (iOS keyboard varies across modifier/space/backspace keys, volume drops slightly at high typing speeds), button press sounds (iPhone 7 Home button uses Taptic Engine + synthesized click sound, synchronized to ±10 ms precision), transition sounds (Apple TV sounds convey depth and directionality).
- **Haptics + sound synchronization** — the same haptic tap sounds and feels very different to users depending on the accompanying sound; 10 ms of timing difference is perceptible; this is a fundamental multisensory design principle.
- **UI sound characteristics** — used sparingly, much lower volume than notifications (phone is in hand), helps users understand state changes, best combined with haptics and animation; silence is a deliberate design choice.
- **Practical guidelines** — keep notifications short (long sounds duck music for too long); synchronize sound to animation/haptics precisely; test on device (iPhone speaker has low-frequency limitations); test with headphones; filter out unused low frequencies.
- **Starting tools** — Voice Memos (recording), Music Memos (musical ideas, integrates with GarageBand), GarageBand (editing, denoising with "iChat Voice" preset, track automation for fade-outs), Logic Pro (professional use).
- **When to involve experts** — a sound designer develops the overall concept; a sound engineer cleans sounds for production use.

## APIs & Frameworks

This is a sound design principles session with no new API introductions. Relevant platform APIs referenced:

- **AVFoundation / AVAudioPlayer** — playback of custom notification and UI sounds
- **UNUserNotificationCenter** — registering custom notification sounds via `UNNotificationSound(named:)`
- **UNNotificationSound** — specifies custom sound files for local and remote notifications
- **CoreHaptics** (context) — haptic engine used in iPhone 7 Home button; Taptic Engine synchronization discussed
- **UIFeedbackGenerator** (`UIImpactFeedbackGenerator`, `UISelectionFeedbackGenerator`, `UINotificationFeedbackGenerator`) — generate haptic feedback synchronized with sound
- **GarageBand / Logic Pro** — used for recording, editing, denoising, and fading custom sounds
- **Voice Memos app** — built-in iOS app for field recording of sound sources
- **Music Memos app** — free app for capturing musical ideas; integrates with GarageBand

Sound file requirements for notifications:
- Format: aiff, wav, or caf
- Duration: ≤ 30 seconds
- Must be in the app bundle or `Library/Sounds` directory

## Code Highlights

Registering a custom notification sound:
```swift
import UserNotifications

let content = UNMutableNotificationContent()
content.title = "New Toast Discovered"
content.sound = UNNotificationSound(named: UNNotificationSoundName("toast_chime.caf"))

let trigger = UNTimeIntervalNotificationTrigger(timeInterval: 5, repeats: false)
let request = UNNotificationRequest(identifier: "toastNotification",
                                    content: content, trigger: trigger)
UNUserNotificationCenter.current().add(request)
```

## Takeaways

- Create a custom notification sound for any app that sends frequent notifications — it is one of the most effective and underused branding and identity opportunities on the platform.
- Sound, haptics, and animation must be synchronized precisely; even 10 ms of drift changes user perception of all three modalities together.
- Use sparingly for UI sounds — silence is a design choice; too many sounds make the app feel like a game when it shouldn't; always provide an option to turn UI sounds off.
- Start with recordings of real-world sources related to your app's concept; most ideas won't work, but the exploration reveals what you actually need; GarageBand provides enough post-processing for prototyping.

---
_Source: WWDC17 Session 803 page (abstract, transcript, and resource links)._
