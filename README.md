# 🤖 n8n Automation Workflows — Open Source Monorepo

A growing collection of production-ready n8n automation workflows, built and documented in public. Each workflow lives in its own folder with a JSON export and a detailed README so you can import, understand, and customize it in minutes.

---

## 📂 Repository Structure

```
n8n-automations/
├── README.md                          ← You are here
└── workflows/
    ├── new-lead-email-sender/
    │   ├── README.md
    │   └── workflow.json
    └── ai-faq-responder-gmail/
        ├── README.md
        └── AI_FAQ_Responder__Gmail_.json
```

---

## 🗂️ Workflow Index

| # | Workflow | Description | Integrations | Folder |
|---|----------|-------------|--------------|--------|
| 01 | **New Lead Email Sender** | Captures Google Form submissions, validates emails, logs leads to Google Sheets, and sends personalized welcome emails | Google Forms · Google Sheets · Gmail | [`new-lead-email-sender/`](./workflows/new-lead-email-sender/) |
| 02 | **AI FAQ Responder (Gmail)** | Monitors your Gmail inbox, reads incoming customer emails, generates AI-powered FAQ replies via OpenRouter, and sends acknowledgment emails automatically | Gmail · OpenRouter · LangChain Agent | [`ai-faq-responder-gmail/`](./workflows/ai-faq-responder-gmail/) |

---

## 🚀 How to Use Any Workflow

1. **Clone the repo**
   ```bash
   git clone https://github.com/your-username/n8n-automations.git
   ```

2. **Open n8n** and navigate to **Workflows → Import from file**

3. **Select the `.json` file** from the workflow's subfolder

4. **Follow the setup checklist** in that workflow's `README.md` — each one lists the exact credentials and configuration steps needed

5. **Activate** the workflow and you're live

---

## 🧱 Design Principles

- **One folder per workflow** — each automation is self-contained with its JSON export and documentation
- **Build in public** — every workflow is openly shared with full node breakdowns and setup instructions
- **Reusable patterns** — field normalization, AI agents, conditional routing, and logging patterns are documented so they're easy to adapt
- **Customization-first docs** — every README ends with ideas for extending the workflow to fit your specific use case

---

## 🛠️ Tech Stack

| Tool | Role |
|------|------|
| [n8n](https://n8n.io) | Workflow automation engine |
| Gmail (OAuth2) | Email trigger & sending |
| Google Forms / Sheets | Lead capture & logging |
| OpenRouter | LLM API gateway (multi-model) |
| LangChain (n8n nodes) | AI agent orchestration |

---

## 📬 Follow Along

These workflows are documented and shared publicly as they're built. If you find them useful, feel free to star the repo, fork a workflow, or suggest ideas for future automations.

---

## 📄 License

MIT — free to use, adapt, and share with attribution.