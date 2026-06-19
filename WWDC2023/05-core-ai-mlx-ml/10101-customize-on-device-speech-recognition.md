# Customize On-Device Speech Recognition
**WWDC23 · Session 10101** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10101/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, watchOS 10, tvOS 17

## Overview
iOS 17 introduces language model customization for `SFSpeechRecognizer`, enabling apps to tune the on-device speech recognizer to their specific domain vocabulary. Previously, all apps shared the same language model trained on general usage patterns, which caused mispredictions for domain-specific terminology (app commands, names, technical jargon). With this feature, developers supply training data—exact phrases, template-generated phrase sets, and custom pronunciations—that are compiled into a per-app language model and used exclusively on-device.

The session walks through the full workflow: building a training data collection using a result-builder DSL, exporting it to a file, compiling it with `prepareCustomLanguageModel`, configuring an on-device `SFSpeechRecognitionRequest` with the custom model, and running recognition. All customization data stays on device and is never sent to Apple's servers.

## Key Topics

### How Speech Recognition Works
A speech recognizer pipeline has two stages:
1. **Acoustic Model**: converts audio into phonetic representations.
2. **Language Model**: scores candidate transcriptions by how likely the word sequence is given patterns learned during training.

When domain-specific phrases are absent from training data (e.g., chess move names like "Albin counter gambit"), the language model incorrectly scores more common alternatives ("Play the album…") higher, causing mispredictions. Language model customization lets apps inject their own phrases to raise the likelihood score for domain-relevant transcriptions.

### Building Training Data
Training data is built using `SFCustomLanguageModelData`, a new class with a result-builder DSL:

- **`PhraseCount(phrase:count:)`**: adds an exact phrase or partial phrase with a repetition weight. Higher `count` = higher boost. Training data has a system-enforced budget; balance boosts accordingly.
- **`PhraseCountsFromTemplates(classes:count:template:)`**: defines word classes (arrays of alternatives) and a template pattern to generate all combinations. The `count` is divided evenly across all generated samples, making it efficient for combinatorial domains (e.g., all chess moves from piece × file × rank).
- **`CustomPronunciation(grapheme:phonemes:)`** (used inside `PhraseCount`): provides X-SAMPA phonetic strings for non-standard spellings. Each locale supports a specific subset of X-SAMPA symbols; consult documentation for the full locale/symbol matrix.

Training data can be built at **development time** (baked into the app bundle) or at **runtime** (from user-specific data such as contacts or history). Runtime generation enables personalization while keeping private data on device.

Training data is locale-bound. For multi-locale support, use standard localization facilities (e.g., `NSLocalizedString`) to select the appropriate phrases per locale.

### Exporting and Preparing the Model
After constructing the `SFCustomLanguageModelData` object, export it to a file:
```swift
try await data.export(to: fileURL)
```

In the app (at launch or before first recognition), call:
```swift
try await SFSpeechLanguageModel.prepareCustomLanguageModel(
    for: fileURL,
    clientIdentifier: "com.example.myapp",
    configuration: config)
```
This compiles the data into two output files used by the recognizer. It has significant latency; call it off the main thread, ideally behind a loading screen. The compilation result is cached and only re-runs when input data changes.

### Configuring On-Device Recognition
```swift
let request = SFSpeechAudioBufferRecognitionRequest()
// REQUIRED: custom LM only works on-device
request.requiresOnDeviceRecognition = true
// Attach the compiled custom LM
request.customizedLanguageModel = preparedModelConfiguration
```
Omitting `requiresOnDeviceRecognition = true` causes the request to be served by the cloud recognizer without customization.

### Privacy and On-Device Guarantee
All language model customization data—including runtime-generated personal data like contacts—stays on device. The framework never uploads customization data to servers. This makes LM customization safe to use with sensitive data such as medical terminology, personal names, or proprietary product names.

## APIs & Frameworks

**Speech Framework (iOS 17 — New)**
- `SFCustomLanguageModelData` **[NEW]** — container for training data, built with result-builder DSL
- `SFCustomLanguageModelData.PhraseCount(phrase:count:)` **[NEW]** — single phrase with boost weight
- `SFCustomLanguageModelData.PhraseCountsFromTemplates(classes:count:template:)` **[NEW]** — template-based combinatorial phrase generation
- `SFCustomLanguageModelData.CustomPronunciation(grapheme:phonemes:)` **[NEW]** — X-SAMPA pronunciation for non-standard words
- `SFCustomLanguageModelData.export(to:)` **[NEW]** — serialize training data to file
- `SFSpeechLanguageModel.prepareCustomLanguageModel(for:clientIdentifier:configuration:)` **[NEW]** — compile training data into recognizer-ready model files
- `SFSpeechRecognitionRequest.customizedLanguageModel` **[NEW]** — attach compiled custom LM to a recognition request
- `SFSpeechRecognitionRequest.requiresOnDeviceRecognition` — must be `true` to use custom LM

## Code Highlights

Building training data with templates and custom pronunciation:
```swift
let data = SFCustomLanguageModelData(locale: Locale(identifier: "en_US")) {
    // Exact phrase boost
    SFCustomLanguageModelData.PhraseCount(phrase: "Albin counter gambit", count: 100)

    // Template: all chess moves (piece × file × rank)
    SFCustomLanguageModelData.PhraseCountsFromTemplates(
        classes: [
            "piece": ["pawn", "knight", "bishop", "rook", "queen", "king",
                      "a", "b", "c", "d", "e", "f", "g", "h"],
            "royalPiece": ["king", "queen"],
            "rank": ["1","2","3","4","5","6","7","8"]
        ],
        count: 10_000,
        template: "<piece> <royalPiece> <rank>"
    )

    // Custom pronunciation (X-SAMPA)
    SFCustomLanguageModelData.PhraseCount(
        phrase: "Winawer variation",
        count: 50
    ) {
        SFCustomLanguageModelData.CustomPronunciation(
            grapheme: "Winawer",
            phonemes: ["wI.n@.w@r"]
        )
    }
}
try await data.export(to: trainingDataURL)
```

Preparing and using the custom model:
```swift
// Off the main thread:
let config = SFSpeechLanguageModel.Configuration(languageModel: outputModelURL)
try await SFSpeechLanguageModel.prepareCustomLanguageModel(
    for: trainingDataURL,
    clientIdentifier: "com.example.chessapp",
    configuration: config)

// Configuring the request:
let request = SFSpeechAudioBufferRecognitionRequest()
request.requiresOnDeviceRecognition = true
request.customizedLanguageModel = config
```

## Resources
- [Speech framework documentation](https://developer.apple.com/documentation/Speech)
- [Recognizing speech in live audio](https://developer.apple.com/documentation/Speech/recognizing-speech-in-live-audio)
- Related: "Advances in Speech Recognition" (WWDC19 256)

## Takeaways
- `SFCustomLanguageModelData` with `PhraseCount` and `PhraseCountsFromTemplates` is the primary API for teaching the recognizer domain vocabulary; `count` controls boost strength relative to the training budget.
- `PhraseCountsFromTemplates` is the efficient path for combinatorial domains (commands with variable slots like move notation, product names with variants, etc.).
- `requiresOnDeviceRecognition = true` is mandatory—omitting it silently skips customization by routing to the cloud recognizer.
- Language model customization is appropriate for private data (contacts, medical terms, proprietary names) because customization data never leaves the device.

---
_Source: WWDC23 Session 10101 page (abstract, transcript, and resource links)._
