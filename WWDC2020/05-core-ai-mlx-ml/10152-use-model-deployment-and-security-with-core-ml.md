# Use Model Deployment and Security with Core ML
**WWDC20 · Session 10152** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10152/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, tvOS 14, watchOS 7

## Overview
iOS 14 and Xcode 12 introduce two major new capabilities around Core ML model lifecycle management: Core ML Model Deployment and model encryption. Previously, updating a model required shipping a new app binary. Core ML Model Deployment breaks this dependency by allowing developers to store, manage, and push updated models to user devices via Apple cloud, independent of any App Store update. Models are grouped into versioned "model collections" and delivered atomically—ensuring that interdependent models stay in sync across deployments. Targeted deployments add rule-based device filtering (by device class, OS version, etc.) so that population-specific models are only sent to the devices that need them.

Model encryption closes the gap between model confidentiality and ease of use. When enabled, the compiled `.mlmodelc` is encrypted at rest both in the app bundle and in Apple cloud storage. Core ML handles key management transparently: on first load it fetches the decryption key from Apple cloud and caches it locally; subsequent loads work offline. Encryption is enabled either via an Xcode build phase compiler flag (`--encrypt path/to/key`) or automatically when generating a `.mlarchive` if a `.mlmodelkey` file is present alongside the model. The session also covers new Xcode UI—a Utilities tab with model details, interactive model preview for a wide variety of task types, and first-class Core ML support in Xcode Playgrounds.

## Key Topics
- **Core ML Model Deployment** — deliver models outside the app binary; manage via Apple cloud dashboard **[NEW]**
  - Independent model update cycle from app updates
  - Model collections — group related models; atomic/versioned delivery
  - Targeted deployments — rule-based filtering by device class, OS version, etc.
- **`MLModelCollection`** — new API to access a deployed model collection **[NEW]**
  - `beginAccessing(identifier:completionHandler:)` — downloads models on first call; registers for future updates
  - `MLModelCollectionEntry.modelURL` — URL to the compiled model on disk
- **Model encryption** — `.mlmodelkey` file created by Xcode; compiled model encrypted at rest and in transit **[NEW]**
  - Key generated via Xcode Utilities tab → "Create Encryption Key"
  - Encrypt bundled models: `--encrypt path/to/key` compiler flag in Build Phases > Compile Sources
  - Encrypt model archives: Xcode pre-selects adjacent `.mlmodelkey` when creating `.mlarchive`
  - Key fetched from Apple cloud on first `load`; cached locally for offline use afterward
- **`MLModel.load(_:completionHandler:)` / type-safe `.load(_:)`** — new asynchronous load method replacing default initializer **[RECOMMENDED]**; handles key fetch; returns `Result<T, Error>`
- **`MLModelError.modelKeyFetch`** — error case for when key retrieval fails (no network); handle gracefully
- **Model archive (`.mlarchive`)** — packaged form for upload to Model Deployment dashboard; created from Xcode Utilities tab
- **Xcode 12 Core ML UI enhancements**:
  - Utilities tab — OS version support, class labels, neural network details
  - Interactive model preview — test image segmentation, pose detection, depth estimation, image classification, etc. directly in Xcode
  - Xcode Playgrounds integration — drag model into Resources; same auto-generated class interface as Xcode project

## APIs & Frameworks

**Core ML**
- `MLModelCollection` **[NEW]** — manages a deployed model collection
  - `static func beginAccessing(identifier:completionHandler:) -> Progress` — begins access/download; calls handler asynchronously with `Result<MLModelCollection, Error>`
  - `entries: [String: MLModelCollectionEntry]` — dictionary keyed by model name
- `MLModelCollectionEntry` **[NEW]** — entry for one model within a collection
  - `modelURL: URL` — URL to the compiled model in the app container
- `MLModel.load(contentsOf:configuration:completionHandler:)` — general async load **[NEW/RECOMMENDED]**
- Generated model class `.load(_:)` — e.g., `FlowerClassifier.load { result in ... }` **[NEW/RECOMMENDED]**; replaces `FlowerClassifier(contentsOf:)` / `FlowerClassifier(configuration:)`
- `MLModelError` — error type for Core ML failures
  - `MLModelError.modelKeyFetch` **[NEW]** — key fetch failure; indicates no network connectivity
- `MLModelConfiguration` — existing configuration object for compute units, batch size, etc.

**Xcode 12 tooling**
- Utilities tab on `.mlmodel` file — model deployment section, encryption section, interactive preview **[NEW]**
- "Create Encryption Key" button — generates `.mlmodelkey` file, stores key on Apple cloud **[NEW]**
- "Create Model Archive" button — generates `.mlarchive` for upload; auto-encrypts if `.mlmodelkey` present **[NEW]**
- Build Phases > Compile Sources > Compiler Flags: `--encrypt "path/to/file.mlmodelkey"` — encrypt bundled model at build time **[NEW]**
- Model Deployment dashboard — web UI to create collections, upload archives, configure targeting rules **[NEW]**

## Code Highlights

Opt in to Model Deployment and load from collection:
```swift
private func classifyFlower(in image: CGImage) {
    if let model = flowerClassifier {
        classify(image, using: model)
        return
    }

    MLModelCollection.beginAccessing(identifier: "FlowerModels") { [self] result in
        var modelURL: URL?
        switch result {
        case .success(let collection):
            modelURL = collection.entries["FlowerClassifier"]?.modelURL
        case .failure(let error):
            handleModelCollectionFailure(for: error)
        }
        let result = loadFlowerClassifier(from: modelURL)
        switch result {
        case .success(let model):
            flowerClassifier = model
            classify(image, using: model)
        case .failure(let error):
            handleModelLoadFailure(for: error)
        }
    }
}
```

Load an encrypted bundled model asynchronously:
```swift
FlowerStylizer.load { [self] result in
    switch result {
    case .success(let model):
        flowerStylizer = model
        DispatchQueue.main.async { applyStyledEffect(using: model) }
    case .failure(let error):
        switch error {
        case MLModelError.modelKeyFetch:
            handleNetworkFailure()  // key not yet fetched, needs connectivity
        default:
            handleModelLoadError(error)
        }
    }
}
```

Compiler flag for encrypting a bundled model (in Build Phases > Compile Sources):
```
--encrypt "$SRCROOT/MyApp/Models/MyModel.mlmodelkey"
```

## Takeaways
- Core ML Model Deployment decouples model updates from app releases; use `MLModelCollection.beginAccessing(identifier:)` to receive downloaded models, and always ship a bundled fallback model for the first-launch / offline case.
- Group models that belong to the same feature into a single collection so they are delivered and updated atomically; the collection identifier in code must match the name on the dashboard.
- Model encryption requires only: (1) generate key in Xcode Utilities tab, (2) add `--encrypt` compiler flag or create a model archive, (3) call `.load` instead of the default initializer; Core ML handles key fetch and caching automatically.
- Always switch from the deprecated synchronous initializer to the new asynchronous `.load` method, especially for encrypted models that require a first-time network key fetch; trap `MLModelError.modelKeyFetch` specifically to show the user a meaningful offline error.

---
_Source: WWDC20 Session 10152 page (abstract, chapter summaries, code samples, and resource links)._
