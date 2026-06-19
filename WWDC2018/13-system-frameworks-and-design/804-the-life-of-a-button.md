# The Life of a Button
**WWDC18 · Session 804** · [Watch](https://developer.apple.com/videos/play/wwdc2018/804/)

_Platforms:_ iOS 12, general design principles (all platforms)

## Overview
Through the design of a single button in a connected-toaster app, this session delivers an in-depth exploration of interaction design, visual design, and sound design principles. The talk is divided between Julian (UI/UX design, custom controls) and Hugo (sound design), with the shared thesis that even the simplest controls deserve deep, considered design — and that the best way to know if a design works is to prototype and try it.

The interaction design half uses a three-phase model (before / during / after) applied to button interaction, introducing the concept of perceived affordance, feedforward (communicating what will happen during interaction), and feedback (communicating what happened after). The sound design half covers how to derive meaningful UI sounds from real-world analogs and introduces four fundamental building blocks: timbre, frequency, duration, and loudness.

## Key Topics

### What Is a Button?
- Buttons are **indirect controllers of action**: tapping causes an effect elsewhere, unlike direct manipulation (dragging a slider)
- Indirect interactions provide clarity and power through their separation from the result
- Design the action and the button separately and connectedly

### Feedback as the Lens for Button Design
- Two types of feedback: **tell** (text, icons — explicit but requires reading) and **show** (visual, audio, haptic change over time — experiential)
- Apply feedback across all three phases of interaction, not just completion

### Phase 1: Before Interaction — Perceived Affordance
- The screen is just glass; nothing about it inherently communicates interactivity
- **Perceived affordance**: what users understand they can interact with, based on prior iOS experience and current context
- Design signals: label text ("Make Toast" vs. ambiguous "Toast"), button shape, color, grouping/proximity with related elements
- Test affordance perception by holding the screen in the context where users will actually use the app

### Phase 2: During Interaction — Feedforward
- **Feedforward**: communicates what is happening now and hints toward what will happen — makes interactions feel fluid and responsive
- Fast response matters: slow animations during initial touch contact feel unresponsive; for instantaneous taps, use immediate visual highlight, haptic, or confirmation sound
- Dragging off the button allows the user to cancel — this **undecided** state is essential for fluid interactions (analogous to mid-gesture paging); dragging back re-activates
- Feedforward can connect the button state to the resulting action state (e.g., showing toast appearing as the button is held)
- Test feedforward on device; what seems obvious in static mockups may be invisible in practice

### Phase 3: After Interaction — Feedback
- The unhighlight timing matters: a slight delay ensures fast taps are still visible
- Consider whether the action requires a delay before showing confirmation (e.g., double-tap detection)
- Connect button feedback to action-result feedback: text + toaster icon animation together are better than either alone (text is legible if animation is missed; animation is experiential)
- Design cancel/undo affordance: a stop button that is visually distinct from the trigger button (position, color intent communicated clearly)

### Sound Design — Hugo's Framework

**Should you add sound?**
- Consider app category, user expectations, and usage context
- Sounds closest to users' real-world analogs of the action (e.g., toaster lever, toasting coil, toast ejection) are immediately meaningful

**Inspiration from the real world**
- Real-world buttons generate sounds as byproducts of materials; software gives total freedom to choose or omit sound
- Use real-world analogs as starting points, not constraints — derive meaning from them but don't copy one-to-one

**Design process**
- Create multiple distinct options (A/B/C tested in this session)
- Two-click sounds (press down + lift off) are more satisfying than single-click for taps
- Listen to all sounds together as a family to ensure they cohere as the "voice" of the app

**Four building blocks of sound**
1. **Timbre** (tone color): determined by material, shape, and how it's played; sets the emotional character (friendly vs. harsh/metallic)
2. **Frequency** (pitch): indicates perceived object size — higher pitch = smaller, lower pitch = larger; use to reinforce scale and weight
3. **Duration**: short sounds for frequently triggered interactions; longer/more elaborate sounds for rare but significant actions
4. **Loudness** (amplitude): UI sounds are subtle layers — never as loud as ringtones; only needs to add nuance to the interaction

### Multimodal Cohesion
- What we see (animation), feel (haptics), and hear (sound) combine into a single experience
- All three modalities must be considered together and timed together

## APIs & Frameworks

- **`UIButton`** — standard button control; `UIButton.State` (normal, highlighted, disabled, selected)
  - `isHighlighted` — animates on touch; avoid slow animations that feel unresponsive
  - `touchDragInside` / `touchDragOutside` — enable cancel gesture (drag off button to cancel)
- **`UIControl`** — base class with touch tracking; `beginTracking(_:with:)`, `continueTracking(_:with:)`, `endTracking(_:with:)`
- **`UIFeedbackGenerator`** — haptic feedback family:
  - `UIImpactFeedbackGenerator` — for feedforward during touch
  - `UINotificationFeedbackGenerator` — for feedback after action completes (success/warning/error)
  - `UISelectionFeedbackGenerator` — for selection changes
- **`AVAudioPlayer`** / **`AVAudioSession`** — for playing custom UI sounds
- **Human Interface Guidelines** — affordance, touch target sizes (minimum 44×44 pt), color and contrast for perceivability

## Code Highlights

Immediate visual response on touch (avoiding slow animations):
```swift
override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
    super.touchesBegan(touches, with: event)
    // Instant highlight — no animation duration
    UIView.animate(withDuration: 0.0) { self.alpha = 0.6 }
    let impact = UIImpactFeedbackGenerator(style: .medium)
    impact.impactOccurred()
}

override func touchesEnded(_ touches: Set<UITouch>, with event: UIEvent?) {
    super.touchesEnded(touches, with: event)
    UIView.animate(withDuration: 0.15) { self.alpha = 1.0 }
    let notification = UINotificationFeedbackGenerator()
    notification.notificationOccurred(.success)
}
```

## Takeaways
- Design buttons in three phases — before (affordance/label), during (feedforward/highlight/haptic), and after (feedback/result connection) — not just the action result.
- Perceived affordance depends heavily on context and prior iOS familiarity; test on device in the actual environment of use.
- Two-click sounds (press + release) feel significantly more satisfying than single-click; the second click provides confirmation that the action was completed.
- Visual, haptic, and audio feedback must be considered and timed together as a single multimodal experience — they are not independent layers.

---
_Source: WWDC18 Session 804 page (abstract, full transcript, and resource links)._
