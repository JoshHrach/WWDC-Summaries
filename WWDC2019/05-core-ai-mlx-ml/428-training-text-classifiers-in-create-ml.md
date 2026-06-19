# Training Text Classifiers in Create ML
**WWDC19 · Session 428** · [Watch](https://developer.apple.com/videos/play/wwdc2019/428/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15

## Overview
Create ML in macOS Catalina adds Transfer Learning with Dynamic Embedding to its Text Classifier template, enabling state-of-the-art natural language classification with far less training data than conventional approaches. Apple pre-trained a model on billions of text samples, shipped that pretrained model as part of the OS, and exposes it as a feature extractor that developers can fine-tune inside the Create ML app with just their own labeled examples.

Transfer Learning with Dynamic Embedding understands semantic context: unlike static word embeddings that treat identical words identically regardless of surrounding text, dynamic embeddings capture how the same word can mean different things in different sentences. This substantially improves accuracy on small custom datasets — the primary scenario for independent app developers.

The session explains all three available text classification algorithms, demonstrates the full Create ML app workflow, covers data organization best practices, and shows the single `ModelParameters` change needed to switch to Transfer Learning in Swift code.

## Key Topics

**Text Classification Use Cases**
- Sentiment analysis (positive / negative)
- Spam detection
- Topic classification (sports, entertainment, nature, etc.)
- "Other" catch-all class for out-of-domain inputs

**Data Organization**
Training data is a folder of subfolders, one per class. Each text file within a class folder is a single labeled example. The label is the folder name (emoji folder names are supported). A dedicated "other" class folder should be included if the app must reject out-of-domain inputs.

**Three Algorithms**
1. MaxEnt (Maximum Entropy) — fast, low data requirement, keyword-oriented; supported since 2018
2. CRF (Conditional Random Field) — sequence-aware; supported since 2018
3. Transfer Learning with Dynamic Embedding **[NEW]** — semantic context-aware; requires more training time but works well with limited data

**Transfer Learning with Dynamic Embedding**
Apple trained a language model on billions of texts and ships it as part of the OS. Create ML uses it as a pretrained feature extractor; the custom model only fine-tunes on top. Because the pretrained model is already on-device, the final `.mlmodel` file stays small and inference is fast.

**Create ML App Interactive Testing**
The Output tab supports real-time classification: typing text with pauses triggers live model predictions. Files and entire folders can also be dragged in for batch evaluation.

**Best Practices**
- Balance the number of examples across classes
- Match training data length distribution to expected runtime inputs (short sentences vs. full articles)
- Include an "other" class for robustness against out-of-domain text

## APIs & Frameworks

**Create ML**
- `MLTextClassifier` — trains a text classification Core ML model
- `MLTextClassifier.ModelParameters` — configuration type for the classifier
- `MLTextClassifier.ModelAlgorithmType.transferLearning(featureExtractor:revision:)` **[NEW]** — Transfer Learning algorithm selector
- `MLTextClassifier.FeatureExtractorType.dynamicEmbedding` **[NEW]** — Dynamic Embedding feature extractor (context-aware)
- `MLTextClassifier.FeatureExtractorType.staticEmbedding` — Static word embedding (carried over from 2018)
- `MLTextClassifier(trainingData:textColumn:labelColumn:parameters:)` — designated initializer
- `MLTextClassifier.makeTrainingSession(trainingData:textColumn:labelColumn:parameters:sessionParameters:)` — async training session
- `MLTextClassifier.write(to:metadata:)` — export to `.mlmodel`
- Create ML app Text Classifier template, interactive live-prediction input field **[NEW]**

**Natural Language**
- `NLModel` — loads the exported `.mlmodel` for inference
- `NLModel.predictedLabel(for:)` — classifies a string
- `NLTagger` — can use a custom `NLModel` for tagging

**Core ML**
- Generated text classifier model class, usable via `NLModel` or directly via `MLModel`

## Code Highlights

Switching to Transfer Learning with Dynamic Embedding in Swift:

```swift
import CreateML

// Load training data (folder-of-folders organization)
let trainingData = try MLDataTable(contentsOf: trainingDataURL)

// Configure Transfer Learning with Dynamic Embedding
let parameters = MLTextClassifier.ModelParameters(
    algorithm: .transferLearning(
        featureExtractor: .dynamicEmbedding,
        revision: 1
    )
)

// Train the classifier
let classifier = try MLTextClassifier(
    trainingData: trainingData,
    textColumn: "text",
    labelColumn: "label",
    parameters: parameters
)

// Export
try classifier.write(to: URL(fileURLWithPath: "TopicClassifier.mlmodel"))
```

Using the model at runtime:

```swift
import NaturalLanguage

let model = try NLModel(contentsOf: modelURL)
let label = model.predictedLabel(for: "The Raptors are on top of the mountain.")
// → "sport" (or whichever custom label matches)
```

## Takeaways
- Transfer Learning with Dynamic Embedding is the recommended algorithm for new text classifiers: it delivers high accuracy on small datasets and understands semantic context.
- The pretrained language model ships with the OS; the exported `.mlmodel` is compact because it only stores the fine-tuned layers.
- Data balance and length consistency are the most important data quality factors; unbalanced classes produce biased models regardless of algorithm.
- The same workflow (folder organization → Create ML app → drag-and-drop model to Xcode) applies to all three algorithms, and the API difference in Swift is a single parameter change.

---
_Source: WWDC19 Session 428 page (abstract, transcript, and resource links)._
