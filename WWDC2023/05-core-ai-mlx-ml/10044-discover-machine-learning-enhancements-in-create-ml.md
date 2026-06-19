# Discover Machine Learning Enhancements in Create ML
**WWDC23 · Session 10044** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10044/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14

## Overview
Create ML engineer David Findlay presents three major improvements in WWDC23: a new multilingual BERT embedding model for text classification, a brand-new multi-label image classifier task, and a new custom augmentation API built on Create ML Components. Together, these advances make it practical to build high-accuracy models with smaller, multilingual, or more varied datasets.

The session is the practical counterpart to "Explore Natural Language multilingual models" (10042) and builds directly on the Create ML Components framework introduced at WWDC22.

## Key Topics

### Text Classification: BERT Multilingual Embedding **[NEW]**
Create ML's text classifier now offers a BERT (Bidirectional Encoder Representations from Transformers) embedding model trained on billions of labeled text examples. Key properties:
- **Multilingual**: training data can contain multiple languages in a single model.
- **Improved monolingual accuracy**: BERT also boosts accuracy for single-language classifiers.
- Available in Create ML app Settings → Model Parameters → Feature Extractor.
- Compatible with iOS 17, iPadOS 17, macOS Sonoma.

### Image Classification: New Feature Extractor **[NEW]**
The latest version of the Apple Neural Scene Analyzer is now available as a feature extractor in Create ML's image classifier. Benefits over previous version:
- Smaller output embedding size.
- Better general accuracy.
- Faster training time.
- Reduced memory for extracted features.

### Multi-Label Image Classifier **[NEW]**
A new classifier task that predicts a *set* of labels per image, not just one. Contrasted with:
- **Single-label classifier**: picks the single best label.
- **Object detector**: locates objects with bounding boxes.
- **Multi-label classifier**: assigns multiple labels (objects, attributes, scene descriptors) to the whole image.

**Training data format**: JSON file where each entry maps a filename to an array of label strings. Single-label images are valid training examples. **Evaluation**: uses **Mean Average Precision (MAP)** score across all labels. The Create ML app shows per-class metrics: Precision, Recall, False Positives, False Negatives, Confidence Threshold.

**Inference code pattern**: create a `VNCoreMLModel` from the compiled `.mlmodel`, use `VNImageRequestHandler`, retrieve `VNClassificationObservation` results, filter by confidence threshold.

**Object detection evaluation**: the existing object detector also received an enhanced evaluation tab with an Explore mode (parallel to the multi-label classifier's new interactive evaluation).

### Custom Augmentation API (Create ML Components) **[NEW]**
New `Augmenter` type with result builder syntax (similar to SwiftUI's `ViewBuilder`) for composing custom training data augmentation pipelines.

**Built-in transformers include**: horizontal flip, vertical flip, crop, rotate, contrast adjustment, and more.

**Custom augmentations**: conform to `RandomTransformer` protocol (takes a random number generator as input). Implement `applied(to:using:)` to produce a transformed output.

**Training with augmentations best practice**:
1. Use `model.update(_:)` (not `model.fit(_:)`) in a loop.
2. Shuffle training data before each epoch so batches contain different images.
3. Augmentations produce an `AsyncSequence`; use `.batches(ofSize:)` to group them.
4. Add **early stopping**: compute validation metrics after each update and stop when validation accuracy stops improving (e.g., no improvement for 5 consecutive iterations).

## APIs & Frameworks

### Create ML App
- **Multi-Label Image Classifier** template **[NEW]**
- **BERT embedding** option in Text Classifier model parameters **[NEW]**
- **New Neural Scene Analyzer feature extractor** in Image Classifier model parameters **[NEW]**
- Interactive evaluation with Explore mode for multi-label and object detection tasks **[NEW]**
- MAP score display in training progress and evaluation metrics **[NEW]**

### Create ML Framework
- `MLImageClassifier` — single-label image classifier (updated feature extractor)
- `MLMultiLabelImageClassifier` — **new** multi-label image classifier **[NEW]**
- `MLTextClassifier` — text classifier (updated with BERT multilingual embedding) **[NEW embedding]**

### Create ML Components Framework **[NEW APIs]**
- `Augmenter` — result-builder type for composing transformation pipelines **[NEW]**
- `ApplyRandomly` — applies a nested transformer with a given probability **[NEW]**
- `RandomTransformer` protocol — protocol for custom random transformations **[NEW]**
- `UniformRandomFloatingPointParameter` — generates a random float in a range each application **[NEW]**
- `HorizontalFlip` — horizontal flip transformer
- `RandomCrop` — random crop transformer
- `RandomRotation` — random rotation transformer
- `Augmenter.applied(to:)` — returns an `AsyncSequence` of augmented samples
- `AsyncSequence.batches(ofSize:)` — groups async sequence into batches for training
- Estimator/Transformer `.update(_:)` method — incremental model update (preferred for augmented training loops)

### Vision / Core ML (Inference)
- `VNCoreMLModel` — wraps compiled `.mlmodel` for Vision inference
- `VNImageRequestHandler` — runs Vision requests on an image
- `VNClassificationObservation` — per-label result with `identifier` and `confidence`
- Confidence threshold filtering — each label has an independently tuned threshold

## Code Highlights

```swift
// Augmenter using result builder
let augmenter = Augmenter<AnnotatedImage> {
    ApplyRandomly(probability: 0.5) {
        HorizontalFlip()
    }
    RandomRotation(angleRange: -30...30)
    RandomCrop(minRatio: 0.7, maxRatio: 1.0)
}

// Training loop with augmentation and early stopping
var model = ImageClassifier()
var bestAccuracy = 0.0
var noImprovementCount = 0

for _ in 0..<100 {
    let shuffled = trainingData.shuffled()
    let augmented = augmenter.applied(to: shuffled)
    for batch in augmented.batches(ofSize: 16) {
        try await model.update(batch)
    }
    let validationAccuracy = model.evaluation(on: validationData).accuracy
    if validationAccuracy > bestAccuracy {
        bestAccuracy = validationAccuracy
        noImprovementCount = 0
    } else {
        noImprovementCount += 1
        if noImprovementCount >= 5 { break }
    }
}
```

## Takeaways
- The new BERT embedding gives text classifiers multilingual capability and better accuracy without changing training data format.
- The multi-label image classifier fills a gap between single-label classification and object detection — ideal for scene understanding where multiple labels apply to the whole image.
- Use the Create ML Components `Augmenter` with `RandomTransformer` conformance to build custom augmentation pipelines that dramatically improve model quality on small datasets.
- Always shuffle before augmenting, use `update()` in a training loop (not `fit()`), and implement early stopping on validation metrics.

---
_Source: WWDC23 Session 10044 page (abstract, chapter summaries, code samples, and resource links)._
