# Review Code and Collaborate in Xcode
**WWDC21 · Session 10205** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10205/)

_Platforms:_ macOS Monterey 12 (Xcode 13)

## Overview
Xcode 13 completely reimagines code review mode and introduces integrated pull request workflows for GitHub and Bitbucket Server. The session walks through a realistic development cycle: using code review mode to investigate a UI bug by comparing commits across branches and tags, creating a pull request, incorporating reviewer feedback, monitoring Xcode Cloud CI results, and merging.

Code review mode gains new capabilities including inline vs. side-by-side diff preferences, a stepper control for jumping between hunks, and a commit selector with support for branches, tags, and recent locations. The new Changes tab in the Source Control Navigator provides an at-a-glance view of all modified files and pull request state.

Pull requests are now first-class Xcode citizens — developers can create, review, comment on, approve, and merge pull requests without leaving the IDE, with discussions anchored directly to source code lines.

## Key Topics

### Reimagined Code Review Mode
- Activated via the Code Review button in the editor bar; continuously reflects local edits vs. last commit.
- **Inline vs. side-by-side diff**: toggle between presentations via the Editor menu.
- **Stepper control** in the bottom bar: shows total change count and jumps between diff hunks.
- **Commit selector**: compare any two points in history — branches, tags, and recent locations; highlighted with distinct colors per commit.
- **Reset button**: one click to return to comparing against the most recent commit.

### Changes Navigator
- New **Changes tab** in the Source Control Navigator lists all locally modified files since the last commit.
- Clicking any file auto-enters code review mode for that file.
- Pull requests appear as nodes in the Changes Navigator; Local Changes section shows uncommitted files not yet included in the PR.

### Pull Request Workflows (GitHub & Bitbucket Server)
- Create pull requests directly from the source control popover; Xcode pre-populates the target branch (default upstream).
- Draft PR support: add title, description, and reviewers (via the Participants button) before publishing.
- The source control popover shows pull requests relevant to the current developer: created by them, and PRs they've been requested to review.
- Xcode automatically associates a PR with the current branch when switching.

### Code Review and Collaboration
- Inline discussion threads anchored to specific source lines; reviewers can comment, reply, and view the full conversation in Xcode.
- PR overview panel shows review status, discussions, CI status (Xcode Cloud integration).
- **Xcode Cloud CI integration**: the PR's CI popover shows pass/fail summaries; clicking jumps into detailed Xcode Cloud reports.
- Merge from Xcode: choose merge strategy (merge commit, squash, rebase); optional custom commit message.

## APIs & Frameworks

This session covers Xcode 13 IDE features rather than runtime APIs. Key tooling:

- **Xcode 13** Code Review mode — inline and side-by-side diff views **[NEW]**
- **Commit selector** — compare across branches, tags, and recent locations **[NEW]**
- **Stepper control** — jump between diff hunks **[NEW]**
- **Changes tab** in Source Control Navigator **[NEW]**
- **Source control branch popover** in Xcode toolbar — shows current branch; quick branch switch; recent branches **[NEW]**
- **Pull request creation** via Xcode source control popover — GitHub and Bitbucket Server **[NEW]**
- **Participants / Reviewers picker** in PR creation **[NEW]**
- **Inline PR comments** anchored to source lines **[NEW]**
- **PR review actions** (approve, request changes, merge) from within Xcode **[NEW]**
- **Xcode Cloud CI status** embedded in PR panel **[NEW]**
- **Merge strategies**: merge commit, squash, rebase **[NEW]**

## Code Highlights

No runtime code samples — this session is entirely IDE workflow-based. Key workflow steps:

1. Click Code Review button in the editor bar to enter diff mode.
2. Use the commit selector in the bottom bar to select comparison branches/tags.
3. Open the Changes tab in the Source Control Navigator to view all modified files.
4. Use the source control branch popover (toolbar) to create a new branch.
5. From the source control popover, create a pull request; set title, description, and reviewers.
6. Review teammate PRs by selecting them in the source control popover → step through files in the Changes Navigator.
7. Leave inline comments via secondary-click → "Insert Comment."
8. Merge via Pull Request actions → Merge → choose strategy.

## Takeaways
- Code review mode now supports full historical comparisons across branches and tags, making it possible to pinpoint exactly when a bug was introduced without leaving Xcode.
- Pull requests with inline discussions keep code review context anchored to source code, enabling richer collaboration within the IDE.
- The Changes tab and source control branch popover streamline everyday Git workflows — branch creation, PR creation, and reviewer assignment.
- Xcode Cloud CI results surface directly in the PR panel, creating a tight loop between code review approval and CI status.

---
_Source: WWDC21 Session 10205 page (abstract, chapter summaries, and resource links)._
