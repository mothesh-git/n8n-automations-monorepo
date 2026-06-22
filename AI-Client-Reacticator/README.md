# 💆 MedSpa AI Client Reactivator System

Runs every morning at 9 AM, reads your client sheet for anyone with a `Lapsed` status, checks whether they've been away 90+ days, and fires a personalized AI-generated SMS via Twilio to bring them back. Every outreach attempt is logged to a separate sheet with delivery status. No manual follow-up list. No copy-pasting. Just a daily reactivation loop that runs itself.

---

## Workflow Diagram

```
Daily 9AM Trigger
(schedule: every day at 9:00 AM)
        │
        ▼
Fetch All Clients
(Google Sheets — filter: Status = "Lapsed")
        │
        ▼
IF Client Lapsed 90+ Days
        │
   ┌────┴────┐
   │ TRUE    │ FALSE
   ▼         ▼
AI Agent   (no action)
   │
   │ ◄─── OpenRouter Chat Model (qwen3-32b)
   │
   ▼
Send SMS — Twilio
        │
        ▼
Log Outreach to Sheet
(Google Sheets — Outreach_Log tab)
```

---

## Node Breakdown

| # | Node | Type | What It Does |
|---|------|------|--------------|
| 1 | **Daily 9AM Trigger** | Schedule Trigger | Fires every day at 9:00 AM; kicks off the full lapsed-client check loop |
| 2 | **Fetch All Clients** | Google Sheets | Reads the `Client Retention Details` sheet; filters rows where `Status = Lapsed` |
| 3 | **IF Client Lapsed 90+ Days** | IF | Evaluates whether `(today − last_visit_date) ≥ 90 days`; TRUE proceeds to outreach, FALSE exits silently |
| 4 | **OpenRouter Chat Model** | LangChain | Provides `qwen/qwen3-32b` as the LLM for the AI Agent |
| 5 | **AI Agent** | LangChain | Drafts a personalized reactivation SMS under 160 characters based on client name, last visit, and preferred treatment |
| 6 | **Send SMS — Twilio** | Twilio | Sends the generated SMS from your Twilio number to the client's phone number |
| 7 | **Log Outreach to Sheet** | Google Sheets | Appends or updates a row in the `Outreach_Log` tab with date, client info, message sent, and Twilio delivery status |

---

## Google Sheets Schema

### Sheet 1 — `Client Retention Details`

This is the source sheet. The workflow filters this for `Status = Lapsed` and reads each matching row.

| Column | Format | Required | Description |
|--------|--------|----------|-------------|
| `client_name` | Text | ✅ | Full client name |
| `phone_number` | Text (E.164) | ✅ | Client phone number — must include country code (e.g. `+12125551234`) |
| `email` | Text | ✅ | Client email address |
| `last_visit_date` | Date (`YYYY-MM-DD`) | ✅ | Date of client's most recent visit — must use this exact format |
| `treatment_type` | Text | ✅ | Type of treatment received at last visit |
| `preferred_treatment` | Text | ✅ | Client's preferred or requested treatment |
| `Status` | Text | ✅ | Set to `Lapsed` for clients eligible for reactivation outreach |

> **Important:** `last_visit_date` must be formatted as `YYYY-MM-DD`. The IF node and AI Agent both compute days-since-visit using JavaScript's `Date` object — any other date format will produce incorrect results.

### Sheet 2 — `Outreach_Log`

Create this tab manually in the same spreadsheet before activating the workflow. The Log Outreach node writes one row per outreach attempt.

| Column | Value Source | Description |
|--------|-------------|-------------|
| `date_sent` | Auto (today's date) | Date the SMS was sent |
| `client_name` | From `Fetch All Clients` | Client's full name |
| `phone_number` | From `Fetch All Clients` | Phone number the SMS was sent to |
| `treatment_type` | From `Fetch All Clients` | Treatment type from the source sheet |
| `days_since_visit` | Computed | Calculated as `(today - last_visit_date)` in days |
| `sms_message_sent` | From AI Agent output | The exact SMS text that was sent |
| `twilio_status` | From Twilio response | Delivery status returned by Twilio (e.g. `queued`, `sent`, `delivered`) |

---

## AI Agent Behaviour

The AI Agent is configured with a system prompt defining its persona and hard constraints:

- **Persona:** Warm, professional client care coordinator at a luxury medical spa in the US
- **Tone:** Caring, not salesy
- **Character limit:** 160 characters (standard SMS)
- **Opening:** Must start with `Hi [first name],`
- **CTA:** Must end with a direct rebooking call-to-action — `Reply BOOK or call us at [CLINIC_PHONE_NUMBER]`
- **Restrictions:** No emojis. Never mention the exact number of days since last visit.

The prompt is built dynamically with the client's name, last visit date, preferred treatment, and computed days since last visit from the `Fetch All Clients` node.

> **Before activating:** Replace `[CLINIC_PHONE_NUMBER]` in the AI Agent's system prompt with your actual clinic phone number.

---

## Setup Checklist

1. **Import the workflow** — In n8n: Import → paste or upload `workflow.json`
2. **Prepare your Google Sheet** — Create a spreadsheet with the `Client Retention Details` tab using the column schema above; add a second tab named exactly `Outreach_Log`
3. **Format `last_visit_date` as `YYYY-MM-DD`** — Any other format will break the 90-day calculation
4. **Connect Google Sheets credentials** — Link your Google Sheets OAuth2 account to both the **Fetch All Clients** and **Log Outreach to Sheet** nodes
5. **Update the Sheet URL in Fetch All Clients** — Replace the spreadsheet URL with your own sheet's URL
6. **Update the Sheet ID in Log Outreach to Sheet** — Replace `YOUR_GOOGLE_SHEET_ID` in the Log Outreach node with your actual spreadsheet URL or ID
7. **Set up a Twilio account** — Get a Twilio phone number at [twilio.com](https://www.twilio.com/); note your Account SID, Auth Token, and Twilio number in E.164 format
8. **Add Twilio credentials in n8n** — Create a Twilio credential with your Account SID and Auth Token; link it to the **Send SMS — Twilio** node
9. **Update the Twilio `from` number** — In the Send SMS node, replace the `from` value with your Twilio phone number in E.164 format (e.g. `+12125551234`)
10. **Connect OpenRouter credentials** — Link your OpenRouter API key to the **OpenRouter Chat Model** node
11. **Update the clinic phone number in the AI Agent** — In the system prompt, replace `[CLINIC_PHONE_NUMBER]` with your real clinic number
12. **Test with one client row** — Add a single test row with `Status = Lapsed` and a `last_visit_date` 90+ days ago; run manually to verify the SMS is generated and sent correctly
13. **Verify the Outreach_Log** — Confirm a row was appended with the correct message text and Twilio status
14. **Activate the workflow** — Toggle it on; it will begin running daily at 9:00 AM

---

## Credentials Required

| Credential | Node(s) | Notes |
|------------|---------|-------|
| Google Sheets OAuth2 | Fetch All Clients, Log Outreach to Sheet | Needs read + write access to your spreadsheet |
| Twilio API | Send SMS — Twilio | Requires Account SID + Auth Token; get from [twilio.com/console](https://console.twilio.com/) |
| OpenRouter API | OpenRouter Chat Model | Uses `qwen/qwen3-32b`; get key from [openrouter.ai](https://openrouter.ai/) |

---

## IF Node Logic

The **IF Client Lapsed 90+ Days** node checks whether the `Status` field exists (i.e. the row was fetched with `Status = Lapsed`) and proceeds accordingly. The 90-day threshold is enforced in the **AI Agent prompt** via the computed `days_since_visit` expression:

```javascript
Math.floor((Date.now() - new Date($json['last_visit_date']).getTime()) / (1000 * 60 * 60 * 24))
```

To change the lapse threshold from 90 days to another value (e.g. 60 days), update this expression in the AI Agent's prompt text and adjust the Google Sheets filter logic accordingly.

---

## Customization Ideas

- **Adjust the lapse window** — Change the 90-day threshold to 60 or 120 days by modifying the IF node condition and the days expression in the prompt
- **Add an email fallback** — After the Twilio node, add a Gmail node to send an email to clients whose SMS delivery status comes back as `failed` or `undelivered`
- **Multi-treatment personalization** — Expand the AI Agent prompt to include specific treatment offers or seasonal promotions based on the `preferred_treatment` field
- **Reactivation tiers** — Add a second IF node to send a different, more urgent SMS to clients lapsed 180+ days versus 90–180 days
- **Opt-out handling** — Add a Twilio webhook listener workflow that catches `STOP` replies and updates the client's `Status` column to `Opted Out` to prevent future outreach
- **WhatsApp instead of SMS** — Swap the Twilio SMS node for a Twilio WhatsApp node to send outreach via WhatsApp for international clients
- **CRM sync** — After the Log Outreach node, add a HubSpot or Airtable node to sync outreach records to your CRM alongside the sheet log

---

## Use Cases

- **Medical spas** — Reactivate clients who haven't booked a facial, laser, or injectable appointment in 90+ days
- **Salons & barbershops** — Bring back clients with a personalized reminder tied to their last service
- **Physiotherapy & wellness clinics** — Follow up with patients who have lapsed between treatment plans
- **Dental practices** — Reach out to patients overdue for a check-up or hygiene appointment
- **Personal trainers & gyms** — Re-engage members who haven't checked in recently with a targeted offer

---

## Tech Stack

| Tool | Role |
|------|------|
| n8n | Workflow automation |
| Google Sheets | Client data source and outreach log |
| OpenRouter (`qwen/qwen3-32b`) | LLM powering personalized SMS generation |
| Twilio | SMS delivery |
| LangChain (n8n) | AI Agent orchestration |