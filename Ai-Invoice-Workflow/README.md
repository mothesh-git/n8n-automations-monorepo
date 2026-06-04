# 🧾 Invoice Workflow — Automated Invoice Processing & Notification

Watches a Google Drive folder for new PDF invoices, extracts structured data using AI, logs it to a Google Sheet, and notifies the internal billing team via a personalized Gmail — all without human intervention.

---

## How It Works

1. **Google Drive Trigger** detects a new PDF file in a specified folder (polls every minute)
2. **Download Binary** fetches the file contents from Google Drive
3. **Extract from File** reads the raw text from the PDF
4. **Information Extractor** (powered by Gemini 2.0 Flash) pulls out 8 structured fields from the invoice text
5. **Update DB** appends the extracted data as a new row in a Google Sheet
6. **Create Email** (powered by GPT-4o Mini) drafts a professional notification email for the billing team
7. **Send Email** delivers the notification via Gmail
8. **No Operation** marks the end of the flow

---

## Node Breakdown

| # | Node | Type | What It Does |
|---|------|------|--------------|
| 1 | **Google Drive Trigger** | Trigger | Polls every minute; fires when a new file is created in the configured folder |
| 2 | **Download Binary** | Google Drive | Downloads the PDF as binary data using the file ID from the trigger |
| 3 | **Extract from File** | Core | Extracts raw text content from the binary PDF |
| 4 | **Google Gemini Chat Model** | AI Sub-node | Provides the LLM (Gemini 2.0 Flash) to the Information Extractor |
| 5 | **Information Extractor** | LangChain | Parses invoice text and returns 8 structured fields (see below) |
| 6 | **Update DB** | Google Sheets | Appends a new row with extracted invoice data to the tracking sheet |
| 7 | **Create Email** | OpenAI (GPT-4o Mini) | Drafts a billing notification email with `Subject` and `Email` fields as JSON |
| 8 | **Send Email** | Gmail | Sends the drafted email to the billing team |
| 9 | **No Operation** | Core | Terminal node — marks successful completion |

---

## Extracted Invoice Fields

The Information Extractor pulls the following fields from every invoice PDF:

| Field | Type | Required |
|-------|------|----------|
| Invoice Number | String | ✅ |
| Client Name | String | ✅ |
| Client Email | String | ✅ |
| Client Address | String | ✅ |
| Client Phone | String | ✅ |
| Total Amount | String | ✅ |
| Invoice Date | Date | ✅ |
| Due Date | Date | ✅ |

---

## Stack

| Service | Purpose |
|---------|---------|
| Google Drive | Invoice PDF storage & trigger |
| Google Gemini 2.0 Flash | AI extraction model |
| OpenAI GPT-4o Mini | Email drafting model |
| Google Sheets | Invoice database / log |
| Gmail | Internal billing notification |

**Trigger:** New file created in Google Drive folder (polls every minute)

---

## Setup Checklist

### 1. Credentials
Connect the following accounts in n8n before activating:

- [ ] **Google Drive OAuth2** — used by both the Trigger and Download Binary nodes
- [ ] **Google Gemini (PaLM) API** — used by the Gemini Chat Model node
- [ ] **OpenAI API** — used by the Create Email node
- [ ] **Google Sheets OAuth2** — used by the Update DB node
- [ ] **Gmail OAuth2** — used by the Send Email node

### 2. Google Drive Trigger
- Open the **Google Drive Trigger** node
- Set **Trigger On** → `Specific Folder`
- Select or paste the folder ID where invoices will be uploaded

### 3. Google Sheets — Update DB
- Open the **Update DB** node
- Set **Document** → select your Google Sheet
- Set **Sheet** → select the target sheet/tab
- Make sure the sheet has columns matching the 8 extracted fields

### 4. Send Email — Billing Recipient
- Open the **Send Email** node
- Update `sendTo` from `billing@example.com` to your actual billing team email

### 5. Create Email — Database Link
- Open the **Create Email** node (system prompt)
- Replace the Google Sheets URL in the prompt with your actual sheet URL

### 6. Activate
- Toggle the workflow to **Active**
- Drop a test PDF invoice into the watched Google Drive folder to verify end-to-end

---

## Customization Ideas

- **Add error handling** — connect an error branch to send a Slack/email alert if extraction fails
- **Support non-PDF formats** — add a Switch node before Extract from File to handle `.docx` or image invoices
- **Client auto-notification** — after Update DB, add a branch to email the client a receipt confirmation using `Client Email`
- **Payment status tracking** — add a column in Google Sheets for payment status and build a follow-up reminder workflow on top
- **Multi-folder routing** — use multiple triggers for different vendor folders and tag rows by source
- **Replace GPT-4o Mini** — swap the Create Email node's model for a cheaper/faster option if email quality allows

---

## Folder Structure

```
workflows/invoice-workflow/
├── workflow.json   ← importable n8n export
└── README.md       ← this file
```

---

## Import Instructions

1. Download `workflow.json`
2. In n8n: **Settings → Import workflow** → upload the file
3. Follow the setup checklist above
4. Connect all credentials, configure the Drive folder and Sheet, then activate