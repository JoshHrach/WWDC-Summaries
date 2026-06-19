# Deep Dive into Xcode Cloud for Teams
**WWDC22 · Session 110375** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110375/)

_Platforms:_ iOS, iPadOS, macOS, tvOS, watchOS (Xcode Cloud / App Store Connect)

## Overview
This session demonstrates how to use Xcode Cloud effectively in team settings of any size. It covers three major areas: integrating Xcode Cloud with existing issue trackers and tools using the App Store Connect API and webhooks; managing code dependencies (Swift Package Manager, CocoaPods, Carthage); and a set of workflow best practices including static analysis with SwiftLint, restricting workflow editing, and consolidating start conditions.

A worked example builds a Swift on Server webhook receiver (using the Vapor framework) that listens for Xcode Cloud build events, calls the App Store Connect REST API to enrich the data, and posts structured build summaries to an issue tracker.

## Key Topics

### Integrating with Existing Tools via Webhooks and the App Store Connect API
Xcode Cloud sends webhook payloads to a configured URL whenever a build completes. In App Store Connect, go to your product → Settings → Webhooks to register the URL. Each payload contains the build run details and per-action results.

The App Store Connect API covers all Xcode Cloud data under the same authentication tokens used for other App Store Connect operations. Key REST endpoints used in this session:
- `GET /v1/ciBuildRuns/{id}` — retrieve build run status and metadata
- `GET /v1/ciBuildActions/{id}/artifacts` — list artifact IDs produced by a build action
- `GET /v1/ciArtifacts/{id}` — get the `downloadUrl` for an artifact
- `GET /v1/ciBuildActions/{id}/issues` — list issues (warnings, errors, test failures) found in a build action

The session recommends generating a strongly typed Swift API client from Apple's published OpenAPI spec for the App Store Connect API using the open-source `openapi-generator` tool.

### Webhook Server Implementation Pattern
1. Decode the incoming JSON webhook payload into a Swift struct containing the build run and its actions
2. Guard on build completion status before proceeding
3. Call `GET /v1/ciBuildActions/{id}/issues` for each action to collect issues
4. Compose a comment string (build number, commit hash, author, per-action issues)
5. POST the comment to the issue tracker API, keying on an issue ID embedded in the commit message

### Code Dependency Management
- **Swift Package Manager**: publicly accessible Swift packages resolve automatically without extra configuration
- **CocoaPods / Carthage**: use custom build scripts (`ci_scripts/post_clone.sh`) to install and resolve dependencies inside the Xcode Cloud build environment; Homebrew is pre-installed

### Static Analysis with SwiftLint
Add a `post_clone.sh` script inside the `ci_scripts/` folder at the project root. The script installs SwiftLint via Homebrew and runs it against the `$CI_WORKSPACE` environment variable (which points to the checked-out repository root, not the `ci_scripts/` directory).

### Workflow Access Control
Workflows can be **restricted** so only administrators, account holders, and app managers can edit them — any team member can still trigger or view builds. Restricted workflows show a key icon; workflows locked by an admin show a lock icon. Useful for protecting high-stakes release or distribution workflows.

Workflows can also be **deactivated** to prevent start conditions from triggering automatic builds while still allowing manual builds. This is useful while triaging static analysis results or resolving linting violations.

### Multiple Start Conditions
A single workflow can have multiple start conditions — for example, triggering on merges to `main`, merges to `release/*`, and a scheduled nightly build — all running the same archive and test actions. This reduces the total number of workflows to maintain and eliminates configuration drift between similar workflows.

Start conditions are configured from the workflow editor in both Xcode and App Store Connect (web UI).

## APIs & Frameworks

### App Store Connect REST API — Xcode Cloud Endpoints
- `POST /v1/ciBuildRuns` — trigger a build run manually
- `DELETE /v1/ciBuildRuns/{id}` — cancel a build run
- `GET /v1/ciBuildRuns/{id}` — get build run status, attributes (buildNumber, commitSha, sourceCommit, createdDate)
- `GET /v1/ciBuildActions/{id}/artifacts` — list artifacts produced for a build action
- `GET /v1/ciArtifacts/{id}` — artifact metadata including `downloadUrl`
- `GET /v1/ciBuildActions/{id}/issues` — list issues (type, message, category) found during the action

### Xcode Cloud Webhook Payload Fields
- `ciWorkflow` — workflow that triggered the build
- `ciBuildRun` — build run details: `buildNumber`, `sourceCommitId`, `authorName`, `completionStatus`
- `ciBuildActions[]` — array of action results; each includes `actionType`, `completionStatus`, `id`

### Custom Build Scripts Environment Variables
- `$CI_WORKSPACE` — absolute path to the cloned repository root
- `$CI_SCRIPTS_DIR` — directory containing the `ci_scripts/` scripts

### Xcode Cloud Build Environment
- Homebrew — pre-installed; use it in `post_clone.sh` to install tools like SwiftLint
- Swift Package Manager — Swift packages with public repos resolve automatically

### App Store Connect API Client Generation
- **openapi-generator** CLI — generates strongly typed Swift (or other language) clients from the App Store Connect OpenAPI spec; the generated output is a Swift package ready to import

## Code Highlights

Custom build script (`ci_scripts/post_clone.sh`) to install and run SwiftLint:
```bash
#!/bin/zsh
brew install swiftlint
swiftlint --path "$CI_WORKSPACE"
```

Webhook payload struct (Swift):
```swift
struct WebhookPayload: Decodable {
    let ciBuildRun: CiBuildRun
    let ciBuildActions: [CiBuildAction]
}
```

Webhook handler in Vapor:
```swift
app.post("webhook") { req -> HTTPStatus in
    let payload = try req.content.decode(WebhookPayload.self)
    guard payload.ciBuildRun.completionStatus == "SUCCEEDED" ||
          payload.ciBuildRun.completionStatus == "FAILED" else {
        return .ok
    }
    // Enrich with API data, post to issue tracker…
    return .ok
}
```

Extension to fetch issues for a build action via the generated API client:
```swift
extension CiBuildActionsAPI {
    func issues(forActionId id: String) async throws -> [CiIssue] {
        let response = try await CiBuildActionsAPI.ciBuildActionsIssuesGetToManyRelated(id: id)
        return response.data
    }
}
```

## Takeaways
- The App Store Connect API provides full programmatic access to Xcode Cloud build data (runs, artifacts, issues); combine it with webhooks to integrate build status into any issue tracker or dashboard.
- Generate a typed Swift API client from Apple's OpenAPI spec with `openapi-generator` to avoid manual JSON parsing.
- Use `post_clone.sh` in `ci_scripts/` for dependency managers or tools (CocoaPods, Carthage, SwiftLint) that Xcode Cloud does not handle natively; Homebrew is pre-installed.
- Consolidate related start conditions (branch push, PR, scheduled) into a single workflow to reduce maintenance overhead.
- Use workflow restrictions (key/lock) to protect production workflows and deactivate workflows temporarily while resolving linting or quality issues.

---
_Source: WWDC22 Session 110375 page (abstract, transcript, and resource links)._
