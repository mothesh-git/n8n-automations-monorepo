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

### 4. 🧾 Invoice Automation
**Folder:** `workflows/invoice-automation/`

Watches a Google Drive folder for new invoice PDFs, extracts structured data (invoice number, client details, amounts, dates) using Google Gemini, logs everything to a Google Sheets database, and automatically notifies your billing team with a generated email via Gmail.

**Stack:** Google Drive · Google Gemini · OpenAI (GPT-4o-mini) · Google Sheets · Gmail
**Trigger:** New file created in Google Drive folder

---

### 5. 🧠 RAG Pipeline & Chatbot
**Folder:** `workflows/rag-pipeline-chatbot/`

A two-part workflow: an automated ingestion pipeline that embeds documents from Google Drive into a Pinecone vector store, and a conversational AI chatbot that retrieves answers from those documents in real time. Drop a file in the folder — it's indexed. Ask a question in the chat — it's answered from your own knowledge base.

**Stack:** Google Drive · OpenAI Embeddings · Pinecone · OpenRouter (Qwen3-32b) · LangChain
**Trigger:** New file in Google Drive folder (ingestion) · Chat message (chatbot)

---

### 6. 🎧 Customer Support Workflow
**Folder:** `workflows/customer-support-workflow/`

Monitors a Gmail inbox, classifies incoming emails as customer support or other using an AI text classifier, and routes only genuine support emails to an AI agent. The agent drafts a grounded reply from a Pinecone knowledge base, labels the original email, and sends the reply — fully automated, zero human effort for routine queries.

**Stack:** Gmail · OpenRouter · OpenAI (GPT-4o-mini) · OpenAI Embeddings · Pinecone · LangChain
**Trigger:** New email received in Gmail (polls every minute)

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
    ├── weekly-report-generator/
    │   ├── workflow.json
    │   └── README.md
    ├── invoice-automation/
    │   ├── workflow.json
    │   └── README.md
    ├── rag-pipeline-chatbot/
    │   ├── workflow.json
    │   └── README.md
    └── customer-support-workflow/
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