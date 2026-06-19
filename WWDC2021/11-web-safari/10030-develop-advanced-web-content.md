# Develop advanced web content
**WWDC21 · Session 10030** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10030/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12

## Overview
This session surveys the most significant JavaScript, WebAssembly, and Web API improvements shipping in Safari 15 and WebKit. The speaker uses a running "Voice Memo" web app demo to illustrate new APIs in context, culminating in a web app that records audio, applies Audio Worklet distortion effects, transcribes speech, and shares the resulting file—all using platform web APIs with no native code.

The three sections cover: (1) JavaScript class field syntax, weak references, top-level await, module workers, and Internationalization API additions; (2) WebAssembly engine upgrades including streaming compilation, reference types, and BigInt/i64 interop; and (3) new/updated Web APIs: WebGL2, expanded WebM/VP9 support, Storage Access API improvements, MediaRecorder, Audio Worklet, Web Share (with file support), Speech Recognition, and Media Session.

## Key Topics

### JavaScript — New Class Field Syntax **[NEW]**
- Private instance fields (`#field`) and private methods (`#method()`) enforced by the language, not by convention.
- Private static fields (`static #field`) and public static fields (`static field`) on classes.
- Prevents accidental access from outside the class; produces a runtime error on violation.

### JavaScript — Weak References **[NEW]**
- `WeakRef` — holds a weak reference to an object; deref it with `.deref()` before use (may return `undefined` if GC'd).
- `FinalizationRegistry` — registers a callback invoked when a registered object is garbage-collected; useful for cleanup tasks.
- Caveat: GC timing in JavaScript is uncertain; `FinalizationRegistry` callback runs on the event loop, not immediately.

### JavaScript — Top-Level Await **[NEW]**
- `await` can now be used outside `async` functions at the top level of ES modules.
- The module itself acts as an async function; modules depending on it wait for it to resolve.
- Only available in modules; produces a syntax error in non-module scripts.

### JavaScript — Module Workers **[NEW]**
- Web Workers, Service Workers, and Worklets now support ES modules.
- Web Workers / Service Workers: pass `{ type: 'module' }` in the constructor options.
- Audio Worklet: use `audioWorklet.addModule(url)`.
- Enables dynamic import, optimized loading, and dependency management in background threads.

### JavaScript — Internationalization API Updates **[NEW/EXPANDED]**
- `Intl.NumberFormat` — language-sensitive number formatting with padding, currency, unit styles.
- `Intl.DateTimeFormat` — fine-grained date/time formatting with per-component style control (including seconds/milliseconds).
- `Intl.Segmenter` — language-sensitive string splitting by grapheme, word, or sentence; useful for languages like Chinese without obvious word boundaries.
- `Intl.ListFormat` — language-sensitive list formatting with `type` (conjunction/disjunction) and `style` options.
- `Intl.DisplayNames` — consistent localized translation of language, region, and script codes.

### WebAssembly Engine Upgrades **[NEW]**
- Bulk memory instructions for faster `memcpy`/`memset`-style operations.
- Non-trapping float-to-int conversion instructions.
- Sign-extension operators for signed integers.
- `i64`/`BigInt` interoperability — simpler and faster interop between Wasm `i64` and JS `BigInt`.
- Reference types — Wasm modules can hold and pass references to JS and DOM objects.
- Streaming download and compilation — Wasm starts compiling while still downloading, reducing startup latency.
- WebGL2 backend migrated from OpenGL to Metal — enables GPU access in iOS Simulator and Metal tool integration (Xcode Frame Debugger).

### WebGL2 **[NEW in Safari]**
- Full WebGL2 now available in Safari on all Apple devices.
- New features: 3D textures, sampler objects, transform feedback (GPU particle systems).
- Backend migrated from OpenGL to Metal; Metal tools (Xcode Frame Debugger) now work with WebGL code.

### Expanded Media Format Support
- WebM with VP8/VP9 video + Vorbis audio: added in macOS 11.3 (streaming playback).
- WebM with Opus audio: macOS 12 **[NEW]**.
- Media Source Extensions for WebM on iPadOS 15 **[NEW]** (previously macOS only).
- VP9 support for streaming and WebRTC on macOS (all Apple Silicon Macs) and iPadOS.
- Use `MediaCapabilities` API to detect support before use.

### Storage Access API Updates **[NEW behavior]**
- Permission is now granted per-page scope: once a third party is granted access, all its subresources on the page get access without individual requests.
- Nested iframes (iframes inside iframes) can now request Storage Access.

### MediaRecorder API **[NEW in WebKit]**
- `MediaRecorder` captures data from `MediaStream` or HTML media elements.
- Options: MIME type, video/audio bit rates.
- Key events: `dataavailable` (chunks of data), `stop` (recording finished).
- Start: `mediaRecorder.start()`, Stop: `mediaRecorder.stop()`.

### Audio Worklet API **[NEW in WebKit]**
- Runs custom audio processing scripts on the audio-rendering thread (low-latency, no main-thread hopping vs. deprecated `ScriptProcessorNode`).
- Supports JavaScript and WebAssembly processors.
- `AudioWorkletProcessor` subclass with `process(inputs, outputs, parameters)` method.
- Register globally with `registerProcessor(name, processorClass)`.
- Use `audioWorklet.addModule(url)` then `new AudioWorkletNode(ctx, name)`.

### Web Share API — File Sharing **[NEW]**
- `navigator.share({ files: [file], title, text })` — shares files (audio, video, image, etc.) via the system share sheet.
- `navigator.canShare({ files })` — checks if the file type can be shared before attempting.

### Speech Recognition API **[NEW in WebKit]**
- `webkitSpeechRecognition` (prefix kept for compatibility) — captures live audio and transcribes to text.
- Uses the same engine as Siri; supports multiple languages; high accuracy.
- Requires Siri or Dictation enabled in system settings; privacy prompt on first use.
- `continuous` property — keeps recognition running until stopped.
- Events: `result` (with transcript alternatives sorted by confidence), `end`.

### Media Session API **[NEW in WebKit]**
- Lets web pages communicate media state and controls to platform components (Now Playing widget, lock screen).
- Set `navigator.mediaSession.metadata` with `MediaMetadata` (title, artist, artwork).
- Register action handlers for transport controls (play, pause, seek, etc.).

## APIs & Frameworks

**JavaScript (ECMAScript 2021 / Stage 4)**
- Private class fields (`#`) and methods **[NEW]**
- Private and public static class fields **[NEW]**
- `WeakRef` **[NEW]**
- `FinalizationRegistry` **[NEW]**
- Top-level `await` in ES modules **[NEW]**
- Module Workers (`{ type: 'module' }` option) **[NEW]**
- `Intl.NumberFormat`, `Intl.DateTimeFormat`, `Intl.Segmenter`, `Intl.ListFormat`, `Intl.DisplayNames` **[NEW/EXPANDED]**

**WebAssembly**
- Bulk memory operations, non-trapping float-to-int, sign-extension operators **[NEW]**
- `i64`/`BigInt` interop **[NEW]**
- Reference types **[NEW]**
- Streaming compilation (`WebAssembly.instantiateStreaming`) **[NEW]**

**Web APIs**
- WebGL2 **[NEW in Safari on all Apple devices]**
- `MediaCapabilities` API — detect codec/format support **[existing]**
- Storage Access API per-page scope and nested iframe support **[UPDATED]**
- `MediaRecorder` — `new MediaRecorder(stream, options)`, `.start()`, `.stop()`, `dataavailable` event **[NEW in WebKit]**
- `AudioWorkletProcessor` / `AudioWorkletNode` — `process(inputs, outputs)` override, `registerProcessor()` **[NEW in WebKit]**
- `audioWorklet.addModule(url)` — loads processor module **[NEW]**
- `navigator.share({ files })` — file sharing via Web Share API **[NEW]**
- `navigator.canShare({ files })` — checks shareability **[NEW]**
- `webkitSpeechRecognition` — speech-to-text, continuous mode, `result`/`end` events **[NEW in WebKit]**
- `navigator.mediaSession.metadata` / `MediaMetadata` — Now Playing integration **[NEW in WebKit]**
- `navigator.mediaSession.setActionHandler(type, handler)` — transport control hooks **[NEW]**

## Code Highlights

Private class field and method syntax:
```javascript
class StopwatchWithOneButton {
    #startTime;
    click() {
        if (!this.#startTime) { this.#start(); }
        else { this.#stop(); }
    }
    #start() { this.#startTime = Date.now(); }
    #stop() {
        const duration = Date.now() - this.#startTime;
        this.#startTime = undefined;
        return duration;
    }
    static #startedCount = 0;
}
```

MediaRecorder recording audio:
```javascript
const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
const mediaRecorder = new MediaRecorder(stream);
const chunks = [];
mediaRecorder.ondataavailable = e => chunks.push(e.data);
mediaRecorder.onstop = () => {
    const blob = new Blob(chunks, { type: 'audio/ogg' });
    audioElement.src = URL.createObjectURL(blob);
};
mediaRecorder.start();
```

Web Share with file:
```javascript
const file = new File([blob], 'memo.ogg', { type: blob.type });
if (navigator.canShare({ files: [file] })) {
    await navigator.share({ files: [file], title: 'Voice Memo' });
}
```

## Takeaways
- Private class fields (`#`) provide real language-enforced encapsulation in JavaScript; prefer them over the `_` naming convention.
- WebGL2 is now available in Safari on all Apple devices with a Metal backend, enabling GPU use in iOS Simulator and Metal tooling.
- `MediaRecorder`, `AudioWorklet`, Web Share file support, Speech Recognition, and Media Session are all new in WebKit this year—enabling rich native-like audio workflows entirely in the browser.
- Use `Intl.Segmenter` for correct word-splitting in CJK languages, and `Intl.DisplayNames` for consistent locale name translation.

---
_Source: WWDC21 Session 10030 page (abstract, full transcript, and resource links)._
