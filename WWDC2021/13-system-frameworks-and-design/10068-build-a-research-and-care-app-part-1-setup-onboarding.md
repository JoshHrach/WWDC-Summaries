# Build a Research and Care App, Part 1: Setup Onboarding
**WWDC21 · Session 10068** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10068/)

_Platforms:_ iOS 15, iPadOS 15

## Overview
This is the first session in a three-part code-along series building "Recover," a physical therapy research app using ResearchKit and CareKit together. Part 1 focuses on participant onboarding and consent — the critical first step in any research app that must explain data collection practices, obtain a signature, and request device permissions before revealing the app's care content.

The session demonstrates a key architectural pattern: gating a CareKit care feed behind an onboarding consent task. The care plan's real content is hidden until the participant completes the ResearchKit consent flow, at which point a `CKOutcome` is saved and the feed reloads to reveal all scheduled tasks.

## Key Topics

### ResearchKit + CareKit Integration Pattern
- ResearchKit: provides consent, survey, and permission request UI as a series of `ORKStep` objects composed into `ORKOrderedTask`
- CareKit: provides the scheduling and persistence layer (`OCKStore`, `OCKTask`, `OCKOutcome`) and the feed UI (`OCKDailyPageViewController`)
- Bridge: `OCKSurveyTaskViewController` **[NEW]** — a CareKit view controller that presents a ResearchKit survey as a CareKit task card and translates ResearchKit results into `OCKOutcomeValue` objects

### Consent-Gated Feed Architecture
1. At app launch, persist a CareKit `OCKTask` with ID `"onboarding"` on a daily schedule (so participants are reminded each day until complete)
2. In `OCKDailyPageViewController.prepareListViewController(for:)`, query for `OCKOutcome` records associated with the onboarding task
3. If none found: show only the consent task card; if found: show all other care tasks
4. `OCKSurveyTaskViewController` presents the ResearchKit survey; on completion, its delegate fires `surveyTaskViewController(_:didFinish:with:)` where you save the outcome and call `reload()`

### ResearchKit Onboarding Best Practices
- **Orientation before consent**: use 1–2 `ORKInstructionStep` screens to explain what participants will be asked BEFORE jumping into the legal consent language — significantly improves participant experience
- **Body items**: instruction steps support `ORKBodyItem` bullet points with optional SF Symbol images instead of plain bullet characters
- **Consent document with signature**: `ORKWebViewStep` renders HTML consent document with `showSignatureAfterContent = true` to append a signature field below the document
- **Permissions up front**: collect all permissions in a single `ORKRequestPermissionsStep` at the end of onboarding so participants understand the full data story in one flow

### New Permission Types (iOS 15)
- `ORKNotificationPermissionType` **[NEW]** — requests UNUserNotification authorization (alerts, badge, sound) within a ResearchKit step
- `ORKMotionActivityPermissionType` **[NEW]** — requests CMMotionActivityManager authorization for device motion/accelerometer data within a ResearchKit step
- Combined with existing `ORKHealthKitPermissionType` (introduced WWDC20) into a single `ORKRequestPermissionsStep`

## APIs & Frameworks

### ResearchKit
- `ORKInstructionStep(identifier:)` — welcome and orientation screens
  - `title`, `detailText`, `image` — header image
  - `bodyItems: [ORKBodyItem]` — SF-Symbol-illustrated bullet points
- `ORKBodyItem(text:detailText:image:learnMoreItem:isBulleted:)` — body item with optional SF Symbol image
- `ORKWebViewStep(identifier:html:)` — consent document rendered as HTML
  - `showSignatureAfterContent: Bool` — append signature field below HTML
- `ORKRequestPermissionsStep(identifier:permissionTypes:)` — batched permissions request
  - `permissionTypes: [ORKPermissionType]`
- `ORKHealthKitPermissionType(sampleTypesToWrite:objectTypesToRead:)` — HealthKit authorization
- `ORKNotificationPermissionType(authorizationOptions:)` **[NEW]** — notification authorization
- `ORKMotionActivityPermissionType()` **[NEW]** — device motion authorization
- `ORKCompletionStep(identifier:)` — final thank-you screen
- `ORKOrderedTask(identifier:steps:)` — chain all steps into a task

### CareKit
- `OCKStore` — local SQLite-backed persistence; call `addTasks(_:callbackQueue:completion:)` to persist tasks
- `OCKTask(id:title:carePlanUUID:schedule:)` — define a scheduled care task
  - `instructions: String?` — description shown in the task card
  - `impactsAdherence: Bool` — set to `false` for onboarding so it doesn't count toward completion rings
- `OCKSchedule.dailyAtTime(hour:minutes:start:end:text:duration:)` — daily schedule element
- `OCKOutcomeQuery` — query outcomes by task ID
- `OCKSurveyTaskViewController(taskID:eventQuery:storeManager:survey:extractOutcomeValues:)` **[NEW]**
  - Bridges a ResearchKit survey into a CareKit task card
  - `extractOutcomeValues`: closure `(ORKTaskResult) throws -> [OCKOutcomeValue]` — convert ResearchKit results to CareKit outcome values
  - Delegate: `OCKSurveyTaskViewControllerDelegate`
    - `surveyTaskViewController(_:didFinish:with:)` — called on completion or cancellation
- `OCKDailyPageViewController` — calendar + scrollable feed
  - `prepareListViewController(for:)` — override to populate feed content per day

## Code Highlights

Persisting the onboarding task in CareKit:
```swift
let schedule = OCKSchedule.dailyAtTime(hour: 8, minutes: 0, start: Date(), end: nil, text: nil)
var onboardingTask = OCKTask(id: TaskID.onboarding, title: "Onboarding",
                             carePlanUUID: nil, schedule: schedule)
onboardingTask.instructions = "Please complete the onboarding steps."
onboardingTask.impactsAdherence = false
store.addTasks([onboardingTask]) { result in print(result) }
```

Checking if onboarding is complete:
```swift
func checkIfOnboardingIsComplete(_ completion: @escaping (Bool) -> Void) {
    var query = OCKOutcomeQuery()
    query.taskIDs = [TaskID.onboarding]
    storeManager.store.fetchAnyOutcomes(query: query, callbackQueue: .main) { result in
        switch result {
        case .failure: completion(false)
        case .success(let outcomes): completion(!outcomes.isEmpty)
        }
    }
}
```

Building the ResearchKit consent task:
```swift
func onboardingSurvey() -> ORKTask {
    let welcomeStep = ORKInstructionStep(identifier: "welcome")
    welcomeStep.title = "Welcome to Recover"
    welcomeStep.image = UIImage(named: "recoverLogo")

    let consentStep = ORKInstructionStep(identifier: "consent.intro")
    consentStep.bodyItems = [
        ORKBodyItem(text: "We'll ask you to share some health data.", detailText: nil,
                    image: UIImage(systemName: "heart.fill"), learnMoreItem: nil, isBulleted: false),
        ORKBodyItem(text: "We'll ask you to complete daily tasks.", detailText: nil,
                    image: UIImage(systemName: "checkmark.circle"), learnMoreItem: nil, isBulleted: false)
    ]

    let signatureStep = ORKWebViewStep(identifier: "consent.signature", html: consentHTML)
    signatureStep.showSignatureAfterContent = true

    let healthTypes: Set<HKSampleType> = [HKObjectType.quantityType(forIdentifier: .stepCount)!]
    let permissionsStep = ORKRequestPermissionsStep(identifier: "permissions", permissionTypes: [
        ORKHealthKitPermissionType(sampleTypesToWrite: healthTypes, objectTypesToRead: healthTypes),
        ORKNotificationPermissionType(authorizationOptions: [.alert, .badge, .sound]),
        ORKMotionActivityPermissionType()
    ])

    let completionStep = ORKCompletionStep(identifier: "complete")
    completionStep.title = "Thank you for joining!"

    return ORKOrderedTask(identifier: "onboarding",
                          steps: [welcomeStep, consentStep, signatureStep, permissionsStep, completionStep])
}
```

Presenting with `OCKSurveyTaskViewController`:
```swift
let surveyVC = OCKSurveyTaskViewController(
    taskID: TaskID.onboarding,
    eventQuery: OCKEventQuery(for: Date()),
    storeManager: storeManager,
    survey: onboardingSurvey(),
    extractOutcomeValues: { _ in [OCKOutcomeValue(Date())] }
)
surveyVC.surveyDelegate = self
listViewController.appendViewController(surveyVC, animated: false)
```

## Takeaways
- `OCKSurveyTaskViewController` (new in 2021) is the key bridge between ResearchKit and CareKit — it presents a ResearchKit survey as a native CareKit task card and handles result-to-outcome conversion via a closure.
- Always orient participants with plain-language instruction steps before the legal consent language; `ORKBodyItem` with SF Symbols makes these screens welcoming rather than intimidating.
- Request all permissions (HealthKit, notifications, motion) in a single `ORKRequestPermissionsStep` at the end of onboarding so participants understand the full data picture in one coherent flow.
- The consent-gated feed pattern (query for the onboarding outcome, show different content if absent) is a reusable architecture for any research app that requires study enrollment before access.

---
_Source: WWDC21 Session 10068 page (abstract and transcript). Part of a three-part series with sessions 10069 and 10282._
