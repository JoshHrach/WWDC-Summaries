# Writing Great Alerts
**WWDC17 · Session 813** · [Watch](https://developer.apple.com/videos/play/wwdc2017/813/)

_Platforms:_ iOS, macOS

## Overview
This short, practical session covers the decision framework and writing techniques for crafting alerts that are clear, helpful, and appropriately rare. The central argument is that alerts are powerful precisely because they are disruptive — and that power is wasted, or worse turned against the app, when alerts appear for low-value reasons, contain too much content, or leave users uncertain about what to do. Good alerts solve real problems efficiently and let users return to their task immediately; bad ones frustrate users and generate one-star reviews.

The session is structured in three parts: when to use (and when not to use) an alert; how to structure one correctly using the title/message/action hierarchy; and how to write the individual components effectively, with before/after rewrites of common failure patterns.

## Key Topics
- **Alerts are disruptive by design** — this is a feature and a cost simultaneously; use that disruption only for situations that genuinely require immediate attention, not passive information or optional prompts
- **Three legitimate uses for alerts** — (1) correct errors that have already occurred; (2) request access to user data (location, contacts, photos, financial information); (3) notify about major updates (significant feature releases or critical security fixes)
- **When not to use alerts** — non-essential information; content the user does not need right now; settings or configuration choices (use settings screens instead); technical data and error codes (diagnostic information belongs in logs or support flows, not alerts); long lists of instructions; preventable problems (fix the root cause in the UI, e.g., show password rules inline rather than alerting after failure)
- **Alerts are lightweight by design** — small format for quick decisions; if the alert is filling the viewport, the content belongs somewhere else
- **Alert structure** — three components with distinct roles: (1) **title**: main point in one sentence or less; (2) **message**: brief additional context if necessary (cause of error, reason for request); (3) **actions**: clear, specific labels that describe exactly what happens when tapped
- **Scan test** — title + actions alone should convey what happened and what will happen; message details are supplementary; if the title + actions don't stand on their own, rewrite them
- **Three-question test** — a good alert answers: What happened? Why is the user seeing this? How should they proceed?
- **Specificity** — name the file, feature, or item involved; "cannot open this file" is worse than "cannot open Report.pdf"; vague alerts slow down decision-making
- **Ambiguity trap** — even well-structured alerts can confuse if button labels are ambiguous; the classic case: a "Cancel" button on an alert about canceling a download is ambiguous (cancel the download or cancel the alert?); rewrite actions to state the consequence, not just a verb
- **Action label guidelines** — use the recommended vocabulary (Cancel, OK for neutral; Delete, Remove for destructive); avoid "No" and "Yes" (replace with consequence-describing labels); avoid "OK" for confirmations of non-neutral actions
- **Navigation obligation** — if an action takes the user out of the app or to a different state, the alert or its action must navigate there; do not leave users to find their own way back
- **Style and quality checklist** — avoid platitudes, jargon, and filler words; use correct spelling, punctuation, and capitalization; match editorial voice to the audience (medical professionals vs. children vs. general consumers)
- **Audit advice** — review every existing alert in an app: Does it still need to exist? Is it providing real value? Can the content or choices be simplified?

## APIs & Frameworks

### UIKit
- **`UIAlertController`** — modal alert or action sheet; `preferredStyle: .alert` for two-button alerts; `preferredStyle: .actionSheet` for sheets (prefer action sheets for three or more choices)
- **`UIAlertAction`** — individual button; `style: .default`, `.cancel`, or `.destructive`; `title` is the action label that must be clear and specific
- **`UIAlertController.title`** — alert headline; keep to one sentence or less; this is the first and most prominent text users read
- **`UIAlertController.message`** — secondary body text; optional; should add context not already in title; omit if title + actions are self-sufficient

## Code Highlights

```swift
// Anti-pattern: vague title, vague message, confusing actions
let bad = UIAlertController(
    title: "Cannot open this file",
    message: "You may need to download the latest updates. Visit the website for troubleshooting?",
    preferredStyle: .alert
)
bad.addAction(UIAlertAction(title: "No", style: .cancel))
bad.addAction(UIAlertAction(title: "Yes", style: .default))

// Corrected: specific title, concise message, clear actions
let good = UIAlertController(
    title: "Cannot Open "Report.pdf"",
    message: "Update App Name to open this file type.",
    preferredStyle: .alert
)
good.addAction(UIAlertAction(title: "Not Now", style: .cancel))
good.addAction(UIAlertAction(title: "Update App Name", style: .default) { _ in
    // Navigate to App Store update page
})

// Anti-pattern: ambiguous actions on a cancellation alert
let ambiguous = UIAlertController(
    title: "Cancel download?",
    message: "If you cancel now, you will not receive this file.",
    preferredStyle: .alert
)
ambiguous.addAction(UIAlertAction(title: "Cancel", style: .cancel))  // which Cancel?
ambiguous.addAction(UIAlertAction(title: "OK", style: .default))

// Corrected: clear consequence-describing actions
let clear = UIAlertController(
    title: "Stop Downloading "Report.pdf"?",
    message: "If you stop now, "Report.pdf" will not be saved to your device.",
    preferredStyle: .alert
)
clear.addAction(UIAlertAction(title: "Keep Downloading", style: .cancel))
clear.addAction(UIAlertAction(title: "Stop Downloading", style: .destructive))
```

## Takeaways
- Before writing a single word, ask: does this alert need to exist at all, and does the user need to see it right now? If the answer is not an unambiguous yes, find a less disruptive UI pattern.
- The scan test is the fastest quality check: cover the message body and read only the title and button labels; if the situation and next steps are still clear, the alert is well-structured.
- Ambiguity in action labels is the most common cause of user confusion, even in otherwise correctly structured alerts; always describe the consequence of the action, not just its generic verb ("Stop Downloading" vs. "Cancel").
- Style quality matters as much as structure: typos, jargon, and mismatched tone undermine trust in the app even if the alert's content is correct.

---
_Source: WWDC17 Session 813 page (abstract, transcript, and resource links)._
