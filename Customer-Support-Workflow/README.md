# 🎧 Customer Support Workflow

Monitors a Gmail inbox for incoming emails, uses an AI classifier to filter out non-support messages, then routes genuine customer support emails to an AI agent. The agent drafts a grounded reply using a Pinecone knowledge base (policies, FAQs), labels the original email in Gmail, and sends the reply — all without any human in the loop.

---

## Workflow Diagram

```
Gmail Trigger
(polls every minute)
        │
        ▼
Text Classifier ◄─── OpenRouter Chat Model
        │
   ┌────┴────┐
   │         │
   ▼         ▼
AI Agent   No Operation
           (non-support)
   │
   │ ◄─── OpenAI Chat Model (gpt-4o-mini)
   │ ◄─── Pinecone Vector Store (retrieve-as-tool)
   │              ▲
   │      Embeddings OpenAI
   │
   ▼
Label
(addLabels: message)
   │
   ▼
Send
(reply: message)
```

---

## Node Breakdown

| # | Node | Type | What It Does |
|---|------|------|--------------|
| 1 | **Gmail Trigger** | Trigger | Polls the inbox every minute for new emails; passes full email data including body text downstream |
| 2 | **OpenRouter Chat Model** | LangChain | Provides the LLM for the Text Classifier to make categorization decisions |
| 3 | **Text Classifier** | LangChain | Categorizes each email into `Customer Support` or `Other`; routes to separate branches |
| 4 | **No Operation** | Core | Silently drops emails that are not customer support related — no reply, no label |
| 5 | **OpenAI Chat Model** | OpenAI | Provides `gpt-4o-mini` as the reasoning LLM for the AI Agent |
| 6 | **Pinecone Vector Store** | LangChain | Exposed as a `knowledgeBase` tool; retrieves relevant policy/FAQ chunks from Pinecone on demand |
| 7 | **Embeddings OpenAI** | OpenAI | Embeds the agent's retrieval query at runtime to match against stored vectors |
| 8 | **AI Agent** | LangChain | Reads the incoming email, calls `knowledgeBase` as needed, and drafts a friendly reply signed as *Mr. Helpful from Tech Haven Solutions* |
| 9 | **Label** | Gmail | Adds a Gmail label to the original email to mark it as handled |
| 10 | **Send** | Gmail | Replies directly to the original email thread with the agent's generated response |

---

## Classification Logic

The **Text Classifier** uses OpenRouter as its LLM and two defined categories:

| Category | Description | Branch |
|----------|-------------|--------|
| `Customer Support` | Emails asking about policies, products, or services | → AI Agent → Label → Send |
| `Other` | Any email not related to customer support | → No Operation (dropped silently) |

Only emails classified as `Customer Support` proceed through the pipeline. Everything else exits at the `No Operation` node with no action taken.

---

## AI Agent Behaviour

The agent is configured with a system prompt that defines its persona and output format:

- **Persona:** Customer support agent for *Tech Haven*
- **Tool available:** `knowledgeBase` — retrieves from Pinecone (`sample` index, `FAQ` namespace)
- **Tone:** Friendly, with emojis
- **Sign-off:** *Mr. Helpful from Tech Haven Solutions*
- **Output:** Email body only (no subject line)

The agent decides autonomously when to call the `knowledgeBase` tool based on the email content. It only retrieves what it needs, then composes a response grounded in your actual policies and FAQ documents.

---

## Pinecone Configuration

| Setting | Value |
|---------|-------|
| Index name | `sample` *(rename to match your index)* |
| Namespace | `FAQ` |
| Retrieval mode | `retrieve-as-tool` |
| Tool name exposed to agent | `knowledgeBase` |
| Tool description | "Call this tool to access information about Policies and FAQ" |

> **Pre-requisite:** This workflow requires an already-populated Pinecone index. Use the [RAG Pipeline & Chatbot](../rag-pipeline-chatbot/) workflow (or equivalent) to ingest your FAQ and policy documents before activating this one.

---

## Setup Checklist

1. **Import the workflow** — In n8n: Import → paste or upload `workflow.json`
2. **Pre-populate Pinecone** — Ensure your `sample` index / `FAQ` namespace already has documents indexed (see the RAG Pipeline & Chatbot workflow)
3. **Add Pinecone credentials** — Link your Pinecone API key to the Pinecone Vector Store node; update the index name if different from `sample`
4. **Add OpenAI credentials** — Link your OpenAI API key to both the **OpenAI Chat Model** and **Embeddings OpenAI** nodes
5. **Add OpenRouter credentials** — Link your OpenRouter API key to the **OpenRouter Chat Model** node (used by the Text Classifier)
6. **Connect Gmail credentials** — Link your Gmail OAuth2 account to the **Gmail Trigger**, **Label**, and **Send** nodes; all three must use the same Gmail account
7. **Create a Gmail label** — In Gmail, create a label to tag handled support emails (e.g. `Support - Handled`); copy its Label ID from Gmail settings
8. **Update the Label node** — Replace the `labelIds` value in the Label node with your actual Gmail Label ID
9. **Customize the agent persona** — In the AI Agent node's system prompt, replace `Tech Haven` and `Mr. Helpful from Tech Haven Solutions` with your own company name and sign-off
10. **Test with a sample email** — Send a test support email to the monitored inbox; verify it gets classified, replied to, and labelled correctly
11. **Test the Other branch** — Send a non-support email (e.g. a newsletter) and confirm it exits at No Operation with no reply sent
12. **Activate the workflow** — Toggle it on; the Gmail Trigger will begin polling every minute

---

## Credentials Required

| Credential | Node(s) | Notes |
|------------|---------|-------|
| Gmail OAuth2 | Gmail Trigger, Label, Send | All three must share the same Gmail account |
| OpenRouter API | OpenRouter Chat Model | Used for classification only; get key from [openrouter.ai](https://openrouter.ai/) |
| OpenAI API | OpenAI Chat Model, Embeddings OpenAI | `gpt-4o-mini` for chat; default embedding model for retrieval |
| Pinecone API | Pinecone Vector Store | Get from [app.pinecone.io](https://app.pinecone.io/); index must be pre-populated |

---

## How to Find Your Gmail Label ID

1. Go to Gmail → Settings → See all settings → Labels
2. Hover over the label you created
3. The Label ID appears in the URL as a long numeric string (e.g. `Label_1594706753190197855`)
4. Paste this value into the `labelIds` array in the **Label** node

---

## Customization Ideas

- **Expand classification categories** — Add more branches to the Text Classifier (e.g. `Billing`, `Technical`, `Returns`) and route each to a differently-configured AI Agent with its own system prompt
- **Add conversation memory** — Connect a Window Buffer Memory node to the AI Agent to handle multi-turn support threads more coherently
- **Slack escalation** — After the Send node, add a conditional check: if the agent's output contains phrases like "I'm not sure" or "please contact us", route to a Slack node to ping a human agent
- **Sentiment routing** — Add a second Text Classifier after the first to detect angry or urgent emails and fast-track them to a human review queue
- **CRM logging** — After the Send node, add a HubSpot or Airtable node to log the support interaction with the sender's email and a summary
- **Swap the knowledge base** — Point the Pinecone node at a different namespace per product line or department to give each agent its own scoped knowledge base

---

## Use Cases

- **E-commerce support** — Auto-respond to questions about shipping, returns, and order status using your policy documents as the source of truth
- **SaaS help desk** — Answer product feature questions from a documentation knowledge base without routing every ticket to a human
- **Internal IT helpdesk** — Monitor an IT support inbox and reply to common questions about tools, access, and procedures
- **Agency client support** — Handle repetitive client questions about deliverables, timelines, and processes with a branded AI persona

---

## Tech Stack

| Tool | Role |
|------|------|
| n8n | Workflow automation |
| Gmail | Email trigger, labelling, and reply |
| OpenRouter | LLM for email classification |
| OpenAI (`gpt-4o-mini`) | LLM powering the AI Agent |
| OpenAI Embeddings | Query vectorization for retrieval |
| Pinecone | Vector knowledge base (policies & FAQ) |
| LangChain (n8n) | Agent orchestration, classification, retrieval |