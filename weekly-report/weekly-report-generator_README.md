# 📊 Weekly Report Generator

Automatically pulls data from a Google Sheet every Monday morning, feeds it to an AI agent, and delivers a fully formatted HTML report straight to your inbox — no manual effort required.

Designed to be **plug-and-play for any tabular data**: sales performance, social media campaigns, employee attendance, support tickets, expenses, and more.

---

## Workflow Overview

```
Schedule Trigger → Google Sheets → AI Agent → Aggregate → Gmail
                                       ↑
                               OpenRouter Chat Model
                               (meta-llama/llama-3.1-8b-instruct)
```

---

## Node Breakdown

### 1. Schedule Trigger
- Fires every **Monday at 7:00 AM**
- Configurable: change `triggerAtDay` (0 = Sun, 1 = Mon … 6 = Sat) and `triggerAtHour` to suit your timezone and preference

### 2. Google Sheets
- Reads all rows from your configured sheet
- **Operation:** Read Sheet
- Replace the `documentId` with your own Google Sheet ID (found in the sheet URL)
- Change `sheetName` to match your tab name (default: `Sheet1`)

### 3. OpenRouter Chat Model *(sub-node of AI Agent)*
- **Model:** `meta-llama/llama-3.1-8b-instruct` via OpenRouter
- Swap this for any supported LLM: OpenAI GPT-4, Anthropic Claude, Mistral, etc.
- Requires an OpenRouter API key credential

### 4. AI Agent
- Receives all sheet rows as JSON
- Prompted to produce a **single clean HTML email report** with:
  - Executive Summary
  - Key Metrics table
  - Notable Trends & Highlights
  - Areas of Concern
  - Action Items & Recommendations
- Inline CSS styling — no external stylesheets needed
- You can customize the prompt to match your data schema and report style

### 5. Aggregate
- Collects all AI Agent output items into a single array
- Ensures the Gmail node receives one consolidated message, not multiple separate emails

### 6. Gmail
- Sends the HTML report to the configured recipient
- **Subject line** auto-includes the current date: `📊 Weekly Report — MM DD`
- Supports CC / BCC — configured by default with a BCC to the sender
- Requires a Gmail OAuth2 credential

---

## Sample Data Schema

The workflow was tested with this Google Sheet structure:

| Sales Rep | Leads | Calls | Deals Closed | Revenue   |
|-----------|-------|-------|--------------|-----------|
| Arun      | 50    | 120   | 8            | ₹1,20,000 |
| Priya     | 40    | 100   | 5            | ₹80,000   |
| Sanjay    | 45    | 110   | 6            | ₹1,00,000 |
| Vijay     | 40    | 100   | 5            | ₹80,000   |

**You can use any schema** — the AI agent dynamically analyzes whatever columns you provide. There's no hardcoded field mapping.

---

## Email Output

The generated email includes:
- A branded header with the company/team name and report date
- An **Executive Summary** in plain prose
- A **Key Metrics** table with alternating row colors
- **Notable Trends & Highlights** as bullet points
- **Areas of Concern** flagging underperforming metrics
- **Action Items & Recommendations** with concrete next steps

---

## Setup Checklist

- [ ] Import `weekly_report_generator.json` into your n8n instance
- [ ] Connect your **Google Sheets OAuth2** credential
- [ ] Connect your **Gmail OAuth2** credential
- [ ] Connect your **OpenRouter API** credential (or swap the model node)
- [ ] Update the `documentId` in the Google Sheets node with your Sheet ID
- [ ] Update the `sheetName` if your tab isn't named `Sheet1`
- [ ] Replace the recipient email in the Gmail node
- [ ] Adjust the Schedule Trigger day/hour to your preferred send time
- [ ] Activate the workflow

---

## Required Credentials

| Credential | Node | Notes |
|---|---|---|
| Google Sheets OAuth2 | Google Sheets | Read access to your sheet |
| Gmail OAuth2 | Gmail | Send access from your Gmail account |
| OpenRouter API | OpenRouter Chat Model | Get key at [openrouter.ai](https://openrouter.ai) |

---

## Customization Ideas

- **Swap the LLM**: Replace OpenRouter with Anthropic Claude or OpenAI for more powerful analysis
- **Multiple sheets**: Add a second Google Sheets node to pull from multiple tabs and merge the data before passing to the AI Agent
- **Richer prompts**: Add your company name, KPI targets, or specific analysis instructions directly in the AI Agent prompt
- **Slack/Teams delivery**: Replace or add a Slack/Teams node alongside Gmail to post the summary to a channel
- **Dynamic recipients**: Pull a recipient list from a separate sheet and loop the Gmail node to send personalized department reports
- **Weekly diff**: Store last week's data in a second sheet tab and pass both to the AI Agent for week-over-week comparison
- **PDF attachment**: Add an HTML-to-PDF conversion node and attach the report as a PDF instead of (or alongside) the email body

---

## Use Cases

This workflow is **data-schema agnostic**. Works out of the box for:

- 📈 Sales performance (leads, calls, deals, revenue)
- 📣 Social media campaigns (impressions, clicks, conversions)
- 🧑‍💼 Employee attendance (present, absent, late, hours)
- 🎫 Support ticket analysis (opened, closed, SLA breaches)
- 💰 Expense & revenue tracking (spend by category, budget vs. actual)
- 🚀 Product/sprint metrics (tasks completed, bugs, velocity)

---

## Tags

`weekly-report` · `automated` · `google-sheets` · `gmail` · `openrouter` · `llm` · `ai-agent`
