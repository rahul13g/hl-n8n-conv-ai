# LetsGo Travel — Hooman Labs n8n Integration

Outbound AI voice agent (Raju) built on Hooman Labs. Two n8n workflows handle the full lead lifecycle — from Google Sheets to call to CRM update.

---

## How It Works

```
New lead in Google Sheets
        ↓
Pre-call workflow creates HL Task → Raju calls the lead
        ↓
Raju qualifies lead in Hinglish (destination, occasion, group, month, budget)
        ↓
Post-call webhook fires → all extracted fields written back to Sheet
```

---

## Files

| File | Purpose |
|------|---------|
| `pre-call-workflow.json` | Import into n8n — triggers on new Sheet row, creates HL outbound task |
| `post-call-workflow.json` | Import into n8n — receives HL webhook after call, updates Sheet row |
| `google-sheets-schema.md` | Exact column names for the CRM sheet |

---

## Google Sheet

Make a copy of the template sheet and use the column schema in `google-sheets-schema.md`.

**Sheet link:** https://docs.google.com/spreadsheets/d/14HB-PDQfAlTY0-0G0bWNhp6M-0IdSMJJIt3pJG0Kj3I/edit

---

## Setup

### 1. Google Sheet
Create a sheet named `Leads` with columns from `google-sheets-schema.md` (order matters).

### 2. Pre-call Workflow
Import `pre-call-workflow.json` into n8n and set:
- Google Sheets credential (OAuth2)
- `YOUR_HL_AGENT_ID` → your Hooman Labs agent ID
- `YOUR_HL_PHONE_NUMBER` → your HL from-number
- `YOUR_CAMPAIGN_ID` → your HL campaign ID
- `YOUR_GOOGLE_SHEET_ID` → your Sheet ID

### 3. Post-call Workflow
Import `post-call-workflow.json` into n8n and set:
- Google Sheets credential (OAuth2)
- Copy the webhook URL from the Webhook node
- Paste that URL into your HL agent config under `actions → callEndConnected` and `callEndNotConnected`

### 4. Activate Both Workflows
Publish both workflows in n8n. Add a row to the Sheet with `Status = New` to test.

---

## Credentials Needed

| Service | Type |
|---------|------|
| Google Sheets | OAuth2 |
| Hooman Labs | API key (raw token, no Bearer prefix) |

---

## Flow Details

**Pre-call guard:** Only processes rows where `Status = New` and phone number is not empty — prevents duplicate calls on re-polls.

**Post-call match:** HL sends `leadId` in the webhook body. The workflow matches it to the `ID` column in the Sheet to update the correct row.

**Fields written after call:** Qualification Status, Destination, Travel Month, Group Type, Budget Per Person, Trip Type, Callback Time, Notes, Call Picked Up, Updated At.
