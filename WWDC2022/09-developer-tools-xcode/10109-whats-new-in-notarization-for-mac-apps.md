# What's New in Notarization for Mac Apps
**WWDC22 · Session 10109** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10109/)

_Platforms:_ macOS Ventura 13

## Overview
Notarization is Apple's automated security scanning service for macOS software distributed outside the App Store. This session covers three developments: a mandatory migration deadline from the old `altool` CLI to `notarytool` (fall 2023), performance improvements in Xcode 14's built-in notarization (now using the same backend as `notarytool`), and a brand-new Notary REST API enabling notarization submissions from any internet-connected machine including Linux-based CI systems.

The Notary REST API is a JSON-based web service authenticated via JWT (same as other App Store Connect APIs). It supports submitting files for notarization, polling submission status, and retrieving notarization logs — enabling fully automated CI/CD notarization pipelines without requiring macOS.

## Key Topics

### altool Sunset (Fall 2023)
`altool` for notarization will stop working in fall 2023. All three affected methods must migrate:
- **Xcode 13 UI**: Will stop working; update to Xcode 14 (no workflow changes needed)
- **altool CLI**: All forms stop working; migrate to `notarytool` CLI (available standalone or bundled in Xcode)
- `notarytool` CLI (including the Xcode 13-bundled version) continues to work past the deadline

### Xcode 14 Notarization Improvements
Xcode 14 migrates its built-in notarization to the same backend as `notarytool`, delivering approximately a 4x performance improvement over Xcode 13. No project or workflow changes required — just update to Xcode 14.

### Notary REST API (New)
A new JSON-based REST API (`https://appstoreconnect.apple.com/notary/v2/`) allows:
- Uploading software for notarization from any platform (including Linux CI)
- Polling submission status
- Retrieving submission history and details
- Webhook support: provide a URL in the upload request to receive a callback when notarization completes (includes submission ID, team ID, and verification signature)

**Authentication**: JSON Web Token (JWT) — same as App Store Connect API.

**Upload flow**:
1. POST to `/notary/v2/submissions` with SHA-256 and filename → receive S3 upload credentials and submission ID
2. Upload file to Amazon S3 using the returned temporary credentials
3. Poll `/notary/v2/submissions/{submissionID}` for status, or use webhook callback

## APIs & Frameworks

**Notary REST API** **[NEW]**
- `POST https://appstoreconnect.apple.com/notary/v2/submissions` — initiate submission; returns AWS S3 credentials + submission ID
- `GET https://appstoreconnect.apple.com/notary/v2/submissions/{submissionId}` — get submission status (In Progress / Accepted / Invalid)
- `GET https://appstoreconnect.apple.com/notary/v2/submissions/{submissionId}/logs` — retrieve notarization log
- `GET https://appstoreconnect.apple.com/notary/v2/submissions` — retrieve submission history
- Webhook support — provide callback URL in submission request body

**Developer Tools**
- `notarytool` CLI — replacement for `altool`; ~4x faster than altool; bundled in Xcode 13+
- Xcode 14 notarization UI — now uses notarytool backend; ~4x performance improvement **[NEW]**
- `altool` notarization — **[DEPRECATED]**, sunset fall 2023

## Code Highlights

```python
# Step 1: initiate notarization and get S3 upload credentials
def upload_file(token, filepath, sha256):
    data = {"sha256": sha256, "submissionName": os.path.basename(filepath)}
    resp = requests.post(
        "https://appstoreconnect.apple.com/notary/v2/submissions",
        json=data,
        headers={"Authorization": "Bearer " + token})
    output = resp.json()
    aws_info = output["data"]["attributes"]
    submission_id = output["data"]["id"]

    # Step 2: upload to S3 with temporary credentials
    client = boto3.client("s3",
        aws_access_key_id=aws_info["awsAccessKeyId"],
        aws_secret_access_key=aws_info["awsSecretAccessKey"],
        aws_session_token=aws_info["awsSessionToken"])
    client.upload_file(filepath, aws_info["bucket"], aws_info["object"])
    return submission_id

# Step 3: poll for completion
def watch_upload(submission_id, token):
    while True:
        resp = requests.get(
            "https://appstoreconnect.apple.com/notary/v2/submissions/" + submission_id,
            headers={"Authorization": "Bearer " + token})
        status = resp.json()["data"]["attributes"]["status"]
        if status != "In Progress":
            return status  # "Accepted" or "Invalid"
        time.sleep(30)
```

## Takeaways

- Migrate from `altool` to `notarytool` or the new REST API before fall 2023 — all `altool` notarization methods and the Xcode 13 UI will stop working.
- Xcode 14 delivers ~4x faster notarization automatically with no project changes needed.
- The new Notary REST API enables notarization from any internet-connected machine (including Linux CI servers), using JWT authentication and Amazon S3 for file upload.
- Use webhook support in the REST API to decouple file upload from post-notarization automation in CI/CD pipelines.

---
_Source: WWDC22 Session 10109 page (abstract, chapter summaries, code samples, and resource links)._
