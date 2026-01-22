# Smart Follow-up Sequences AI Agent

An importable n8n workflow that nurtures inbound and outbound leads with personalized, automated follow-ups based on behavior across email, calendar, forms, and website activity.

**Target Outcome:** Save 5+ hours per week by removing manual follow-ups.

---

## Quick Start (30-Minute Setup)

### Step 1: Import the Workflow (2 minutes)

1. Open your n8n instance
2. Go to **Workflows** > **Add Workflow** > **Import from File**
3. Select `smart-followup-sequences-workflow.json`
4. Click **Import**

### Step 2: Create Google Sheet for Logging (3 minutes)

1. Create a new Google Sheet
2. Name the first tab: `Lead Follow-up Log`
3. Add these column headers in Row 1:
   ```
   timestamp | leadId | email | sequence | step | channel | subject | message_preview | trigger_type | last_activity_type | confidence | compliance_ok | action_taken | error | dedupe_key | test_mode | value_prop_used | personalization_fields
   ```
4. Copy the Sheet ID from the URL (the long string between `/d/` and `/edit`)

### Step 3: Connect Credentials (10 minutes)

Navigate to **Settings** > **Credentials** and create:

| Credential | Required | How to Get |
|------------|----------|------------|
| **Google Sheets OAuth2** | Yes | n8n will guide OAuth flow |
| **Gmail OAuth2** | Yes | Enable Gmail API in Google Cloud Console |
| **OpenAI API** | Yes | Get from platform.openai.com |
| **Google Calendar OAuth2** | For meeting detection | Same Google account as Gmail |
| **Twilio** | Optional (for SMS) | Get from twilio.com/console |

### Step 4: Configure the Workflow (10 minutes)

Open the **"Set Config"** node and update these values:

```javascript
// REQUIRED - Update these:
sheetId: "YOUR_GOOGLE_SHEET_ID_HERE"
ownerNotificationEmail: "your-email@company.com"

// OPTIONAL - Adjust as needed:
crmProvider: "hubspot"          // or "airtable"
emailProvider: "gmail"          // or "smtp"
useSms: false                   // true to enable Twilio SMS
testMode: true                  // START WITH TRUE!
timezone: "America/New_York"
quietHoursStart: "20:00"
quietHoursEnd: "08:00"
confidenceThreshold: 0.70
maxTouchesPerDayPerLead: 2
minHoursBetweenTouches: 16
inactivityThresholdDays: 3
```

### Step 5: Test the Workflow (5 minutes)

1. **Keep `testMode: true`** - this sends all messages to your owner email instead of leads
2. Click **"Test workflow"** button
3. Check your inbox for the test email
4. Check Google Sheet for the log entry
5. Once confirmed working, set `testMode: false` for production

---

## Required Lead Fields

Your CRM or lead source must provide these fields:

| Field | Required | Description |
|-------|----------|-------------|
| `leadId` | Yes | Unique identifier |
| `email` | Yes | Lead's email address |
| `first_name` | Recommended | For personalization |
| `last_name` | Optional | For full name |
| `company` | Recommended | For personalization |
| `role` | Optional | Job title for context |
| `phone` | For SMS | Mobile number with country code |
| `status` | Recommended | lead, customer, closed_lost, etc. |
| `opt_out` | Important | Boolean - respect this! |
| `consent_sms` | For SMS | Boolean - required for SMS |
| `lead_source` | Optional | Where they came from |
| `last_activity_type` | Optional | form_submit, page_visit, etc. |
| `last_activity_ts` | Optional | ISO timestamp |
| `last_touch_ts` | Optional | Last message sent timestamp |

### HubSpot Property Mapping

If using HubSpot, the workflow maps these properties:
- `firstname` → first_name
- `lastname` → last_name
- `company` → company
- `jobtitle` → role
- `phone` → phone
- `lifecyclestage` → status
- `hs_email_optout` → opt_out

### Airtable Column Names

If using Airtable, use these exact column names:
`Email`, `FirstName`, `LastName`, `Company`, `Role`, `Phone`, `Status`, `OptOut`, `ConsentSMS`, `LeadSource`, `LastActivityType`, `LastActivityTimestamp`

---

## Webhook Integration

The workflow includes a universal webhook trigger. Send POST requests to:

```
https://your-n8n-instance.com/webhook/lead-event
```

### Webhook Payload Format

```json
{
  "event_type": "new_lead",
  "lead": {
    "leadId": "lead_123",
    "email": "jane@company.com",
    "first_name": "Jane",
    "last_name": "Smith",
    "company": "Acme Corp",
    "role": "Marketing Director",
    "phone": "+15551234567",
    "status": "lead",
    "opt_out": false,
    "consent_sms": true,
    "lead_source": "Website Form"
  },
  "activity": {
    "type": "form_submission",
    "timestamp": "2024-01-15T10:30:00Z",
    "details": "Downloaded pricing guide"
  }
}
```

### Supported Event Types

| event_type | Triggers |
|------------|----------|
| `new_lead` | New Lead Sequence |
| `form_submit` | New Lead Sequence |
| `engagement` | Engaged Lead Sequence |
| `email_open` | Engaged Lead Sequence |
| `email_click` | Engaged Lead Sequence |
| `page_visit` | Engaged Lead Sequence |
| `inactivity` | Nudge Sequence |
| `meeting_booked` | Post-Meeting Confirmation |

---

## Sequences Reference

### 1. New Lead Sequence
Triggered when a new lead is created.

| Step | Delay | Channel | Template |
|------|-------|---------|----------|
| 1 | Immediate | Email | Welcome |
| 2 | 2 days | Email | Value Prop |
| 3 | 4 days | SMS (if enabled) or Email | Soft CTA |

### 2. Engaged Lead Sequence
Triggered when a lead engages (opens, clicks, visits, submits).

| Step | Delay | Channel | Template |
|------|-------|---------|----------|
| 1 | Immediate | Email | Engagement Response |
| 2 | 1 day | Email | Deeper Value |
| 3 | 3 days | SMS (if enabled) or Email | Meeting Ask |

### 3. Nudge Sequence
Triggered after X days of inactivity (default: 3 days).

| Step | Delay | Channel | Template |
|------|-------|---------|----------|
| 1 | Immediate | Email | Gentle Nudge |
| 2 | 3 days | Email | Value Reminder |
| 3 | 5 days | SMS (if enabled) or Email | Last Chance |

### 4. Post-Meeting Confirmation
Triggered when a meeting is booked via Google Calendar.

| Step | Delay | Channel | Template |
|------|-------|---------|----------|
| 1 | Immediate | Email | Meeting Confirmation |

---

## How to Edit Sequences

1. Open the **"Set Sequences & Value Props"** node
2. Edit the `sequences` JSON object
3. Structure for each sequence:

```javascript
{
  "sequence_key": {
    "name": "Display Name",
    "steps": [
      {
        "step": 1,
        "delayDays": 0,
        "delayHours": 0,
        "channel": "email",        // "email", "sms", or "sms_or_email"
        "template": "template_name"
      }
    ]
  }
}
```

### Adding a New Sequence

1. Add a new key to the sequences object
2. Add the trigger condition in **"Normalize Lead Data"** code node
3. The LLM will automatically generate appropriate messages

---

## How to Add Channels

### Adding SMTP Instead of Gmail

1. Create SMTP credentials in n8n
2. In **"Set Config"**, change `emailProvider` to `"smtp"`
3. Replace Gmail nodes with SMTP nodes
4. Update the node references in the workflow

### Enabling SMS (Twilio)

1. Set `useSms: true` in **"Set Config"**
2. Add `twilioFromNumber: "+1XXXXXXXXXX"` to config
3. Connect Twilio credentials to the **"Twilio: Send SMS"** node
4. Ensure leads have `consent_sms: true` and valid `phone` number

---

## Stop Conditions

The workflow automatically stops sequences when:

1. **Lead opted out** - `opt_out: true`
2. **Meeting booked** - Detected via Google Calendar
3. **Status changed** - To Customer, Closed Lost, or Disqualified
4. **Lead replied** - If reply tracking is implemented
5. **Compliance issue** - LLM detects opt-out request in context

---

## Notifications

The workflow sends alerts to `ownerNotificationEmail` when:

| Condition | Subject Prefix |
|-----------|----------------|
| LLM confidence below threshold | `[Alert] Low Confidence Follow-up` |
| Compliance block (opt-out, stop requested) | `[Alert] Compliance Block` |
| Send failure | Via error handling |

---

## Value Props Library

The workflow includes 5 B2B value propositions that the LLM selects based on lead context:

1. **time_savings** - For operations/admin roles
2. **revenue_growth** - For sales/BD roles
3. **team_efficiency** - For management/directors
4. **data_insights** - For analytics/marketing roles
5. **easy_setup** - For small business/non-technical

Edit these in the **"Set Sequences & Value Props"** node.

---

## Test Plan

### Test 1: New Lead (Basic Flow)
1. Set `testMode: true`
2. Click "Test workflow"
3. **Expected:** Email arrives at owner email with [TEST MODE] prefix
4. **Check:** Google Sheet has log entry with `action_taken: sent`

### Test 2: Engaged Lead Event
1. Send webhook with `event_type: "engagement"`
2. **Expected:** Engaged Lead Sequence Step 1 email sent
3. **Check:** Log shows `sequence: Engaged Lead Sequence`

### Test 3: Inactivity Trigger
1. Let the Schedule trigger run (every 30 min)
2. Or manually trigger with `event_type: "inactivity"`
3. **Expected:** Nudge Sequence initiated
4. **Check:** Log shows `trigger_type: inactivity`

### Test 4: Meeting Booked (Stop Condition)
1. Create a calendar event with a lead's email as attendee
2. **Expected:** All sequences stop, Post-Meeting Confirmation sent
3. **Check:** Log shows `sequence: Post-Meeting Confirmation`

### Test 5: Opt-Out Block
1. Modify Test Lead Data: set `opt_out: true`
2. Run workflow
3. **Expected:** No message sent, compliance alert received
4. **Check:** Log shows `action_taken: blocked`, `error: Lead has opted out`

### Test 6: Duplicate Prevention
1. Run the workflow twice with same lead + sequence + step
2. **Expected:** Second run is skipped
3. **Check:** Log shows `action_taken: skipped_duplicate`

### Test 7: Quiet Hours
1. Set `quietHoursStart: "00:00"` and `quietHoursEnd: "23:59"` (all day quiet)
2. Run workflow
3. **Expected:** Workflow waits until quiet hours end
4. **Check:** Execution shows wait node active

---

## Troubleshooting

### "Sheets: Dedupe Lookup" fails
- Ensure Google Sheets credential has access to the sheet
- Verify sheetId and sheetTabName in config
- Check that the tab exists with correct headers

### LLM returns malformed JSON
- The Parse LLM Response node has fallback handling
- Check OpenAI API key is valid
- Review the LLM response in execution data

### Emails not sending
- Check Gmail credential is connected and authorized
- Verify testMode setting
- Check execution logs for errors

### SMS not sending
- Verify `useSms: true` in config
- Check lead has `consent_sms: true` and valid `phone`
- Verify Twilio credentials and `twilioFromNumber`

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                         TRIGGER LAYER                                │
├─────────────┬─────────────┬─────────────────┬──────────────────────┤
│  Webhook    │  Schedule   │ Google Calendar │   Manual Test        │
│ (Forms,Web) │ (Inactivity)│ (Meeting Book)  │                      │
└──────┬──────┴──────┬──────┴────────┬────────┴──────────┬───────────┘
       │             │               │                    │
       └─────────────┴───────────────┴────────────────────┘
                              │
                    ┌─────────▼─────────┐
                    │    Set Config     │
                    │  (Configuration)  │
                    └─────────┬─────────┘
                              │
                    ┌─────────▼─────────┐
                    │ Normalize Lead    │
                    │ (Data Transform)  │
                    └─────────┬─────────┘
                              │
       ┌──────────────────────┼──────────────────────┐
       │                      │                      │
┌──────▼──────┐      ┌────────▼────────┐    ┌───────▼───────┐
│ Stop Check  │      │  Quiet Hours    │    │ Dedupe Check  │
└──────┬──────┘      └────────┬────────┘    └───────┬───────┘
       │                      │                     │
       └──────────────────────┴─────────────────────┘
                              │
                    ┌─────────▼─────────┐
                    │   Rate Limit      │
                    │     Check         │
                    └─────────┬─────────┘
                              │
                    ┌─────────▼─────────┐
                    │  OpenAI LLM       │
                    │ (Generate Msg)    │
                    └─────────┬─────────┘
                              │
                    ┌─────────▼─────────┐
                    │ Compliance Check  │
                    └─────────┬─────────┘
                              │
              ┌───────────────┼───────────────┐
              │               │               │
       ┌──────▼──────┐ ┌──────▼──────┐ ┌──────▼──────┐
       │  Test Mode  │ │   Gmail     │ │   Twilio    │
       │   Email     │ │  Live Send  │ │  SMS Send   │
       └──────┬──────┘ └──────┬──────┘ └──────┬──────┘
              │               │               │
              └───────────────┴───────────────┘
                              │
                    ┌─────────▼─────────┐
                    │  Google Sheets    │
                    │    (Logging)      │
                    └─────────┬─────────┘
                              │
                    ┌─────────▼─────────┐
                    │  Schedule Next    │
                    │      Step         │
                    └─────────┬─────────┘
                              │
                        ┌─────▼─────┐
                        │   Wait    │───► Loop back to
                        │  (Delay)  │     Stop Check
                        └───────────┘
```

---

## Support

For issues with this workflow:
1. Check the Troubleshooting section above
2. Review n8n execution logs for error details
3. Test with `testMode: true` first

---

*Built for n8n v1.0+ | Uses standard nodes only | No custom code servers required*
