# Training Sound Classification Models in Create ML
**WWDC19 · Session 425** · [Watch](https://developer.apple.com/videos/play/wwdc2019/425/)

_Platforms:_ iOS 13, macOS Catalina 10.15

## Overview
Create ML in macOS Catalina gains a Sound Classifier template, allowing developers to train on-device Core ML models that identify sounds in audio files and live audio streams — all without writing code. A companion new framework, SoundAnalysis, handles the plumbing of feeding audio buffers to the model, including channel mapping, sample-rate conversion, and windowed overlap analysis.

The session explains three conceptual axes of sound classification (object, environment, and attribute), walks through the data preparation requirements, and demos a real-time musical instrument classifier trained in under a minute inside the Create ML app. The microphone recording feature in the Output tab lets developers validate model performance live before export.

The SoundAnalysis framework is a high-level abstraction over Core ML that lets an app analyze audio files or streaming microphone input with just a few lines of Swift, receiving time-stamped classification callbacks as results arrive.

## Key Topics

**Data Collection and Organization**
Audio training data must be organized into labeled folders — one folder per class. A single audio file may only contain one sound class; mixed-content recordings must be pre-split. Including a "background" or ambient noise class is essential if the model will run in real-world environments.

**Training in the Create ML App**
The Sound Classifier template accepts labeled directories, trains using audio feature extraction across entire files, and shows live training/validation accuracy. The Output tab supports drag-and-drop file analysis and a live Record Microphone mode that streams microphone data through the trained model in real time.

**SoundAnalysis Framework**
A new high-level framework released alongside iOS 13 and macOS Catalina. Internally handles channel mapping (e.g., stereo to mono), sample-rate conversion (e.g., to 16 kHz), and audio buffering/re-blocking to the model's required window size. Results are delivered asynchronously via `SNResultsObserving`. The default analysis window uses 50% overlap to avoid sounds falling between windows.

**Analysis Window and Overlap**
The sound classifier processes approximately 1-second audio chunks. Results carry an associated `CMTimeRange` indicating which segment of audio was analyzed. 50% overlap is the default; this is configurable through the API.

**AVAudioSession Modes**
Selecting the correct `AVAudioSession.Mode` is important to match the microphone processing applied during data collection; mismatched processing can degrade inference accuracy.

## APIs & Frameworks

**Create ML**
- `MLSoundClassifier` **[NEW]** — trains a Core ML model from labeled audio directories
- `MLSoundClassifier.ModelParameters` **[NEW]** — configuration (iterations, validation split)
- Create ML app Sound Classifier template **[NEW]**
- Real-time microphone recording in the Output tab **[NEW]**

**SoundAnalysis** **[NEW framework]**
- `SNAudioFileAnalyzer` **[NEW]** — analyzes a complete audio file
- `SNAudioStreamAnalyzer` **[NEW]** — analyzes a real-time audio stream (e.g., from AVAudioEngine)
- `SNClassifySoundRequest` **[NEW]** — wraps a Core ML model for use with SoundAnalysis
- `SNResultsObserving` **[NEW]** — protocol for receiving classification callbacks
  - `request(_:didProduce:)` — called with each new `SNClassificationResult`
  - `request(_:didFailWithError:)` — called if analysis encounters an error
  - `requestDidComplete(_:)` — called when a file analysis finishes
- `SNClassificationResult` **[NEW]** — contains ranked classifications and a `CMTimeRange`
- `SNClassification` **[NEW]** — individual label + confidence pair

**Core ML**
- Generated model class (e.g., `MySoundClassifier`) integrated via `SNClassifySoundRequest`

**AVFoundation**
- `AVAudioEngine` — feeds microphone PCM buffers to `SNAudioStreamAnalyzer`
- `AVAudioSession.Mode` — controls microphone processing pipeline; must match training conditions

## Code Highlights

Analyzing an audio file with SoundAnalysis:

```swift
import SoundAnalysis

let analyzer = try SNAudioFileAnalyzer(url: audioFileURL)
let request = try SNClassifySoundRequest(mlModel: MySoundClassifier().model)
try analyzer.add(request, withObserver: self)
analyzer.analyze()
```

Implementing the results observer:

```swift
extension MyClass: SNResultsObserving {
    func request(_ request: SNRequest, didProduce result: SNResult) {
        guard let classification = result as? SNClassificationResult else { return }
        let topLabel = classification.classifications.first?.identifier ?? "unknown"
        let timeRange = classification.timeRange
        print("[\(timeRange.start)] \(topLabel)")
    }

    func request(_ request: SNRequest, didFailWithError error: Error) {
        print("Analysis failed: \(error)")
    }

    func requestDidComplete(_ request: SNRequest) {
        print("Analysis complete")
    }
}
```

## Takeaways
- The Create ML Sound Classifier template and the new SoundAnalysis framework form an end-to-end pipeline: train in the app, integrate with four lines of Swift.
- SoundAnalysis internally manages channel mapping, sample-rate conversion, and buffer windowing, eliminating boilerplate audio plumbing from app code.
- Training data organization (one class per folder, one class per file) and inclusion of a background noise class are the most impactful data quality decisions.
- Match the `AVAudioSession.Mode` at inference time to the microphone processing used during data recording to preserve model accuracy.

---
_Source: WWDC19 Session 425 page (abstract, transcript, and resource links)._
