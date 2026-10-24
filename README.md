# AI Inbox Triage Starter

**An n8n workflow that labels every incoming email and writes reply drafts for the ones that need an answer. It never sends anything.**

![License: MIT](https://img.shields.io/badge/License-MIT-green.svg) ![n8n](https://img.shields.io/badge/built%20for-n8n-EA4B71.svg) ![Drafts only](https://img.shields.io/badge/safety-drafts--only-blue.svg)

Leads sit unanswered, invoices get buried, newsletters crowd the important mail. This is the starter version of an email-triage pattern used in paid retainer builds.

## What it does

- Classifies each new email as `urgent`, `lead`, `invoice`, `newsletter` or `other`.
- Writes a draft reply for urgent, lead and invoice emails and saves it to your Drafts folder.
- Flags anything involving money, legal matters or complaints with `needs_human`.
- Builds a daily digest of what was triaged and which drafts are ready.
- Routing rules are plain JSON in `rules/triage-rules.example.json`; the exact AI prompts are in `docs/PROMPTS.md`.
- Bring your own AI key: OpenAI, Groq (free tier) or Gemini by changing the base URL and model.

## Quickstart

1. Run n8n: [self-host with Docker](https://docs.n8n.io/hosting/) or use n8n Cloud.
2. **Workflows > Import from File** and pick `workflows/inbox-triage-starter.n8n.json`.
3. Open the two Gmail nodes and add your Gmail OAuth2 credential (IMAP also works, see `docs/SETUP.md`).
4. Open the two LLM nodes and add an HTTP Header Auth credential with `Authorization: Bearer YOUR_KEY`.
5. Run **Execute Workflow** on recent unread mail, check your Drafts folder, then set the workflow **Active**.

Full walkthrough and safety notes: [docs/SETUP.md](docs/SETUP.md).

## Safety

- The workflow creates drafts only. It has no send step, so you are always the last click.
- It reads and labels. It does not delete or archive existing mail.
- Credentials stay in your own n8n instance; none are committed here.

## For you if

You run a small business, agency, clinic or shop, your inbox is the front door, and you want triage without handing over send access.

## Status and limits

- Classification quality depends on the model you pick and on tuning the rules to your mail. Review drafts before sending.
- It is a starter, not a full support desk.
- Last reviewed October 2026.

## Built by SkynetLabs

Built by [SkynetLabs](https://skynetjoe.com). Questions: info@skynetjoe.com

Need this wired into your business? SkynetLabs builds voice agents, n8n back offices and sites that convert → https://skynetjoe.com
