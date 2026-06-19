# Swan's Quest, Chapter 4: The sequence completes
**WWDC20 · Session 10684** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10684/)

_Platforms:_ iPadOS 14, macOS Big Sur 11 (Swift Playgrounds)

## Overview
Chapter 4 of Swan's Quest is the finale, introducing the Swift Playgrounds content SDK's sampled instrument API and a **step sequencer** architecture. Instead of generating sine-wave tones with `ToneOutput`, this chapter uses `playInstrument(_:note:volume:)` with pre-sampled instruments (piano, guitar, bass guitar, bells, and synths) and MIDI note codes. Players build a multi-track step sequencer to play two-part harmony — bass and piano — simultaneously over a repeating timer loop.

The step sequencer pattern generalizes the timer approach from Chapters 2–3 into a multi-track architecture: each "track" stores a sequence of MIDI notes for a specific instrument; a single timer fires at a fixed interval for each "step" (beat slot); on each step, the timer iterates all tracks and calls `playInstrument` once per track. The side quest challenges players to integrate `ToneOutput` as a third instrument track, combining all four chapters' skills.

## Key Topics

### Step Sequencer Architecture
- A step sequencer divides time into equal **steps** (beats) and iterates all tracks simultaneously on each step
- Single timer drives all tracks: `interval = totalDuration / numberOfBeats`
- At 120 BPM with 8 beats: `interval = 4.0 / 8 = 0.5s` per step
- After completing one loop, reset `index` to 0 and call `owner.endPerformance()` to credit the first playthrough, then continue looping (or `timer.invalidate()` to stop)
- Each track conforms to `TrackProtocol`: declares its `instrument`, `length` (total beats), and a `note(for:)` method returning the MIDI note for a given step index
- Use `.rest` (MIDI code 0) to represent silence for a track at a given step

### Sampled Instruments via MIDI
- `playInstrument(_:note:volume:)` — SDK method that plays one of seven pre-sampled instruments at a MIDI note pitch
- Seven available `Instrument.Kind` cases: `.electricGuitar`, `.bassGuitar`, `.piano`, `.warmBells`, `.sevenSynth`, `.bassSynth`, `.crystalSynth`
- `MIDINoteProtocol` — requires `var midiCode: UInt8`; implement as an enum with raw values matching standard MIDI codes
  - Standard MIDI: C2 = 36, D2 = 38, E2 = 40, F2 = 41, G2 = 43, A2 = 45, B2 = 47; rest = 0
- Instruments were sampled in GarageBand, exported as uncompressed WAV, trimmed per note, and bundled into the SDK

### Sampling Custom Instruments in GarageBand
- Open GarageBand → select Keyboard (or other instrument) → choose a preset (e.g., Koto)
- Adjust properties (tone, resonance) and select a scale to simplify recording individual notes
- Record each note individually → export as uncompressed WAV → trim per note in editor → import into Xcode/Playgrounds project as custom instrument samples

## APIs & Frameworks

**Swift Playgrounds content SDK — Sequencer module**
- `playInstrument(_ kind: Instrument.Kind, note: MIDINoteProtocol, volume: Double = 75)` — play a sampled instrument at a MIDI pitch
- `Instrument.Kind` enum — `.electricGuitar`, `.bassGuitar`, `.piano`, `.warmBells`, `.sevenSynth`, `.bassSynth`, `.crystalSynth`
- `MIDINoteProtocol` — `var midiCode: UInt8 { get }`
- `TrackProtocol` — `var instrument: Instrument.Kind`, `var length: Int`, `func note(for frame: Int) -> NoteType`

**Swift Playgrounds content SDK — Audio**
- `ToneOutput` (from Chapter 2/3) — still usable as a custom "instrument track" in the sequencer (side quest)

**Foundation**
- `Timer.scheduledTimer(withTimeInterval:repeats:block:)` — step sequencer timing loop

**Swift Playgrounds runtime**
- `owner.endPerformance()` — signal challenge completion (call after first loop completes)

## Code Highlights

MIDI note enum implementation:
```swift
enum MIDINotes: UInt8, MIDINoteProtocol {
    case rest = 0
    case C2 = 36, D2 = 38, E2 = 40, F2 = 41
    case G2 = 43, A2 = 45, B2 = 47
    var midiCode: UInt8 { rawValue }
}
```

Track implementation conforming to `TrackProtocol`:
```swift
struct Track: TrackProtocol {
    var instrument: Instrument.Kind
    var length: Int
    var notes: [MIDINotes]? = nil

    func note(for frame: Int) -> MIDINotes {
        guard let n = notes, frame < n.count else { return .rest }
        return n[frame]
    }
}
```

Two-track step sequencer (bass + piano, 8 beats, 4 seconds):
```swift
let numberOfBeats = 8
let duration = 4.0

var bass  = Track(instrument: .bassGuitar, length: numberOfBeats)
var piano = Track(instrument: .piano,      length: numberOfBeats)

bass.notes  = [.rest, .C2, .A2, .rest, .C2, .A2, .D2, .C2]
piano.notes = [.A2,   .A2, .C2, .F2,  .A2, .C2, .rest, .F2]

let tracks = [bass, piano]
let interval = duration / Double(numberOfBeats)
var index = 0

Timer.scheduledTimer(withTimeInterval: interval, repeats: true) { timer in
    for track in tracks {
        playInstrument(track.instrument, note: track.note(for: index))
    }
    if index + 1 < numberOfBeats {
        index += 1
    } else {
        index = 0
        owner.endPerformance()  // credit awarded after first complete loop
    }
}
```

## Takeaways

- A step sequencer needs only a single timer: divide total duration by number of beats to get the interval, then call `playInstrument` for each track at every timer fire — one timer drives arbitrarily many instrument tracks.
- Use MIDI code 0 (`.rest`) to represent silence; guard against out-of-bounds access in `note(for:)` and return `.rest` as the safe default.
- The seven pre-sampled instruments in the Swift Playgrounds SDK (piano, electric guitar, bass guitar, warm bells, seven synth, bass synth, crystal synth) are available via `Instrument.Kind` — no audio file handling required.
- The side quest unifies all four Swan's Quest chapters: add `ToneOutput` as a custom frequency-based track running alongside the MIDI instrument tracks in the same timer loop, blending synthesized and sampled audio.

---
_Source: WWDC20 Session 10684 page (transcript and code samples)._
