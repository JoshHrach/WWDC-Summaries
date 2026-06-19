# What's new in web accessibility
**WWDC22 · Session 10153** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10153/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13 (Safari / WebKit)

## Overview
This session covers modern web accessibility techniques for Safari and WebKit, with a focus on three areas: building accessible custom controls with ARIA attributes, using Speech Synthesis Markup Language (SSML) in the Web Speech API for rich audio experiences, and adopting the HTML `<dialog>` element for accessible modal dialogs.

Custom web controls that go beyond semantic HTML require supplemental ARIA attributes to expose correct semantics to assistive technologies like VoiceOver, Switch Control, Voice Control, and Full Keyboard Access. The session provides a worked example of a custom slider with full keyboard, pointer, and VoiceOver support using `role="slider"` and `aria-value*` attributes.

Safari now supports SSML in `SpeechSynthesisUtterance`, enabling developers to control pronunciation, pauses, rate, pitch, and locale-specific voices. The `<dialog>` element (newly supported in Safari) provides built-in accessibility-friendly modal behavior including focus management, iOS scrub gesture support, and Escape key handling.

## Key Topics

### Assistive Technologies Overview
Apple's assistive technologies — VoiceOver, Switch Control, Voice Control, Full Keyboard Access — all interact with web content through Safari's accessibility tree. Semantic HTML provides a solid baseline; ARIA supplements custom components. Approximately 1 in 7 people worldwide have a disability affecting device interaction.

### Custom Controls with ARIA
When building custom interactive components in JavaScript, the ARIA `role` attribute establishes the component type (e.g., `role="slider"`). Required companion attributes: `tabindex="0"` for keyboard focusability; `aria-valuemin`, `aria-valuemax`, `aria-valuenow`, `aria-valuetext` for slider state. On iOS, VoiceOver swipe-up/down gestures are translated by Safari into `ArrowRight`/`ArrowLeft` key events, so a `keydown` listener handles both keyboard and VoiceOver input. When updating visuals, always update corresponding ARIA attributes synchronously.

### SSML in Web Speech API (New in Safari)
`SpeechSynthesisUtterance` now accepts SSML markup in Safari. Key SSML elements:
- `<speak>` — required root wrapper for SSML content
- `<break time="Xs"/>` — inserts a pause of specified duration
- `<phoneme alphabet="ipa" ph="...">` — controls pronunciation
- `<prosody pitch="..." rate="..." volume="...">` — adjusts speech delivery
- `<lang xml:lang="es-MX">` — selects a locale-specific voice

### HTML dialog Element (New in Safari)
The `<dialog>` element provides accessible modal behavior out of the box: `showModal()` method to open, `<form method="dialog">` to auto-close on submit, Escape key handling, iOS scrub gesture handling. Accessibility best practices: add `aria-labelledby` pointing to the modal's content, use `autofocus` attribute on the appropriate initial focus element.

## APIs & Frameworks

**Web Accessibility / ARIA (HTML/CSS/JS)**
- `role="slider"` — ARIA slider role for custom controls
- `tabindex="0"` — makes element focusable in tab order
- `aria-valuemin` — slider minimum value
- `aria-valuemax` — slider maximum value
- `aria-valuenow` — slider current value
- `aria-valuetext` — human-readable description of current value
- `aria-modal="true"` — marks a container as a modal (existing)
- `aria-labelledby` — links an element to its label by ID
- `aria-hidden="true"` — hides decorative elements from assistive tech
- `autofocus` attribute — sets initial focus within a dialog

**Web Speech API (JavaScript)**
- `SpeechSynthesis` — text-to-speech interface (`window.speechSynthesis`)
- `SpeechSynthesisUtterance` — speech request object
- `window.speechSynthesis.speak(utterance)` — queues utterance for playback
- SSML support in `SpeechSynthesisUtterance` **[NEW in Safari]**

**SSML Elements (W3C Speech Synthesis Markup Language)**
- `<speak>` — root SSML element **[NEW Safari support]**
- `<break time="Xs"/>` — timed pause **[NEW Safari support]**
- `<phoneme alphabet="ipa" ph="...">` — IPA pronunciation **[NEW Safari support]**
- `<prosody pitch rate volume>` — speech delivery control **[NEW Safari support]**
- `<lang xml:lang="...">` — locale-specific voice selection **[NEW Safari support]**

**HTML (WebKit)**
- `<dialog>` element **[NEW Safari support]**
- `dialog.showModal()` — opens as modal
- `<form method="dialog">` — submit closes dialog
- `HTMLDialogElement.close()` — programmatic close

**Keyboard Events (for VoiceOver/Keyboard integration)**
- `ArrowRight` / `ArrowUp` key events — VoiceOver slider increment (Safari simulates these)
- `ArrowLeft` / `ArrowDown` key events — VoiceOver slider decrement

## Code Highlights

Accessible custom slider with ARIA and keyboard support:
```html
<div id="pizza-input"
     role="slider" tabindex="0"
     aria-valuemin="0" aria-valuemax="8"
     aria-valuenow="4" aria-valuetext="4 slices">
</div>
```
```javascript
this.control.addEventListener("keydown", (event) => {
    if (event.key === "ArrowRight" || event.key === "ArrowUp")
        this.update(this.sliceCount + 1);
    else if (event.key === "ArrowLeft" || event.key === "ArrowDown")
        this.update(this.sliceCount - 1);
});
// In update(): always sync both aria-valuenow and aria-valuetext
this.control.setAttribute("aria-valuenow", this.sliceCount);
this.control.setAttribute("aria-valuetext", `${this.sliceCount} slices`);
```

SSML with locale-specific voice via Web Speech API:
```javascript
function wrapWithSSML(phrase, locale) {
    return `<break time="100ms"/>
            <prosody rate="80%">
                <lang xml:lang="${locale}">${phrase}</lang>
            </prosody>`;
}
const ssml = `<speak>
    How do you say ${wrapWithSSML("the water", "en-US")} in Spanish?
    ${wrapWithSSML("El agua", "es-MX")}
</speak>`;
const utterance = new SpeechSynthesisUtterance(ssml);
window.speechSynthesis.speak(utterance);
```

Accessible dialog with aria-labelledby and autofocus:
```html
<dialog id="score-modal" aria-labelledby="modal-content">
    <form method="dialog">
        <span id="modal-content">You got all six questions correct!</span>
        <button type="submit" autofocus>Close</button>
    </form>
</dialog>
```
```javascript
document.getElementById("show-score-btn").addEventListener("click", () => {
    document.getElementById("score-modal").showModal();
});
```

## Takeaways
- Custom interactive controls must supplement JavaScript event handling with ARIA roles, `tabindex`, and state attributes (`aria-valuenow`, `aria-valuetext`) — keep ARIA in sync with visual state on every update.
- VoiceOver slider swipe gestures are automatically translated by Safari into `ArrowRight`/`ArrowLeft` key events, so a single `keydown` listener covers both keyboard users and VoiceOver users.
- Safari now supports SSML in `SpeechSynthesisUtterance`, enabling rich text-to-speech with locale-specific voices, pronunciation control, and timed pauses.
- The HTML `<dialog>` element provides built-in focus management, Escape key handling, and iOS VoiceOver scrub gesture support — prefer it over manual `aria-modal` patterns.

---
_Source: WWDC22 Session 10153 page (abstract, chapter summaries, code samples, and resource links)._
