# Extend your Xcode Cloud Workflows
**WWDC24 · Session 10200** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10200/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia 15, Xcode 16, Xcode Cloud

## Overview
Xcode Cloud is Apple's continuous integration and delivery service built into Xcode and App Store Connect, providing cloud-based tools for building, testing, and distributing apps. This session dives into how to scale and extend workflows beyond the basics, demonstrating powerful features that help teams adapt Xcode Cloud to their specific development needs.

The session walks through three major capabilities: configuring manual start conditions so workflows only run when explicitly triggered, creating custom aliases to keep macOS and Xcode version settings synchronized across multiple workflows, and writing custom scripts that run at key points in the build pipeline. It then covers how to connect Xcode Cloud to external systems using the App Store Connect API and webhooks.

A practical demo builds an end-to-end pipeline: integration tests triggered automatically by server changes via the App Store Connect API, with test results driving automated deployment decisions through a Vapor-based webhook listener.

## Key Topics

**Essential Workflow Concepts**
- Workflows consist of four elements: Environment, Start Conditions, Build Actions, and Post Actions
- Environment defines variables, Xcode version, and macOS version; aliases simplify version management
- Start Conditions: branch, pull request, git tag, schedule, or manual
- Build Actions: Build, Test, Analyze, Archive; Post Actions: notify, notarize, distribute to TestFlight

**Scale Your Workflows**
- Manual start conditions (new in Xcode 15.1): workflows run only when explicitly triggered — ideal for integration or expensive test suites
- Custom aliases (new in Xcode 15.3): define named version aliases (e.g., "Team Preference") that resolve to a specific Xcode or macOS version; update the alias and all associated workflows get the new version automatically
- Custom scripts placed in `ci_scripts/` directory; filename determines execution point (`ci_pre_xcodebuild.sh`, `ci_post_xcodebuild.sh`, `ci_post_clone.sh`)
- Environment variables (`CI_XCODEBUILD_ACTION`, `CI_WORKFLOW_ID`) allow scripts to be conditional

**Connect Other Systems**
- App Store Connect API enables automating Xcode Cloud: trigger builds, query workflows, and retrieve git references programmatically
- Uses Swift OpenAPI Generator to produce strongly typed Swift client code for all API endpoints
- Three-step API flow to start a build: fetch `CiWorkflows` resource → query `ScmRepositories` for git references → call `CiBuildRuns` to create the build
- Webhooks deliver JSON payloads for build events to any HTTP endpoint; configure via App Store Connect Settings > Webhooks
- Webhook payload contains `ciWorkflow.id`, `ciBuildRun.id`, `executionProgress`, and `completionStatus`

## APIs & Frameworks

**Xcode Cloud**
- Workflow environment, start conditions, build actions, post actions
- **[NEW]** Manual start conditions (Xcode 15.1)
- **[NEW]** Custom aliases for Xcode and macOS versions (Xcode 15.3)
- `ci_scripts/ci_pre_xcodebuild.sh` — custom script executed before xcodebuild
- `ci_scripts/ci_post_xcodebuild.sh` — custom script executed after xcodebuild
- `ci_scripts/ci_post_clone.sh` — custom script executed after repo clone
- Environment variables: `CI_XCODEBUILD_ACTION`, `CI_WORKFLOW_ID`, and many others (see Environment Variable Reference documentation)

**App Store Connect API**
- `CiWorkflows` resource — query workflow details including repository relationship
- `ScmRepositories` resource — list git references (branches, tags, PRs) for a repository
- `CiBuildRuns` resource — create new Xcode Cloud builds
- `ciWorkflowsGetInstance(path:query:)` — fetch a specific workflow
- `scmRepositoriesGitReferencesGetToManyRelated(path:)` — fetch git references for a repo
- `ciBuildRunsCreateInstance(body:)` — trigger a build
- Swift OpenAPI Generator — generates typed Swift client from OpenAPI spec

**Webhooks (Vapor / Hummingbird)**
- `WebhookPayload` struct (app-defined) with fields: `ciWorkflow`, `ciBuildRun` (id, executionProgress, completionStatus)
- `executionProgress` — `running` or `complete`
- `completionStatus` — `success`, `failure`, etc.
- Configured in App Store Connect > App > Settings > Webhooks

## Code Highlights

Custom script that checks server health before running integration tests:
```sh
#!/bin/sh
set -e
if [[ $CI_XCODEBUILD_ACTION == "test-without-building" && \
      $CI_WORKFLOW_ID == "82D89C93-B69C-46B5-A794-A2BCFD3EE487" ]]; then
    curl https://example.com/health --fail
fi
```

Swift extension to start a build via App Store Connect API:
```swift
func startBuild(workflowID: String, gitReferenceID: String) async throws {
    _ = try await ciBuildRunsCreateInstance(body: .json(.init(
        data: .init(
            _type: .ciBuildRuns,
            relationships: .init(
                workflow: .init(data: .init(_type: .ciWorkflows, id: workflowID)),
                sourceBranchOrTag: .init(data: .init(_type: .scmGitReferences, id: gitReferenceID))
            )
        )
    ))).created
}
```

## Takeaways
- Use **manual start conditions** to prevent expensive integration tests from running on every push.
- Use **custom aliases** to keep Xcode/macOS versions synchronized across all workflows with a single update point.
- Use **custom scripts** in `ci_scripts/` to add pre/post build logic (health checks, environment validation, artifact processing).
- Combine the **App Store Connect API** and **webhooks** to build fully automated pipelines that span Xcode Cloud and external systems.

---
_Source: WWDC24 Session 10200 page (abstract, chapter summaries, code samples, and resource links)._
