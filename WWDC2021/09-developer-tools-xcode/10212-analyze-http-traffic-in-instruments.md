# Analyze HTTP Traffic in Instruments
**WWDC21 · Session 10212** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10212/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12

## Overview
Instruments 13 introduces a new HTTP Traffic instrument within the Network template that taps directly into the URL Loading System (URLSession) to capture, visualize, and analyze all HTTP traffic from an app. Unlike packet captures or proxy tools, this instrument requires no external setup, works with HTTPS, HTTP/3, VPN traffic, and on-device background network activity, and organizes everything in terms of familiar API objects: `URLSession`, `URLSessionTask`, and individual transactions.

This session uses a dog photo social app as the demo vehicle to diagnose four real-world networking issues: HTTP/1 head-of-line blocking, an expired authentication cookie, stale cache serving outdated data, and a third-party SDK secretly sending location coordinates to a remote server.

## Key Topics

### HTTP Traffic Instrument Overview
- **New** instrument in Instruments 13 — part of the Network template **[NEW]**
- Taps URLSession directly; no proxy setup, certificate pinning bypass, or entitlement needed
- Captures: all URLSession traffic (including HTTP/3, VPN), background process traffic, requests that hit the on-disk cache, and networking errors
- Track hierarchy: HTTP Traffic → Process → URLSession → Domain → Tasks/Transactions

### Track Hierarchy and Key Concepts
- **URLSession track**: set `URLSession.sessionDescription` to give named labels
- **URLSessionTask**: set `URLSessionTask.taskDescription` for per-task labels; shows task ID, error description
- **Transaction**: one HTTP request + response pair; tasks can have multiple (e.g., after a redirect)
- **Transaction states**: Cache Lookup → Blocked → Sending Request → Waiting for Response → Receiving Response
- **Transaction views**: Tasks view (grouped by task) and "HTTP Transactions by Connection" view

### Demo 1 — Head-of-Line Blocking (HTTP/1 vs HTTP/2)
- Staircase pattern of blocked transactions reveals HTTP/1 connection reuse limitation
- Switch to "HTTP Transactions by Connection" to see which connection each transaction used
- Enabling HTTP/2 on the server reduced load time from 7+ seconds to under 3 seconds with a single connection

### Demo 2 — Authentication / Cookie Bug
- Transaction label icons reveal `Cookie` sent and `Set-Cookie` received at a glance
- Inspecting response headers in the extended detail view revealed the server sent a `Set-Cookie` with an expiry date in the past — URLSession correctly refused to store it
- Gray task intervals = canceled tasks; orange transactions = non-2xx status codes

### Demo 3 — Stale Cache
- An extremely short task duration revealed the response was served from "Local Cache" rather than a connection
- `URLRequest.cachePolicy = .reloadRevalidatingCacheData` — sends a conditional request; server returns 304 (use cache) or new data
- Backtrace in the detail view links directly to the source code that issued the request

### Demo 4 — Third-Party SDK Privacy Audit
- SDK was making network requests before any user action (logged in background analytics)
- Backtrace showed CoreLocation was in the call stack — location data was in the request body
- Demonstrates using HTTP Traffic instrument to audit SDK dependencies for unexpected or privacy-violating behavior

### Exporting Traces
- `xctrace export --input <trace> --output <file> --har` — exports to HTTP Archive (HAR) format **[NEW]**
- HAR is a JSON-based industry standard readable by any HTTP analysis tool; useful for sharing traces without Instruments

## APIs & Frameworks

### Instruments / Xcode
- **HTTP Traffic Instrument** **[NEW]** — instrument in Instruments 13 Network template
  - Track hierarchy: process → URLSession → domain → tasks/transactions
  - Views: "Tasks" and "HTTP Transactions by Connection"
  - Detail pane: request/response headers, backtrace, timing
- `xctrace export --har` **[NEW]** — command-line export to HAR format

### URLSession / Foundation
- `URLSession.sessionDescription: String?` — names the session in Instruments track labels
- `URLSessionTask.taskDescription: String?` — names individual tasks in Instruments track labels
- `URLRequest.cachePolicy` — controls caching behavior
  - `.reloadRevalidatingCacheData` — send conditional request; use cached data only if server confirms unchanged
  - `.reloadIgnoringLocalCacheData` — always bypass cache
  - `.returnCacheDataElseLoad` — use cache if available
- Transaction states surfaced by instrument: cache lookup, blocked, sending, waiting, receiving

## Code Highlights

Naming a URLSession for Instruments:
```swift
let session = URLSession(configuration: .default)
session.sessionDescription = "My App Main Session"
```

Naming a URLSessionTask:
```swift
let task = session.dataTask(with: request)
task.taskDescription = "Load Favorites List"
task.resume()
```

Fixing stale cache with revalidation:
```swift
var request = URLRequest(url: favoritesURL)
request.cachePolicy = .reloadRevalidatingCacheData
let task = session.dataTask(with: request)
task.resume()
```

Exporting a trace to HAR format from Terminal:
```bash
xctrace export --input PrivacyViolation.trace --output traffic.har --har
```

## Takeaways
- The HTTP Traffic instrument gives unprecedented visibility into URLSession behavior — task/transaction timelines, connection reuse, cache behavior, headers, and backtraces — all with no setup.
- Name your `URLSession` and `URLSessionTask` objects for much more readable traces and easier debugging.
- The instrument is a powerful privacy auditing tool: use it to verify that SDKs and third-party frameworks only send the data you expect.
- `xctrace --har` export makes sharing trace findings with server teams or third parties easy, even if they don't have Instruments.

---
_Source: WWDC21 Session 10212 page (abstract, chapter summaries, and resource links)._
