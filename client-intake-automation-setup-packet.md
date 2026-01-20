# Client Intake Automation Setup Packet

## Complete Self-Serve Guide for n8n

---

# Section 0: What This Automation Does

## Overview

This automation responds to new leads around the clock, qualifies them automatically, schedules consultations, and keeps your CRM updated without manual work.

## What You Get

- **Instant lead response**: Every lead gets a personalized email within 2 minutes, even at 3 AM
- **Automatic qualification**: Leads are scored and routed based on fit, urgency, and budget signals
- **Self-service scheduling**: Qualified leads book directly on your calendar using your booking link
- **CRM always current**: HubSpot contacts, notes, and deal stages update automatically
- **Smart follow-ups**: 4-hour, 24-hour, and 3-day follow-ups until they book or reply
- **No leads lost**: Stale leads get flagged and you get notified

## Time Savings

- **Setup time**: About 1 hour
- **Weekly time saved**: 8-12 hours of manual lead handling

## What You Need Before Starting

1. An n8n account (cloud or self-hosted)
2. A Google Workspace account (Gmail + Google Calendar)
3. A HubSpot account (free tier works)
4. A booking link (Google Appointment Schedule, Calendly, or similar)
5. 45-60 minutes of focused setup time

---

# Section 1: Setup Checklist (10 Minutes)

## Accounts Needed

| Account | Purpose | Where to Get It |
|---------|---------|-----------------|
| n8n | Runs the automation | n8n.io |
| Google Workspace | Email and calendar | workspace.google.com |
| HubSpot | CRM for contacts | hubspot.com |

## Permissions Required

### Google Account
- Gmail: Send emails on your behalf
- Calendar: Read and write calendar events

### HubSpot
- Contacts: Read and write
- Notes: Create
- Deals: Create (optional)

## Information to Gather

Before you start, collect these items and write them down:

| Item | Example | Your Value |
|------|---------|------------|
| HubSpot Private App Token | pat-na1-xxxxx | ___________ |
| Google Account Email | you@yourcompany.com | ___________ |
| Calendar Name | Consultations | ___________ |
| Calendar ID | your-email@gmail.com or calendar ID | ___________ |
| Booking Link | https://calendar.app.google/abc123 | ___________ |
| Internal Owner Email | owner@yourcompany.com | ___________ |
| Your Company Name | Acme Consulting | ___________ |
| Your Timezone | America/New_York | ___________ |

## How to Get Your HubSpot Private App Token

1. Log into HubSpot
2. Click the gear icon (Settings) in the top right
3. In the left sidebar, click "Integrations" then "Private Apps"
4. Click "Create a private app"
5. Name it "n8n Intake Automation"
6. Go to the "Scopes" tab
7. Select these scopes:
   - crm.objects.contacts.read
   - crm.objects.contacts.write
   - crm.objects.deals.read (optional)
   - crm.objects.deals.write (optional)
8. Click "Create app"
9. Copy the token that appears and save it somewhere safe

## How to Get Your Google Calendar ID

1. Go to calendar.google.com
2. Find your calendar in the left sidebar
3. Click the three dots next to it
4. Click "Settings and sharing"
5. Scroll down to "Integrate calendar"
6. Copy the "Calendar ID" (looks like an email address)

---

# Section 2: Step-by-Step Install Guide

## Step 1: Import the Workflow JSON into n8n

1. Log into your n8n instance
2. Click "Workflows" in the left sidebar
3. Click the "Add Workflow" button (or the + icon)
4. Click the three dots menu in the top right
5. Select "Import from File" or "Import from URL"
6. Paste or upload the workflow JSON (provided at the end of this document)
7. Click "Import"
8. You should see the workflow appear with all nodes

**Do this twice**: Once for "Workflow 1: Intake Automation" and once for "Workflow 2: Calendar Watcher"

## Step 2: Connect Google Credentials

1. In your imported workflow, click on any Gmail or Google Calendar node
2. In the credentials dropdown, click "Create New"
3. Select "Google OAuth2"
4. Follow the prompts to sign in with your Google account
5. Grant the requested permissions
6. Name the credential "Google OAuth2" (important: use this exact name)
7. Click "Save"

**Note**: You only need to do this once. All Google nodes will use the same credential.

## Step 3: Connect HubSpot Credentials

1. Click on any HubSpot node in the workflow
2. In the credentials dropdown, click "Create New"
3. Select "HubSpot Private App"
4. Paste your Private App Token (from the checklist)
5. Name the credential "HubSpot Private App" (use this exact name)
6. Click "Save"

## Step 4: Set Required Variables

Several nodes have placeholder values you need to replace. Here is exactly where to find them:

### In the "Set Config" Node (first node after the webhook):

Double-click the "Set Config" node and update these values:

| Field | Replace This | With Your Value |
|-------|--------------|-----------------|
| bookingLink | https://your-booking-link-here | Your actual booking link |
| internalOwnerEmail | owner@yourdomain.com | Your email address |
| companyName | Your Company | Your company name |
| calendarId | primary | Your calendar ID |
| timezone | America/New_York | Your timezone |

### In the Calendar Watcher Workflow:

Double-click the "Set Config" node and update the same values.

## Step 5: Create HubSpot Custom Properties

Before testing, create these custom properties in HubSpot:

1. Go to HubSpot Settings (gear icon)
2. Click "Properties" in the left sidebar
3. Select "Contact properties"
4. Click "Create property" for each of these:

| Property Name | Internal Name | Field Type |
|---------------|---------------|------------|
| Intake Status | intake_status | Dropdown select |
| Intake Score | intake_score | Number |
| Intake ID | intake_id | Single-line text |
| Last Follow-up | last_followup_date | Date picker |
| Follow-up Count | followup_count | Number |

For "Intake Status" dropdown, add these options:
- new
- qualifying
- qualified
- nurture
- disqualified
- booked
- stale

## Step 6: Test with Sample Lead

1. In the Intake Automation workflow, click the "Webhook" node
2. Click "Listen for Test Event"
3. Copy the webhook URL that appears
4. Open a new browser tab
5. Use a tool like webhook.site or simply use this curl command in terminal, or use the test feature in n8n

**Test Payload** (copy this exactly):

```json
{
  "email": "test@example.com",
  "firstName": "Test",
  "lastName": "Lead",
  "phone": "555-123-4567",
  "company": "Test Company",
  "message": "I need help with automation",
  "source": "website",
  "useCase": "lead management",
  "timeline": "this month",
  "budget": "5000-10000"
}
```

6. Send this to your webhook URL
7. Watch the workflow execute
8. Check that:
   - The workflow completes without errors (green checkmarks)
   - A test email arrives (check spam folder)
   - A contact appears in HubSpot

## Step 7: Turn It On

1. In each workflow, toggle the "Active" switch in the top right to ON
2. The Intake workflow is now listening for leads
3. The Calendar Watcher will run every 10 minutes

**Congratulations! Your intake automation is live.**

---

# Section 3: Where to Paste the Webhook Link

Your webhook URL looks like this:
```
https://your-n8n-instance.com/webhook/intake-automation
```

Copy this URL and paste it into your lead sources:

## Website Contact Form

Most form builders have a "webhook" or "send data to URL" option:

1. Open your form builder settings
2. Look for "Integrations" or "Actions after submit"
3. Choose "Webhook" or "Send to URL"
4. Paste your webhook URL
5. Make sure the form fields map to: email, firstName, lastName, phone, company, message

## Chat Widget (Intercom, Drift, Crisp)

1. Go to your chat tool settings
2. Find "Integrations" or "Webhooks"
3. Create a new webhook for "New conversation" or "New lead"
4. Paste your webhook URL

## Typeform

1. Open your form in Typeform
2. Go to "Connect" tab
3. Click "Webhooks"
4. Click "Add a webhook"
5. Paste your webhook URL
6. Click "Save webhook"

## Webflow

1. Open your site in Webflow
2. Go to Site Settings > Forms
3. Under "Form submission URL" add your webhook URL
4. Or use Webflow Logic to send to your webhook

## Zapier (Connecting Other Tools)

1. Create a new Zap
2. Choose your trigger app (the source of leads)
3. For the action, choose "Webhooks by Zapier"
4. Select "POST"
5. Paste your webhook URL
6. Map the fields to match the expected format

## Required Fields in Your Payload

Your webhook expects JSON with at least these fields:

```json
{
  "email": "required",
  "firstName": "optional but recommended",
  "lastName": "optional but recommended",
  "phone": "optional",
  "company": "optional",
  "message": "optional",
  "source": "optional",
  "useCase": "optional for scoring",
  "timeline": "optional for scoring",
  "budget": "optional for scoring"
}
```

---

# Section 4: Editable Settings

All settings you might want to change are in one place: the "Set Config" node at the start of each workflow.

## General Settings

| Setting | Default Value | What It Does |
|---------|---------------|--------------|
| bookingLink | https://your-booking-link-here | Link included in emails for scheduling |
| internalOwnerEmail | owner@yourdomain.com | Gets notifications for qualified leads, bookings, errors |
| companyName | Your Company | Used in email signatures |
| calendarId | primary | Which calendar to watch for bookings |
| timezone | America/New_York | Used for business hours awareness |

## Qualification Scoring Thresholds

| Threshold | Default | What It Means |
|-----------|---------|---------------|
| qualifiedMin | 7 | Score 7-10 = Qualified (ready to book) |
| nurtureMin | 4 | Score 4-6 = Nurture (needs warming) |
| disqualifiedMax | 3 | Score 0-3 = Disqualified (poor fit) |

## Follow-up Timing

| Follow-up | Default Delay | When It Sends |
|-----------|---------------|---------------|
| First | 4 hours | Same day, afternoon if morning lead |
| Second | 24 hours | Next day (required) |
| Third | 72 hours | 3 days later |
| Stale Alert | 168 hours | 7 days, notifies internal owner |

## Email Settings

| Setting | Default | Notes |
|---------|---------|-------|
| Subject: Initial | "Thanks for reaching out, {{firstName}}" | First response |
| Subject: Follow-up 1 | "Quick follow-up, {{firstName}}" | 4 hour follow-up |
| Subject: Follow-up 2 | "Still interested, {{firstName}}?" | 24 hour follow-up |
| Subject: Follow-up 3 | "Last chance to connect" | 3 day follow-up |
| Signature | Best regards, [Your Company] | Appears in all emails |

## Scoring Rules (Editable in Function Node)

Open the "Calculate Score" node to adjust these rules:

### Use Case Fit (0-3 points)
- 3 points: Perfect match keywords (automation, integration, workflow)
- 2 points: Good match keywords (efficiency, process)
- 1 point: Partial match
- 0 points: No match or unclear

### Timeline Urgency (0-2 points)
- 2 points: "immediately", "asap", "this week", "urgent"
- 1 point: "this month", "this quarter"
- 0 points: "next year", "just researching", or blank

### Budget Signal (0-2 points)
- 2 points: Budget mentioned and > $5,000
- 1 point: Budget mentioned and > $1,000
- 0 points: No budget mentioned or < $1,000

### Role/Seniority (0-2 points)
- 2 points: Owner, CEO, Director, VP, Head of
- 1 point: Manager, Lead
- 0 points: Other or not specified

### Data Completeness (0-1 point)
- 1 point: Phone number provided
- 0 points: No phone number

---

# Section 5: Email Templates

Copy and customize these templates. They are pre-loaded in your workflow but you can edit them in the Gmail nodes.

## Instant Response Email

**Subject**: Thanks for reaching out, {{firstName}}

```
Hi {{firstName}},

Thank you for contacting us. I received your message and wanted to reach out personally.

To make sure we can help you effectively, I have a few quick questions:

1. What is the main challenge you are trying to solve?
2. What does your timeline look like for getting started?
3. Have you set aside a budget for this project?
4. What is your role at {{company}}?

You can reply to this email with your answers, or if you prefer, book a time to chat directly:

[Book a 15-minute call]({{bookingLink}})

If you already have times in mind, just reply with 2-3 options that work for you and I will confirm one.

Looking forward to connecting.

Best regards,
{{companyName}} Team
```

## Qualified Lead Email (Booking Push)

**Subject**: Let's schedule your consultation, {{firstName}}

```
Hi {{firstName}},

Based on what you shared, it sounds like we could be a great fit to help you with {{useCase}}.

I would love to learn more about your goals and show you how we have helped similar companies.

Pick a time that works for you:

[Book Your Consultation]({{bookingLink}})

The call takes about 15-20 minutes. We will cover your current situation, goals, and whether our approach makes sense for you.

Talk soon,
{{companyName}} Team
```

## Nurture Lead Email

**Subject**: Some resources for you, {{firstName}}

```
Hi {{firstName}},

Thank you for your interest. Based on what you shared, I wanted to send over some resources that might help.

[Add relevant content links here]

When you are ready to discuss further, feel free to book a time:

[Schedule a Call]({{bookingLink}})

No pressure. We are here when the timing is right.

Best,
{{companyName}} Team
```

## Disqualified Lead Email

**Subject**: Thanks for your interest, {{firstName}}

```
Hi {{firstName}},

Thank you for reaching out to us.

After reviewing your inquiry, it looks like we might not be the best fit for what you need right now. We specialize in [your specialty], and it sounds like your needs might be better served by [alternative suggestion].

If things change in the future, feel free to reach back out.

Wishing you the best,
{{companyName}} Team
```

## Follow-up 1 (4 Hours)

**Subject**: Quick follow-up, {{firstName}}

```
Hi {{firstName}},

Just following up on my earlier email. I know things get busy.

If you have a moment, I would love to hear back about those quick questions, or feel free to grab a time on my calendar:

[Book a Call]({{bookingLink}})

Let me know if you have any questions.

Best,
{{companyName}} Team
```

## Follow-up 2 (24 Hours) - REQUIRED

**Subject**: Still interested, {{firstName}}?

```
Hi {{firstName}},

I wanted to check in one more time. I sent over some questions yesterday and wanted to make sure my email did not get buried.

If you are still interested in discussing {{useCase}}, I am happy to help. Just reply to this email or book a time here:

[Book a Call]({{bookingLink}})

If now is not the right time, no worries at all. Just let me know and I will follow up later.

Best,
{{companyName}} Team
```

## Follow-up 3 (3 Days)

**Subject**: Last chance to connect, {{firstName}}

```
Hi {{firstName}},

I have reached out a few times and have not heard back, so I wanted to send one final note.

If you are still interested in exploring how we can help with {{useCase}}, I am here. If not, I completely understand and will close out your inquiry.

Either way, feel free to reach out anytime in the future.

Best,
{{companyName}} Team
```

## Stale Lead Internal Alert

**Subject**: [Intake Alert] Lead went stale: {{firstName}} {{lastName}}

```
A lead has not responded or booked after 7 days.

Lead Details:
- Name: {{firstName}} {{lastName}}
- Email: {{email}}
- Company: {{company}}
- Score: {{score}}
- Status: stale

Action needed: Review and decide whether to manually follow up or close.

View in HubSpot: [Link to contact]
```

---

# Section 6: Troubleshooting FAQ

## "Webhook test works but emails not sending"

**Likely causes:**
1. Google credentials not connected or expired
2. Recipient email is invalid
3. Gmail sending limits reached

**Fix steps:**
1. Open any Gmail node and check if credentials show a green checkmark
2. If not, click the credential and re-authenticate
3. Check the execution log for specific error messages
4. Try sending a test email directly from the Gmail node

## "HubSpot contact not created"

**Likely causes:**
1. HubSpot credentials invalid or expired
2. Missing required scopes on the private app
3. Email address already exists (might be updating instead)

**Fix steps:**
1. Check HubSpot credentials in n8n
2. Go to HubSpot > Settings > Private Apps and verify scopes include contacts.read and contacts.write
3. Search HubSpot for the email address to see if it exists
4. Check the execution log for the specific error

## "Calendar booking not detected"

**Likely causes:**
1. Calendar ID is wrong
2. The booking email does not match the lead email
3. Calendar Watcher workflow is not active
4. Calendar credentials not connected

**Fix steps:**
1. Verify the Calendar ID in the Set Config node matches your actual calendar
2. Check that your booking tool adds the attendee email to the calendar event
3. Make sure the Calendar Watcher workflow toggle is set to Active
4. Re-authenticate Google credentials if needed

## "Duplicates happening"

**Likely causes:**
1. Webhook being triggered multiple times
2. HubSpot search not finding existing contact
3. intake_id not being set correctly

**Fix steps:**
1. Check your form or source tool for duplicate webhook sends
2. Verify the HubSpot search node is configured to search by email
3. Check that intake_id property exists in HubSpot
4. Look at execution logs to see if the same lead is processing twice

## "Follow-ups not stopping"

**Likely causes:**
1. Booking not being detected (see above)
2. intake_status not updating in HubSpot
3. Calendar Watcher workflow not running

**Fix steps:**
1. Check the lead's intake_status in HubSpot - it should be "booked"
2. Manually set intake_status to "booked" to stop follow-ups immediately
3. Check Calendar Watcher workflow execution history
4. Verify the email on the calendar event matches the lead email exactly

## "Permissions error"

**Likely causes:**
1. OAuth token expired
2. Private app token invalid
3. Scopes missing

**Fix steps:**
1. Re-authenticate the credential that is failing
2. For HubSpot: regenerate the private app token and update in n8n
3. For Google: disconnect and reconnect the OAuth credential
4. Check that all required scopes are enabled

## "Workflow errors out randomly"

**Likely causes:**
1. API rate limits
2. Network timeouts
3. Invalid data in a specific lead

**Fix steps:**
1. Check the execution log for the specific error message
2. If rate limit, the workflow has retry logic built in
3. For invalid data, check what fields the problematic lead had
4. You should receive an error notification email with details

---

# Section 7: Testing Checklist

Run through each of these scenarios to verify your automation works correctly.

## Test Scenarios

| # | Scenario | How to Test | Expected Result |
|---|----------|-------------|-----------------|
| 1 | New lead | Send test payload with new email | Contact created in HubSpot, instant email sent |
| 2 | Duplicate lead | Send same payload again | Contact updated (not duplicated), email sent |
| 3 | Missing phone | Send payload without phone field | Works normally, completeness score lower |
| 4 | Qualified routing | Send payload with budget="10000", timeline="asap" | Score 7+, qualified email sent, internal notification |
| 5 | Nurture routing | Send payload with budget="1000", timeline="next quarter" | Score 4-6, nurture email sent |
| 6 | Disqualified routing | Send payload with only email, no other fields | Score 0-3, disqualified email sent |
| 7 | Booking detection | Book a meeting using your booking link with test email | HubSpot status changes to "booked" |
| 8 | Follow-ups stop | After booking, wait and verify | No more follow-up emails sent |
| 9 | Reply detection | Reply to the thread from your test email | Follow-ups should stop (check intake_status) |
| 10 | Error notification | Temporarily break a credential and send lead | You receive error alert email |

## Quick Test Payloads

### Qualified Lead (Score 8+)
```json
{
  "email": "qualified-test@example.com",
  "firstName": "Sarah",
  "lastName": "Director",
  "phone": "555-987-6543",
  "company": "Big Corp",
  "message": "We need automation help urgently",
  "useCase": "workflow automation",
  "timeline": "this week",
  "budget": "15000",
  "role": "Director of Operations"
}
```

### Nurture Lead (Score 5)
```json
{
  "email": "nurture-test@example.com",
  "firstName": "Mike",
  "lastName": "Manager",
  "company": "Medium Co",
  "message": "Exploring options",
  "useCase": "maybe automation",
  "timeline": "next quarter",
  "budget": "2000"
}
```

### Disqualified Lead (Score 2)
```json
{
  "email": "disqualified-test@example.com",
  "firstName": "Pat",
  "message": "Just curious"
}
```

---

# Section 8: Technical Appendix

## Data Schema

### Incoming Webhook Payload

```json
{
  "email": "string (required)",
  "firstName": "string",
  "lastName": "string",
  "phone": "string",
  "company": "string",
  "message": "string",
  "source": "string",
  "useCase": "string",
  "timeline": "string",
  "budget": "string or number",
  "role": "string"
}
```

### HubSpot Contact Properties

| Property | Internal Name | Type | Purpose |
|----------|---------------|------|---------|
| Email | email | string | Primary identifier |
| First Name | firstname | string | Personalization |
| Last Name | lastname | string | Personalization |
| Phone | phone | string | Contact info |
| Company | company | string | Context |
| Intake Status | intake_status | enum | Workflow state |
| Intake Score | intake_score | number | Qualification score |
| Intake ID | intake_id | string | Idempotency key |
| Last Follow-up | last_followup_date | date | Follow-up tracking |
| Follow-up Count | followup_count | number | Follow-up tracking |

### Intake Status Values

| Status | Meaning |
|--------|---------|
| new | Just received, not yet processed |
| qualifying | Being scored |
| qualified | Score 7+, ready for sales |
| nurture | Score 4-6, needs warming |
| disqualified | Score 0-3, not a fit |
| booked | Meeting scheduled |
| stale | No response after 7 days |

## Node List (Intake Workflow)

1. Webhook (trigger)
2. Set Config (configuration variables)
3. Normalize Data (clean and format input)
4. Generate Intake ID (idempotency)
5. Search HubSpot Contact (dedupe check)
6. IF Contact Exists (routing)
7. Create HubSpot Contact (new leads)
8. Update HubSpot Contact (existing leads)
9. Merge (combine paths)
10. Calculate Score (qualification logic)
11. Switch by Score (routing)
12. Send Qualified Email (Gmail)
13. Send Nurture Email (Gmail)
14. Send Disqualified Email (Gmail)
15. Notify Owner - Qualified (Gmail)
16. Create HubSpot Note (logging)
17. Error Handler (catch errors)
18. Send Error Alert (Gmail)

## Node List (Calendar Watcher Workflow)

1. Schedule Trigger (every 10 minutes)
2. Set Config (configuration)
3. Get Recent Calendar Events (Google Calendar)
4. Filter New Bookings (identify new events)
5. Extract Attendee Email (get lead email)
6. Search HubSpot by Email (find contact)
7. IF Contact Found (routing)
8. Update HubSpot Status to Booked (update)
9. Notify Owner - Booking (Gmail)
10. Check Follow-up Queue (find pending follow-ups)
11. Send Due Follow-ups (Gmail)
12. Update Follow-up Count (HubSpot)
13. Check Stale Leads (7 day check)
14. Mark Stale and Notify (update + alert)

## Idempotency

Each lead gets a unique intake_id generated from:
- Email address (lowercase)
- Timestamp (date only)
- Hash function

This prevents duplicate processing if the same lead submits twice on the same day.

## Error Handling

All API nodes (HubSpot, Gmail, Calendar) have:
- Retry on failure: 3 attempts
- Retry delay: 5 seconds
- On final failure: route to error handler

Error handler actions:
- Log error details
- Email internal owner with execution link
- Attempt to create HubSpot note (if possible)

## Rate Limits

| Service | Limit | How We Handle It |
|---------|-------|------------------|
| HubSpot API | 100 requests/10 seconds | Built-in retry |
| Gmail API | 100 emails/day (free), higher for Workspace | Monitor usage |
| Google Calendar | 1,000,000 requests/day | Not a concern |

---

# End of Setup Packet

The workflow JSON files follow below. Import these into n8n to get started.

