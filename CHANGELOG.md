# Changelog

All notable changes to this project are documented in this file.

## [2026.09] - 2026-09-16

- Maintenance review of ai-inbox-triage-starter — a public, MIT-licensed starter kit that triages an inbox with AI and writes reply drafts inside n8n, without ever auto-sending.
- Status: documentation + assets, not an application — one importable workflow (`workflows/inbox-triage-starter.n8n.json`), editable routing rules (`rules/triage-rules.example.json`), `docs/SETUP.md` and `docs/PROMPTS.md`, a CONTRIBUTING guide and a README covering the drafts-only safety model, the five classification buckets and bring-your-own-key setup (OpenAI, Groq or Gemini). No secrets are committed.
- Reviewed September 2026: README Status section added and a CHANGELOG created; the repo is versioned as v2026.09. The workflow JSON, rules and prompts were not changed.
- Known gaps: no CHANGELOG before this release; no CI or JSON validation for the workflow file; the workflow is pinned to whatever n8n node versions it was exported with in August 2026 and has not been re-tested against a newer n8n release.
