# What's New in Create ML
**WWDC24 · Session 10183** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10183/)

_Platforms:_ macOS Sequoia 15, visionOS 2

## Overview
Create ML gains three major additions: interactive data source previews for visualizing and debugging annotations before training, a brand-new Spatial category with an Object Tracking template for building visionOS anchor experiences, and new Create ML Components for general-purpose time-series forecasting and classification with a `DateFeatureExtractor` to harness temporal patterns in data.

The ecosystem consists of three layers: the Create ML app (click-to-train, no code), the Create ML framework (automate training, on-device personalization), and underlying domain frameworks (Vision, Natural Language, Sound Analysis) that the models deploy into.

## Key Topics

### App Enhancements: Data Source Preview
The Create ML app's data source panel now has an "Explore" option that drills into a specific object or class label and visualizes all annotations for it. For image-based templates (object detection, image classification, hand pose classification), you can inspect bounding boxes overlaid on each training image. This helps catch annotation inconsistencies — e.g., some images annotating the surface of a coffee cup vs. the whole cup — before training begins.

### Object Tracking Template (New Spatial Category)
A brand-new "Spatial" category appears in the Create ML app template chooser, containing an **Object Tracking** template. To train: import a USDZ 3D asset of your object, configure a viewing angle (All Angles, Upright, or Front), and click Train. Training runs entirely locally on Apple Silicon Mac. The output is a `.referenceobject` file that integrates with ARKit and RealityKit on visionOS for real-time 3D tracking. The full workflow is covered in "Explore object tracking for visionOS" (WWDC24 Session 10101).

### Time-Series Forecasting (New Component)
`TimeSeriesForecaster` is a new general-purpose Create ML component that learns from historical numerical data to predict future values over a configurable window. Key parameters: `inputWindowSize` (historical context) and `forecastWindowSize` (prediction horizon). The forecaster is trained with the `fitted(_:)` method and evaluated with `applied(_:)`. Use case in the session: predicting food truck sales per day from transaction history, enabling on-device personalization per individual food truck.

### Time-Series Classification (New Component)
A new general-purpose `TimeSeriesClassifier` component supplements the existing Activity Classification support. It answers "what does this data represent?" — suitable for gesture classification from accelerometer data (pinch, snap, clench), audio pattern recognition, and other applications not covered by the Activity Classification template.

### DateFeatureExtractor Component
`DateFeatureExtractor` extracts temporal components from a date column to feed the time-series forecaster with rich features: day of week (weekly patterns), month of year (seasonal patterns), and others. It composes with `ColumnSelector` and `ColumnConcatenator` in a pipeline, and is fitted to the training data frame to produce a reusable preprocessor.

### Tabular Data Preprocessing
The session uses the Tabular Data framework to group transactions by date and sum quantities before passing to the forecaster. Demonstrates composing multiple Create ML Components into a preprocessing pipeline.

## APIs & Frameworks

**Create ML App**
- Data source "Explore" view **[NEW]** — interactive annotation visualization for image-based templates
- Spatial template category **[NEW]**
- Object Tracking template **[NEW]** — produces `.referenceobject` output file

**Create ML Components** (`CreateMLComponents`)
- `TimeSeriesForecaster` **[NEW]** — general-purpose time-series prediction component
  - `inputWindowSize: Int` — historical context window
  - `forecastWindowSize: Int` — prediction horizon
  - `fitted(_:)` — trains the component
  - `applied(_:)` — generates forecast predictions
- `TimeSeriesClassifier` **[NEW]** — general-purpose time-series classification component
- `DateFeatureExtractor` **[NEW]** — extracts date components (weekday, month, etc.) as ML features
  - `DateFeatureExtractor(components:)` init with `.month`, `.weekday`, etc.
- `ColumnSelector` — selects specific columns from a data frame (existing, used in pipeline)
- `ColumnConcatenator` — combines multiple columns into a shaped array (existing, used in pipeline)
- `MLShapedArray` — output type for feature and target columns used in time-series training (existing)

**Tabular Data Framework**
- Group-by and aggregation operations on `DataFrame` — used to preprocess transaction data
- Column extraction to `MLShapedArray`

**ARKit / RealityKit (consumption side)**
- `ReferenceObject` — loads the `.referenceobject` file produced by the Object Tracking template
- `ObjectTrackingProvider` — ARKit data provider that uses trained reference objects

## Code Highlights

DateFeatureExtractor pipeline for preprocessing sales data:
```swift
let featureExtractor = DateFeatureExtractor(components: [.month, .weekday])
let pipeline = ColumnSelector(columns: ["Date"])
    .appending(featureExtractor)
    .appending(ColumnConcatenator())
let preprocessor = try pipeline.fitted(on: dataFrame)
let features = preprocessor.applied(to: dataFrame)["features"] as! Column<MLShapedArray<Float>>
let targets = dataFrame["Quantity"] as! Column<MLShapedArray<Float>>
```

Training and inference with the time-series forecaster:
```swift
var forecaster = TimeSeriesForecaster()
forecaster.inputWindowSize = 15   // 15 days of historical context
forecaster.forecastWindowSize = 3  // predict 3 days ahead

let trainedForecaster = try forecaster.fitted(to: trainingFeatures, targets: trainingTargets,
                                               validateOn: validationFeatures,
                                               validationTargets: validationTargets)
let predictions = trainedForecaster.applied(to: latestFeatures)
```

## Takeaways
- Use the new "Explore" data source view before every training run to catch annotation inconsistencies; bad labels are the most common cause of poor model performance.
- The Object Tracking template requires only a USDZ file and a viewing angle choice — no code, no custom data collection — to produce a deployment-ready `.referenceobject`.
- Time-series forecasting requires `inputWindowSize` to exceed `forecastWindowSize`; the `DateFeatureExtractor` is essential for giving the model temporal context (weekly/seasonal patterns).
- The `TimeSeriesForecaster` is ideal for on-device personalization: train on each user's data locally to produce forecasts that are specific to their patterns.

---
_Source: WWDC24 Session 10183 page (abstract, chapter summaries, and resource links)._
