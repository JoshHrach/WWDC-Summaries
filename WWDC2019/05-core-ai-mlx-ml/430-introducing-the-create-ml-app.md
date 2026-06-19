# Introducing the Create ML App
**WWDC19 · Session 430** · [Watch](https://developer.apple.com/videos/play/wwdc2019/430/)

_Platforms:_ macOS Catalina 10.15

## Overview
The Create ML app, introduced at WWDC19, provides a completely new graphical workflow for training machine learning models on the Mac without writing any code. It expands on the previous Swift Playgrounds/script-based approach by offering an intuitive template-driven UI covering nine model types across five input domains: Images, Sound, Activity, Text, and Tabular Data.

The app guides developers through three sequential phases — input, training, and output — displaying live accuracy graphs, precision/recall breakdowns per class, and an interactive preview tab where trained models can be tested directly before integration into an app. Continuity Camera support allows importing live photos from an attached iPhone for real-time testing.

Developers can create multiple training experiments per project, tweak augmentation settings, adjust validation and testing data splits, and compare runs. Finished models are small Core ML files that can be dragged directly into Xcode, replacing larger generic models with task-specific, on-device models.

## Key Topics
- **Template selection** — nine templates spanning Image Classifier, Object Detector, Sound Classifier, Activity Classifier, Text Classifier, Word Tagger, Tabular Classifier, Tabular Regressor, and Recommender
- **Live training metrics** — accuracy-vs-iterations chart, per-class precision and recall table with interactive filtering
- **Model Preview tab** — drag-and-drop images, audio, or text to see predictions and confidence scores without writing code
- **Continuity Camera integration** — capture photos from an iPhone directly into Create ML for live preview
- **Augmentation controls** — toggle image/audio augmentations to improve model robustness
- **Experiment management** — multiple runs per project with configurable data splits and algorithm options
- **Transfer learning** — Image Classifier and Sound Classifier both use transfer learning (Vision Feature Print and audio feature extraction) for faster training and smaller output models

## APIs & Frameworks
- **Create ML** framework (underlying engine) **[NEW in macOS Catalina]**
  - `MLImageClassifier` **[NEW]**
  - `MLObjectDetector` **[NEW]**
  - `MLSoundClassifier` **[NEW]**
  - `MLActivityClassifier` **[NEW]**
  - `MLTextClassifier`
  - `MLWordTagger`
  - `MLDataTable`
  - `MLClassifier` (tabular) **[NEW automatic algorithm selection]**
  - `MLRegressor` (tabular) **[NEW automatic algorithm selection]**
  - `MLRecommender` **[NEW]**
- **Core ML** — `.mlmodel` output consumed by apps
- **Vision** — `VisionFeaturePrint` model used internally for image transfer learning
- **CreateML app** — new standalone macOS application (launched from Xcode > Developer Tools)

## Code Highlights
This session focuses on the GUI app; no code authoring is required. The workflow is entirely drag-and-drop within the Create ML app. The resulting `.mlmodel` file is dragged into an Xcode project and used via the standard Core ML API:

```swift
// After dragging FlowerClassifier.mlmodel into Xcode:
let model = try FlowerClassifier(configuration: MLModelConfiguration())
let prediction = try model.prediction(image: pixelBuffer)
print(prediction.classLabel) // e.g. "hibiscus"
```

## Takeaways
- Create ML is now a full Mac app with GUI-based model training — no Swift code needed.
- Nine templates cover the five domains: Images, Sound, Activity, Text, and Tabular Data.
- Transfer learning keeps training fast and output models small (e.g., 66 KB image classifier vs. 100 MB generic model).
- The Preview tab enables immediate end-to-end testing of trained models before Xcode integration.

---
_Source: WWDC19 Session 430 page (abstract, chapter summaries, code samples, and resource links)._
