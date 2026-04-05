# SamFlow AI — Lead Qualification Workflow

An AI-powered, production-grade lead qualification workflow built in [n8n](https://n8n.io). Automatically scores, routes, and follows up with inbound leads from a Tally form — with full error handling and Slack observability.

---

## How it works

```
Tally Form → Validate → AI Score → Route → HubSpot + Gmail + Slack
```

1. A lead submits the Tally enquiry form
2. Fields are normalised and validated (rejects spam/incomplete submissions)
3. OpenAI `gpt-4o-mini` scores the lead 0–100 and assigns a priority (`hot` / `warm` / `cold`)
4. **Qualified leads** → HubSpot contact created → confirmation email sent → Slack hot-lead alert fired
5. **Unqualified leads** → HubSpot contact created → 2-day wait → personalised follow-up email sent
6. Any invalid submission or workflow error triggers a Slack alert for manual review

---

## Workflow diagram

```
[Tally Trigger]
      │
[Normalise Fields]
      │
[Validate Input] ──FAIL──► [Slack: Invalid Lead Alert]
      │
   PASS
      │
[AI Lead Qualifier (gpt-4o-mini)]
      │
[Parse AI Output] ──ERROR──► [Slack: Parse Error Alert]
      │
   OK
      │
[Is Qualified?]
   │           │
  YES          NO
   │           │
[HubSpot]   [HubSpot]
   │           │
[Gmail:      [Wait 2d]
 Confirm]      │
   │         [Gmail:
[Slack:      Follow-up]
 Alert]

[Error Trigger] ──► [Slack: Workflow Error Alert]
```

---

## Nodes

| Node | Type | Purpose |
|---|---|---|
| Lead Submission | Tally Trigger | Receives form submissions |
| Normalise Fields | Set | Trims strings, lowercases email, adds timestamp |
| Validate Input | Code (JS) | Guards against missing fields and bad email format |
| Is Valid? | If | Routes invalid leads to Slack alert |
| AI Lead Qualifier | LangChain Agent | Scores and qualifies the lead using GPT |
| OpenAI gpt-4o-mini | LLM | Powers the AI agent |
| Parse AI Output | Code (JS) | Safely parses JSON, enforces types, sets defaults |
| Parse Error? | If | Catches unparseable AI responses |
| Is Qualified? | If | Routes on `is_qualified` boolean |
| HubSpot (×2) | HubSpot | Creates/updates contact for both branches |
| Send Confirmation Email | Gmail | Personalised reply to qualified leads |
| Slack: Hot Lead Alert | Slack | Notifies team with full lead details and score |
| Wait 2 Days | Wait | Delays follow-up for unqualified leads |
| Send Follow-Up Email | Gmail | Nurture email after 2-day wait |
| Workflow Error Trigger | Error Trigger | Catches any unhandled workflow failure |
| Alert Workflow Error | Slack | Posts error details to Slack |

---

## AI scoring rules

| Condition | Points |
|---|---|
| Base score | +40 |
| Business email (not Gmail/Yahoo/Hotmail) | +20 |
| Business name present | +10 |
| Each specific service selected (max 3) | +10 each |
| Budget > 0 | +10 |
| Note/message included | +5 |
| Spam email detected | −30 |

**Priority mapping:**
- `hot` → score ≥ 80
- `warm` → score 50–79
- `cold` → score < 50

---

## Setup

### Prerequisites

- n8n (self-hosted v1.x+ or n8n Cloud)
- Accounts and API credentials for:
  - [Tally](https://tally.so)
  - [OpenAI](https://platform.openai.com)
  - [HubSpot](https://hubspot.com) (OAuth2)
  - Gmail (OAuth2)
  - [Slack](https://api.slack.com/apps) (OAuth2)

### Import

1. Clone this repo or download `samflow_lead_workflow_v2.json`
2. In n8n, go to **Workflows → Import from file**
3. Select the JSON file

### Configure credentials

Open each node and connect your credentials:

| Node | Credential type |
|---|---|
| Lead Submission | Tally API |
| OpenAI gpt-4o-mini | OpenAI API |
| HubSpot nodes | HubSpot OAuth2 |
| Gmail nodes | Gmail OAuth2 |
| Slack nodes | Slack OAuth2 |

### Update Slack channel ID

All Slack nodes target channel ID `C0A5CK4V0MQ`. Replace this with your own channel ID in each Slack node.

### Activate

Toggle the workflow to **Active** in n8n. Submit a test lead via your Tally form to verify all branches.

---

## Testing checklist

- [ ] Valid lead with business email and services → qualified branch fires
- [ ] Lead missing business name → unqualified branch fires
- [ ] Incomplete submission (no email) → validation gate drops it, Slack alert fires
- [ ] Spam email (test@test.com) → AI scores low, unqualified branch
- [ ] HubSpot contact created with correct first name, last name, company, phone
- [ ] Confirmation email received with personalised message
- [ ] Slack hot-lead alert includes score, priority, reasoning
- [ ] Unqualified lead receives follow-up email after 2-day wait
- [ ] Simulate workflow error → Slack error alert fires

---

## File structure

```
.
├── samflow_lead_workflow_v2.json   # n8n workflow — import this
└── README.md
```

---

## Built with

- [n8n](https://n8n.io) — workflow automation
- [OpenAI gpt-4o-mini](https://platform.openai.com) — lead scoring
- [HubSpot](https://hubspot.com) — CRM
- [Tally](https://tally.so) — form trigger
- [Gmail](https://mail.google.com) — transactional email
- [Slack](https://slack.com) — team alerts

---

## Author

**Adetuyi Samuel** — [SamFlow AI](https://samflow.ai)
