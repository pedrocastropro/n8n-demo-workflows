# Email Analysis — Thread Export to Google Sheets

## How it works

This workflow reads emails from a Gmail label, reconstructs full conversation threads, and exports them to a formatted Google Sheets report. It handles the complexity of Gmail's thread model — where a single thread can contain multiple messages — by fetching each message individually, grouping them back into threads sorted chronologically, and cleaning up quoted reply text. The result is a clean spreadsheet with one row per conversation, ready for manual review or further processing.

## Key features

- Fetches emails by Gmail label and deduplicates by thread ID
- Reconstructs full threads from individual messages sorted by date
- Strips quoted reply blocks (On ..., El ..., -----Original Message-----) to keep only the actual content
- Identifies support vs. user roles based on configurable email domains
- Creates a new sheet tab named with a timestamp so each run is isolated
- Formats the output sheet with styled headers, alternating row colors, borders, and column widths

## Step by step

**Manual Trigger:**
Click "Execute workflow" to start. This lets you run the analysis on demand.

**Get Emails by Label:**
Fetches all messages matching the configured Gmail label. Change the label filter to target the emails you want to analyze.

**Remove Duplicates:**
Multiple messages can belong to the same thread. This node deduplicates by `threadId` so each thread is processed once.

**Get Threads:**
Fetches the full thread object for each unique thread ID from Gmail.

**Thread Parser:**
Extracts individual `messageId` + `threadId` pairs from each thread's message array, flattening them into separate items.

**Get Messages:**
Fetches the full content of each individual message (headers, body, metadata) using `simple: false` for the raw data.

**Rebuild Threads:**
Groups all fetched messages back into threads using a Map keyed by `threadId`, then sorts messages within each thread by `internalDate`.

**Format for LLM:**
Processes each thread into a clean structure:
- Strips quoted reply text (On ..., El ..., etc.)
- Labels each message as "support" or "user" based on sender domain
- Identifies who initiated the conversation and who the client is
- Outputs `{ threadId, subject, initiatedBy, identifiedUser, conversationHistory }`.

**Create Analysis Sheet:**
Creates a new sheet tab in the target spreadsheet, named with the current timestamp (e.g., `20260302153045`).

**Create Headers + Append Headers:**
Writes the column headers (SUMMARY, FULL CONVERSATION, AREA) as the first row.

**Conversation to Text:**
Converts each thread's conversation history into a readable text block with timestamps and role labels.

**Update Rows:**
Writes the formatted conversation text into the spreadsheet rows.

**Wait + Format Sheet:**
After all rows are written, the workflow pauses briefly then calls the Google Sheets API directly to apply formatting: header styling, column widths, alternating row colors, text wrapping, and borders.

## Credentials needed

- **Gmail OAuth2** — for reading emails and thread data
- **Google Sheets OAuth2** — for creating sheets, writing data, and formatting via the Sheets API

Both use the same Google Cloud project. Set the OAuth redirect URI to `http://localhost:5678/rest/oauth2-credential/callback` and complete the sign-in through n8n.

## Setup

1. Update the label filter in the "Get Emails by Label" node to match your Gmail label
2. Edit `supportDomains` in the "Format for LLM" node with your support team's email addresses
3. Set your target spreadsheet ID in the "Create Analysis Sheet" node