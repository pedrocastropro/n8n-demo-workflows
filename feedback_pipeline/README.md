# Customer Feedback Pipeline and Daily Digest

## How it works

This workflow handles customer feedback in two ways. First, it receives feedback in real time through a webhook — every submission gets saved to Google Sheets and, if the rating is low, an urgent alert goes straight to Slack. Second, a daily schedule trigger runs every morning at 9 AM, pulls all of yesterday's feedback, calculates stats like average rating and urgency counts, and posts a digest to Slack. It shows how you can combine a webhook and a schedule trigger in a single workflow.

## Key features

- Accepts feedback submissions via POST webhook with instant confirmation
- Saves every entry to Google Sheets with a timestamp and unique ID
- Routes low ratings (1-2 out of 5) to a Slack alert channel immediately
- Logs standard feedback to a separate Slack channel
- Runs a daily digest at 9 AM with average rating, urgency breakdown, and highlights
- Skips the digest if there was no feedback the previous day
- Includes pinned demo data so you can test both paths without any external setup

## Step by step

### Real-time path (webhook)

**Webhook — New Feedback:**
Listens for POST requests at `/webhook/feedback`. Expects a JSON body with `name`, `email`, `rating` (1-5), `message`, and `source`.

**Add Metadata:**
Stamps each submission with a `received_at` timestamp and a unique `feedback_id`, then passes everything along.

**Save to Google Sheets:**
Appends the feedback entry to the "Feedback" sheet using a Service Account.

**Urgency Check:**
If the rating is 2 or below, the feedback is treated as urgent and routed to the alert path. Ratings of 3 and above go to the standard log.

**Slack Urgent Alert:**
Posts a detailed message to #feedback-alerts with the customer's name, email, rating, message, source, and feedback ID.

**Slack Log Entry:**
Posts a lighter message to #feedback-log with the name, rating, and message.

**Respond OK:**
Returns a 200 response to the caller confirming the feedback was received.

### Daily digest path (schedule)

**Daily at 9:00 AM:**
A schedule trigger fires every morning to generate the digest.

**Read All Feedback:**
Reads every row from the "Feedback" sheet.

**Aggregate Yesterday:**
A code node filters to only yesterday's entries and calculates the total count, average rating, number of urgent and positive responses, source breakdown, and top three highlights.

**Has Feedback?:**
Checks whether any feedback came in yesterday. This prevents sending an empty digest.

**Slack Daily Digest:**
If there is data, posts a summary to #daily-reports with all the aggregated stats.

**Slack No Data:**
If nothing came in, posts a short "all quiet" message instead.

## Credentials needed

- **Google Service Account** — for saving and reading feedback in Google Sheets. Create a Service Account in Google Cloud Console, download the JSON key, and share your sheet with the service account email.
- **Slack OAuth2** — for posting alerts and digest messages. Create a Slack App, add bot token scopes (`chat:write`, `channels:read`, `channels:join`), and complete the OAuth flow.
