# Customize Your Advanced Xcode Cloud Workflows
**WWDC21 · Session 10269** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10269/)

_Platforms:_ macOS (Xcode Cloud / CI service)

## Overview
Xcode Cloud, introduced at WWDC21, integrates with Apple's developer ecosystem and major Git providers out of the box. This session covers four advanced customization mechanisms for teams needing to connect external tooling, proprietary services, or additional source repositories: environment variables (including secrets), custom shell scripts at precise build lifecycle points, additional repository access for private Swift package dependencies, and webhooks for receiving real-time build lifecycle events.

The session demonstrates each feature using the Fruta smoothie app: custom scripts swap in a beta app icon for pull request builds distributed via TestFlight; Swift package manager is used to pull in a private `InvitationsKit` dependency; and webhooks fire to an AWS Lambda function to post a tweet when a release build succeeds.

## Key Topics

### Environment Variables
Environment variables are key-value pairs configured in the Environment section of a workflow. They are injected into every action's execution environment. Use cases include: pointing tests at a staging vs. production API endpoint, or passing feature flags.

**Secret environment variables**: Check the "Secret" checkbox to encrypt and store the value securely. Secrets are decrypted only in the action's temporary environment, redacted from logs, and cannot be viewed in the workflow editor after saving. Use for API keys, tokens, credentials.

Built-in Xcode Cloud variables include:
- `CI_PULL_REQUEST_NUMBER` — set if the build is triggered by a pull request
- `CI_XCODEBUILD_ACTION` — the current action: `build`, `test`, `analyze`, or `archive`
- `CI_WORKFLOW` — the workflow name
- `CI_PRODUCT_PLATFORM` — `iOS`, `macOS`, `tvOS`, `watchOS`
- `CI_WORKSPACE` — path to the cloned source root

### Custom Build Scripts
Custom scripts are shell scripts placed in a `ci_scripts/` folder at the same directory level as the `.xcodeproj` or `.xcworkspace`. Three script names are recognized (exact names required):

| Script | Runs when |
|---|---|
| `ci_post_clone.sh` | After source clone, before dependency resolution |
| `ci_pre_xcodebuild.sh` | After dependency resolution, before xcodebuild |
| `ci_post_xcodebuild.sh` | After xcodebuild completes |

Key behaviors:
- All environment variables (both user-defined and built-in) are available inside scripts.
- Scripts are auto-discovered — no workflow configuration needed.
- stdout/stderr are captured into action logs and downloadable artifacts.
- A non-zero exit code fails the action, enabling gates in the pipeline.
- In test actions: only the build environment has source code cloned; test-runner environments only receive the `ci_scripts/` folder. `ci_post_clone.sh` does not run in test-runner environments. All script dependencies must be self-contained in `ci_scripts/`.

### Additional Repositories
Xcode Cloud auto-detects when a build fails because a private Git dependency (Swift package, submodule, or any repository cloned in a script) is inaccessible. The build results UI shows a warning banner with a "Manage Repositories" button. Clicking through redirects to the Settings → Additional Repositories page, where you grant access repository by repository. Once granted, rebuilds proceed with access. This works for any Git operation: `SPM` dependencies, Git submodules, or manual `git clone` calls in custom scripts.

### Webhooks
Webhooks send HTTP POST requests with a JSON payload to a URL you configure when build lifecycle events occur. Up to 5 webhooks per product.

**Trigger points**:
1. Build created (push or manual trigger)
2. Build starting (actions beginning execution)
3. Build completed (success or failure)

**Payload** includes: App Store Connect app info, workflow name, product, build number, build state, and more.

**Configuration**: Settings → Webhooks → + button; supply a name and endpoint URL.

**Delivery inspection**: Settings → Webhooks → select webhook → view delivery list with timestamps, request headers/body, and response status.

If the endpoint returns a non-2xx status code, Xcode Cloud retries the request.

## APIs & Frameworks

**Xcode Cloud** (CI service, configured in Xcode or App Store Connect)

**Custom Script Conventions**:
- Folder: `ci_scripts/` — placed alongside `.xcodeproj` or `.xcworkspace`
- Script names (exact): `ci_post_clone.sh`, `ci_pre_xcodebuild.sh`, `ci_post_xcodebuild.sh`
- Must be executable shell scripts

**Built-in Environment Variables** (partial list):
- `CI_PULL_REQUEST_NUMBER` — PR number if triggered by pull request
- `CI_XCODEBUILD_ACTION` — `build` | `test` | `analyze` | `archive`
- `CI_WORKFLOW` — workflow name string
- `CI_PRODUCT_PLATFORM` — `iOS` | `macOS` | `tvOS` | `watchOS`
- `CI_WORKSPACE` — absolute path to checked-out source root
- `CI_BRANCH` — current branch name
- `CI_COMMIT` — current commit SHA
- `CI_BUILD_NUMBER` — Xcode Cloud build number
- `CI_APP_STORE_SIGNED_APP_PATH` — path to signed archive product (archive actions)

**Webhook JSON Payload fields**: `app`, `workflow` (name, id), `product`, `build` (number, state: `SUCCEEDED`/`FAILED`/`ERRORED`), `sourceChanges`

## Code Highlights

Pre-xcodebuild script swapping app icon for PR builds (`ci_scripts/ci_pre_xcodebuild.sh`):
```bash
#!/bin/sh
# ci_pre_xcodebuild.sh — runs before xcodebuild in each action

if [[ -n $CI_PULL_REQUEST_NUMBER && $CI_XCODEBUILD_ACTION = 'archive' ]]; then
    echo "Setting Fruta Beta App Icon"
    APP_ICON_PATH=$CI_WORKSPACE/Shared/Assets.xcassets/AppIcon.appiconset

    # Remove existing App Icon
    rm -rf $APP_ICON_PATH

    # Replace with Fruta Beta App Icon
    mv "$CI_WORKSPACE/ci_scripts/AppIcon-Beta.appiconset" $APP_ICON_PATH
fi
```

Webhook handler in Swift on AWS Lambda (conceptual):
```swift
// Receive POST from Xcode Cloud webhook
let request = try event.body
let payload = try JSONDecoder().decode(XcodeCloudPayload.self, from: request)

if payload.workflow.name == "Release" && payload.build.buildRun.completionStatus == "SUCCEEDED" {
    // Post tweet notifying beta testers
    try await twitterClient.post("New TestFlight build available: \(payload.build.buildRun.buildNumber)")
}

return APIGatewayV2Response(statusCode: .ok)
```

Environment variable usage in a custom script:
```bash
#!/bin/sh
# Only run during iOS archive actions on the 'Release' workflow
if [[ $CI_PRODUCT_PLATFORM == 'iOS' && $CI_XCODEBUILD_ACTION == 'archive' && $CI_WORKFLOW == 'Release' ]]; then
    # Use $MY_API_KEY (secret environment variable)
    curl -H "Authorization: $MY_API_KEY" https://api.example.com/notify
fi
```

## Takeaways
- Environment variables (including secrets) inject configuration into builds without adding secrets to source control; built-in variables like `CI_PULL_REQUEST_NUMBER` and `CI_XCODEBUILD_ACTION` enable powerful conditional logic.
- Custom scripts in a `ci_scripts/` folder (with exact naming: `ci_post_clone.sh`, `ci_pre_xcodebuild.sh`, `ci_post_xcodebuild.sh`) need no workflow configuration — Xcode Cloud discovers and runs them automatically; a non-zero exit fails the action.
- Private Swift package dependencies and Git submodules trigger a friendly "Grant Access" UI flow in App Store Connect when Xcode Cloud encounters a repository it cannot access.
- Webhooks deliver a rich JSON payload at build-created, build-starting, and build-completed points; up to 5 per product; ideal for cross-posting to Slack, JIRA, Twitter, or downstream CI systems.
- Test actions in Xcode Cloud split across multiple environments: only the build environment has source code; test-runner environments only have `ci_scripts/` — design scripts and their dependencies accordingly.

---
_Source: WWDC21 Session 10269 page (abstract, chapter summaries, code samples, and resource links)._
