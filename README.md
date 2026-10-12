# Client Onboarding Orchestrator

A Docker-hosted n8n automation that takes a new client from signup to internal project handoff without anyone doing the admin by hand.

**Demo:** https://lnkd.in/p/dFpNi6zq

---

## The problem

Onboarding a new client is a pile of small tasks: send the welcome email, make the folder, ask the questions, read the answers, update the team. None of it is hard, but repeating it for every client is slow and easy to do inconsistently.

## What it does

Two connected workflows handle the whole process.

### Workflow A: New client onboarding

```
Client Signup Form
→ Normalize Data
→ Check Duplicate by Email
→ Create Google Drive Folder
→ Add Client Record to Google Sheets
→ Send Welcome Email with Intake Form
```

If the email already exists, the workflow stops, no duplicate folder or record is created, and an internal alert is sent.

**Signup form collects:** Client Name, Email, Service Tier, Project Goal

### Workflow B: Intake processing

```
Client Intake Form
→ Normalize Answers
→ Send Data to Gemini API
→ Generate Project Brief
→ Find Client by Email
→ Update Existing Sheet Row
→ Send Internal Notification
```

**Intake form collects:** Client Name, Email, Success Criteria, Constraints / Must-haves, Communication Preference

Gemini turns the raw answers into a concise project brief. The same client row is then updated:

| Field | Before | After |
|---|---|---|
| Status | Onboarding started | Intake complete |
| Notes/Brief | Pending intake | AI-generated project brief |

## Google Sheet structure

| Column | Purpose |
|---|---|
| Client Name | From signup form |
| Email | Unique key for duplicate checks and row updates |
| Service Tier | From signup form |
| Project Goal | From signup form |
| Status | Tracks onboarding stage |
| Notes/Brief | Holds the AI-generated brief |
| Drive Folder URL | Link to the client's folder |
| Created At | Timestamp |

## Result

- Captures client details
- Prevents duplicate clients
- Creates a Drive folder per client
- Stores client records
- Sends onboarding emails
- Collects detailed requirements
- Summarizes requirements with AI
- Updates the client record
- Notifies the internal team

## Tech stack

- n8n (self-hosted)
- Docker / Docker Compose
- Google Drive
- Google Sheets
- Gmail
- Gemini API (HTTP Request node)
- OAuth 2.0
- n8n Forms

## Setup

1. Run n8n with Docker Compose. `WEBHOOK_URL` and `N8N_HOST` must match the URL you open n8n at, otherwise Google OAuth redirects fail.
2. Create credentials in n8n: Google Drive, Google Sheets and Gmail (OAuth 2.0), plus a Gemini API key.
3. Create the Google Sheet with the columns listed above.
4. Import the two workflow JSON files into n8n.
5. Point the Drive, Sheets and Gmail nodes at your own folder, sheet and sender address, and link the welcome email to your intake form URL from Workflow B.
6. Activate both workflows and submit a test client through the signup form.

Never commit credentials or `.env` files.

## Limitations and next steps

- The Gemini brief goes straight into the client record with no confidence check. Next step: flag short or contradictory intake answers for a quick human review before the team sees the brief.
- Signup is a form trigger. It could be swapped for a payment or contract webhook.
