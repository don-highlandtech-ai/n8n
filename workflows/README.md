# Appointment Scheduler - n8n Workflow

An automated appointment scheduling system that saves approximately 3 hours per week by handling booking, calendar sync, reminders, and rescheduling.

## Overview

This solution consists of three n8n workflows:

1. **Booking Intake** (`appointment-scheduler-booking.json`) - Handles new appointment requests
2. **Reschedule/Cancel** (`appointment-scheduler-reschedule-cancel.json`) - Handles appointment changes
3. **Reminders** (`appointment-scheduler-reminders.json`) - Sends automated reminders

## Features

- Webhook-based booking intake (works with any form tool)
- Google Calendar integration with conflict detection
- Automatic Google Meet link generation
- Email confirmations and notifications via Gmail
- Optional SMS via Twilio
- Automated reminders (24h and 2h before appointments)
- Secure reschedule/cancel links with HMAC tokens
- Google Sheets as system of record
- Quiet hours for reminders (no notifications 8pm-8am)
- Dedupe protection against double bookings
- Comprehensive error logging

---

## Quick Start (20 minutes)

### Step 1: Create Google Sheet (2 minutes)

1. Create a new Google Sheet
2. Create three tabs:
   - `Appointments` (main data)
   - `ErrorLog` (error tracking)
   - `ReminderLog` (reminder tracking)

3. Add headers to the **Appointments** tab (Row 1):
```
timestamp_created | appointment_id | status | client_first_name | client_last_name | client_email | client_phone | service_type | start_time | end_time | timezone | calendar_event_id | meeting_link | reschedule_count | dedupe_key | last_action | last_action_ts | notes | reschedule_url | cancel_url | reminder_24h_sent | reminder_2h_sent | consent_sms
```

4. Add headers to the **ErrorLog** tab:
```
timestamp | error_type | error_message | client_email | request_data
```

5. Add headers to the **ReminderLog** tab:
```
timestamp | appointment_id | client_email | reminder_type | appointment_time | email_sent | sms_sent
```

6. Copy the Sheet ID from the URL: `https://docs.google.com/spreadsheets/d/YOUR_SHEET_ID_HERE/edit`

### Step 2: Set Up n8n Credentials (5 minutes)

Create these credentials in n8n (Settings > Credentials):

1. **Google OAuth2** - For Calendar, Gmail, and Sheets
   - Create OAuth2 credentials in Google Cloud Console
   - Enable: Google Calendar API, Gmail API, Google Sheets API
   - Scopes needed: `calendar`, `gmail.send`, `spreadsheets`

2. **Twilio** (Optional) - For SMS
   - Get Account SID and Auth Token from Twilio Console

### Step 3: Import Workflows (3 minutes)

1. In n8n, go to **Workflows > Import from File**
2. Import all three JSON files:
   - `appointment-scheduler-booking.json`
   - `appointment-scheduler-reschedule-cancel.json`
   - `appointment-scheduler-reminders.json`

### Step 4: Configure Each Workflow (5 minutes)

Open each workflow and update the **Set Config** node:

| Setting | Description | Default |
|---------|-------------|---------|
| `timezone` | Your timezone | `America/New_York` |
| `calendarId` | Google Calendar ID | `primary` |
| `bookingWindowDays` | Days ahead for booking | `30` |
| `workingHoursStart` | Business hours start | `09:00` |
| `workingHoursEnd` | Business hours end | `17:00` |
| `workingDays` | Working days (1=Mon, 5=Fri) | `[1,2,3,4,5]` |
| `appointmentDurationMinutes` | Default appointment length | `30` |
| `bufferBeforeMinutes` | Buffer before appointments | `10` |
| `bufferAfterMinutes` | Buffer after appointments | `10` |
| `maxDaysAhead` | Maximum days in advance | `60` |
| `minNoticeHours` | Minimum notice required | `2` |
| `allowWeekends` | Allow weekend bookings | `false` |
| `useSms` | Enable SMS notifications | `false` |
| `requireSmsConsent` | Require SMS consent | `true` |
| `ownerNotificationEmail` | Your email for notifications | (required) |
| `fromEmailName` | Email sender name | `Appointment Scheduler` |
| `sheetId` | Your Google Sheet ID | (required) |
| `sheetTabName` | Main data tab name | `Appointments` |
| `testMode` | Send all emails to owner | `true` |
| `maxReschedules` | Max reschedules allowed | `2` |
| `tokenSecret` | Secret for HMAC tokens | (change this!) |
| `webhookBaseUrl` | Your n8n webhook URL | (required) |
| `businessName` | Your business name | `Your Business Name` |
| `quietHoursStart` | No reminders after | `20:00` |
| `quietHoursEnd` | No reminders before | `08:00` |

**IMPORTANT**: The `tokenSecret` must be identical across all three workflows!

### Step 5: Connect Credentials (3 minutes)

In each workflow, click on nodes that require credentials and select your configured credentials:
- Google Calendar nodes: Google OAuth2
- Gmail nodes: Google OAuth2
- Google Sheets nodes: Google OAuth2
- Twilio nodes (if using): Twilio

### Step 6: Activate Workflows (2 minutes)

1. Test in testMode first (all emails go to ownerNotificationEmail)
2. Activate all three workflows
3. Copy the webhook URLs from the Booking and Reschedule/Cancel webhooks

---

## Connecting Your Booking Form

### Webhook Endpoint

```
POST https://your-n8n-instance.com/webhook/appointment-booking
```

### Required Payload

```json
{
  "first_name": "John",
  "last_name": "Doe",
  "email": "john@example.com",
  "requested_start": "2024-01-15T10:00:00-05:00"
}
```

### Optional Fields

```json
{
  "phone": "+15551234567",
  "service_type": "Consultation",
  "requested_end": "2024-01-15T10:30:00-05:00",
  "notes": "First time client",
  "consent_sms": true
}
```

### Success Response (200)

```json
{
  "success": true,
  "appointment_id": "APT-ABC123-XYZ",
  "message": "Appointment confirmed! Check your email for details.",
  "start": "2024-01-15T10:00:00-05:00",
  "end": "2024-01-15T10:30:00-05:00",
  "meeting_link": "https://meet.google.com/xxx-xxxx-xxx"
}
```

### Conflict Response (409)

```json
{
  "success": false,
  "conflict": true,
  "message": "The requested time slot is not available.",
  "alternatives": [
    {
      "start": "2024-01-15T11:00:00-05:00",
      "end": "2024-01-15T11:30:00-05:00",
      "formatted": "Monday, January 15 at 11:00 AM"
    }
  ]
}
```

### Error Response (400)

```json
{
  "success": false,
  "error": "Missing required fields",
  "code": "VALIDATION_ERROR"
}
```

### Form Integration Examples

**Webflow Form:**
Use Webflow's native webhook or Zapier to POST to the n8n webhook.

**Typeform:**
Use Typeform's webhook integration to send responses to n8n.

**HTML Form:**
```html
<form id="booking-form">
  <input name="first_name" required>
  <input name="last_name" required>
  <input name="email" type="email" required>
  <input name="phone" type="tel">
  <input name="requested_start" type="datetime-local" required>
  <textarea name="notes"></textarea>
  <button type="submit">Book Appointment</button>
</form>

<script>
document.getElementById('booking-form').addEventListener('submit', async (e) => {
  e.preventDefault();
  const formData = new FormData(e.target);
  const data = Object.fromEntries(formData);

  // Convert datetime-local to ISO
  data.requested_start = new Date(data.requested_start).toISOString();

  const response = await fetch('https://your-n8n-instance.com/webhook/appointment-booking', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data)
  });

  const result = await response.json();
  if (result.success) {
    alert('Appointment booked! Check your email.');
  } else if (result.conflict) {
    alert('Time not available. Alternatives: ' + result.alternatives.map(a => a.formatted).join(', '));
  } else {
    alert('Error: ' + result.error);
  }
});
</script>
```

---

## Token Format and Security

### HMAC SHA256 Token Structure

Reschedule and cancel links use secure tokens to prevent unauthorized access.

**Token Format:**
```
base64url(payload).signature
```

**Payload Structure:**
```json
{
  "appointment_id": "APT-ABC123-XYZ",
  "client_email": "john@example.com",
  "expires_at": "2024-02-15T10:00:00.000Z"
}
```

**Signature Generation:**
```javascript
const payload = JSON.stringify({ appointment_id, client_email, expires_at });
const payloadBase64 = Buffer.from(payload).toString('base64url');
const signature = crypto
  .createHmac('sha256', tokenSecret)
  .update(payloadBase64)
  .digest('base64url');
const token = `${payloadBase64}.${signature}`;
```

**Configuration:**
- Set `tokenSecret` in the Set Config node (minimum 32 characters recommended)
- Must be identical across all workflows
- Tokens expire in 30 days by default

### Reschedule URL Format

```
https://your-n8n-instance.com/webhook/appointment-reschedule?token=TOKEN&action=reschedule&new_start=2024-01-16T14:00:00-05:00
```

### Cancel URL Format

```
https://your-n8n-instance.com/webhook/appointment-reschedule?token=TOKEN&action=cancel
```

---

## Test Plan

### Test 1: Available Booking

1. Set `testMode: true` in config
2. Send POST to booking webhook:
```bash
curl -X POST https://your-n8n/webhook/appointment-booking \
  -H "Content-Type: application/json" \
  -d '{
    "first_name": "Test",
    "last_name": "User",
    "email": "test@example.com",
    "requested_start": "2024-01-15T10:00:00-05:00"
  }'
```
3. Verify:
   - Response shows success with appointment_id
   - Calendar event created
   - Row added to Sheets
   - Confirmation email sent to ownerNotificationEmail

### Test 2: Conflict Booking with Alternatives

1. Send same request again (same time slot)
2. Verify:
   - Response returns 409 with alternatives array
   - No duplicate calendar event
   - No duplicate Sheets row (dedupe working)

### Test 3: Reschedule Success

1. Copy reschedule_url from Sheets or confirmation email
2. Add `&new_start=2024-01-16T14:00:00-05:00` to URL
3. Visit the URL or GET request
4. Verify:
   - Response shows success
   - Calendar event updated (same event ID)
   - Sheets row updated with new time and incremented reschedule_count
   - Confirmation email sent

### Test 4: Cancellation Success

1. Copy cancel_url from Sheets or confirmation email
2. Visit the URL or GET request
3. Verify:
   - Response shows success
   - Calendar event deleted
   - Sheets row status changed to "Cancelled"
   - Cancellation email sent

### Test 5: Reminder Run

1. Create a test appointment 24-26 hours in the future
2. Manually execute the Reminders workflow
3. Verify:
   - Reminder email sent
   - reminder_24h_sent flag set to "true" in Sheets
   - Entry added to ReminderLog tab

---

## Troubleshooting

### Common Issues

**"Missing credentials" error:**
- Ensure all Google OAuth2 credentials are connected to the nodes
- Re-authenticate if tokens expired

**"Sheet not found" error:**
- Verify sheetId matches your Google Sheet URL
- Ensure the sheet has the correct tab names (Appointments, ErrorLog, ReminderLog)

**Emails not sending:**
- Check Gmail API is enabled in Google Cloud Console
- Verify OAuth scopes include gmail.send

**Calendar events not creating:**
- Check Calendar API is enabled
- Verify calendarId is correct (use "primary" for default calendar)

**Token validation failing:**
- Ensure tokenSecret is identical across all workflows
- Check token has not expired (30 day limit)

### Viewing Logs

- Check the **ErrorLog** tab in your Google Sheet for error details
- Check n8n execution logs for detailed debugging

---

## Swapping Components

### Using Outlook Calendar Instead of Google Calendar

1. Replace Google Calendar nodes with Microsoft Outlook nodes
2. Update credential to Microsoft OAuth2
3. Adjust event creation/update parameters as needed

### Using SendGrid Instead of Gmail

1. Replace Gmail nodes with SendGrid nodes
2. Create SendGrid credentials with API key
3. Update email parameters for SendGrid format

### Using Different Form Providers

The webhook accepts standard JSON, so any service that can POST JSON works:
- Tally, JotForm, Google Forms (via Apps Script)
- Make/Integromat, Zapier
- Custom frontend applications

---

## Support

For issues or questions:
1. Check the ErrorLog in your Google Sheet
2. Review n8n execution logs
3. Verify all credentials are properly connected
4. Ensure tokenSecret matches across workflows

---

## License

MIT License - Use freely for personal and commercial projects.
