# 📬 AI FAQ Responder (Gmail)

An n8n automation that monitors your Gmail inbox, reads incoming customer emails, and uses an AI agent to generate context-aware FAQ replies — then immediately sends an acknowledgment back to the sender.

---

## 🔁 Workflow Overview

```
Gmail Trigger
    └──▶ Set Node (Extract Email Data)
              └──▶ AI FAQ Agent  ◀──── OpenRouter Chat Model (GPT)
                        │         ◀──── Simple Memory (Buffer Window)
                        └──▶ Send a message (Gmail)
```

**Trigger:** Gmail inbox poll (every minute)
**Output:** Automated acknowledgment email sent to the original sender

---

## 🧩 Node Breakdown

### 1. Gmail Trigger
- **Type:** `n8n-nodes-base.gmailTrigger`
- **Poll interval:** Every minute
- **Filters:** None (captures all new inbox emails)
- Outputs the raw Gmail message object including `From`, `Subject`, `snippet`, and `id`

---

### 2. Set Node — Extract Email Data
- **Type:** `n8n-nodes-base.set`
- Normalizes the raw Gmail payload into clean, named fields for downstream nodes

| Output Field    | Source               | Description                        |
|-----------------|----------------------|------------------------------------|
| `sender_email`  | `$json.From`         | Sender's email address             |
| `email_subject` | `$json.Subject`      | Subject line of the incoming email |
| `question_text` | `$json.snippet`      | Preview/body snippet of the email  |
| `received_at`   | `$now`               | Timestamp when the workflow ran    |

---

### 3. AI FAQ Agent
- **Type:** `@n8n/n8n-nodes-langchain.agent`
- **Model sub-node:** OpenRouter Chat Model (see below)
- **Memory sub-node:** Simple Memory (see below)

**System Prompt behaviour:**
The agent is instructed to act as a professional customer support assistant. It answers strictly from a hardcoded FAQ knowledge base embedded in the system prompt. If a question isn't covered, it politely informs the customer that their query will be forwarded to the support team.

**Current FAQ Knowledge Base topics:**
- Business hours
- Password reset instructions
- Shipping timelines (standard & express)
- Refund policy (30-day money-back guarantee)
- Order tracking
- How to reach a human agent

**User Prompt template:**
```
The customer sent the following email:

Subject: {{ email_subject }}

Message:
{{ question_text }}

Please write a helpful email reply to their question.
```

---

### 4. OpenRouter Chat Model
- **Type:** `@n8n/n8n-nodes-langchain.lmChatOpenRouter`
- **Model:** `openai/gpt-oss-120b`
- **Credential:** OpenRouter API key
- Connected to the AI FAQ Agent as the language model sub-node

---

### 5. Simple Memory
- **Type:** `@n8n/n8n-nodes-langchain.memoryBufferWindow`
- **Session key:** Gmail message `id` (unique per email thread)
- **Context window:** Last 50 messages
- Gives the AI agent per-thread conversational context so follow-up emails in the same thread are handled consistently

---

### 6. Send a Message (Gmail)
- **Type:** `n8n-nodes-base.gmail`
- **To:** `$('Gmail Trigger').item.json.From` (dynamic — replies to the original sender)
- **Subject:** `We've received your inquiry`
- Sends an immediate acknowledgment email confirming receipt and setting expectations for a full response within 24 business hours

---

## ⚙️ Setup Checklist

- [ ] **Gmail OAuth2 credential** — Connect your Gmail account under n8n Credentials
- [ ] **OpenRouter API key** — Create an account at [openrouter.ai](https://openrouter.ai) and add the credential in n8n
- [ ] **Update the FAQ knowledge base** — Edit the system prompt inside the AI FAQ Agent node to reflect your actual FAQ content
- [ ] **Update contact details** — Replace `support@yourcompany.com` and `+1-800-000-0000` in the system prompt
- [ ] **Adjust poll interval** (optional) — Default is every minute; change under Gmail Trigger settings
- [ ] **Activate the workflow** — Toggle it on in n8n once credentials are set

---

## 🔐 Required Credentials

| Credential         | n8n Type          | Where to get it                          |
|--------------------|-------------------|------------------------------------------|
| Gmail account      | `gmailOAuth2`     | Google Cloud Console (OAuth2 app)        |
| OpenRouter account | `openRouterApi`   | [openrouter.ai/keys](https://openrouter.ai/keys) |

---

## 💡 Customization Ideas

- **Swap the model** — Replace `openai/gpt-oss-120b` with any OpenRouter-supported model (e.g. `anthropic/claude-3.5-sonnet`, `meta-llama/llama-3-70b-instruct`)
- **Load FAQs dynamically** — Instead of hardcoding the FAQ in the system prompt, fetch it from a Google Sheet or Notion database using a node before the AI agent
- **Send the AI-generated reply directly** — Pipe `$json.output` from the AI FAQ Agent into the Gmail send node's message field to send the actual AI answer instead of the acknowledgment template
- **Add a logging step** — Insert a Google Sheets node after the AI agent to log every inquiry (sender, subject, AI response, timestamp) for review
- **Escalation routing** — Add an If node to check if the AI response contains a fallback phrase (e.g. "forward your query") and route those to a Slack alert or a separate email to your support team
- **Label handled emails** — Use the Gmail node to apply a label (e.g. `AI-Handled`) after sending, so your inbox stays organized

---

## 📁 Files

```
ai-faq-responder-gmail/
├── README.md                          ← You are here
└── AI_FAQ_Responder__Gmail_.json      ← n8n workflow export
```

To import: open n8n → **Workflows** → **Import from file** → select the JSON.

---

## 🏷️ Tags

`AI Automation` · `Gmail` · `Customer Support` · `LangChain` · `OpenRouter` · `n8n`
