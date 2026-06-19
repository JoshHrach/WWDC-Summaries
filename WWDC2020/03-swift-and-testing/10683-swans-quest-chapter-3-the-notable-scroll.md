# Swan's Quest, Chapter 3: The notable scroll
**WWDC20 · Session 10683** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10683/)

_Platforms:_ iPadOS 14, macOS Big Sur 11 (Swift Playgrounds)

## Overview
Chapter 3 of Swan's Quest extends the tone sequencing approach from Chapter 2 by introducing variable-length notes and musical tempo. The Swan's scroll contains a piece ("Ode to Joy") with quarter notes, half notes, and dotted half notes — requiring players to implement protocol-based `Pitch` and `Note` types, calculate timer intervals from a tempo (120 BPM), and use a "subdivide" pattern to flatten variable-length notes into a flat array of equally-spaced pitch instructions.

The key insight is that a single fixed-interval timer can play notes of different durations by subdividing longer notes into repeated pitch entries. A half note at 120 BPM equals two quarter-note slots; a dotted half note equals three slots. The `NoteProtocol.subdivide()` method returns an array of pitches proportional to the note's length, which is then flattened into the playback array before the timer starts. A side quest challenges players to layer bass chords under the melody using multiple simultaneous timers.

## Key Topics

### Music Theory: Tempo and Note Length
- Tempo is defined in **beats per minute (BPM)**, where one beat = one quarter note
- At 120 BPM: one quarter note = 500 ms (60s / 120 = 0.5s per beat)
- Note length multiples (as fractions of a quarter note):
  - Eighth note: 0.5
  - Quarter note: 1.0
  - Half note: 2.0
  - Dotted half note: 3.0
  - Whole note: 4.0
- Timer interval = `shortestSupportedNoteLength × (60.0 / BPM)` in seconds

### Protocol-Based Design
- `PitchProtocol` — requires `var frequency: Double { get }` for conversion to `Tone`
- `NoteProtocol` — requires `var tone: Tone`, `var length: Float`, `static var shortestSupportedNoteLength: Float`, and `func subdivide() -> [PitchType]`
- Concrete types are enums: `Pitch: PitchProtocol` with cases like `.a4` (rawValue = 440.0); `Note: NoteProtocol` with associated-value cases like `.quarter(pitch: Pitch)`, `.half(pitch: Pitch)`

### Subdivide Pattern for Variable-Length Notes
- A single fixed-interval timer fires at the shortest note's interval; longer notes must consume multiple timer ticks
- `subdivide()` returns an array of pitches whose count equals `Int(self.length / shortestSupportedNoteLength)`:
  - `.quarter(pitch: .a4).subdivide()` → `[.a4]` (1 element)
  - `.half(pitch: .a4).subdivide()` → `[.a4, .a4]` (2 elements)
  - `.dottedHalf(pitch: .a4).subdivide()` → `[.a4, .a4, .a4]` (3 elements)
- Pre-flatten all notes into a `[Pitch]` array before the timer starts; the timer iterates over this flat array, playing one pitch per tick
- Advantage: the timer code stays simple (no note-length math inside the closure)

### Layering with Multiple Timers (Side Quest)
- Multiple concurrent `Timer` instances with the same interval can drive independent `ToneOutput` instances simultaneously
- One timer for melody, another for bass chords — produces multi-part harmony with minimal additional code

## APIs & Frameworks

**SPCAudio module (Swift Playgrounds content SDK)**
- `ToneOutput` — audio generator; `play(tone:)`, `stopTones()`
- `Tone(pitch: Double, volume: Double)` — pitch in Hz, volume 0.0–1.0

**Swift Playgrounds content SDK — Music module**
- `PitchProtocol` — `var frequency: Double { get }`
- `NoteProtocol` — `var tone: Tone`, `var length: Float`, `static var shortestSupportedNoteLength: Float`, `func subdivide() -> [PitchType]`

**Foundation**
- `Timer.scheduledTimer(withTimeInterval:repeats:block:)` — fixed-interval playback loop
- `Timer.invalidate()` — end playback

**Swift Playgrounds runtime**
- `owner.endPerformance()` — signals challenge completion

## Code Highlights

Protocol definitions for structured note representation:
```swift
// Music.swift (provided in the playground)

protocol PitchProtocol {
    var frequency: Double { get }
}

protocol NoteProtocol {
    associatedtype PitchType: PitchProtocol
    var tone: Tone { get }
    var length: Float { get }
    static var shortestSupportedNoteLength: Float { get }
    func subdivide() -> [PitchType]
}
```

Example `Pitch` and `Note` enum implementations:
```swift
enum Pitch: Double, PitchProtocol {
    case a4 = 440.0, b4 = 493.88, c4 = 261.63
    var frequency: Double { rawValue }
}

enum Note: NoteProtocol {
    case quarter(pitch: Pitch)
    case half(pitch: Pitch)

    var tone: Tone {
        switch self {
        case .quarter(let p), .half(let p):
            return Tone(pitch: p.frequency, volume: 0.3)
        }
    }
    var length: Float {
        switch self { case .quarter: return 1.0; case .half: return 2.0 }
    }
    static var shortestSupportedNoteLength: Float { 1.0 }
    func subdivide() -> [Pitch] {
        let count = Int(length / Note.shortestSupportedNoteLength)
        switch self {
        case .quarter(let p), .half(let p): return Array(repeating: p, count: count)
        }
    }
}
```

Timer sequencing with pre-flattened pitch array (120 BPM):
```swift
let toneOutput = ToneOutput()
let notes: [Note] = [.quarter(pitch: .a4), .half(pitch: .a4), .quarter(pitch: .a4)]

// Flatten variable-length notes into equally-spaced pitch slots
var pitches = [Pitch]()
for note in notes { pitches.append(contentsOf: note.subdivide()) }

let interval = TimeInterval(Note.shortestSupportedNoteLength * 0.5) // 120 BPM
var index = 0
Timer.scheduledTimer(withTimeInterval: interval, repeats: true) { timer in
    guard index < pitches.count else {
        toneOutput.stopTones()
        timer.invalidate()
        owner.endPerformance()
        return
    }
    toneOutput.play(tone: Tone(pitch: pitches[index].frequency, volume: 0.3))
    index += 1
}
```

## Takeaways

- Set the timer interval to the **shortest note length × tempo duration**, then represent all longer notes as repeated entries in a flattened pitch array — this keeps the timer closure simple and decouples note duration from playback logic.
- The `subdivide()` method on `NoteProtocol` is the key abstraction: it converts a single variable-length note into an array of equally-timed pitch instructions proportional to the note's length.
- Multiple concurrent timers driving independent `ToneOutput` instances produce multi-part harmony — use this pattern to layer bass, melody, and percussion without synchronization complexity.
- Protocol-based design (`PitchProtocol`, `NoteProtocol`) makes the music model extensible; adding a new note length or pitch only requires a new enum case, not a change to the sequencing logic.

---
_Source: WWDC20 Session 10683 page (transcript and code samples)._
