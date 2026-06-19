# What's New in Machine Learning
**WWDC19 · Session 209** · [Watch](https://developer.apple.com/videos/play/wwdc2019/209/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, watchOS 6, tvOS 13

## Overview
This session provides a high-level overview of the entire Apple ML ecosystem in 2019, organized around three pillars: the new Create ML app for training models without code, expanded Domain APIs across Vision, Natural Language, Speech, and Sound, and Core ML 3 with 100+ new neural-network layer types and on-device model personalization.

The overarching design philosophy is "Machine Learning for everyone" — making ML easy for developers new to the field while keeping it flexible and powerful enough for ML researchers. Every capability runs on-device, preserving user privacy and eliminating the need for servers.

## Key Topics

### Create ML App **[NEW]**
- A new standalone macOS app (replacing the Swift Playground-based workflow) for training Core ML models covering image classification, object detection, sound classification, activity classification, text classification, and recommendation.
- Point-and-click training pipeline: drag in training data, configure parameters, watch live training graphs, export a `.mlmodel`.
- Discussed in depth in Sessions 406, 407, 425, 426, and 430.

### Vision Framework Additions **[NEW]**
- **Image Saliency**: identifies the most visually relevant region in an image; use for smart cropping, thumbnail generation, and memory curation.
  - `VNGenerateAttentionBasedSaliencyImageRequest` — attention-based saliency **[NEW]**
  - `VNGenerateObjectnessSaliencyImageRequest` — objectness-based saliency **[NEW]**
- **Text Recognition**: full on-device document text recognition with perspective and lighting correction.
  - `VNRecognizeTextRequest` **[NEW]**
- Built-in image classifier, human body detector, pet (cat/dog) detector updated.
- Details in Sessions 222 and 234.

### Natural Language Framework Additions **[NEW]**
- **Sentiment Analysis**: on-device, real-time, privacy-friendly positive/negative/neutral classification.
  - `NLTagger` with `.sentimentScore` scheme **[NEW]**
- **Word Embeddings**: exposing Apple's built-in word embedding vectors for semantic similarity search.
  - `NLEmbedding` **[NEW]**
  - `NLEmbedding.wordEmbedding(for:)` — load built-in embedding for a language
  - `NLEmbedding.neighbors(for:maximumCount:)` — find semantically similar words
- Details in Session 232.

### Speech and Sound **[NEW]**
- **On-Device Speech Recognition**: transcribe speech without a network connection.
  - `SFSpeechRecognizer` with on-device `SFSpeechRecognitionRequest.requiresOnDeviceRecognition = true` **[NEW]**
- **Voice Analytics API**: characterize speech properties beyond transcription (jitter, shimmer, pitch, voicing).
  - `SFVoiceAnalytics` **[NEW]**
- **Sound Analysis Framework**: classify arbitrary audio (e.g., dog barking, car horn); train custom classifiers in Create ML.
  - `SNClassifySoundRequest` **[NEW]**
  - `SNAudioStreamAnalyzer` / `SNAudioFileAnalyzer` **[NEW]**
- Details in Sessions 425 and 256.

### Core ML 3 **[NEW]**
- **Expanded Neural Network Layer Support**: 100+ new layer types enabling state-of-the-art architectures including ELMo, BERT, WaveNet, instance segmentation models, and audio generation models.
- **Updated Converters**: new TensorFlow → Core ML converter; ONNX converter update coming soon.
- **Model Gallery**: curated research models pre-converted to `.mlmodel` for immediate use.
- **On-Device Model Personalization**: fine-tune a Core ML model on the device using the user's own labeled data; no server required; no data leaves the device.
  - Supports Neural Network fine-tuning and Nearest Neighbor classification.
  - Background processing available (e.g., overnight).
  - Powers use cases like personalized photo search ("My Dog"), Face ID, Watch face personalization.
- Core ML runs on all Apple platforms (iOS, iPadOS, macOS, tvOS, watchOS) with hardware acceleration (CPU, GPU, Neural Engine).

### Combining Domain APIs
- Vision + Natural Language: tag images with the built-in image classifier, then use `NLEmbedding` to expand user search queries to semantically similar tags — enabling semantic image search.
- Custom Create ML models can be combined with built-in Domain API models in the same pipeline.

## APIs & Frameworks

### Core ML
- `MLModel` — load and run any Core ML model
- `MLModelConfiguration` **[NEW]** — configure compute units (CPU, GPU, Neural Engine)
- `MLUpdateTask` **[NEW]** — perform on-device personalization / fine-tuning
- `MLArrayBatchProvider` / `MLBatchProvider` — batch input for training
- Support for 100+ new neural network layer types **[NEW]**

### Vision
- `VNGenerateAttentionBasedSaliencyImageRequest` **[NEW]**
- `VNGenerateObjectnessSaliencyImageRequest` **[NEW]**
- `VNSaliencyImageObservation` **[NEW]**
- `VNRecognizeTextRequest` **[NEW]**
- `VNRecognizedTextObservation` **[NEW]**
- `VNDetectHumanRectanglesRequest` **[NEW]**
- `VNDetectAnimalRectanglesRequest` (cats/dogs) **[NEW]**
- `VNClassifyImageRequest` (updated)
- `VNImageRequestHandler`, `VNSequenceRequestHandler`

### Natural Language
- `NLTagger` — multi-purpose tagger
- `NLTagScheme.sentimentScore` **[NEW]**
- `NLEmbedding` **[NEW]**
- `NLEmbedding.wordEmbedding(for:)` **[NEW]**
- `NLEmbedding.neighbors(for:maximumCount:distanceType:)` **[NEW]**
- `NLDistance` / `NLDistanceType` **[NEW]**

### Speech
- `SFSpeechRecognizer` (updated)
- `SFSpeechRecognitionRequest.requiresOnDeviceRecognition` **[NEW]**
- `SFVoiceAnalytics` **[NEW]** — jitter, shimmer, pitch, voicing
- `SFTranscriptionSegment.voiceAnalytics` **[NEW]**

### Sound Analysis
- `SNClassifySoundRequest` **[NEW]**
- `SNAudioStreamAnalyzer` **[NEW]**
- `SNAudioFileAnalyzer` **[NEW]**
- `SNResultsObserving` protocol **[NEW]**
- `SNClassificationResult` **[NEW]**

### Create ML
- `MLImageClassifier` (updated)
- `MLObjectDetector` **[NEW]**
- `MLSoundClassifier` **[NEW]**
- `MLActivityClassifier` **[NEW]**
- `MLTextClassifier` (updated)
- `MLRecommender` **[NEW]**

## Code Highlights

Semantic image search combining Vision and Natural Language:
```swift
// Step 1: classify images to get tags
let classifyRequest = VNClassifyImageRequest()
let handler = VNImageRequestHandler(cgImage: cgImage)
try handler.perform([classifyRequest])
let tags = classifyRequest.results?.map { $0.identifier } ?? []

// Step 2: expand search query using word embeddings
let embedding = NLEmbedding.wordEmbedding(for: .english)!
let similarWords = embedding.neighbors(for: "thunderstorm", maximumCount: 5)
    .map { $0.0 }  // ["cloudy", "lightning", "storm", "sky", ...]

// Step 3: match images whose tags intersect with expanded query
let matches = imageTagMap.filter { !$0.value.isDisjoint(with: Set(similarWords)) }
```

On-device sentiment analysis:
```swift
let tagger = NLTagger(tagSchemes: [.sentimentScore])
tagger.string = "I was so excited about the season finale"
let sentiment = tagger.tag(at: tagger.string!.startIndex,
                            unit: .paragraph,
                            scheme: .sentimentScore).0?.rawValue
// sentiment is a string like "0.85" (positive)
```

## Takeaways
- Core ML 3's 100+ new layer types unlock cutting-edge architectures (BERT, ELMo, WaveNet, instance segmentation) directly on device.
- On-device model personalization via `MLUpdateTask` enables user-specific models without any data leaving the device.
- Domain APIs now cover sentiment analysis, word embeddings, on-device speech, voice analytics, and sound classification — all out-of-the-box with no model training required.
- The new Create ML app and updated converters (TensorFlow, ONNX) dramatically lower the barrier to bringing custom ML models into apps.

---
_Source: WWDC19 Session 209 page (abstract, chapter summaries, code samples, and resource links)._
