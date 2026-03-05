# Client Onboarding — Automated Welcome Sequence

## How it works

When a new client signs up, this workflow takes care of the entire onboarding process. It receives the signup data through a webhook, saves the client information to Google Sheets, creates a dedicated folder in Google Drive, and sends a series of three welcome emails spread over five days. The whole thing runs hands-free after the initial trigger.

## Key features

- Receives signups in real time via a POST webhook and responds immediately
- Stores client details (name, email, company, plan, date, status) in Google Sheets
- Creates a Google Drive folder named "[Company] - [Name]" for each new client
- Sends three emails over five days: welcome, getting started, and a check-in
- Uses email as a matching key so re-submissions update instead of duplicating rows

## Step by step

**Webhook:**
Listens for POST requests at `/client-onboarding`. Expects a JSON body with `name`, `email`, `company`, and `plan`.

**Respond to Webhook:**
Returns a 200 OK right away so the caller gets instant confirmation while the rest of the pipeline runs in the background.

**Google Sheets — Add Client:**
Writes the client's name, email, company, plan, today's date, and "Active" status to the "Clients" sheet. If the email already exists it updates the row instead of adding a duplicate.

**Google Drive — Create Folder:**
Creates a folder called "[Company] - [Name]" (for example "Acme Corp - Jane Doe") where documents for this client can be stored.

**Welcome Email (Day 0):**
Sends an HTML email introducing the client to the platform, confirming their plan, and letting them know what to expect over the next few days.

**Wait 2 Days:**
Pauses the workflow for two days.

**Getting Started Email (Day 2):**
Sends a second email with practical first steps — exploring the Drive folder, setting up a profile, inviting teammates, and checking out guides.

**Wait 3 Days:**
Pauses for three more days.

**Check-in Email (Day 5):**
Sends a final follow-up asking how things are going and offering help.

## Credentials needed

- **Google Sheets OAuth2** — for reading and writing client data to the spreadsheet
- **Google Drive OAuth2** — for creating client folders
- **Gmail OAuth2** — for sending the three onboarding emails

All three use the same Google Cloud project. Set the OAuth redirect URI to `http://localhost:5678/rest/oauth2-credential/callback` and complete the sign-in through n8n.
