# n8n Workflow Templates

![n8n](https://img.shields.io/badge/n8n-workflow%20automation-FF6D5A?logo=n8n&logoColor=white)
![Workflows](https://img.shields.io/badge/workflows-6%20demos-blue)
![OpenAI](https://img.shields.io/badge/OpenAI-GPT--4o-412991?logo=openai&logoColor=white)
![Gmail](https://img.shields.io/badge/Gmail-integration-EA4335?logo=gmail&logoColor=white)
![Google Drive](https://img.shields.io/badge/Google%20Drive-integration-4285F4?logo=googledrive&logoColor=white)
![Slack](https://img.shields.io/badge/Slack-integration-4A154B?logo=slack&logoColor=white)
![License](https://img.shields.io/badge/license-MIT-green)

A curated collection of n8n workflow demos inspired by real production workflows — covering common automation patterns from AI-powered email support to resilient API pipelines.

Each workflow is a self-contained demo with its own documentation, credential placeholders, and (where applicable) pinned test data so you can import and run immediately.

---

## Workflows

### AI & Email

| Workflow | Description |
|----------|-------------|
| [**Email Responder**](./email_responder/) | Full AI support pipeline — classifies emails with 3 parallel agents, searches a Google Drive knowledge base, and drafts responses using GPT-4o |
| [**Email Analysis**](./email_analysis/) | Reconstructs Gmail threads, cleans quoted replies, and exports a formatted report to Google Sheets |

### Data Pipelines

| Workflow | Description |
|----------|-------------|
| [**Multi-Platform Data Sync**](./multi_platform_data_sync/) | Hourly lead enrichment — pulls from Sheets, scores leads, routes to Airtable (Hot vs Nurturing), notifies Slack |
| [**Resilient API Pipeline**](./resilient_api_pipeline/) | API polling with retries, fallback endpoints, anomaly detection, error logging, and tiered Slack alerts |

### Customer Operations

| Workflow | Description |
|----------|-------------|
| [**Client Onboarding**](./client-onboarding/) | Automated welcome sequence — webhook signup, Sheets storage, Drive folder creation, 3 drip emails over 5 days |
| [**Feedback Pipeline**](./feedback_pipeline/) | Real-time feedback collection via webhook with low-rating Slack alerts and a daily digest at 9 AM |

---

## Quick start

### Option A: Run locally with n8nLauncher

The fastest way to try these workflows is with [**n8nLauncher**](https://github.com/pedrocastropro/n8nLauncher) — a local n8n instance manager that spins up isolated Docker environments with one command.

```bash
# 1. Clone the launcher
git clone https://github.com/pedrocastropro/n8nLauncher.git
cd n8nLauncher

# 2. Start a local n8n instance
./manager.sh start

# 3. Clone the workflows into the instance
git clone https://github.com/pedrocastropro/n8n-demo-workflows.git workflows

# 4. Import a workflow
./manager.sh update
```

Your n8n instance will be available at `http://localhost:5678`. Open it, add your credentials, and execute any workflow.

### Option B: Import into an existing n8n instance

```bash
# 1. Clone
git clone https://github.com/pedrocastropro/n8n-demo-workflows.git
cd n8n-demo-workflows

# 2. Pick a workflow and import it into your n8n instance
#    Copy/paste the workflow.json into n8n's import dialog
#    or use the n8n CLI:
n8n import:workflow --input=email_responder/workflow.json

# 3. Add your credentials in n8n (Settings → Credentials)
# 4. Execute the workflow
```

## Credentials overview

Each workflow lists its required credentials in its own README. Here's a quick reference:

| Credential | Used by |
|------------|---------|
| Gmail OAuth2 | email_responder, email_analysis, client-onboarding |
| Google Sheets OAuth2 | email_analysis, client-onboarding |
| Google Service Account | feedback_pipeline, multi_platform_data_sync, resilient_api_pipeline |
| Google Drive OAuth2 | email_responder, client-onboarding |
| OpenAI API | email_responder |
| Slack OAuth2 | feedback_pipeline, multi_platform_data_sync, resilient_api_pipeline |
| Airtable PAT | multi_platform_data_sync |

All credential files are **empty placeholders** — no API keys or tokens are included. Configure them through your n8n instance.

## Project structure

```
workflows/
├── README.md
├── client-onboarding/
│   ├── workflow.json
│   ├── README.md
│   └── credentials/
├── email_analysis/
│   ├── workflow.json
│   ├── README.md
│   └── credentials/
├── email_responder/
│   ├── workflow.json
│   ├── README.md
│   └── credentials/
├── feedback_pipeline/
│   ├── workflow.json
│   ├── README.md
│   └── credentials/
├── multi_platform_data_sync/
│   ├── workflow.json
│   ├── README.md
│   └── credentials/
└── resilient_api_pipeline/
    ├── workflow.json
    ├── README.md
    └── credentials/
```

## License

MIT
