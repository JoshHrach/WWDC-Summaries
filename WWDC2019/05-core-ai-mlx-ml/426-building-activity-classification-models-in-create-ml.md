# Building Activity Classification Models in Create ML
**WWDC19 · Session 426** · [Watch](https://developer.apple.com/videos/play/wwdc2019/426/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, watchOS 6

## Overview
Create ML gains a brand-new template in 2019: Activity Classification. This model type uses motion sensor data (accelerometer, gyroscope) from iPhone and Apple Watch to recognize a developer-defined set of physical actions in real time. The session walks through the full pipeline — data collection with Core Motion, training in the Create ML app or via the Swift framework, per-class evaluation, and on-device inference — using a frisbee-throw classifier as the live demonstration.

The underlying model is a deep learning neural network that uses a sliding window over the time-series sensor data to extract both spatial and temporal motion patterns. The resulting Core ML model is small (the demo model is 1.1 MB) and runs efficiently on-device, including on Apple Watch.

## Key Topics

**What Activity Classification Does**
- Recognizes a user-defined set of physical actions from continuous motion sensor data
- Works with data from accelerometer, gyroscope, device motion (sensor fusion), and other Core Motion sources
- Applicable to sports detection, gesture/game-control recognition, posture analysis, exercise form feedback

**Data Collection**
- Use Core Motion framework to record sensor data; same APIs used for both training data capture and on-device inference
- Data organized as files in per-class folders (folder name = label); supports CSV, JSON, and plain text
- Include a "no activity" / "other" class for robust real-world runtime behavior
- Balance classes by both number of recording files and total recording duration

**Training in Create ML App**
- Select the Activity Classifier template; drag-and-drop training data folder
- Choose `selectedSensors` from the headers of the CSV files (e.g., `userAcceleration`, `rotationRate`)
- Set `predictionWindowSize` based on activity speed; example: 100 samples for 50 Hz data = 2-second window
- Training, evaluation, and model export follow the same flow as other Create ML templates

**Evaluation**
- Per-class precision and recall matrix in the Testing tab
- Batch prediction on unlabeled data directly in Create ML app with label, confidence, and data preview

**Programmatic API (Create ML framework)**
- `MLActivityClassifier` in the Create ML framework
- Specify `selectedSensors` as a required parameter (unlike other models)
- Training, evaluation, and `write(to:)` model export each in one line of code
- Usable from Xcode Playgrounds, Swift scripts, or command-line tools

**On-Device Inference**
- Export as a `.mlmodel` (Core ML) neural network classifier
- Model metadata shows `selectedSensors` and `predictionWindowSize` — use identical config at runtime
- Deploy on iPhone or Apple Watch via Core ML's standard prediction APIs

## APIs & Frameworks

### Create ML (NEW template)
- `MLActivityClassifier` **[NEW]** — new Create ML model type for motion-based activity recognition
- `MLActivityClassifier.DataSource` **[NEW]** — folder-based data source (per-class subfolders)
- `MLActivityClassifier(trainingData:sessionIdentifier:featureColumns:predictionWindowSize:parameters:)` **[NEW]** — programmatic training initializer
- `selectedSensors` / `featureColumns` parameter **[NEW]** — specifies which sensor columns from CSV to use
- `predictionWindowSize` parameter **[NEW]** — sliding window length in samples
- `MLActivityClassifier.ModelParameters` **[NEW]** — training hyperparameters
- `evaluation(on:)` **[NEW]** — per-class precision/recall evaluation
- `write(to:metadata:)` — export to `.mlmodel` (standard Create ML method)

### Core Motion (data collection)
- `CMMotionManager` — start/stop sensor streams; set `accelerometerUpdateInterval`, `gyroUpdateInterval`, etc.
- `CMAccelerometerData` / `CMAcceleration` — raw accelerometer readings (`x`, `y`, `z`)
- `CMGyroData` / `CMRotationRate` — raw gyroscope readings
- `CMDeviceMotion` — sensor-fused data including `userAcceleration`, `rotationRate`, `attitude`, gravity; provides normalization and bias removal **[recommended for activity classification]**
- `startAccelerometerUpdates(to:withHandler:)` / `startDeviceMotionUpdates(to:withHandler:)` — streaming callbacks

### Core ML (inference)
- Generated `MLModel` — neural network classifier; standard `prediction(from:)` API
- Prediction input: fixed-size window of sensor feature vectors matching `predictionWindowSize`
- Prediction output: label (activity class name) + confidence scores

## Code Highlights

Programmatic training with Create ML framework:
```swift
import CreateML

let trainingData = MLActivityClassifier.DataSource.labeledDirectories(at: trainingFolderURL)

let classifier = try MLActivityClassifier(
    trainingData: trainingData,
    sessionIdentifier: "frisbeeSession",
    featureColumns: ["userAcceleration.x", "userAcceleration.y", "userAcceleration.z",
                     "rotationRate.x", "rotationRate.y", "rotationRate.z"],
    predictionWindowSize: 100,  // 2 seconds at 50 Hz
    parameters: .init()
)

let evaluation = classifier.evaluation(on: testingData)
try classifier.write(to: URL(fileURLWithPath: "FrisbeeMotion.mlmodel"))
```

Data collection with Core Motion:
```swift
let motionManager = CMMotionManager()
motionManager.deviceMotionUpdateInterval = 1.0 / 50.0  // 50 Hz
motionManager.startDeviceMotionUpdates(to: .main) { data, error in
    guard let data = data else { return }
    let row = [data.userAcceleration.x, data.userAcceleration.y, data.userAcceleration.z,
               data.rotationRate.x, data.rotationRate.y, data.rotationRate.z]
    // append row to recording buffer
}
```

## Takeaways
- Activity Classification is a new Create ML template that requires only labeled CSV/JSON recordings from Core Motion — no ML expertise needed.
- Choose `predictionWindowSize` based on the cadence of activities; use the same window size at training time and inference time.
- Always include a "no activity" or "other" class for robust real-world performance.
- Use `CMDeviceMotion` (sensor fusion) rather than raw accelerometer/gyro when possible for better normalization and bias removal.

---
_Source: WWDC19 Session 426 page (abstract, full transcript, and resource links)._
