# Resilient API Pipeline — Error Handling and Fallback

## How it works

This workflow polls an external API every four hours and shows how to handle failures gracefully. The primary API call retries automatically up to three times. If it still fails, the workflow switches to a fallback API endpoint. Successful responses go through validation, transformation, and anomaly detection before being stored in Google Sheets. Any failures — whether from the primary API, the fallback, or an unexpected crash — get logged and reported to Slack so the team always knows what happened.

## Key features

- Polls an external API every 4 hours on a schedule
- Retries the primary endpoint 3 times with a 2-second gap between attempts
- Falls back to a secondary endpoint when the primary is completely down
- Validates and transforms every response before storing it
- Detects anomalies by comparing values against configurable thresholds
- Logs all errors to a dedicated Google Sheets tab for auditing
- Sends Slack notifications at three levels: anomaly alerts, API failure alerts, and unhandled crash alerts
- Includes a global error trigger as a safety net for anything unexpected
- Ships with pinned demo data for both success and failure scenarios

## Step by step

### Happy path (API responds)

**Schedule Trigger:**
Fires every 4 hours.

**Fetch API Data:**
Calls the primary API endpoint with HTTP header authentication. Configured with 3 retries and 2-second backoff. If all retries fail, the error goes to a separate output branch instead of stopping the workflow.

**Validate and Transform:**
Checks the response structure, makes sure required fields are present, and normalizes each data point into a consistent format with fields like `id`, `metric`, `value`, `unit`, `source`, `fetched_at`, and an `is_anomaly` flag.

**Store Data in Google Sheets:**
Appends the transformed data to the "API Data" sheet, building up a historical record.

**Anomaly Detection:**
Checks the `is_anomaly` flag on each data point. If a value exceeds its threshold, it gets routed to the alert path.

**Slack Anomaly Alert:**
Posts a detailed message to #api-alerts with the metric name, value, timestamp, and source.

### Error path (API fails)

**Format Error Details:**
When the primary API fails after all retries, this node extracts and structures the error information: type, message, endpoint, timestamp, and retry count.

**Fallback API:**
Calls a secondary endpoint. If it succeeds, the data is routed back to the Validate and Transform node and rejoins the happy path.

**Error Log:**
If the fallback also fails, the error details are appended to the "Error Log" sheet in Google Sheets for auditing.

**Slack API Failure Alert:**
Posts a critical alert to #api-alerts saying both endpoints are down and action is needed.

### Safety net

**Global Error Trigger:**
A separate error trigger catches any unhandled exception anywhere in the workflow — things that bypass the explicit error handling above.

**Slack Workflow Error:**
Posts a crash notification to #api-alerts with the error message and execution ID.

## Credentials needed

- **Google Service Account** — for storing API data and error logs in Google Sheets. Create a Service Account in Google Cloud Console, download the JSON key, and share your sheet with the service account email.
- **Slack OAuth2** — for posting anomaly alerts, failure notifications, and crash reports. Create a Slack App, add bot token scopes (`chat:write`, `channels:read`, `channels:join`), and complete the OAuth flow.
- **External API Key (optional)** — an HTTP header auth token for the external data service. Both the primary and fallback endpoints use the same credential. The workflow works with pinned demo data if you skip this.
