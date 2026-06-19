# Swan's Quest, Chapter 2: A time for tones
**WWDC20 · Session 10682** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10682/)

_Platforms:_ iPadOS 14, macOS Big Sur 11 (Swift Playgrounds)

## Overview
Chapter 2 of Swan's Quest introduces audio synthesis through the Swift Playgrounds content SDK's `ToneOutput` and `Tone` types. The challenge requires players to decode a musical scroll and play a C-major scale by generating individual tones at specific frequencies, using a `Timer` to sequence multiple notes in order and stop playback correctly. The chapter demonstrates how to convert musical notes into Hz frequencies, chain tones using scheduled timers, and signal completion to the playground runtime via `endPerformance()`.

Building on the accessibility concepts from Chapter 1, this chapter focuses on the `SPCAudio` module and the programming patterns needed to produce real-time audio output from a Swift Playground. A side quest challenges attendees to extend the C-major scale solution to play an F-major scale, requiring them to apply the octave-doubling principle (multiplying a frequency by 2 raises it by one octave).

## Key Topics

### ToneOutput and Tone
- `ToneOutput` — a class in the `SPCAudio` module that generates audio at 44,100 samples/second; produces a continuous tone until explicitly stopped
- `Tone` — a simple `Codable` struct with `pitch: Double` (frequency in Hz) and `volume: Double` (0.0–1.0)
- `ToneOutput.play(tone:)` — begins outputting the specified tone continuously; calling it again replaces the previous tone immediately
- `ToneOutput.stopTones()` — stops audio output
- For a single note with a defined duration, use `DispatchQueue.main.asyncAfter` to call `stopTones()` after the desired interval
- For sequences, a `Timer` is cleaner and more reliable than chained `asyncAfter` calls

### Timer-Based Note Sequencing
- `Timer.scheduledTimer(withTimeInterval:repeats:block:)` — fires a closure at a fixed interval; use `repeats: true` for a sequence loop
- Pattern: maintain an array of `Tone` values and an index; play `tones[index]` on each timer fire; when index exceeds the array, call `stopTones()`, `timer.invalidate()`, and `owner.endPerformance()`
- `endPerformance()` — Swift Playgrounds SDK method that signals the playground runtime that the performance is complete and credit should be awarded; must be called at the end of every challenge

### Music Theory Fundamentals
- Middle A (A4) = 440 Hz; Middle B = 493.88 Hz; Middle C (C4) = 523.25 Hz
- To raise a note by one octave: multiply its frequency by 2 (e.g., A4 = 440 Hz → A5 = 880 Hz)
- Side quest note: B-flat4 ≈ 466.16 Hz (required for the F-major scale, which uses: F4, G4, A4, B♭4, C5, D5, E5, F5)

## APIs & Frameworks

**SPCAudio module (Swift Playgrounds content SDK)**
- `ToneOutput` class:
  - `let sampleRate = 44100.0` — samples per second
  - `func play(tone: Tone)` — begin playing a continuous tone
  - `func stopTones()` — stop audio output
- `Tone` struct:
  - `var pitch: Double` — frequency in Hz
  - `var volume: Double` — amplitude (0.0–1.0)
  - `init(pitch: Double, volume: Double)`

**Foundation**
- `Timer.scheduledTimer(withTimeInterval:repeats:block:)` — schedule repeating note playback
- `Timer.invalidate()` — stop a repeating timer
- `DispatchQueue.main.asyncAfter(deadline:execute:)` — single-shot delay for stopping a single note

**Swift Playgrounds runtime**
- `owner.endPerformance()` — signals successful completion of the challenge to the playground evaluator

## Code Highlights

Play a single middle A for 400 ms then stop:
```swift
import SPCAudio

let toneOutput = ToneOutput()
let a4 = Tone(pitch: 440.0, volume: 0.3)
toneOutput.play(tone: a4)

DispatchQueue.main.asyncAfter(deadline: .now() + .milliseconds(400)) {
    toneOutput.stopTones()
}
```

Play a sequence of notes (A4, B4, C5) with 400 ms per note using a timer:
```swift
let toneOutput = ToneOutput()
let tones = [
    Tone(pitch: 440.00, volume: 0.3),  // A4
    Tone(pitch: 493.88, volume: 0.3),  // B4
    Tone(pitch: 523.25, volume: 0.3)   // C5
]
var toneIndex = 0

Timer.scheduledTimer(withTimeInterval: 0.4, repeats: true) { timer in
    guard toneIndex < tones.count else {
        toneOutput.stopTones()
        timer.invalidate()
        owner.endPerformance()
        return
    }
    toneOutput.play(tone: tones[toneIndex])
    toneIndex += 1
}
```

## Takeaways

- `ToneOutput` produces continuous audio — always pair each `play(tone:)` call with a subsequent `stopTones()` call, either via `asyncAfter` for single notes or `timer.invalidate()` at the end of a sequence.
- Use `Timer.scheduledTimer(withTimeInterval:repeats:block:)` rather than chained `asyncAfter` calls for multi-note sequences; the timer interval directly maps to the per-note duration.
- Musical frequency follows a pattern: doubling the Hz raises pitch by one octave; use this to extend any scale across multiple octaves without looking up every frequency.
- Always call `owner.endPerformance()` at the sequence's end — without it, the playground will not record the challenge as complete.

---
_Source: WWDC20 Session 10682 page (transcript and code samples)._
