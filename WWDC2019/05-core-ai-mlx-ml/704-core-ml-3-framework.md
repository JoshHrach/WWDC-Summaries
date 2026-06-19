# Core ML 3 Framework
**WWDC19 · Session 704** · [Watch](https://developer.apple.com/videos/play/wwdc2019/704/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, watchOS 6, tvOS 13

## Overview
Core ML 3 is a major release with three headline capabilities: on-device model personalization (updating a model with user-specific training data without a server), expanded neural network support (control flow, dynamic shapes, 100+ new layer types, enabling state-of-the-art architectures like BERT), and several new convenience APIs (linked models, CGImage/URL input initializers, GPU configuration options).

The session walks through a hand-drawn sticker recognition demo (K-Nearest Neighbor classifier personalized with Apple Pencil sketches) and a question-answering BERT model demo (TensorFlow → Core ML conversion in 3 lines of Python, fully on-device NLP with Speech + Natural Language + AVFoundation).

## Key Topics

**On-Device Model Personalization**
- Traditional server-side personalization requires a cloud backend, exposes user data, and needs connectivity; Core ML 3 moves all of this on-device
- Personalization workflow: start with a base model that has updatable layers/parameters → collect training data on-device → run `MLUpdateTask` → receive an updated model in the completion handler; use immediately for prediction
- Data privacy: training data never leaves the device
- Supported updatable model types: **K-Nearest Neighbor classifier** and **neural networks** (convolution + fully connected layers); pipelines containing an updatable model are also updatable
- KNN use case: base model contains a frozen feature extractor (pretrained neural network) feeding into an empty KNN classifier; update task adds labeled neighbors
- After the update task completes, write the new model to disk with `MLUpdateContext.model.write(to:)` and load it for future predictions

**`MLUpdateTask`**
- Manages the lifecycle of a model update: `.suspended`, `.running`, `.cancelling`, `.completed`
- Two modes: completion-handler only, or progress handler + completion handler
- `MLUpdateProgressHandler(requestedMetrics:progressHandler:completionHandler:)` — subscribe to per-epoch events (`trainingBegin`, `miniBatchEnd`, `epochEnd`, `completed`)
- `MLUpdateProgressEvent` identifies which event fired
- `MLUpdateContext` provides the updated model and training metrics (loss, accuracy)
- Override update hyperparameters at runtime via `MLModelConfiguration` + `MLParameterKey` dictionary

**Neural Network Spec Expansions (Core ML 3)**
- Control flow: **branch** (if/else) and **loop** nodes in the model graph — enables recurrent/dynamic architectures
- Dynamic shapes: allocate intermediate tensors at runtime based on input shape (variable-length sequences)
- 100+ new layer types: expanded basic math operations, new activation functions, normalization layers; most state-of-the-art architectures can now be expressed in Core ML format
- Newly supported architectures: BERT, GPT, GAN, SqueezeNet variants, EfficientNet, among others highlighted in session

**BERT on Device**
- Bidirectional Encoder Representations from Transformers (BERT) — state-of-the-art NLP architecture requiring control flow + dynamic shapes → enabled by Core ML 3
- Conversion from TensorFlow: `coremltools.converters.tensorflow.convert()` → save as `.mlmodel` — 3 lines of Python
- App pipeline: Speech Framework (speech-to-text) → Natural Language Framework (tokenizer) → Core ML BERT model (question answering) → AVFoundation (text-to-speech); fully offline, no network required

**Linked Models (NEW)**
- `MLLinkedModel` — a model type that is a reference to another model on disk rather than an embedded copy
- Functions like a dynamic library link: one model on disk, multiple pipeline models referencing it by name and search path
- Eliminates redundant copies of shared feature extractors across multiple pipelines
- Particularly useful when a shared model is also updatable (one update propagates to all consumers)

**Image Input Enhancements (NEW)**
- Previously: images must be provided as `CVPixelBuffer`
- New initializers: load directly from `CGImage` or `URL`
- `MLFeatureValue(cgImage:pixelFormatType:size:options:)` **[NEW]**
- `MLFeatureValue(imageAt:pixelFormatType:size:options:)` **[NEW]**
- Note: Vision framework's `VNCoreMLRequest` is still preferred for typical inference; direct initializers needed when calling update APIs

**`MLModelConfiguration` GPU Options (NEW)**
- `preferredMetalDevice: MTLDevice?` **[NEW]** — pin model execution to a specific GPU (useful on Mac with multiple GPUs)
- `allowLowPrecisionAccumulationOnGPU: Bool` **[NEW]** — float16 accumulation on GPU for speed gains; always verify accuracy after enabling

**Background Training via BackgroundTasks Framework**
- Use `BGTaskScheduler` + `BGProcessingTaskRequest` to schedule model updates when the app is backgrounded or the device is idle
- System grants several minutes of runtime even when the user is not interacting with the app

**Core ML Tools (Python)**
- `coremltools` — open-source Python package for model conversion and specification building
- Pass `trainable=True` flag when converting to make layers updatable in the resulting `.mlmodel`
- Converters updated for Core ML 3 spec: TensorFlow, PyTorch (via ONNX), Keras, scikit-learn — fewer missing-layer errors

## APIs & Frameworks

### Core ML (Updated — iOS 13 / macOS 10.15)
- `MLUpdateTask` **[NEW]** — manages on-device model update
  - `init(forModelAt:configuration:progressHandlers:)` **[NEW]**
  - `init(forModelAt:configuration:completionHandler:)` **[NEW]**
  - `resume()`, `cancel()`
  - `taskState: MLUpdateTask.TaskState`
- `MLUpdateProgressHandler` **[NEW]** — per-event callbacks during update
- `MLUpdateProgressEvent` **[NEW]** — `.trainingBegin`, `.miniBatchEnd`, `.epochEnd`, `.completed`
- `MLUpdateContext` **[NEW]** — result of update; `.model: MLModel`, `.metrics: [MLMetricKey: Any]`
- `MLMetricKey` **[NEW]** — `.lossValue`, `.epochIndex`, `.miniBatchIndex`
- `MLParameterKey` **[NEW]** — keys for overriding hyperparameters: `.learningRate`, `.numberOfEpochs`, `.miniBatchSize`, `.momentum`, `.beta1`, `.beta2`, `.eps`
- `MLModelConfiguration.preferredMetalDevice: MTLDevice?` **[NEW]**
- `MLModelConfiguration.allowLowPrecisionAccumulationOnGPU: Bool` **[NEW]**
- `MLFeatureValue(cgImage:pixelFormatType:size:options:)` **[NEW]**
- `MLFeatureValue(imageAt:pixelFormatType:size:options:)` **[NEW]**
- `MLLinkedModel` **[NEW]** — model reference type for shared/linked models
- `MLModel.write(to:)` — save updated model variant to disk (unchanged API)
- Auto-generated training input class (e.g., `MyModelTrainingInput`) **[NEW]** — Xcode generates alongside prediction input class for updatable models

### Core ML Neural Network Spec (Updated)
- Branch node **[NEW]** — if/else control flow within model graph
- Loop node **[NEW]** — while/for control flow within model graph
- Dynamic layers **[NEW]** — runtime shape allocation
- 100+ new layer types **[NEW]** including expanded math ops, activations, normalizations

### Vision (referenced)
- `VNCoreMLRequest` — preferred for image inference; handles preprocessing, format conversion

### Natural Language (referenced)
- `NLTokenizer` — used for BERT tokenization

### Speech (referenced)
- `SFSpeechRecognizer` — speech-to-text for voice query input

### AVFoundation (referenced)
- `AVSpeechSynthesizer` — text-to-speech for reading answers aloud

### BackgroundTasks (referenced)
- `BGTaskScheduler`, `BGProcessingTaskRequest` — schedule background model updates

## Code Highlights

Kicking off a model update with completion handler:
```swift
let updateTask = try MLUpdateTask(
    forModelAt: updatableModelURL,
    configuration: nil,
    completionHandler: { context in
        print("Training loss: \(context.metrics[.lossValue] ?? "N/A")")
        let updatedModel = context.model
        try? updatedModel.write(to: updatedModelURL)
        self.currentModel = updatedModel
    }
)
updateTask.resume()
```

Using a progress handler for per-epoch monitoring:
```swift
let progressHandlers = MLUpdateProgressHandlers(
    forEvents: [.trainingBegin, .epochEnd],
    progressHandler: { context in
        if context.event == .epochEnd {
            let loss = context.metrics[.lossValue] as? Double ?? 0
            print("Epoch \(context.metrics[.epochIndex]!), loss: \(loss)")
        }
    },
    completionHandler: { context in
        try? context.model.write(to: updatedModelURL)
    }
)
let updateTask = try MLUpdateTask(
    forModelAt: updatableModelURL,
    configuration: nil,
    progressHandlers: progressHandlers
)
updateTask.resume()
```

Overriding hyperparameters at runtime:
```swift
let config = MLModelConfiguration()
config.parameters = [
    .learningRate: 0.001,
    .numberOfEpochs: 5
]
let updateTask = try MLUpdateTask(
    forModelAt: updatableModelURL,
    configuration: config,
    completionHandler: { _ in }
)
```

Converting a TensorFlow model to Core ML 3 (Python):
```python
import coremltools.converters.tensorflow as tf_converter
mlmodel = tf_converter.convert("bert_model.pb", outputs=["output"])
mlmodel.save("BERT.mlmodel")
```

## Takeaways
- On-device model personalization is the right architecture whenever user-specific training data is sensitive, the dataset is per-user, or offline use is required — it eliminates cloud backend costs and protects user privacy.
- K-Nearest Neighbor classifiers are the easiest entry point for personalization: add labeled examples at runtime with no gradient computation required; update takes seconds even on device.
- Core ML 3's control flow (branch + loop nodes) and dynamic shapes unlock BERT-class architectures that previously required server-side inference; check `coremltools` for updated converters.
- Use `BGProcessingTaskRequest` to schedule heavier neural network update tasks during device idle time — the system grants several minutes of background runtime even with the screen off.
- Always test accuracy after enabling `allowLowPrecisionAccumulationOnGPU` — float16 accumulation can degrade model output quality depending on architecture.

---
_Source: WWDC19 Session 704 page (abstract and full transcript)._
