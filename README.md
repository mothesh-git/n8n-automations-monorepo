# 🤖 n8n Automation Workflows

A public monorepo of production-ready n8n automation workflows — built, documented, and shared openly.

Each workflow lives in its own folder with a JSON export and a full README covering setup, node breakdown, and customization ideas. The goal is to make every automation genuinely reusable, not just a screenshot.

---

## Workflows

### 1. 📬 New Lead Email Sender
**Folder:** `workflows/new-lead-email-sender/`

Handles Google Form submissions end-to-end: normalizes lead fields, validates email addresses, logs valid leads to Google Sheets, and sends personalized welcome emails via Gmail.

**Stack:** Google Forms · Google Sheets · Gmail
**Trigger:** Google Form submission (webhook)

---

### 2. 🤖 AI FAQ Responder
**Folder:** `workflows/ai-faq-responder/`

Monitors a Gmail inbox for incoming questions, uses an AI agent to generate accurate answers based on a predefined FAQ knowledge base, and replies automatically — keeping response times near zero.

**Stack:** Gmail · AI Agent
**Trigger:** New email received in Gmail

---

### 3. 📊 Weekly Report Generator
**Folder:** `workflows/weekly-report-generator/`

Pulls data from any Google Sheet every Monday morning, passes it to an AI agent, and delivers a fully formatted HTML report straight to your inbox. Works for sales performance, social media, attendance, support tickets, expenses, and more — no schema changes required.

**Stack:** Google Sheets · OpenRouter (LLM) · Gmail
**Trigger:** Schedule (every Monday at 7:00 AM)

---

## Monorepo Structure

```
/
├── README.md
└── workflows/
    ├── new-lead-email-sender/
    │   ├── workflow.json
    │   └── README.md
    ├── ai-faq-responder/
    │   ├── workflow.json
    │   └── README.md
    └── weekly-report-generator/
        ├── workflow.json
        └── README.md
```

Each workflow folder contains:
- `workflow.json` — importable n8n workflow export
- `README.md` — node breakdown, schema, setup checklist, credentials, and customization ideas

---

## How to Use Any Workflow

1. Download the `workflow.json` from the folder
2. In n8n: **Import** → paste or upload the JSON
3. Follow the setup checklist in that workflow's README
4. Connect your credentials and activate

---

## Philosophy

**Build in public.** Every workflow here was built for real use and documented for real reuse. If something saves time, it's worth sharing — and worth documenting well enough that someone else can actually use it.

---

## Contributing

Found a bug or have an improvement? Open an issue or PR. New workflow ideas are welcome too.

---

## Author

Built by [@mothesh](https://github.com/mothesh) · Shared on [LinkedIn](https://linkedin.com/in/mothesh)