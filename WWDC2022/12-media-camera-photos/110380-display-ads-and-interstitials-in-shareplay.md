# Display Ads and Interstitials in SharePlay
**WWDC22 · Session 110380** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110380/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, tvOS 16

## Overview
This session addresses a core challenge of SharePlay coordinated playback: when participants receive different ad schedules (different ad durations, or some participants having no ads at all), how do you maintain a synchronized group viewing experience? The session explains the two main synchronization models, provides concrete APIs for both stitched-in ads and HLS Interstitials, and offers best practices for minimizing disruption to the viewing experience.

The fundamental principle is that the `AVPlayerPlaybackCoordinator` needs to know which time ranges in the media timeline are "interstitials" (ads, banners, warnings) so it can treat them differently from primary content. For stitched-in ads, you provide these ranges via a delegate method. For HLS Interstitials, AVFoundation handles them automatically.

## Key Topics

### The Coordination Challenge
- Coordinated playback in SharePlay syncs playback commands across all participants in a FaceTime group.
- When participants have different ad durations (or one has no ads), their presentation timelines diverge.
- Without ad-aware coordination, participants end up watching completely different parts of the content at the same time.

### Waiting Policies
Two experiences are possible when participants have different-duration ads:
1. **Wait for all**: The group pauses main content until all participants finish their ads, then resumes together. No content is skipped.
2. **Skip to catch up** (default): Participants who finish ads early continue; those who finish late skip ahead to catch up. Some program content is missed.

Configured via `AVPlayerPlaybackCoordinator.suspensionReasonsThatTriggerWaiting`:
- **Default** (no waiting): Participants who finish ads catch up by skipping content.
- **Wait policy**: Include `.playingInterstitial` in `suspensionReasonsThatTriggerWaiting` to pause and wait.
- While waiting: `AVPlayer.timeControlStatus == .waitingToPlayAtSpecifiedRate` and `waitingReason == .waitingForCoordinatedPlayback`.

### Stitched-In Ads (Discontinuity-Based VOD)
- For ads stitched into primary content playlists using HLS discontinuity tags:
- Implement `AVPlayerPlaybackCoordinatorDelegate` and the new delegate method **[NEW]**:
  `playbackCoordinator(_:interstitialTimeRangesFor:)` — return an array of `NSValue`-wrapped `CMTimeRange` objects representing ad/interstitial time ranges.
- Populate by summing `EXTINF` tag durations from the playlist.
- When a participant enters a declared time range, coordination uses the waiting policy.
- Seeking into a declared time range snaps the entire group to the start of that range.
- For VOD: all participants' assets must have matching **primary content duration** (ad durations may differ).
- For live content with stitched-in ads: ad break durations must match across all participants.

### HLS Interstitials
- Ads and interstitials treated as separate objects outside the primary content timeline.
- Scheduled via Server Side Ad Insertion (date range tags in the media playlist) or client-side via AVFoundation APIs.
- With HLS Interstitials: AVFoundation automatically handles coordination — just specify the waiting policy.
- Primary content durations must still match across participants for SharePlay compatibility.
- For mixed groups (some have ads, some don't) watching live content: use HLS Interstitials with the default (no-wait) policy. Participants without ads continue watching; those with ads sync up after their ad completes.

### Best Practices
- **Align ad break durations** across participants to minimize wait/skip times.
- **Mixed groups watching live**: Use HLS Interstitials + default policy. Premium subscribers continue the live feed; ad viewers rejoin in sync when ad ends (e.g., just in time for sports play to resume).
- **VOD where missing content matters**: Use the wait policy. Share ad schedules via `GroupSessionMessenger` so the app knows exactly how long each participant will wait — use this to show "upcoming attractions" or other engaging content to waiting participants instead of a blank screen.
- Keep primary content durations equal across all participants for SharePlay compatibility.

## APIs & Frameworks

### Group Activities / SharePlay
- `GroupSession` — SharePlay session management (existing)
- `GroupSessionMessenger` — send custom messages to share ad schedules across participants **[used for best practice]**
- `AVPlayerPlaybackCoordinator` — coordinates playback across SharePlay group

### AVFoundation
- `AVPlayerPlaybackCoordinator.suspensionReasonsThatTriggerWaiting: [AVPlayerPlaybackCoordinator.SuspensionReason]` — configure waiting policy
- `AVPlayerPlaybackCoordinator.SuspensionReason.playingInterstitial` — include to wait for interstitial participants
- `AVPlayerPlaybackCoordinatorDelegate` — delegate protocol
- `AVPlayerPlaybackCoordinatorDelegate.playbackCoordinator(_:interstitialTimeRangesFor:) -> [NSValue]` — provide interstitial time ranges **[NEW]**
- `AVPlayer.timeControlStatus` — `.waitingToPlayAtSpecifiedRate` during wait
- `AVPlayer.reasonForWaitingToPlay` — `.waitingForCoordinatedPlayback` during coordinated wait
- HLS Interstitials — AVFoundation manages automatically during coordinated playback
- `AVPlayerItem` — associated with coordinator via player item

## Code Highlights

```swift
// Provide interstitial time ranges for stitched-in ads
class MyCoordinatorDelegate: NSObject, AVPlayerPlaybackCoordinatorDelegate {
    func playbackCoordinator(
        _ coordinator: AVPlayerPlaybackCoordinator,
        interstitialTimeRangesFor playerItem: AVPlayerItem
    ) -> [NSValue] {
        return interstitialTimeRanges  // Array of NSValue-wrapped CMTimeRange
    }
}

// Enable waiting for other participants' ads before resuming
coordinator.suspensionReasonsThatTriggerWaiting = [.playingInterstitial]

// While waiting, detect the state:
if player.timeControlStatus == .waitingToPlayAtSpecifiedRate,
   player.reasonForWaitingToPlay == .waitingForCoordinatedPlayback {
    // Show "upcoming attractions" or other content
}
```

## Takeaways
- Declare interstitial time ranges via the new `AVPlayerPlaybackCoordinatorDelegate` method for stitched-in ads, or use HLS Interstitials and let AVFoundation handle coordination automatically.
- Choose the waiting policy based on content type: VOD where missing content matters → wait; live content with mixed subscribers → default (skip/catch-up).
- For best UX, align ad durations when possible, and use `GroupSessionMessenger` to share schedules so waiting participants can be shown engaging content rather than a blank screen.
- Assets in a SharePlay group must have matching primary content duration regardless of ad duration differences.

---
_Source: WWDC22 Session 110380 page (abstract, chapter summaries, code samples, and resource links)._
