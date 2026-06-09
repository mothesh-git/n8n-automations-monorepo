# 🧠 RAG Pipeline & Chatbot

A two-part workflow that builds a fully automated knowledge base and a conversational AI chatbot on top of it. New documents uploaded to Google Drive are automatically chunked, embedded, and stored in Pinecone. A separate chat interface lets users query that knowledge base through an AI agent powered by OpenRouter — no manual indexing, no copy-pasting.

---

## How It Works

This workflow is split into two independent pipelines that share a single Pinecone index:

**Pipeline 1 — Document Ingestion:** Triggered whenever a new file lands in a Google Drive folder. Downloads it, splits it into chunks, embeds those chunks with OpenAI, and upserts them into Pinecone under a dedicated namespace.

**Pipeline 2 — Chat Interface:** Activated when a user sends a message via n8n's built-in chat UI. An AI agent receives the question, calls the Pinecone vector store as a retrieval tool, and returns a grounded answer based on the indexed documents.

---

## Workflow Diagram

### Pipeline 1 — Document Ingestion

```
Google Drive Trigger
  (new file in folder)
        │
        ▼
  Download File
  (Google Drive)
        │
        ▼
Pinecone Vector Store ◄─── Embeddings OpenAI
     (insert)          ◄─── Default Data Loader
                                    ▲
                        Recursive Character
                           Text Splitter
```

### Pipeline 2 — Chat Interface

```
When chat message received
   (n8n Chat Trigger)
          │
          ▼
       AI Agent ◄─── OpenRouter Chat Model (qwen3-32b)
                ◄─── Pinecone Vector Store (retrieve-as-tool)
                              ▲
                     Embeddings OpenAI
```

---

## Node Breakdown

### Pipeline 1 — Ingestion Nodes

| # | Node | Type | What It Does |
|---|------|------|--------------|
| 1 | **Google Drive Trigger** | Trigger | Polls every minute for new files created in the watched `FAQ` folder |
| 2 | **Download File** | Google Drive | Downloads the new file as binary data |
| 3 | **Embeddings OpenAI** | OpenAI | Converts text chunks into vector embeddings using OpenAI's embedding model |
| 4 | **Default Data Loader** | LangChain | Reads the binary file and passes document content downstream |
| 5 | **Recursive Character Text Splitter** | LangChain | Splits document text into overlapping chunks for optimal retrieval |
| 6 | **Pinecone Vector Store** (insert) | LangChain | Upserts embedded chunks into the `FAQ` namespace of the Pinecone index |

### Pipeline 2 — Chat Nodes

| # | Node | Type | What It Does |
|---|------|------|--------------|
| 7 | **When chat message received** | Chat Trigger | Opens n8n's built-in chat UI; fires when a user sends a message |
| 8 | **OpenRouter Chat Model** | LangChain | Provides `qwen/qwen3-32b` as the reasoning LLM for the agent |
| 9 | **Pinecone Vector Store1** (retrieve-as-tool) | LangChain | Exposes the `FAQ` namespace as a callable `knowledgeBase` tool for the agent |
| 10 | **Embeddings OpenAI1** | OpenAI | Embeds the user's query at retrieval time so it can be matched against stored vectors |
| 11 | **AI Agent** | LangChain | Orchestrates the conversation: receives the question, decides when to call `knowledgeBase`, and composes the final answer |

---

## Pinecone Configuration

| Setting | Value |
|---------|-------|
| Index name | `sample` *(rename to match your index)* |
| Namespace | `FAQ` |
| Ingestion mode | `insert` |
| Retrieval mode | `retrieve-as-tool` |
| Tool name exposed to agent | `knowledgeBase` |
| Tool description | "Call this tool to access the policy and FAQ database" |

Both the ingestion and retrieval Pinecone nodes must point to the **same index + namespace** for retrieval to work.

---

## Setup Checklist

1. **Import the workflow** — In n8n: Import → paste or upload `workflow.json`
2. **Create a Pinecone account** — Set up an index at [pinecone.io](https://www.pinecone.io/); note your index name and region
3. **Add Pinecone credentials** — In n8n credentials, add your Pinecone API key; update the index name in both Pinecone nodes if you rename it from `sample`
4. **Add OpenAI credentials** — Link your OpenAI API key to both **Embeddings OpenAI** and **Embeddings OpenAI1** nodes (used for embedding only, not chat)
5. **Add OpenRouter credentials** — Link your OpenRouter API key to the **OpenRouter Chat Model** node
6. **Create a Google Drive folder** — Create a folder that will hold your knowledge base documents (PDFs, DOCX, TXT, etc.); note its folder ID from the URL
7. **Configure Google Drive Trigger** — Set the watched folder to your new folder's ID; connect Google Drive OAuth2 credentials to both the Trigger and Download File nodes
8. **Set the Pinecone namespace** — Confirm both Pinecone nodes use the same namespace (default: `FAQ`); change this if you want to segment different knowledge bases
9. **Upload a test document** — Drop a PDF or text file into the watched Google Drive folder and manually trigger or wait for the poll to run; verify embeddings appear in your Pinecone index
10. **Test the chatbot** — Click **Open Chat** in n8n and ask a question about the document you uploaded; the agent should retrieve and answer from it
11. **Activate the workflow** — Toggle it on so new documents are ingested automatically going forward

---

## Credentials Required

| Credential | Node(s) | Notes |
|------------|---------|-------|
| Google Drive OAuth2 | Google Drive Trigger, Download File | Needs read access to the watched folder |
| OpenAI API | Embeddings OpenAI, Embeddings OpenAI1 | Used for embeddings only — no chat completions |
| Pinecone API | Pinecone Vector Store, Pinecone Vector Store1 | Get from [app.pinecone.io](https://app.pinecone.io/) |
| OpenRouter API | OpenRouter Chat Model | Uses `qwen/qwen3-32b`; get key from [openrouter.ai](https://openrouter.ai/) |

---

## Supported Document Types

The Default Data Loader handles any file type n8n can parse as binary, including:

- PDF (`.pdf`)
- Plain text (`.txt`)
- Markdown (`.md`)
- Word documents (`.docx`)
- CSV (`.csv`)

For best results, use clean text-heavy documents. Scanned image PDFs without embedded text will produce poor extractions.

---

## Customization Ideas

- **Swap the LLM** — Replace `qwen/qwen3-32b` with any OpenRouter-supported model (e.g. `anthropic/claude-3.5-haiku`, `meta-llama/llama-3.1-8b-instruct`) by changing the model string in the OpenRouter Chat Model node
- **Multiple namespaces** — Run separate ingestion pipelines for different document categories (e.g. `HR-Policies`, `Product-Docs`) and give each its own Pinecone node and retrieval tool in the agent
- **Add memory** — Connect a Window Buffer Memory node to the AI Agent to maintain conversation context across multi-turn chats
- **Embed the chatbot** — Use n8n's webhook-based chat trigger instead of the built-in UI to embed the chatbot in an external website or Slack
- **Auto-delete on file removal** — Add a second Google Drive Trigger watching for `fileDeleted` events and pair it with a Pinecone delete operation to keep the index in sync
- **Chunk size tuning** — Adjust the Recursive Character Text Splitter's `chunkSize` and `chunkOverlap` parameters to optimize retrieval quality for your document types

---

## Use Cases

- **Internal knowledge base** — Let employees ask natural language questions about company policies, HR docs, or SOPs without digging through folders
- **Customer-facing FAQ bot** — Index your product documentation and support articles; surface accurate answers to customer questions instantly
- **Legal / compliance Q&A** — Upload contracts, regulations, or compliance guides and query them conversationally
- **Research assistant** — Drop research papers or reports into the Drive folder and query findings across multiple documents at once
- **Onboarding chatbot** — Index onboarding materials so new hires can ask questions and get answers grounded in actual company content

---

## Tech Stack

| Tool | Role |
|------|------|
| n8n | Workflow automation & chat UI |
| Google Drive | Document storage & ingestion trigger |
| OpenAI Embeddings | Text vectorization (ingestion + retrieval) |
| Pinecone | Vector database & similarity search |
| OpenRouter (`qwen/qwen3-32b`) | LLM powering the AI agent |
| LangChain (n8n) | Agent orchestration, document loading, text splitting |