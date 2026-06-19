# Discover built-in sound classification in SoundAnalysis
**WWDC21 · Session 10036** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10036/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, tvOS 15, watchOS 8

## Overview
iOS 15 and macOS Monterey introduce a **built-in sound classifier** in the SoundAnalysis framework, eliminating the need to train a custom Core ML model to identify common sounds. With a single API call, apps can now classify over 300 categories of sound—animals, musical instruments, human vocalizations, vehicles, alarms, tools, and more—entirely on-device, in real time or from audio files.

The session also introduces **Audio Feature Print**, a new, smaller, faster feature extractor now used as the default in CreateML sound classification training. Custom models built with Audio Feature Print are more accurate and support flexible window durations inherited from the built-in classifier architecture.

## Key Topics

### Built-in Sound Classifier **[NEW in iOS 15]**
- A pre-trained, on-device sound classifier shipped with the OS; no model download or training required.
- Supports 300+ sound categories across domains:
  - Animals: domestic, livestock, wild.
  - Music: keyboard, percussion, string, wind instruments.
  - Human sounds: group activities, respiratory sounds, vocalizations.
  - Environmental: vehicles, alarms, tools, liquids.
- All computation is on-device; audio is never sent to the cloud, preserving user privacy.
- Available on iOS, iPadOS, macOS, tvOS, and watchOS.
- Accessed via `SNClassifySoundRequest(classifierIdentifier: .version1)`.

### Window Duration and Time of Detection
- Audio is broken into overlapping windows; each window produces an `SNClassificationResult` with a `timeRange` and confidence scores.
- Window duration is configurable via `SNClassifySoundRequest.windowDuration`.
- Supported window durations: 0.5 to 15 seconds (check `windowDurationConstraint` for the allowed range).
- Short windows (≥0.5 s) work well for brief sounds (drum tap); longer windows better capture sustained sounds (siren). A 1-second starting point is recommended.
- Changing window duration affects confidence score magnitudes; re-tune thresholds after changing it.

### Confidence Thresholds
- Unlike custom single-label classifiers, built-in classifier label scores are independent and do not sum to 1; multiple labels can have high confidence simultaneously.
- Choose per-label thresholds based on the acceptable false-positive vs. false-negative tradeoff for your use case.
- A starting value of `0.5` is reasonable for many categories.

### Audio Feature Print **[NEW in CreateML]**
- The built-in classifier's feature extractor, now exposed for use as the default backbone when training custom models in CreateML.
- Replaces the previous generation extractor: smaller, faster, and higher accuracy across all benchmarks.
- Models trained with Audio Feature Print also support flexible window durations (0.5–15 seconds), matching the built-in classifier.
- Automatically used when training a new Sound Classification model in CreateML (no additional configuration required).

## APIs & Frameworks

**SoundAnalysis**
- `SNClassifySoundRequest(classifierIdentifier: .version1)` — creates a request targeting the built-in classifier **[NEW]**
- `SNClassifySoundRequest.knownClassifications` — returns the list of supported sound label strings **[NEW]**
- `SNClassifySoundRequest.windowDuration` — set a custom analysis window duration **[existing, now supported by built-in classifier]**
- `SNClassifySoundRequest.windowDurationConstraint` — query supported window durations **[existing]**
- `SNAudioFileAnalyzer(url:)` — classify sounds from an audio/video file **[existing]**
- `SNAudioStreamAnalyzer` — classify sounds from a live audio stream **[existing]**
- `SNResultsObserving` protocol — `request(_:didProduce:)` callback receives `SNClassificationResult` **[existing]**
- `SNClassificationResult.classification(forIdentifier:)` — look up a label's confidence **[existing]**
- `SNClassificationResult.timeRange` — `CMTimeRange` indicating the analyzed audio window **[existing]**

**CreateML**
- Audio Feature Print feature extractor — new default for Sound Classification model training **[NEW]**
- `MLSoundClassifier` with Audio Feature Print backbone **[NEW]**
- Window Duration parameter: 0.5–15 seconds, default 3 seconds **[NEW flexibility]**

## Code Highlights

Get the list of all sounds the built-in classifier can recognize:
```swift
func getListOfRecognizedSounds() throws -> [String] {
    let request = try SNClassifySoundRequest(classifierIdentifier: .version1)
    return request.knownClassifications
}
```

Set up a file-based classification pipeline:
```swift
let request = try SNClassifySoundRequest(classifierIdentifier: .version1)
let analyzer = try SNAudioFileAnalyzer(url: url)
let observer = FirstDetectionObserver(label: "cowbell")

try analyzer.add(request, withObserver: observer)
analyzer.analyze()
```

Observer that records the first time a specific sound is detected:
```swift
class FirstDetectionObserver: NSObject, SNResultsObserving {
    var firstDetectionTime = CMTime.invalid
    var label: String

    init(label: String) {
        self.label = label
    }

    func request(_ request: SNRequest, didProduce result: SNResult) {
        if let result = result as? SNClassificationResult,
           let classification = result.classification(forIdentifier: label),
           classification.confidence > 0.5,
           firstDetectionTime == CMTime.invalid {
            firstDetectionTime = result.timeRange.start
        }
    }
}
```

## Takeaways
- The built-in sound classifier removes the biggest barrier to sound classification: no data collection, no training, no custom model—just one API call to classify 300+ sounds on-device.
- Because confidence scores for different labels are independent (not a probability distribution), use per-label thresholds rather than comparing labels against each other.
- Window duration is a key tuning parameter: match it to the typical duration of the sounds you care about; shorter for transient sounds, longer for sustained ones.
- Custom CreateML sound models now automatically use Audio Feature Print as their backbone, giving smaller and more accurate models with flexible window duration support—no extra work required.

---
_Source: WWDC21 Session 10036 page (abstract, full transcript, code samples, and resource links)._
