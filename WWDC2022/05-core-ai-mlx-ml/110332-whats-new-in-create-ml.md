# What's New in Create ML
**WWDC22 · Session 110332** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110332/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, tvOS 16

## Overview
Create ML receives significant updates to both the app and the framework in 2022. In the Create ML app, new interactive evaluation tools let developers visually explore model performance on test data, drill into mislabeled examples, understand top confusions, and preview models live using Continuity Camera on macOS Ventura. These improvements help developers understand not just accuracy metrics but specific strengths and weaknesses in their training data.

On the framework side, Create ML gains tvOS 16 support and introduces a new member of the family — Create ML Components — which exposes the underlying building blocks for creating custom ML pipelines. A highlight new runtime capability is **Repetition Counting**, a class-agnostic API for counting repeated human body actions in live video, requiring no training data.

The Create ML Components framework unlocks deep customization beyond predefined task templates, and combined with the new repetition counting capability, opens up fitness, dance, sports, and other use cases that were previously difficult to implement.

## Key Topics

### Interactive Model Evaluation (Create ML App)
A redesigned Evaluation pane includes a high-level metrics summary and a new interactive Explore tab. Developers can filter test results by correct/incorrect predictions, explore per-class precision/recall, and visually browse individual images with prediction details. This is available for image classifier, hand pose classifier, and object detection templates.

### Live Preview with Continuity Camera
Live preview is expanded to image classifier, hand action classifier, and body action classifier templates. Developers can now select any attached webcam and use iPhone as a Continuity Camera on macOS Ventura for real-time model testing directly in the app.

### Create ML Components Framework (New)
A new framework exposing the underlying building blocks of Create ML. Developers can compose and pipeline these components to create custom ML models beyond the predefined templates. Enables async temporal components and custom training pipelines.

### Repetition Counting (New Runtime API)
A new runtime capability in Create ML Components that counts repeated full-body actions in live video. Class-agnostic (no training required), based on a pretrained model. Can be combined with Action Classification to simultaneously count and categorize repetitions. Available on iOS, iPadOS, and macOS.

### tvOS 16 Support
The Create ML framework now supports tvOS 16, including tabular classifiers and regressors. Video-based tasks still require macOS due to data/computation requirements.

## APIs & Frameworks

**Create ML (CreateML)**
- `MLImageClassifier` — image classification model training
- `MLObjectDetector` — object detection model training
- `MLHandPoseClassifier` — hand pose classification
- `MLActionClassifier` — action classification from video
- `MLBodyActionClassifier` — body action classification

**Create ML Components (CreateMLComponents)** **[NEW]**
- `CreateMLComponents` framework — exposes compositional ML building blocks **[NEW]**
- `RepetitionCounting` / repetition counting API — class-agnostic body action repetition counter **[NEW]**
- Temporal components — async components for video/time-series processing **[NEW]**
- Composable pipeline APIs — combine components into custom training/inference pipelines **[NEW]**

**Create ML App Features**
- Evaluation Explore tab **[NEW]** — interactive per-image exploration of test results
- Metrics summary with expandable per-class statistics **[NEW enhanced]**
- Live Preview with Continuity Camera support **[NEW]** — macOS Ventura + iPhone as webcam
- Support for external webcam selection in Live Preview **[NEW]**

## Code Highlights

Repetition counting requires no training data and can be implemented in a few lines using the Create ML Components framework. See the Apple sample code "Counting human body action repetitions in a live video feed" linked from this session.

## Takeaways

- The new interactive Evaluation Explore tab makes it fast to visually identify mislabeled data, underrepresented classes, and edge cases that statistics alone would miss.
- Create ML Components is a major new framework exposing compositional ML primitives — use it when predefined tasks don't fit your use case.
- Repetition Counting is a pretrained, class-agnostic runtime API: no training data needed to count full-body repetitive actions like jumping jacks, squats, or dance moves.
- Create ML framework now supports tvOS 16; video-based tasks remain macOS-only due to compute requirements.

---
_Source: WWDC22 Session 110332 page (abstract, chapter summaries, code samples, and resource links)._
