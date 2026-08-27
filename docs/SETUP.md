# Setup Guide

Get the AI Inbox Triage Starter running in about 20 minutes. No prior n8n experience needed.

Everything here is **drafts-only**: the workflow reads and labels your mail and writes reply drafts. It never sends. You are always the last step.

---

## 1. Prerequisites

You need three things:

1. **An n8n instance.** Two options:
   - **Self-hosted (free):** the fastest path is Docker. See the [official guide](https://docs.n8n.io/hosting/). One command:
     ```bash
     docker run -it --rm -p 5678:5678 -v n8n_data:/home/node/.n8n docker.n8n.io/n8nio/n8n
     ```
     Then open `http://localhost:5678`.
   - **n8n Cloud:** start a [trial](https://n8n.io/cloud/) - no server to manage.
2. **An email account** you want triaged. This guide uses **Gmail**; IMAP works too (see section 3b).
3. **An AI API key.** Any OpenAI-compatible provider works:
   - **Groq** - has a generous free tier, fastest to start: [console.groq.com](https://console.groq.com)
   - **OpenAI** - [platform.openai.com](https://platform.openai.com)
   - **Google Gemini** - via its [OpenAI-compatible endpoint](https://ai.google.dev/gemini-api/docs/openai)

## 2. Import the workflow

1. In n8n, click **Workflows** -> the **...** menu (top right) -> **Import from File**.
2. Choose `workflows/inbox-triage-starter.n8n.json` from this repo.
3. You will see the graph: **Gmail Trigger -> Config -> Classify -> Parse -> Needs a Draft? -> Draft -> Create Draft**, with a **Daily Digest** branch. The nodes show credential warnings until you complete the next steps - that is expected.

## 3a. Connect Gmail (recommended)

1. Click the **Gmail Trigger** node -> **Credential to connect with** -> **Create New Credential**.
2. Choose **Gmail OAuth2**. Follow n8n's [Gmail credential guide](https://docs.n8n.io/integrations/builtin/credentials/google/) to create Google OAuth credentials and paste the Client ID / Secret, then sign in.
3. Do the same on the **Create Gmail Draft (no send)** node - or just pick the credential you just made from the dropdown.

That is it - both Gmail nodes now use your inbox.

## 3b. Prefer IMAP instead of Gmail?

If you are not on Gmail (Outlook, a cPanel mailbox, etc.):

1. Delete the **Gmail Trigger** node and add an **Email Trigger (IMAP)** node in its place.
2. Add IMAP credentials (host, port 993, your email + app password).
3. Wire it into **Config** exactly as the Gmail Trigger was.
4. For drafts: replace **Create Gmail Draft** with your provider's draft/append method, or an **IMAP -> Append** to the Drafts folder. The rest of the workflow is unchanged.

## 4. Swap in your own AI key

The two nodes named **Classify (LLM)** and **Draft Reply (LLM)** call an OpenAI-compatible chat endpoint. You add your key once as a credential and both nodes use it.

1. Open **Classify (LLM)** -> **Credential for Header Auth** -> **Create New Credential** -> type **Header Auth**.
2. Set:
   - **Name:** `Authorization`
   - **Value:** `Bearer YOUR_API_KEY`
3. Save. Select the same credential on **Draft Reply (LLM)**.
4. Open the **Config** node and set the base URL + model for your provider:

| Provider    | `LLM_BASE_URL`                                                             | `LLM_MODEL` (example)     |
| ----------- | -------------------------------------------------------------------------- | ------------------------- |
| Groq (free) | `https://api.groq.com/openai/v1/chat/completions`                          | `llama-3.3-70b-versatile` |
| OpenAI      | `https://api.openai.com/v1/chat/completions`                               | `gpt-4o-mini`             |
| Gemini      | `https://generativelanguage.googleapis.com/v1beta/openai/chat/completions` | `gemini-2.0-flash`        |

Also in **Config**, set `OWNER_NAME` and `BUSINESS_NAME` so drafts sign off correctly.

> Note: if your provider does not support `response_format: json_object` (some do not), remove that one line from the **Classify (LLM)** node's JSON body. The parser tolerates plain-text JSON too.

## 5. The 12 routing rules explained

Classification is driven by the labels and rules in [`rules/triage-rules.example.json`](../rules/triage-rules.example.json). The AI reads each email and picks the best-fitting label using this logic. The rules, in plain English:

| #   | Label      | Fires when                                                            | Draft? | Human flag |
| --- | ---------- | --------------------------------------------------------------------- | ------ | ---------- |
| 1   | urgent     | Sender asks for a reply today / says urgent / same-day cancellation   | yes    | yes        |
| 2   | urgent     | Existing customer reports something broken or a failed payment        | yes    | yes        |
| 3   | lead       | New person asks about services, pricing, or a quote                   | yes    | no         |
| 4   | lead       | Someone requests a meeting, demo, or call                             | yes    | no         |
| 5   | lead       | Referral or warm introduction                                         | yes    | no         |
| 6   | invoice    | An invoice, bill, receipt, or supplier chasing payment                | yes    | yes        |
| 7   | invoice    | Customer asks about their invoice, a refund, or billing               | yes    | yes        |
| 8   | newsletter | Bulk marketing, product updates, content digests                      | no     | no         |
| 9   | newsletter | Automated no-reply platform notifications                             | no     | no         |
| 10  | other      | Internal chatter, personal mail, anything unmatched                   | no     | no         |
| 11  | urgent     | Legal, lawyer, chargeback, or public-review threat -> human, no draft | no     | yes        |
| 12  | other      | AI confidence below 0.5 -> default to other and flag for a human      | no     | yes        |

To retune: edit the `signals` keywords, flip `draft` true/false, or change `needs_human`. Rules 11 and 12 are safety nets - keep them.

## 6. Test it, then go live

1. Make sure you have a couple of **unread** emails in your inbox.
2. Click **Execute Workflow** (manual run). Watch the nodes light up green.
3. Open your **Drafts** folder. You should see AI-drafted replies for anything labelled urgent / lead / invoice, each tagged `[AI-drafted by Inbox Triage Starter - review before sending]`.
4. Read the **Build Daily Digest** node output for the summary.
5. Happy with it? Toggle the workflow **Active** (top right). The Gmail Trigger polls for new mail on a schedule.

## 7. Safety notes (read these)

- **Drafts-only, by design.** The workflow has no send node. It calls Gmail's _create draft_ action. To keep it safe, never swap that for a _send_ action unless you fully understand the consequences.
- **Review before sending.** AI drafts are a starting point, not final copy. Always read them - especially anything flagged `[NEEDS HUMAN]` (money, legal, complaints).
- **Your keys stay yours.** Credentials live in your n8n instance only. Nothing in this repo contains secrets, and your `.env` / credentials should never be committed (see `.gitignore`).
- **Start conservative.** Run it manually for a day or two before flipping it Active, so you trust what it drafts.
- **Rate + cost.** Every email triggers 1-2 AI calls. On a busy inbox, set the trigger poll interval sensibly and use a cheap model (e.g. `gpt-4o-mini` or Groq's free Llama) for classification.

---

## Troubleshooting

- **Classify node errors with 401** -> your Header Auth value must be exactly `Bearer YOUR_KEY` (with the word `Bearer` and a space).
- **Draft body is empty** -> your provider returned an unexpected shape; open the **Extract Draft Text** node output. Most OpenAI-compatible APIs use `choices[0].message.content`, which the code handles.
- **JSON parse error on classify** -> remove `response_format` from the Classify node body (some models reject it) and re-run.
- **No drafts appear** -> confirm the emails were labelled urgent / lead / invoice (only those three draft), and that the Gmail draft node uses the same credential as the trigger.

Stuck, or want it wired into a real business inbox with monitoring and escalation? **[Book a free systems audit.](https://calendly.com/skynetlabs/schedule-a-free-consultation)**
