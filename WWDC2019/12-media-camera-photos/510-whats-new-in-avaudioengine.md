# What's New in AVAudioEngine
**WWDC19 · Session 510** · [Watch](https://developer.apple.com/videos/play/wwdc2019/510/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15

## Overview
AVAudioEngine in iOS 13 and macOS Catalina receives three significant additions: a Voice Processing mode for echo cancellation in VoIP applications, two new nodes (`AVAudioSourceNode` and `AVAudioSinkNode`) that let apps inject and extract audio from the processing graph via user-defined blocks, and improvements to spatial audio rendering including an automatic algorithm selector and multichannel spatialization support (including higher-order Ambisonics). The session also covers new `AVAudioSession` additions: a prompt style hint API and an option to allow haptics and system sounds during audio recording.

## Key Topics

**Voice Processing Mode**
Enables acoustic echo cancellation and noise suppression suitable for VoIP apps. When activated, both the input and output `AVAudioIONode` switch to voice processing mode; the engine ensures both nodes are configured together. Voice processing is only available in real-time (non-manual) rendering mode. The engine must be stopped before enabling/disabling. Demonstrated by the AVEchoTouch sample project.

**AVAudioSourceNode**
A new node initialized with a render block that produces audio from app-supplied data. The block receives the required format, frame count, and audio buffer list to fill. Runs under real-time constraints in audio device mode — no blocking calls inside the block. Supports linear PCM conversions (sample rate, bit depth). Has one output bus and no input bus. Eliminates the need to implement a full Audio Unit just to generate custom audio.

**AVAudioSinkNode**
The symmetrical counterpart of `AVAudioSourceNode`. Initialized with a receive block that processes audio coming from the input node chain. Must be connected downstream of the input node (restricted to the input chain). Does not support format conversions — the block format must match the hardware input format. More suitable than `AVAudioNode.installTap` for real-time VoIP processing because the block runs in real-time context.

**Spatial Audio Rendering Improvements**
Two key additions to `AVAudio3DMixing`:
1. `AVAudio3DMixingRenderingAlgorithm.auto` — automatically selects the best spatialization algorithm for the current audio route (near-field/in-head for headphones, virtual surround for built-in speakers). Available on iOS devices and MacBooks from 2018 and newer.
2. Multichannel spatialization via `AVAudio3DMixing.sourceMode`:
   - `.spatializeIfMono` — legacy; multichannel streams pass through or are downmixed
   - `.pointSource` — sums to mono and renders at the player node's 3D position
   - `.ambienceBed` — anchors to the 3D world; rotatable with listener orientation; supports channel-based formats and higher-order Ambisonics up to third order

**AVAudioSession Additions**
- `AVAudioSession.promptStyle` — a new hint enum (`.none`, `.short`, `.normal`) that navigation and voice-prompt apps should observe; reflects system audio state (e.g., Siri speaking or call active) to suppress or shorten verbal prompts
- `AVAudioSession.allowHapticsAndSystemSoundsDuringRecording` — boolean property (settable via `setAllowHapticsAndSystemSoundsDuringRecording(_:)`) that permits haptic feedback and system sounds while audio input is active. Default is `false` (haptics muted during recording).

## APIs & Frameworks

**AVAudioEngine**
- `AVAudioInputNode.setVoiceProcessingEnabled(_:)` **[NEW]** — enables voice processing mode; engine must be stopped
- `AVAudioOutputNode.setVoiceProcessingEnabled(_:)` **[NEW]** — symmetric setter; both I/O nodes are switched together
- `AVAudioInputNode.isVoiceProcessingEnabled` **[NEW]** — read current state
- `AVAudioSourceNode` **[NEW]** — render-block audio generator node
  - `AVAudioSourceNode(renderBlock:)` — initializer; block signature: `(UnsafeMutablePointer<ObjCBool>, UnsafePointer<AudioTimeStamp>, AVAudioFrameCount, UnsafeMutablePointer<AudioBufferList>) -> OSStatus`
  - `AVAudioSourceNode(format:renderBlock:)` — initializer with explicit format
- `AVAudioSinkNode` **[NEW]** — receive-block audio sink node
  - `AVAudioSinkNode(receiverBlock:)` — initializer; block signature: `(UnsafePointer<AudioTimeStamp>, AVAudioFrameCount, UnsafePointer<AudioBufferList>) -> OSStatus`

**AVAudio3DMixing Protocol (AVAudioPlayerNode, AVAudioEnvironmentNode)**
- `AVAudio3DMixingRenderingAlgorithm.auto` **[NEW]** — automatic route-appropriate spatialization
- `AVAudio3DMixing.sourceMode: AVAudio3DMixingSourceMode` **[NEW]** — `.spatializeIfMono`, `.pointSource`, `.ambienceBed`
- `AVAudio3DMixing.pointSourceInHeadMode: AVAudio3DMixingPointSourceInHeadMode` **[NEW]** — controls in-head rendering for `.pointSource`
- `AVAudio3DMixing.outputType: AVAudio3DMixingOutputType` **[NEW]** — override output type; `.auto` for real-time auto-detection

**AVAudioSession**
- `AVAudioSession.promptStyle: AVAudioSession.PromptStyle` **[NEW]** — `.none`, `.short`, `.normal`
- `AVAudioSessionPromptStyleDidChangeNotification` **[NEW]** — observe prompt style changes
- `AVAudioSession.allowHapticsAndSystemSoundsDuringRecording: Bool` **[NEW]**
- `AVAudioSession.setAllowHapticsAndSystemSoundsDuringRecording(_:)` **[NEW]**

## Code Highlights

Creating an AVAudioSourceNode (signal generator):

```swift
let sourceNode = AVAudioSourceNode { _, _, frameCount, audioBufferList -> OSStatus in
    let buffers = UnsafeMutableAudioBufferListPointer(audioBufferList)
    for frame in 0..<Int(frameCount) {
        let sample = sin(2.0 * .pi * frequency * Double(currentPhase) / sampleRate)
        currentPhase += 1
        for buffer in buffers {
            let buf = buffer.mData!.assumingMemoryBound(to: Float.self)
            buf[frame] = Float(sample)
        }
    }
    return noErr
}
engine.attach(sourceNode)
engine.connect(sourceNode, to: engine.mainMixerNode, format: format)
```

Enabling voice processing:

```swift
// Engine must be stopped
try engine.inputNode.setVoiceProcessingEnabled(true)
// Both input and output nodes are automatically set
try engine.start()
```

Multichannel ambience bed spatialization:

```swift
playerNode.renderingAlgorithm = .auto
playerNode.sourceMode = .ambienceBed
// Connect with multichannel format
let multichanFormat = AVAudioFormat(standardFormatWithSampleRate: 44100, channelLayout: layout)
engine.connect(playerNode, to: environmentNode, format: multichanFormat)
```

## Takeaways
- `AVAudioSourceNode` and `AVAudioSinkNode` replace the boilerplate of full Audio Unit wrappers for custom generation and real-time capture, with the constraint that block code must be real-time safe.
- Voice Processing mode handles echo cancellation transparently; both I/O nodes must be configured together and the engine must be stopped before enabling.
- `AVAudio3DMixingRenderingAlgorithm.auto` removes the need to write route-detection logic for selecting headphone vs. speaker spatialization.
- Apps playing navigation or voice prompts should observe `AVAudioSession.promptStyle` to mute or shorten prompts when Siri or a call is active.

---
_Source: WWDC19 Session 510 page (abstract, transcript, resource links, and sample code references)._
