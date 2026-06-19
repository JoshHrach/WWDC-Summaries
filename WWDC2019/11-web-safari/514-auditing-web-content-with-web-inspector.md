# Auditing Web Content with Web Inspector
**WWDC19 · Session 514** · [Watch](https://developer.apple.com/videos/play/wwdc2019/514/)

_Platforms:_ macOS (Safari 13), iOS (Safari/WebDriver)

## Overview
Safari 13's Web Inspector introduces a new Audit tab that gives web developers a fast, built-in way to run structured compliance checks on in-progress web content — without needing to configure an external continuous integration system. Audits are collections of test groups and individual tests described in a portable JSON format, executed as JavaScript functions inside Web Inspector.

The session walks through the complete audit workflow: running the built-in accessibility audit, interpreting result levels (pass, warning, fail, error, unsupported), navigating from a failing test directly to the offending DOM node in the Elements tab, and exporting/importing audit files. It also covers how to write custom audits tailored to team-specific coding standards.

## Key Topics

**Audit Tab Overview**
- New tab in Web Inspector available in Safari 13 on macOS **[NEW]**
- Audits are organized hierarchically: Audit → Test Groups → Individual Tests
- Run all audits via Start button, individual audits via right-click context menu, hover play button, or Space Bar
- Edit mode lets you toggle individual tests, groups, or entire audits on/off
- Results persist across page reloads within a Web Inspector session; cleared on close
- Results exportable to and importable from JSON files

**Result Levels**
- **Pass** — code met test expectations
- **Warning** — soft pass; code passed but improvements possible
- **Failed** — code did not meet test expectations
- **Error** — test's JavaScript threw a runtime error **[NEW concept for audits]**
- **Unsupported** — indicates data/API tested is not present on the current page **[NEW concept for audits]**

**Built-in Accessibility Audit**
- Ships with Safari 13 by default; tests common accessibility guidelines
- Results include interactive DOM node references; hovering highlights elements on page
- One-click jump from failing result to the node in the Elements tab for immediate editing

**Custom Audits**
- JSON format makes audits portable and shareable across teams
- Each test is a stringified JavaScript function with access to extended data beyond normal page JavaScript
- Can check CSS class naming conventions, markup patterns, or any team-specific standard
- Import by dragging a JSON file onto Web Inspector
- ESLint audit example published on the WebKit blog as a reference

**Relationship to WebDriver**
- WebDriver (available on macOS; now also iOS **[NEW]**) suits automated CI regression testing
- Web Inspector Audit tool is complementary — designed for developer convenience during active development

## APIs & Frameworks

### Web Inspector Audit System (NEW in Safari 13)
- **Audit tab** **[NEW]** — new Web Inspector panel for running content audits
- **Audit JSON format** **[NEW]** — portable, shareable JSON structure defining audits, test groups, and tests
- Audit test function — stringified JavaScript function executed in Web Inspector context with elevated data access
- Result levels: `pass`, `warn`, `fail`, `error`, `unsupported` **[NEW]**
- DOM node result data — tests can return DOM node references rendered as interactive trees in results
- Export/Import — JSON-based audit and result files, drag-and-drop import supported

### WebDriver
- `WebDriver` — Safari's automation protocol for CI testing; available macOS; **now also iOS [NEW]**

### Web Inspector (existing panels referenced)
- Elements tab — DOM tree inspection; jumped to from audit result nodes
- Develop menu → Show Web Inspector (keyboard shortcut: `Cmd-Option-I`)
- Safari Preferences → Advanced → Show Develop Menu in Menu Bar

### WAI-ARIA (referenced in accessibility audit context)
- `role` attribute — e.g., `role="menu"`, `role="menuitem"` — tested by built-in accessibility audit

## Code Highlights

Audit JSON structure (conceptual):
```json
{
  "type": "audit",
  "name": "My Team Audit",
  "tests": [
    {
      "type": "test",
      "name": "Check CSS class convention",
      "test": "function() { /* JS that inspects the page */ }"
    }
  ]
}
```

Test function return values signal result level:
```javascript
function() {
    const elements = document.querySelectorAll('[role="menu"] > *');
    const failures = [...elements].filter(el => !el.hasAttribute('role'));
    if (failures.length === 0) return { level: "pass" };
    return { level: "fail", domNodes: failures };
}
```

## Takeaways
- The new Audit tab in Safari 13's Web Inspector is a zero-setup alternative to CI automation for quickly catching compliance issues during active development.
- The built-in accessibility audit surfaces ARIA/role violations with interactive DOM node links, making fixes immediate and obvious.
- Custom audits in portable JSON format let teams encode and share their own coding standards across the organization.
- WebDriver on iOS is now available for automated testing, complementing the developer-focused Audit tool.

---
_Source: WWDC19 Session 514 page (abstract, full transcript, and resource links)._
