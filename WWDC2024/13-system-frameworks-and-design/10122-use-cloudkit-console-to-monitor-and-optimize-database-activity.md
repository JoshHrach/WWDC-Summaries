# Use CloudKit Console to Monitor and Optimize Database Activity
**WWDC24 · Session 10122** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10122/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia 15, visionOS 2, watchOS 11

## Overview
CloudKit Console gains a comprehensive set of observability and monitoring features designed to help development teams stay on top of their CloudKit containers in production. The four new capability areas are: Developer Notifications (real-time in-console and email alerts for key container events), Telemetry (interactive graphs for requests, errors, error rate, latency, and bandwidth with filtering and group-by), Logging (per-operation event explorer with privacy-preserving user identifiers and CSV/JSON export), and Alerts (custom threshold-based rules that trigger notifications when container metrics meet specified criteria).

The session uses a cross-platform app (iPhone, iPad, Apple Watch) as the running example, demonstrating how to pivot from a received alert notification through telemetry charts to root-cause a spike in Apple Watch errors, then creating a custom alert so the same condition never goes undetected again.

This session is about tooling in the CloudKit Console web interface, not a new SDK API surface.

## Key Topics

### Developer Notifications
The notification bell in the CloudKit Console header delivers real-time notifications for database alerts, schema changes, promotions, resets, and authorization token status changes. Clicking a notification navigates directly to the relevant console page (e.g., a schema change notification navigates to the new Schema History page, which shows team members' schema changes over time). Notification settings allow enabling in-console display and/or email delivery per event type, and notifications can be scoped to specific containers (toggling "manage containers" and selecting relevant ones).

### Telemetry
Telemetry graphs visualize Requests, Errors, Error Rate, Bandwidth, and Latency over configurable time ranges. A "Group By" selector breaks data down by platform, operation, or other dimensions—essential for cross-platform apps where error distribution by platform reveals device-specific bugs. A query builder allows adding filters (e.g., `platform = Watch`) to isolate specific subsets of traffic. Telemetry queries can be shared with teammates via a URL. Usage graphs show active users, storage composition (records vs. assets), and device distribution.

### Logging
Database Logs expose individual CloudKit operations with full request/response data in a privacy-preserving way (user IDs are pseudonymized). Navigating from a Telemetry chart to Logs via "Query in Logs" preserves all active filters and time range, providing immediate relevance. Logs are displayed in a customizable table view—columns like `userId`, `operation`, and `error` can be added individually. Expanding a log entry shows complete event details. Results can be exported as CSV or JSON for external analysis or historical comparison. Saved log queries and shareable links enable team-wide debugging collaboration.

### Alerts
Alerts let developers define threshold-based rules against Requests, Errors, or Error Rate metrics. Alerts can be scoped by telemetry filters (e.g., only Watch platform traffic) and configured with custom thresholds (fixed value or average-based). Time period granularity (per hour, per day, etc.) controls how quickly an alert fires. A preview chart shows when the configured alert would have fired historically. Alerts are managed from a dedicated Alerts page under the Monitor section. When triggered, alerts generate a Developer Notification (and optionally an email based on notification settings).

## APIs & Frameworks

- `CloudKit Console` — web-based developer tool at icloud.developer.apple.com
- `CloudKit` framework — Apple's cloud database and sync framework
- Developer Notifications **[NEW]** — real-time in-console bell for container events
  - Database Alerts notifications **[NEW]**
  - Schema Changes notifications **[NEW]**
  - Promotions notifications **[NEW]**
  - Resets notifications **[NEW]**
  - Authorization Token Status notifications **[NEW]**
  - Email notifications option **[NEW]**
  - Container-scoped notification filtering **[NEW]**
- Schema History page **[NEW]** — CloudKit Console page showing team schema changes over time
- Telemetry **[NEW]** — interactive time-series graphs in CloudKit Console
  - Requests metric
  - Errors metric
  - Error Rate metric (errors as % of total requests)
  - Bandwidth metric
  - Latency metric
  - Group By selector (platform, operation, etc.) **[NEW]**
  - Filter query builder **[NEW]**
  - Shareable query URLs **[NEW]**
  - Usage graphs (active users, device distribution, storage composition) **[NEW]**
- Database Logs **[NEW]** — per-operation event log explorer
  - Customizable table column selector **[NEW]**
  - "Query in Logs" button from Telemetry page **[NEW]**
  - CSV export **[NEW]**
  - JSON export **[NEW]**
  - Saved log queries **[NEW]**
  - Shareable log query links **[NEW]**
- Alerts **[NEW]** — custom threshold-based monitoring rules
  - Alerts creation dialog with preview chart **[NEW]**
  - Threshold-based alerts (fixed value or average-based) **[NEW]**
  - Filter-scoped alerts **[NEW]**
  - Alerts management page **[NEW]**

## Code Highlights

This session is entirely about the CloudKit Console web interface. No code samples were provided. Key developer actions are:

- Navigate to icloud.developer.apple.com → CloudKit Console → Database
- Click the notification bell to view Developer Notifications
- Configure email + container-scoped notifications in Settings → Notifications
- Use Telemetry → Group By platform → filter by specific platform to diagnose cross-device error spikes
- Click "Query in Logs" from Telemetry to pivot to filtered log view; add `userId` column to identify which users are affected
- Export logs as CSV/JSON for external analysis
- Click "Create Alert" under a Telemetry chart to create a threshold-based alert rule with filter carryover

## Takeaways

- CloudKit Console now delivers real-time Developer Notifications for critical container events (schema changes, database alerts, token status) both in-browser and optionally via email, scoped to containers you care about.
- The Telemetry "Group By platform" and filter query builder make it straightforward to isolate device-specific or operation-specific anomalies—essential for cross-platform apps.
- Database Logs provide per-operation details with pseudonymized user IDs, exportable as CSV/JSON, and shareable with teammates for collaborative debugging without compromising user privacy.
- Custom Alerts with configurable thresholds, filter scoping, and time-period granularity let teams be proactively notified of production regressions before users report them.

---
_Source: WWDC24 Session 10122 page (abstract, chapter summaries, transcript, and resource links)._
