# Get to know Create ML Components
**WWDC22 · Session 10019** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10019/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, tvOS 16

## Overview
Create ML Components is a brand-new Swift framework introduced in 2022 that exposes the building blocks of Create ML tasks as individual, composable types. Rather than picking a fixed task template (image classification, sound classification, etc.), developers can now mix and match feature extractors, transformers, and estimators — and write their own — to construct entirely new ML pipelines.

The session covers the two core protocol abstractions (Transformer and Estimator), demonstrates how to build a custom image regressor from scratch, explores tabular tasks using the TabularData framework, and discusses deployment strategies (native task parameters vs. Core ML export).

## Key Topics

**Transformers vs. Estimators** — A `Transformer` maps an input type to an output type with no training (e.g., `ImageFeaturePrint`, `SaliencyCropper`). An `Estimator` learns from labeled data and produces a `Transformer` via its `fitted(to:validateOn:eventHandler:)` method (e.g., `LogisticRegressionClassifier`, `LinearRegressor`, `BoostedTreeRegressor`). Components are composed via the `appending(_:)` method; output type of step N must match input type of step N+1.

**Image tasks** — The classic image classifier is `ImageFeaturePrint().appending(LogisticRegressionClassifier())`. Switching to regression only requires replacing the classifier with `LinearRegressor()`. A custom image regressor is built in ~30 lines of Swift: load annotated files with `AnnotatedFiles`, map to `CIImage` with `ImageReader.read`, map annotation strings to `Float`, split with `randomSplit`, call `fitted`, and save with `write`.

**Data augmentation** — Multiply training examples by passing a `flatMap` augmentation function to the data pipeline before `fitted`. Augmentations apply at training time only; they do not affect inference. The `AnnotatedFeature<Feature, Annotation>` type holds feature + label pairs.

**Custom Transformers** — Conform to the `Transformer` protocol by implementing a single `applied(to:eventHandler:)` method. Custom transformers can be inserted into the composition chain and are applied at both training and inference time, which guarantees consistency.

**Tabular tasks** — Use `ColumnSelector` to apply per-column preprocessing (e.g., `StandardScaler`, `OptionalUnwrapper`, `OneHotEncoder`) and then append a `BoostedTreeRegressor` or `BoostedTreeClassifier`. Training data is provided as `DataFrame` (TabularData framework). `ColumnID<T>` identifies the annotation column for predictions.

**Deployment** — Two strategies: (1) bundle the composed task definition + trained parameters file (`.pkg`) together, optionally in a Swift package; or (2) export to Core ML via `exportToCore ML(url:)` for optimized tensor operations. Core ML export does not support custom transformers/estimators or non-standard types.

## APIs & Frameworks

### CreateMLComponents (new framework in iOS 16 / macOS 13)

**Protocols**
- `Transformer` **[NEW]** — single-input-to-output mapping protocol; implement `applied(to:eventHandler:)`
- `Estimator` **[NEW]** — learns from data; produces a `Transformer` via `fitted(to:validateOn:eventHandler:)`
- `SupervisedEstimator` **[NEW]** — estimator variant requiring annotated training data
- `SupervisedTabularEstimator` **[NEW]** — tabular variant of supervised estimator

**Transformers (built-in)**
- `ImageFeaturePrint` **[NEW]** — Vision feature extractor for images; produces `MLShapedArray<Float>`
- `SaliencyCropper` **[NEW]** — crops `CIImage` to salient object using Vision attention saliency
- `ImageReader` **[NEW]** — reads `URL` to `CIImage`; use `.read` as a function reference
- `OptionalUnwrapper<T>` **[NEW]** — unwraps `Optional<T>` values from DataFrame columns
- `StandardScaler<T>` **[NEW]** — normalizes numeric values to mean=0, std=1
- `OneHotEncoder` **[NEW]** — encodes categorical string values as binary arrays
- `OrdinalEncoder` **[NEW]** — encodes categorical string values as consecutive integers

**Estimators / Supervised Estimators**
- `LogisticRegressionClassifier<Label>` **[NEW]** — multi-class classifier; output is `ClassificationDistribution<Label>`
- `FullyConnectedNetworkClassifier` **[NEW]** — neural-network-based classifier
- `LinearRegressor` **[NEW]** — linear regression estimator; output is `Float`
- `BoostedTreeRegressor<AnnotationColumn>` **[NEW]** — gradient-boosted tree regressor for tabular data
- `BoostedTreeClassifier<AnnotationColumn>` **[NEW]** — gradient-boosted tree classifier for tabular data

**Data loading / annotation**
- `AnnotatedFeature<Feature, Annotation>` **[NEW]** — feature + annotation pair
- `AnnotatedFiles` **[NEW]** — loads annotated `URL` collection from file system
  - `init(labeledByNamesAt:separator:index:type:)` — annotate by filename component
  - `init(labeledByDirectoriesAt:)` — annotate by containing directory name
- `mapFeatures(_:)` **[NEW]** — transform the feature side of an annotated collection
- `mapAnnotations(_:)` **[NEW]** — transform the annotation side of an annotated collection
- `randomSplit(by:)` **[NEW]** — split a collection into training and validation fractions

**Column selection (tabular)**
- `ColumnSelector<Columns, Estimator>` **[NEW]** — apply an estimator to specific DataFrame columns
- `ColumnID<T>` — column identifier with type; from TabularData framework

**Composition**
- `appending(_:)` **[NEW]** — compose two components; output type of first must match input type of second

**Persistence**
- `Estimator.write(_:to:)` **[NEW]** — save trained parameters to a file URL
- `Estimator.read(from:)` **[NEW]** — load trained parameters from a file URL

**Metrics**
- `meanAbsoluteError(_:_:)` **[NEW]** — compute MAE between predictions and ground truth
- Training event `.metrics` dictionary keys: `.trainingMaximumError`, `.validationMaximumError`, `.validationError`

### TabularData
- `DataFrame` — in-memory tabular data structure; used as input to tabular estimators
- `DataFrame.randomSplit(by:)` — split DataFrame into train/validation

### Vision (referenced)
- `VNGenerateAttentionBasedSaliencyImageRequest` — used internally by `SaliencyCropper`

### Core ML
- `MLShapedArray<Scalar>` — output type of `ImageFeaturePrint`
- Core ML export from Create ML Components — `exportToCoreML(url:)` (unsupported for custom transformers)

## Code Highlights

Custom image regressor (core training loop):
```swift
import CoreImage
import CreateMLComponents

struct ImageRegressor {
    static let trainingDataURL = URL(fileURLWithPath: "~/Desktop/bananas")
    static let parametersURL   = URL(fileURLWithPath: "~/Desktop/parameters")

    static func train() async throws -> some Transformer<CIImage, Float> {
        let estimator = SaliencyCropper()
            .appending(ImageFeaturePrint())
            .appending(LinearRegressor())

        let data = try AnnotatedFiles(labeledByNamesAt: trainingDataURL,
                                      separator: "-", index: 1, type: .image)
            .mapFeatures(ImageReader.read)
            .mapAnnotations { Float($0)! }
            .flatMap(augment)

        let (training, validation) = data.randomSplit(by: 0.8)
        let transformer = try await estimator.fitted(to: training, validateOn: validation) { event in
            if let err = event.metrics[.validationMaximumError] {
                print("Validation max error: \(err)")
            }
        }
        try estimator.write(transformer, to: parametersURL)
        return transformer
    }
}
```

Tabular regressor with ColumnSelector:
```swift
static var task: some SupervisedTabularEstimator {
    let numeric = ColumnSelector(
        columns: ["volume"],
        estimator: OptionalUnwrapper().appending(StandardScaler<Double>())
    )
    let regression = BoostedTreeRegressor<String>(
        annotationColumnName: "price",
        featureColumnNames: ["type", "region", "volume"]
    )
    return numeric.appending(regression)
}
```

## Takeaways
- Create ML Components exposes every step of built-in Create ML tasks as composable `Transformer` and `Estimator` values; combine them with `appending(_:)` to build novel ML pipelines.
- Building a custom task (e.g., image regression) requires only substituting one component (replace `LogisticRegressionClassifier` with `LinearRegressor`) — no architecture code needed.
- Custom `Transformer` conformances automatically apply at both training and inference time, guaranteeing consistency between the two.
- For tabular data, `ColumnSelector` applies per-column preprocessing (scaling, encoding) before passing to tree-based estimators; use `StandardScaler` for numeric and `OneHotEncoder` for low-cardinality categorical columns.

---
_Source: WWDC22 Session 10019 page (abstract, transcript, code samples, and resource links)._
