# What's New in ResearchKit
**WWDC20 · Session 10216** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10216/)

_Platforms:_ iOS 14

## Overview
ResearchKit gained several practical improvements in 2020 across onboarding, survey design, active tasks, and 3D model presentation. Key additions include an inline consent signature capture in the web view step, a new request permission step for HealthKit access within the ResearchKit flow, the "I Don't Know" button for scale-based questions, a post-survey review view controller, and new 3D model step classes powered by USDZ files and a third-party BioDigital integration.

The session also walked through building a fully custom active task — a front-facing camera recording step — illustrating the `ORKActiveStep`/`ORKActiveStepViewController` subclass pattern for developers who need data collection beyond the built-in tasks.

## Key Topics

### Onboarding Enhancements
- `ORKInstructionStep` body items (`ORKBodyItem`) with SF Symbols icons for rich, structured onboarding screens
- `ORKWebViewStep.showSignatureAfterContent = true` — consent signature capture inline with HTML consent document (no separate step needed)
- `ORKRequestPermissionStep` — requests HealthKit read/write permissions within the ResearchKit flow, eliminating custom permission UI

### Survey Enhancements
- `ORKSESAnswerFormat` — socioeconomic scale answer format for ladder/rung-style health questions
- `ORKScaleAnswerFormat.shouldShowDontKnowButton` + `customDontKnowButtonText` — "I Don't Know" / "Prefer not to answer" option on scale questions
- `ORKTextAnswerFormat` — new maximum character count label and Clear button (`hideWordCountLabel`, `hideClearButton`)
- `ORKReviewViewController` — lets participants review all answers and edit before final submission; initialized with task + result

### Active Task: Hearing Test Updates
- Environment SPL meter: new animation clearly indicating whether background noise is within threshold
- dB HL tone audiometry step: updated button UI, improved haptics, progress label instead of indicator, AirPods Pro calibration data added

### 3D Model Step
Two new classes simplify presenting interactive USDZ 3D models:
- `ORK3DModelStep` — ResearchKit step that hosts a 3D model viewer
- `ORKUSDZModelManager` — concrete manager class; loads a USDZ file from the app bundle; supports selection, highlight color, pre-highlighted objects, continue-after-selection control
- `ORK3DModelManager` — abstract base class; can be subclassed for custom model engines
- Third-party: `ORKBioDigitalModelManager` (via BioDigital's HumanKit SDK) — web-loaded anatomy models with programmatic labels, colors, annotations, and admin portal content

### Custom Active Task: Front-Facing Camera Step
Demonstrates the full pattern for building a custom active task:
- Subclass `ORKActiveStep` → `ORKFrontFacingCameraStep` (add maxDuration, allowReview, allowRetry)
- Override `stepViewControllerClass` to return `ORKFrontFacingCameraStepViewController`
- Subclass `ORKActiveStepViewController` for step UI with AVCapture live preview
- Subclass `ORKFileResult` → `ORKFrontFacingCameraStepResult` (adds retryCount, fileURL)
- Override `result` in view controller to return the custom result object

## APIs & Frameworks

### ResearchKit
- `ORKInstructionStep` — instruction/onboarding step
  - `bodyItems: [ORKBodyItem]` **[NEW]** — structured icon + text body items
- `ORKBodyItem` **[NEW]** — body item with text, detailText, image (SF Symbols), learnMoreItem, bodyItemStyle
- `ORKWebViewStep` — HTML content step
  - `showSignatureAfterContent: Bool` **[NEW]** — inline consent signature
- `ORKRequestPermissionStep` **[NEW]** — HealthKit permission request within ResearchKit flow
- `ORKHealthKitPermissionType` **[NEW]** — wraps HKSampleType write set and HKObjectType read set
- `ORKSESAnswerFormat` **[NEW]** — socioeconomic scale (ladder/rung) answer format
- `ORKScaleAnswerFormat`
  - `shouldShowDontKnowButton: Bool` **[NEW]**
  - `customDontKnowButtonText: String` **[NEW]**
- `ORKTextAnswerFormat`
  - `maximumLength: Int` (now shows visual label)
  - `hideWordCountLabel: Bool` **[NEW]**
  - `hideClearButton: Bool` **[NEW]**
- `ORKReviewViewController` **[NEW]** — post-survey answer review and edit
  - `init(task:result:delegate:)`
  - `reviewTitle`, `text` properties
  - Delegate: `didUpdateResult`, `didSelectIncompleteCell`
- `ORK3DModelStep` **[NEW]** — 3D model viewer step; takes an `ORK3DModelManager`
- `ORK3DModelManager` **[NEW]** — abstract base class for model managers
- `ORKUSDZModelManager: ORK3DModelManager` **[NEW]** — USDZ file-based model manager
  - `init(usdzFileName:)`
  - `allowsSelection: Bool`
  - `highlightColor: UIColor`
  - `enableContinueAfterSelection: Bool`
  - `identifiersOfObjectsToHighlight: [String]`
- `ORKActiveStep` — base class for timed active tasks
- `ORKActiveStepViewController` — base view controller for active steps
- `ORKFrontFacingCameraStep: ORKActiveStep` (custom, not built-in) — example custom step
- `ORKFrontFacingCameraStepResult: ORKFileResult` (custom) — retryCount + fileURL
- `ORKTaskViewController` — task presentation controller; delegate: `ORKTaskViewControllerDelegate`
- `ORKTaskResult` — aggregate result for a completed task
- `ORKOrderedTask`, `ORKNavigableOrderedTask` — concrete task implementations
- `ORKFormStep`, `ORKFormItem` — multi-question form step
- `ORKQuestionStep` — single-question step

### HealthKit (used with ORKRequestPermissionStep)
- `HKSampleType` — write access types
- `HKObjectType` — read access types

### AVFoundation (used in custom camera active task)
- `AVCaptureSession` — camera session
- `AVCaptureVideoPreviewLayer` — live preview

## Code Highlights

Inline consent signature in web view step:
```swift
let webViewStep = ORKWebViewStep(identifier: "webViewStep", html: exampleHtml)
webViewStep.showSignatureAfterContent = true
```

Scale question with "prefer not to answer":
```swift
let scaleAnswerFormat = ORKScaleAnswerFormat(maximumValue: 10, minimumValue: 1, defaultValue: 11, step: 1)
scaleAnswerFormat.shouldShowDontKnowButton = true
scaleAnswerFormat.customDontKnowButtonText = "Prefer not to answer"
```

Presenting a USDZ 3D model step:
```swift
let usdzModelManager = ORKUSDZModelManager(usdzFileName: "toy_drummer")
usdzModelManager.allowsSelection = false
usdzModelManager.highlightColor = .yellow
usdzModelManager.enableContinueAfterSelection = false
usdzModelManager.identifiersOfObjectsToHighlight = arrayOfIdentifiers
let threeDimensionalModelStep = ORK3DModelStep(identifier: drummerModelIdentifier,
                                               modelManager: usdzModelManager)
```

## Takeaways

- `ORKRequestPermissionStep` brings HealthKit permission requests inline with the ResearchKit onboarding flow, eliminating the need for custom permission UI code.
- The "I Don't Know" / "Prefer not to answer" option on scale questions reduces survey dropout and improves data quality for sensitive questions.
- The `ORK3DModelStep` + `ORKUSDZModelManager` pair makes presenting interactive 3D anatomical or product models a three-step process: add USDZ, configure manager, create step.
- Custom active tasks require subclassing `ORKActiveStep`, `ORKActiveStepViewController`, and `ORKStepResult`; the task infrastructure handles presentation, timing, and result aggregation automatically.

---
_Source: WWDC20 Session 10216 page (abstract, transcript, code samples, and resource links)._
