# Lead Generation — Google Forms Trigger

Automatically captures new leads from a Google Form, logs them to a Google Sheet, and fires a personalised welcome email — all within seconds of submission.

---

## What It Does

```
Google Form submitted
   └─► Google Sheet (trigger)
         └─► Normalize Lead Fields
               └─► Email Exists? (guard)
                     ├─► [TRUE]  Add to Leads Sheet  +  Send Welcome Email
                     └─► [FALSE] Skip — No Email
```

| Step | Node | Purpose |
|------|------|---------|
| 1 | **Google Forms — New Response** | Polls the Google Sheet linked to your Form every minute for new rows |
| 2 | **Normalize Lead Fields** | Maps raw form column names (`Full Name`, `Email Address`, `Company`) to clean internal fields |
| 3 | **Email Exists?** | Guards against incomplete submissions — only valid leads proceed |
| 4 | **Add to Leads Sheet** | Appends a timestamped row to your master `Leads` tracking sheet |
| 5 | **Send Welcome Email** | Sends a personalised HTML email to the lead via Gmail |
| 6 | **Skip — No Email** | Silently drops submissions with no email address |

---

## Google Sheet Schema

Your **source form sheet** (where Google Forms writes responses) must have columns matching your form labels:

| Column | Default label used |
|--------|--------------------|
| A | `Full Name` |
| B | `Email Address` |
| C | `Company` |
| D | `Number` |
| E | `Timestamp` |

Your **leads tracking sheet** (`Leads` tab) needs these headers in row 1:

```
Name | Email | Company | Source | Date Added
```

---

## Setup Checklist

1. **Import** `lead-gen-google-forms.json` into your n8n instance via *Workflows → Import from file*.
2. **Replace placeholders** in the workflow:
   - `YOUR_GOOGLE_SHEET_ID` — ID from your Google Form responses sheet URL
   - `YOUR_LEADS_SHEET_ID` — ID of your leads tracking sheet
   - `YOUR_GOOGLE_CREDENTIAL_ID` — your n8n Google Sheets OAuth2 credential
   - `YOUR_GMAIL_CREDENTIAL_ID` — your n8n Gmail OAuth2 credential
3. **Update field mappings** in *Normalize Lead Fields* if your form question labels differ from the defaults above.
4. **Edit the welcome email** copy and sign-off name in *Send Welcome Email* to match your brand.
5. **Activate** the workflow — the trigger will poll every minute automatically.

---

## Credentials Required

| Service | n8n Credential Type |
|---------|-------------------|
| Google Sheets | `googleSheetsOAuth2Api` |
| Gmail | `gmailOAuth2` |

---

## Customisation Ideas

- Replace **Skip — No Email** with a Slack alert so you're notified of incomplete submissions.
- Add a **CRM node** (HubSpot, Notion, Airtable) after *Add to Leads Sheet* to sync leads automatically.
- Extend **Normalize Lead Fields** to capture additional form questions (budget, message, etc.).
- Change the poll interval from `everyMinute` to `everyHour` if submission volume is low.

---

## Tags

`lead-generation` · `google-forms` · `gmail` · `google-sheets` · `n8n`
