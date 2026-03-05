# Multi-Platform Data Sync — Lead Enrichment Pipeline

## How it works

Every hour, this workflow pulls new leads from a Google Sheet, enriches each one with company data from an external API, scores them based on company size, industry, and data completeness, and then routes them to separate Airtable tables depending on whether they qualify for sales outreach or need more nurturing. After routing, it updates the original sheet with the sync status and posts a summary to Slack so the team knows the pipeline ran.

## Key features

- Runs automatically every hour with no manual work
- Enriches leads with company details like employee count, industry, revenue, LinkedIn, and phone
- Calculates a lead score using weighted signals across multiple dimensions
- Routes qualified leads (score 50+) to the "Hot Leads" Airtable table and the rest to "Nurturing"
- Updates the Google Sheet with sync status and qualification results
- Sends a summary notification to Slack after each run
- Includes pinned demo data so you can test the whole pipeline without real API connections

## Step by step

**Schedule Trigger:**
Fires every hour to check for new leads.

**Read Leads from Google Sheets:**
Connects to Google Sheets via a Service Account and reads all rows from the "Leads" sheet. Expected columns: `name`, `email`, `company`, `company_domain`, `source`, `status`.

**Filter New Leads:**
Only passes through leads where `status` is "new", skipping anything that has already been processed.

**Enrich Company Data:**
For each new lead, calls an enrichment API with the company domain. Returns details like employee count, industry, revenue, LinkedIn URL, phone, and headquarters. If a lookup fails, the pipeline keeps going — one bad request won't block the rest.

**Transform and Score:**
Merges the original lead data with the enrichment results and calculates a lead score:
- Company size: 500+ employees (+30), 100-500 (+20), 20-100 (+10)
- High-value industry like Technology, Finance, Healthcare, or SaaS (+25), others (+10)
- Has LinkedIn URL (+10), has phone (+10), has revenue data (+15)
- Score of 50 or above means "Qualified", below that means "Needs Nurturing"

**Qualification Routing:**
An IF node checks the score. Leads at 50 or above go to the "Hot Leads" Airtable table. The rest go to the "Nurturing" table.

**Merge Results:**
Combines the output from both Airtable branches so every processed lead moves to the next step regardless of which table it landed in.

**Update Sheet Status:**
Writes back to Google Sheets, setting each lead's status to "synced" along with the sync timestamp, lead score, and qualification result. Uses email as the matching column.

**Notify Team on Slack:**
Posts a message to #leads-pipeline with the number of leads processed and the timestamp.

## Credentials needed

- **Google Service Account** — for reading leads and writing sync status to Google Sheets. Create a Service Account in Google Cloud Console, download the JSON key, and share your sheet with the service account email.
- **Airtable Personal Access Token** — for creating records in your Airtable base. Create a token at airtable.com/create/tokens with scopes `data.records:read`, `data.records:write`, and `schema.bases:read`.
- **Slack OAuth2** — for posting pipeline notifications. Create a Slack App, add bot token scopes (`chat:write`, `channels:read`, `channels:join`), and complete the OAuth flow.
- **Enrichment API Key (optional)** — an HTTP header auth token for the company enrichment service. The workflow works fine with pinned demo data if you skip this.
