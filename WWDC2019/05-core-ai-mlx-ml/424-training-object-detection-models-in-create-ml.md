# Training Object Detection Models in Create ML
**WWDC19 · Session 424** · [Watch](https://developer.apple.com/videos/play/wwdc2019/424/)

_Platforms:_ macOS Catalina 10.15

## Overview
Create ML in macOS Catalina adds an Object Detection template, enabling developers to train custom Core ML models that identify multiple objects in a scene — complete with bounding boxes and labels — directly inside the Create ML app without writing any code.

Unlike image classification, which assigns a single label to an entire image, object detection locates every instance of each class within a photograph. This makes it ideal for applications that need to reason about the position, size, and count of real-world objects captured by a device camera.

The session walks through collecting and annotating training data, configuring the Create ML app for object detection, evaluating model performance via a live loss graph and per-class accuracy metrics, and testing results in the app with Continuity Camera before exporting a `.mlmodel` file ready for use in Xcode.

## Key Topics

**Image Annotation Format**
Training data for object detection must pair images with a JSON annotation file. Each annotation specifies a label, and a bounding box as center-x, center-y, width, and height — measured in pixels from the top-left corner of the image.

**Create ML App Workflow**
After selecting the "Object Detector" template, developers drag a folder containing images and the companion JSON file into the training-data well. The app validates the dataset (checking image format and JSON structure) and displays initial statistics such as class count and image count before training begins.

**Training and Evaluation**
Training is initiated with a single click. A live loss graph shows convergence over time. The Evaluation tab shows overall accuracy (the demo model reached 92%) and per-class breakdowns, ensuring balanced performance across all classes — critical for use cases like fair dice-rolling games.

**Continuity Camera Testing**
The Create ML app supports importing test photos directly from an iPhone via Continuity Camera, using the same optics the final app will use — providing a realistic end-to-end test without leaving the desktop.

**Training Data Best Practices**
- Aim for at least 30 annotated images per class; more complex subjects require more images.
- Balance the number of annotations across all classes.
- Capture data under varied conditions (lighting, background, angle) to help the model generalize.
- Only label the objects you want to detect; unlabeled regions are implicitly treated as negatives.

## APIs & Frameworks

**Create ML (macOS)**
- `MLObjectDetector` **[NEW]** — Core ML model type produced by the Object Detection template
- Create ML app Object Detector template **[NEW]**
- JSON annotation format for bounding boxes (label, x, y, width, height in pixels)
- Training parameters panel (iteration count, augmentations)
- Continuity Camera import **[NEW]** — live photo capture from iPhone into Create ML for testing

**Core ML**
- `.mlmodel` output artifact from Create ML, importable into Xcode projects

**Vision Framework**
- `VNRecognizedObjectObservation` — bounding-box and label output when running an object-detection Core ML model via Vision
- `VNCoreMLRequest` — recommended integration path for live camera and video pipelines

## Code Highlights

Annotation JSON structure expected by Create ML:

```json
[
  {
    "image": "dice_001.jpg",
    "annotations": [
      { "label": "3", "coordinates": { "x": 120, "y": 85, "width": 60, "height": 60 } },
      { "label": "5", "coordinates": { "x": 240, "y": 90, "width": 58, "height": 58 } }
    ]
  }
]
```

No Swift API code was shown; the session is entirely GUI-driven through the Create ML app.

## Takeaways
- The Create ML app's Object Detector template lets you train production-quality Core ML models with zero code, using only annotated images and a JSON file.
- Per-class accuracy metrics and Continuity Camera testing inside the app make it easy to catch distribution mismatches before shipping.
- Balance and diversity in training data (varied lighting, backgrounds, object varieties) are the primary levers for improving model generalization.
- Use the Vision framework's `VNCoreMLRequest` / `VNRecognizedObjectObservation` pipeline to integrate the exported model with live camera feeds in your app.

---
_Source: WWDC19 Session 424 page (abstract, transcript, and resource links)._
