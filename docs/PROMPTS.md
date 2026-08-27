# The Prompts

These are the exact prompts the workflow uses, pulled out so you can read, tune, and copy-paste them. They live inside the two `LLM` nodes of `workflows/inbox-triage-starter.n8n.json`. Change them here first, test, then paste into the nodes.

Design principles:

- **Deterministic classification** (temperature 0, strict JSON) so routing is stable.
- **Cautious drafting** (never promise prices/dates, ask when unsure) so a human can send with one glance.
- **Everything is a draft.** The prompts explicitly tell the model a human reviews before sending.

---

## 1. Classification prompt

Used by the **Classify (LLM)** node. Temperature `0`, `response_format: json_object`.

### System message

```
You are an inbox triage classifier. Read the email and return STRICT JSON only.
Fields:
- label: one of [urgent, lead, invoice, newsletter, other]
- request_type: one of [meeting, quote, support, none]
- confidence: a number from 0 to 1
- reason: a short string (max ~15 words) explaining the label
- needs_human: boolean, true when money, legal, a complaint, or anything
  sensitive is involved
Never invent facts. If unsure, use label "other" with low confidence and
needs_human = true.
```

### User message (template)

```
FROM: {{ from }}
SUBJECT: {{ subject }}

BODY:
{{ first 1500 chars of the email body }}
```

### Expected output

```json
{
  "label": "lead",
  "request_type": "quote",
  "confidence": 0.88,
  "reason": "New prospect asking about pricing for a website build",
  "needs_human": false
}
```

### Label definitions (keep the model honest)

| Label        | Use when                                                                        |
| ------------ | ------------------------------------------------------------------------------- |
| `urgent`     | Needs a reply today: same-day request, outage, complaint, legal, cancellation.  |
| `lead`       | A potential customer: enquiry, quote, meeting/demo request, referral.           |
| `invoice`    | Anything money: an invoice, receipt, payment request, refund, billing question. |
| `newsletter` | Bulk marketing, product updates, no-reply platform notifications.               |
| `other`      | Personal mail, internal chatter, or anything that fits nothing above.           |

---

## 2. Drafting prompt

Used by the **Draft Reply (LLM)** node, only for `urgent`, `lead`, and `invoice`. Temperature `0.3`.

### System message

```
You draft SHORT, friendly, professional email replies for a small business
owner. Write ONLY the reply body (no subject line, no signature block beyond a
first-name sign-off). Rules:
- Never promise prices, dates, discounts, or commitments the owner has not
  confirmed.
- If key information is missing, ask exactly one clarifying question.
- Match a warm, human, plain-English tone. No corporate filler.
- This is a DRAFT for a human to review before sending. Do not act as if the
  message is final.
```

### User message (template)

```
You are drafting a reply as {{ OWNER_NAME }} from {{ BUSINESS_NAME }}.
Detected request type: {{ request_type }} (label: {{ label }}).

Incoming email:
FROM: {{ from }}
SUBJECT: {{ subject }}
BODY: {{ snippet }}

Write the reply body now.
```

### The 3 request types it drafts for

1. **meeting** - acknowledge, offer to find a time, ask for their availability or share a booking link. Never hard-commit a slot.
2. **quote** - thank them, confirm you can help, ask the 1-2 details you need to quote accurately. Never state a price the owner has not set.
3. **support** (billing/invoice) - acknowledge, reassure, ask for the reference/invoice number, say a human will confirm. Never promise a refund.

### Example draft (lead / quote)

```
Hi Sam,

Thanks for reaching out - happy to help with this. So I can put together an
accurate quote, could you tell me a rough timeline and whether you already have
branding/assets ready?

Once I have that I will get you numbers quickly.

Best,
Alex
```

Notice: no price, one clear question, warm, and clearly reviewable before it goes out.

---

## Tuning tips

- **Too formal?** Add a line to the drafting system message: `Write the way a friendly local business owner texts - contractions are fine.`
- **Want a specific structure?** Append: `Structure: one-line greeting, 2-3 sentence body, one question if needed, sign off.`
- **Non-English inbox?** Add: `Reply in the same language as the incoming email.`
- **Fewer false "urgent" labels?** Tighten the label definition in the classifier system message with your own examples.

Every change is free to test - run the workflow manually and read the Drafts folder before flipping it Active. Want these prompts tuned to your real voice and monitored in production? **[Book a free systems audit.](https://calendly.com/skynetlabs/schedule-a-free-consultation)**
