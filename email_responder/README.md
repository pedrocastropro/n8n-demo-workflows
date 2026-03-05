# Email Responder — AI-Powered Support Pipeline

## How it works

This workflow implements a complete AI-powered support email pipeline. It reads unread emails from Gmail, reconstructs conversation threads, classifies them with three parallel AI agents, searches a Google Drive knowledge base for relevant documentation, and generates a professional response draft — all automatically.

## Architecture

```
BLOCK 1: MAIL → CONVERSATION (8 nodes)
  ManualTrigger → GetUnreadInbox → RemoveDuplicates → GetThreads
  → ThreadParser → GetMessages → RebuildThreads → FormatForLLM

BLOCK 2: INTENT RESOLVER (13 nodes)
  Conversations → [Area + Intent + Status + Merge(0)] in parallel
  Each agent: OpenAI Chat Model + Structured Output Parser → Extract → Merge

BLOCK 3: KNOWLEDGE BASE SEARCH (10 nodes)
  Merge → Conversation+Meta → Build Search Query
  → [Search Drive → Download → File to Text → Merge Docs(1)] + Merge Docs(0)
  → KB Search Agent → Extract KB Results → Merge All(1)

BLOCK 5: RESPONSE BUILDER (5 nodes)
  Merge All → Response Agent (gpt-4o) → Extract Response → Create Gmail Draft
```

Block 4 (Gmail Search Queries) is excluded — it was a disconnected mock/test section in the original design that doesn't feed into any downstream node.

## Block details

### Block 1: Mail → Conversation

Fetches unread inbox messages, deduplicates by thread ID, reconstructs full threads with messages sorted chronologically, and cleans the text for LLM consumption.

- Strips quoted reply blocks (On ..., El ..., -----Original Message-----)
- Labels messages as "support" or "user" based on sender domain
- Outputs `{ threadId, subject, initiatedBy, identifiedUser, conversationHistory }`

### Block 2: Intent Resolver

Three AI agents run in parallel on each conversation using gpt-4o-mini:

- **Area**: classifies which product module the email relates to (accounts, billing, product, support) with sub-areas
- **Intent**: determines what the user wants (how_to, not_working, information, other)
- **Status**: evaluates conversation state (new, waiting, solved, reopened)

All results merge back with the original conversation data via a 4-input position merge.

### Block 3: Knowledge Base Search

Takes the classified conversation, builds a search query from the area/sub_area, and searches Google Drive for matching documentation files. Downloads and converts them to text, then runs a KB Search Agent (gpt-4o-mini) to extract the most relevant sections for the user's question. Returns up to 3 search results with relevance scores.

### Block 5: Response Builder

Uses gpt-4o with the full context — conversation history, classification, and KB results — to generate a professional support response. The agent:

- Replies in the same language the user wrote in
- Cites knowledge base information naturally
- Adjusts tone based on status (new vs reopened vs solved)
- Creates a Gmail draft in the original thread, ready for human review before sending

## Output structure

Each processed thread results in a Gmail draft containing:

```json
{
  "response": {
    "subject": "Re: Original subject",
    "body": "Professional support response...",
    "signature": "Best regards,\nSupport Team"
  }
}
```

The intermediate enriched object includes:

```json
{
  "threadId": "...",
  "subject": "...",
  "identifiedUser": "client@example.com",
  "conversationHistory": [...],
  "area": { "description": "product", "sub_area": ["api"], "confidence": 0.95 },
  "intent": { "description": "not_working", "confidence": 0.9 },
  "status": { "value": "new", "confidence": 0.95 },
  "searchResults": [
    { "title": "API Troubleshooting", "content": "...", "relevance": 0.92 }
  ]
}
```

## Credentials needed

- **Gmail OAuth2** — for reading emails, threads, and creating drafts
- **OpenAI API** — for all AI agents (gpt-4o-mini for classification/KB, gpt-4o for response)
- **Google Drive OAuth2** — for searching and downloading knowledge base documents

Set the Gmail and Google Drive OAuth redirect URI to `http://localhost:5678/rest/oauth2-credential/callback` and complete the sign-in through n8n. For OpenAI, add your API key in n8n credentials.

## Setup

1. Edit `supportDomains` in the "Format for LLM" node with your support team's email addresses
2. Customize the Area agent's system prompt with your product's actual modules and sub-areas
3. Upload knowledge base documents to Google Drive, named by area (e.g., `accounts-password_reset.md`)
4. Add your OpenAI API key as a credential in n8n
5. Connect Gmail and Google Drive OAuth2 credentials
