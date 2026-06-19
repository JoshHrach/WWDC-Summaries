# Identify Trends with the Power and Performance API
**WWDC20 · Session 10057** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10057/)

_Platforms:_ iOS 14, iPadOS 14

## Overview
This session introduces the Power and Performance Metrics and Diagnostics REST API — a new App Store Connect API that provides programmatic access to the same aggregated field data that drives the Xcode Organizer's power and performance analysis. It enables teams to build custom dashboards, automated monitoring pipelines, and bug triage workflows around real-world app performance data collected from consented user devices.

Four new REST endpoints are covered: aggregated metrics + smart insights for recent app versions, per-build metrics download, top diagnostic signatures per build, and diagnostic logs per signature. The diagnostic logs include anonymized call stack trees that share the same JSON structure as MetricKit, enabling cross-tool compatibility.

## Key Topics

**API Overview**
- New set of App Store Connect REST API resources released as part of the broader App Store Connect API **[NEW]**
- Data is collected from consented devices, aggregated server-side, and exposed via REST
- Same data that drives Xcode Organizer's power and performance views
- Authentication: App Store Connect API key → JWT bearer token

**Four REST Endpoints**

1. `GET /v1/apps/{id}/perfPowerMetrics` — aggregated metrics + smart insights for up to 8 most recent app versions
2. `GET /v1/builds/{id}/perfPowerMetrics` — metrics for a specific build (requires build ID from App Store Connect API)
3. `GET /v1/builds/{id}/diagnosticSignatures` — top diagnostic signatures for a specific build version
4. `GET /v1/diagnosticSignatures/{id}/logs` — diagnostic logs (with call stack trees) for a signature

**Aggregated Metrics (productData)**
- JSON response keyed by metric type: `hangRate`, `launchTime`, `memoryUsage`, `diskWrites`, `batteryUsage`, etc.
- Each metric includes: unit metadata, per-device-group data, 50th and 90th percentile values
- Covers up to 8 most recent app versions for trend comparison
- Supports iPhone and iPad device groups

**Smart Insights (New)**
- Automatically flags regressions and upticks in metric values without requiring custom analytics **[NEW]**
- Each insight contains:
  - `metric` type and human-readable `summary` string describing the regression
  - `population` object listing impacted percentiles (p50, p90) and device types
  - `latestVersion` and `previousVersions` for comparison context
- Example: "Launch time regressed at 90th percentile for iPhone users in version 2.1"

**Diagnostic Signatures**
- Signatures group similar exceptions/issues from field reports
- `signature` string: the representative call frame for the group (same as Xcode Organizer display)
- `weight`: normalized relative importance of this signature
- Includes `links.logs` URL pointing to the corresponding logs endpoint

**Diagnostic Logs**
- Per-device anonymized logs for root cause analysis
- `diagnosticMetadata`: `deviceType`, `osVersion`, `buildVersion`, `platformVersion`
- `callStacks`: recursive tree structure of `callStackFrame` objects
  - `rawFrame` — binary frame info
  - `subFrames` — child frames; traverse to walk the full call stack tree
- JSON structure shared with MetricKit's call stack format — same parsing code works for both

**Response Content-Type**
- Request must set `Accept: application/vnd.apple.xcode-metrics+json,application/json`
- Without this header, the server may not return the metrics format

## APIs & Frameworks

### App Store Connect REST API (New)
- Base URL: `https://api.appstoreconnect.apple.com`
- Authentication: `Authorization: Bearer <JWT>` — JWT generated from App Store Connect API key
- `GET /v1/apps/{id}/perfPowerMetrics` **[NEW]** — recent-version metrics + insights; `id` = app ID
- `GET /v1/builds/{id}/perfPowerMetrics` **[NEW]** — build-specific metrics; `id` = build ID from App Store Connect API
- `GET /v1/builds/{id}/diagnosticSignatures` **[NEW]** — top signatures for a build; supports `filter[diagnosticType]`
- `GET /v1/diagnosticSignatures/{id}/logs` **[NEW]** — diagnostic logs; `id` = signature ID from diagnosticSignatures response

### Response Schema
- `productData[]` — array of metric objects
  - `metricCategories[]` — grouped metrics (e.g., Power, Performance)
    - `metrics[]` — individual metric with `identifier`, `unit`, `datasets[]`
      - `datasets[]` — per-device data with `filterCriteria` (deviceMarketingName), `points[]`
        - `points[]` — `percentile` (50/90), `value`, `version`, `errorMargin`
- `insights` — object containing regression flags
  - `regressions[]` — list of smart insight objects with `metric`, `summaryString`, `population`, `latestVersion`, `previousVersions`
- `diagnosticSignatures[]` — signature objects with `id`, `attributes.diagnosticType`, `attributes.signature`, `attributes.weight`, `links.logs`
- `diagnosticLogs[]` — log objects with `id`, `attributes.diagnosticMetaData`, `attributes.callStacks[]`

## Code Highlights

Fetching metrics and insights:
```bash
JWT="<your-jwt-token>"
APP_ID="<your-app-id>"

curl -X GET \
  -H "Authorization: Bearer ${JWT}" \
  -H "Accept: application/vnd.apple.xcode-metrics+json,application/json" \
  "https://api.appstoreconnect.apple.com/v1/apps/${APP_ID}/perfPowerMetrics"
```

Fetching top diagnostic signatures for a build:
```bash
BUILD_ID="<build-id-from-app-store-connect>"

curl -X GET \
  -H "Authorization: Bearer ${JWT}" \
  -H "Accept: application/vnd.apple.xcode-metrics+json,application/json" \
  "https://api.appstoreconnect.apple.com/v1/builds/${BUILD_ID}/diagnosticSignatures"
```

Fetching diagnostic logs for a signature:
```bash
SIGNATURE_ID="<signature-id-from-previous-response>"

curl -X GET \
  -H "Authorization: Bearer ${JWT}" \
  -H "Accept: application/vnd.apple.xcode-metrics+json,application/json" \
  "https://api.appstoreconnect.apple.com/v1/diagnosticSignatures/${SIGNATURE_ID}/logs"
```

Workflow for automated monitoring (pseudocode):
```python
# 1. Get latest metrics and check smart insights
response = get("/v1/apps/{id}/perfPowerMetrics")
for insight in response["insights"]["regressions"]:
    if insight["metric"] == "diskWrites":
        alert_team(insight["summaryString"])

# 2. Find top diagnostic signature for disk writes
sigs = get(f"/v1/builds/{build_id}/diagnosticSignatures?filter[diagnosticType]=diskWriteExceptions")
top_sig = sigs["data"][0]

# 3. Get call stacks and find root cause
logs = get(top_sig["links"]["logs"])
for log in logs["data"]:
    print_call_stack(log["attributes"]["callStacks"])
```

## Takeaways
- The Power and Performance API provides the same aggregated metrics and diagnostics as Xcode Organizer, but programmatically — use it to build CI/CD performance gates, team dashboards, or automated regression alerts without manual Xcode review.
- Smart insights (`/v1/apps/{id}/perfPowerMetrics` → `insights.regressions[]`) flag regressions automatically with human-readable summary strings — a zero-effort starting point for monitoring before custom analytics are built.
- Diagnostic logs share the same call stack JSON structure as MetricKit — a single parser can handle both on-device (MetricKit) and server-side (App Store Connect API) diagnostic reports.
- Always set `Accept: application/vnd.apple.xcode-metrics+json,application/json` on all requests to receive the metrics-specific JSON format.

---
_Source: WWDC20 Session 10057 page (abstract, transcript, code samples, and resource links)._
