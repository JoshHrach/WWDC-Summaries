# Immerse Your App in Spatial Audio
**WWDC21 · Session 10265** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10265/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, tvOS 15

## Overview
Spatial audio is a psychoacoustic technology that creates a virtual soundstage providing a theater-like listening experience. It works with multichannel and stereo content, both audiovisual and audio-only. For apps using AVPlayer with HLS content containing multichannel audio alternates, spatial audio is enabled automatically with no code changes required. iOS/iPadOS 15 and tvOS 15 expand API support to `AVSampleBufferAudioRenderer` and introduce new properties for detecting spatial audio availability and advertising multichannel content support.

## Key Topics

**How Spatial Audio Works**
Unlike classic stereo (where audio is tied to the headphones and moves with head movement), spatial audio maintains a static virtual soundstage. On headphones (AirPods Pro, AirPods Max), inertial measurement units track head pose relative to the playback device and dynamically adjust audio rendering to maintain the fixed soundstage — producing an "out-of-head" experience. Stereo sources can be up-mixed to simulate 5.1 channels.

**Adaptive Spatial Audio (HLS)**
For HLS streams, spatial audio is adaptive to bandwidth. When bandwidth is insufficient for full multichannel delivery, the audio degrades seamlessly to a spatially up-mixed stereo treatment (head-tracking maintained). When bandwidth recovers, full multichannel treatment is restored. Important: normalize volume levels between stereo and multichannel renditions and include DRC (Dynamic Range Control) and dialnorm metadata in encodings.

**AVAudioSpatializationFormats**
`AVAudioSpatializationFormats` is an `OptionSet` controlling which audio source formats are spatialized:
- `.monoAndStereo` — spatialize mono and stereo sources
- `.multichannel` — spatialize multichannel sources only
- `.monoStereoAndMultichannel` — spatialize all formats
- `0` / empty — inhibit spatialization

Set via `allowedAudioSpatializationFormats` on `AVPlayerItem` or (new in iOS 15) `AVSampleBufferAudioRenderer`.

**Spatial Audio Availability Detection (NEW in iOS 15)**
`AVAudioSessionPortDescription.isSpatialAudioEnabled` indicates whether a connected audio port supports spatial audio AND the user has enabled it. Apps should observe `AVAudioSession.routeChangeNotification` and re-check this property on each route change. A new `AVAudioSession.spatialPlaybackCapabilitiesChangedNotification` fires when the user changes spatial audio preferences in Control Center or Bluetooth settings; read `AVAudioSessionSpatialAudioEnabledKey` from the notification's `userInfo`.

**Multichannel Content Advertising (NEW in iOS 15)**
`AVAudioSession.setSupportsMultichannelContent(_:)` advertises to the system that the app can deliver multichannel audio. This is reflected in Control Center and Bluetooth settings to inform users that a spatial experience is available. `AVPlayer`-based apps get this management automatically; apps using `AVSampleBufferAudioRenderer` must call this explicitly.

**Platform Support History**
- iOS/iPadOS 13, macOS Catalina: built-in speakers, `AVPlayerItem`, WebKit `<video>` tag; 2018+ devices
- iOS/iPadOS 14, macOS Big Sur: added AirPods Pro / AirPods Max head-tracking support; 2016+ iPhone/iPad paired devices
- iOS/iPadOS 15, macOS Monterey, tvOS 15 (new): `AVSampleBufferAudioRenderer` support; stereo up-mix spatialization enabled by default for all sources; limited WebKit MSE support (no spatialization API, but `AudioConfiguration` in Media Capabilities API can detect support)

## APIs & Frameworks

- **AVFoundation** — `AVPlayerItem`, `AVSampleBufferAudioRenderer`, `AVAudioSession`
- `AVAudioSpatializationFormats` — `OptionSet` **[UPDATED]**
  - `.monoAndStereo`
  - `.multichannel`
  - `.monoStereoAndMultichannel`
- `AVPlayerItem.allowedAudioSpatializationFormats: AVAudioSpatializationFormats` (macOS 11+, iOS 14+)
- `AVSampleBufferAudioRenderer.allowedAudioSpatializationFormats: AVAudioSpatializationFormats` **[NEW]** (iOS 15, tvOS 15)
- `AVAudioSessionPortDescription.isSpatialAudioEnabled: Bool` **[NEW]** (iOS 15)
- `AVAudioSession.spatialPlaybackCapabilitiesChangedNotification: NSNotification.Name` **[NEW]** (iOS 15)
- `AVAudioSessionSpatialAudioEnabledKey: String` **[NEW]** — key for notification `userInfo`
- `AVAudioSession.setSupportsMultichannelContent(_ inValue: Bool) throws` **[NEW]** (iOS 15)
- `AVAudioSession.supportsMultichannelContent: Bool` **[NEW]** (iOS 15)
- **HLS** — multichannel audio alternates in variant playlists (no code change needed for `AVPlayer`)
- **WebKit MSE** — limited spatial audio support via W3C Media Capabilities API `AudioConfiguration` (iOS 15)
- **Core Audio** — underlying audio framework

## Code Highlights

Spatialization formats API:
```swift
public struct AVAudioSpatializationFormats: OptionSet {
    public static var monoAndStereo: AVAudioSpatializationFormats { get }
    public static var multichannel: AVAudioSpatializationFormats { get }
    public static var monoStereoAndMultichannel: AVAudioSpatializationFormats { get }
}
// Set on AVPlayerItem or AVSampleBufferAudioRenderer:
playerItem.allowedAudioSpatializationFormats = .monoStereoAndMultichannel
```

Detect spatial audio availability on current route (iOS 15):
```swift
let session = AVAudioSession.sharedInstance()
for output in session.currentRoute.outputs {
    if output.isSpatialAudioEnabled {
        // spatial audio is available and enabled on this port
    }
}
// Observe changes:
NotificationCenter.default.addObserver(forName: AVAudioSession.spatialPlaybackCapabilitiesChangedNotification, ...) { note in
    let enabled = note.userInfo?[AVAudioSessionSpatialAudioEnabledKey] as? Bool
}
```

Advertise multichannel support (iOS 15):
```swift
try AVAudioSession.sharedInstance().setSupportsMultichannelContent(true)
```

## Takeaways

- For HLS-based apps using `AVPlayer`, spatial audio with multichannel audio alternates works with zero code changes — simply include multichannel audio renditions in the HLS playlist.
- iOS 15 extends spatial audio API to `AVSampleBufferAudioRenderer`, enabling custom player pipelines to participate in spatial audio.
- Use `AVAudioSessionPortDescription.isSpatialAudioEnabled` and `spatialPlaybackCapabilitiesChangedNotification` to adapt your UI when spatial audio is available or becomes unavailable.
- Volume normalization (stereo vs. multichannel) and DRC/dialnorm metadata are critical for a smooth adaptive spatial audio experience.

---
_Source: WWDC21 Session 10265 page (abstract, chapter summaries, code samples, and resource links)._
