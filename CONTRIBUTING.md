# Contributing

Thanks for wanting to make this starter kit better. It is meant to stay small, safe, and genuinely useful for non-technical small business owners.

## Ground rules

- **Drafts-only, always.** Any change that could cause the workflow to auto-send, auto-delete, or auto-archive email will not be merged. The workflow must end at a draft a human reviews.
- **No secrets.** Never commit an API key, credential, `.env`, or a real inbox export. Use the placeholder credential names already in the workflow.
- **Keep it importable.** The n8n JSON must stay valid and pure ASCII so it imports cleanly on any instance. Validate before opening a PR:
  ```bash
  python -c "import json; json.load(open('workflows/inbox-triage-starter.n8n.json'))"
  ```
- **Plain language.** Docs are written for owners, not engineers. Prefer a clear sentence over jargon.

## Good first contributions

- IMAP / Outlook variants of the workflow.
- A digest node that posts to Slack, Discord, or a Google Doc.
- More routing-rule examples for specific niches (clinics, trades, e-commerce).
- Translations of the README and SETUP guide.

## How to submit

1. Fork the repo and create a branch.
2. Make your change; test the import in a real n8n instance.
3. Open a PR describing what changed and why, and confirming the drafts-only rule still holds.

Questions or a bigger idea? Open an issue.
