# Build Dynamic iOS Apps with the Create ML Framework
**WWDC21 · Session 10037** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10037/)

_Platforms:_ iOS 15, iPadOS 15

## Overview
The Create ML framework, previously limited to macOS, is now available on iOS 15 and iPadOS 15, enabling on-device model training directly within apps. This allows developers to build adaptive, personalized experiences without ever sending user data off the device, preserving privacy by design. Training and inference happen entirely on-device, eliminating the need for model deployment infrastructure.

The session demonstrates two concrete use cases: a photo booth app that trains a custom Style Transfer model from a single style image (such as a child's drawing), and a restaurant ordering app that trains a Tabular Linear Regressor to learn user preferences across meal contexts. Both examples show the breadth of Create ML tasks now available on iOS.

Most Create ML task types available on macOS — Image Classification, Sound Classification, Text Classification, Style Transfer, Hand Pose/Action Classifiers (new in 2021), Tabular Classifiers, and Tabular Regressors — are now trainable on iOS and iPadOS.

## Key Topics

### On-Device Training with the Create ML Framework
The Create ML framework arrives on iOS 15/iPadOS 15 with the same programmatic API surface previously available on macOS. Apps can call training APIs directly without any server round-trips.

### Style Transfer on iOS
A style transfer model is trained from a single style image and a content image set. Users can provide any photo as a style source, and the model applies that artistic style to new photos. Parameters include style strength, style density, filter type (image vs. video), and iteration count.

### Tabular Classifiers and Regressors
Four algorithm options are provided for both classifiers and regressors. A linear regressor can be trained incrementally as users interact with the app, learning preference scores for items based on content keywords and contextual signals (e.g., time of day / meal). Positive and negative training examples are both required for the model to discriminate preferences.

### Best Practices
- Always evaluate model performance on held-out data.
- For long-running tasks, use asynchronous training control and checkpointing.
- Account for computational intensity and memory usage in UI design.

## APIs & Frameworks

**Create ML** (`import CreateML`) — **[NEW on iOS/iPadOS]**

- `MLStyleTransfer` **[NEW on iOS]** — trains style transfer models on-device
  - `MLStyleTransfer.DataSource.images(styleImage:contentDirectory:)` — specifies style and content images
  - `MLStyleTransfer.train(trainingData:sessionParameters:)` — returns a training job
- `MLTrainingSessionParameters(sessionDirectory:)` — configures checkpoint save location
- `MLLinearRegressor` **[NEW on iOS]** — trains a linear regression model from tabular data
  - `MLLinearRegressor(trainingData:targetColumn:)` — creates model from a `DataFrame`
  - `model.predictions(from:)` — runs inference on a `DataFrame`
- `MLImageClassifier` — image classification (available on iOS)
- `MLSoundClassifier` — audio/sound classification (available on iOS)
- `MLTextClassifier` — text classification (available on iOS)
- `MLHandPoseClassifier` **[NEW]** — hand pose classification
- `MLHandActionClassifier` **[NEW]** — hand action sequence classification

**Tabular Data** (`import TabularData`)
- `DataFrame` — columnar in-memory data structure used as training and prediction input
  - `DataFrame.append(column:)` — appends a typed column
  - `Column<T>(name:contents:)` — typed column initializer

**Core ML** (`import CoreML`)
- `MLModel.compileModel(at:)` — compiles a written `.mlmodel` file to a compiled bundle
- `MLModel(contentsOf:)` — loads a compiled Core ML model
- `MLDictionaryFeatureProvider(dictionary:)` — wraps input features for prediction
- `mlModel.prediction(from:)` — runs Core ML inference
- `model.write(to:)` — writes trained Create ML model to disk

## Code Highlights

Training a Style Transfer model:
```swift
let data = MLStyleTransfer.DataSource.images(styleImage: styleUrl, contentDirectory: contentUrl)
let sessionParameters = MLTrainingSessionParameters(sessionDirectory: sessionUrl)
let job = try MLStyleTransfer.train(trainingData: data, sessionParameters: sessionParameters)

// On successful completion:
try model.write(to: writeToUrl)
let compiledURL = try MLModel.compileModel(at: writeToUrl)
let mlModel = try MLModel(contentsOf: compiledURL)
let inputImage = try MLDictionaryFeatureProvider(dictionary: ["image": image])
let stylizedImage = try mlModel.prediction(from: inputImage)
```

Training a Tabular Linear Regressor with positive/negative examples:
```swift
var trainingData = DataFrame()
trainingData.append(column: Column(name: "keywords", contents: trainingKeywords))
trainingData.append(column: Column(name: "target", contents: trainingTargets))
let model = try MLLinearRegressor(trainingData: trainingData, targetColumn: "target")

// Prediction:
var inputData = DataFrame()
inputData.append(column: Column(name: "keywords", contents: dishKeywords))
let predictions = try model.predictions(from: inputData)
```

## Takeaways
- Create ML framework is now available on iOS 15 and iPadOS 15, enabling fully on-device model training.
- Style Transfer and Tabular Regressors/Classifiers can be trained at runtime from user data, creating truly personalized app experiences.
- User data never leaves the device — privacy is preserved by construction.
- The same API patterns used on macOS apply on iOS; most existing Create ML task types are now cross-platform.

---
_Source: WWDC21 Session 10037 page (abstract, chapter summaries, code samples, and resource links)._
