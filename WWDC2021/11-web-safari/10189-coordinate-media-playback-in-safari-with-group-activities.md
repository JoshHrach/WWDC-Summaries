# Coordinate Media Playback in Safari with Group Activities
**WWDC21 · Session 10189** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10189/)

_Platforms:_ macOS Monterey 12, Safari 15

## Overview
This session shows how to extend a SharePlay / Group Activities experience to companion websites in Safari 15 on macOS Monterey, enabling users on Mac to join a synchronized watch session without installing a native app. The session covers two pieces: (1) preparing the native iOS/iPadOS app to specify a `fallbackURL` in its `GroupActivityMetadata`, and (2) implementing the W3C Media Session API plus a new experimental `Coordinator` property on the webpage to handle synchronized playback from the browser side.

When an iPhone user in a FaceTime call shares a Group Activity that has a `fallbackURL`, Mac users are notified and can open the URL directly in Safari 15. The website then uses the Media Session Coordinator to synchronize play, pause, and seek commands with all participants — including those on the native iOS app.

## Key Topics

### App-Side: fallbackURL in GroupActivityMetadata
The native app's `GroupActivity` must include a `fallbackURL` in its `GroupActivityMetadata` pointing to the specific content page on the companion website. When a Mac user receives the Group Activity invitation, Safari opens that URL.

### Media Session API (Web Standard)
The Media Session API (`navigator.mediaSession`) tells the browser about the current media state. Set `playbackState` ('playing' or 'paused'), call `setPositionState({duration, playbackRate, position})`, and set `metadata` (title, artwork URL) with `MediaMetadata`. Register action handlers for 'play', 'pause', 'seekto' so the browser can respond to system controls like Now Playing on macOS.

### Media Session Coordinator (Experimental Web API)
`navigator.mediaSession.coordinator` — a new experimental Safari 15 property — implements the synchronization layer on the web. Listen for `coordinatorchange` events to detect when a session becomes available. Call `coordinator.join()` to participate. Replace direct `video.play()` / `video.pause()` / `video.currentTime` calls in UI event handlers with `coordinator.play()` / `coordinator.pause()` / `coordinator.seekTo(time)`. The coordinator negotiates timing with all participants and then calls the page's registered Media Session action handlers when everyone is ready.

Key caveats:
- The Coordinator API is experimental and not yet standardized; the API may change.
- Currently only implemented in Safari.
- A Group Session must be initiated from an iPhone, iPad, or macOS native app; web users can only join, not start.

### Now Playing Integration
Safari automatically populates the macOS menu bar Now Playing widget using the metadata and position state provided through Media Session, and routes Now Playing control events back through the page's registered action handlers.

## APIs & Frameworks

**Group Activities / Swift** (native app side)
- `GroupActivityMetadata.fallbackURL` **[NEW]** — URL pointing to companion website content page; used to launch Safari on Mac when native app is not installed

**Web APIs — Media Session (W3C Standard, Safari 15)**
- `navigator.mediaSession` — the `MediaSession` object **[available in Safari 15]**
- `navigator.mediaSession.setActionHandler(type, handler)` — register handlers for 'play', 'pause', 'seekto', 'previoustrack', 'nexttrack', etc.
- `navigator.mediaSession.playbackState` — string: 'playing' | 'paused' | 'none'
- `navigator.mediaSession.setPositionState({duration, playbackRate, position})` — current playback position info
- `navigator.mediaSession.metadata` — `MediaMetadata` object (title, artist, artwork array)
- `MediaMetadata({title, artist, artwork})` — constructor for session metadata

**Web APIs — Media Session Coordinator (Experimental, Safari 15)** **[NEW]**
- `navigator.mediaSession.coordinator` — `MediaSessionCoordinator` | null
- `coordinatorchange` event on `navigator.mediaSession` — fires when coordinator is set or cleared
- `MediaSessionCoordinator.join()` — join the active group session **[NEW]**
- `MediaSessionCoordinator.leave()` — leave the group session **[NEW]**
- `MediaSessionCoordinator.play()` — signal intent to play (negotiated) **[NEW]**
- `MediaSessionCoordinator.pause()` — signal intent to pause **[NEW]**
- `MediaSessionCoordinator.seekTo(time)` — signal intent to seek **[NEW]**
- `MediaSessionCoordinator.state` — 'joining' | 'joined' | 'closed' **[NEW]**

## Code Highlights

Native app — specify fallbackURL:
```swift
struct WatchTogether: GroupActivity {
    var contentIdentifier: String
    func metadata() async -> GroupActivityMetadata {
        var metadata = GroupActivityMetadata()
        metadata.fallbackURL = URL(string: "https://example.com/title/\(contentIdentifier)")
        return metadata
    }
}
```

Web page — adopt Media Session:
```javascript
if (navigator.mediaSession) {
    navigator.mediaSession.setActionHandler('play', () => video.play());
    navigator.mediaSession.setActionHandler('pause', () => video.pause());
    navigator.mediaSession.setActionHandler('seekto', details => {
        video.currentTime = details.seekTime;
    });
    navigator.mediaSession.metadata = new MediaMetadata({
        title: myPlayer.titleString,
        artwork: [{ src: myPlayer.artworkURL }]
    });
}

function updateMediaSessionState() {
    navigator.mediaSession.playbackState = video.paused ? 'paused' : 'playing';
    navigator.mediaSession.setPositionState({
        duration: video.duration,
        playbackRate: video.playbackRate,
        position: video.currentTime
    });
}
for (const event of ['playing', 'pause', 'durationchange', 'ratechange'])
    video.addEventListener(event, updateMediaSessionState);
```

Web page — adopt Coordinator:
```javascript
navigator.mediaSession.addEventListener('coordinatorchange', event => {
    const coordinator = navigator.mediaSession.coordinator;
    if (coordinator) coordinator.join();
    inSessionIcon.hidden = !coordinator;
});

playButton.addEventListener('click', () => {
    const c = navigator.mediaSession.coordinator;
    c ? c.play() : video.play();
});
pauseButton.addEventListener('click', () => {
    const c = navigator.mediaSession.coordinator;
    c ? c.pause() : video.pause();
});
timeline.addEventListener('change', event => {
    const c = navigator.mediaSession.coordinator;
    c ? c.seekTo(event.target.value) : (video.currentTime = event.target.value);
});
```

## Takeaways
- A single `fallbackURL` in `GroupActivityMetadata` is all the native app changes required to enable Safari-based SharePlay participation.
- Adopt the W3C Media Session API first — it provides Now Playing integration and the action handlers that the Coordinator later calls.
- Replace UI control handlers' direct media element calls with `coordinator.play()`, `coordinator.pause()`, and `coordinator.seekTo()` to propagate commands to all session participants.
- The Coordinator API is experimental (Safari 15 only at time of session); group sessions must still be started from native apps — web users can only join.

---
_Source: WWDC21 Session 10189 page (abstract, chapter summaries, code samples, and resource links)._
