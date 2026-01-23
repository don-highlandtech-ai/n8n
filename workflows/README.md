# Invoice & Billing Automation for n8n

Automated invoice generation, sending, payment tracking, and reminder system.
**Target outcome:** Save 4+ hours per week on billing operations.

## Quick Start (1 Hour Setup)

### Step 1: Import Workflows (5 min)

1. Open your n8n instance
2. Go to **Workflows** > **Import from File**
3. Import `invoice-billing-automation-complete.json`
4. This creates 3 workflows:
   - **Invoice & Billing Automation - Main Workflow** (invoice creation)
   - **Invoice & Billing - Payment Status Tracking** (6-hour polling)
   - **Invoice & Billing - Payment Reminders** (daily at 9 AM)

### Step 2: Create Google Sheet (10 min)

1. Create a new Google Sheet
2. Name the first tab: `Invoice Log`
3. Add these column headers in row 1:

```
timestamp_created | billable_event_id | dedupe_key | client_name | client_email | total | currency | invoice_number | provider | provider_invoice_id | invoice_url | due_date | status | amount_paid | balance_due | last_reminder_type | last_reminder_ts | reminder_count | drive_url | notes | error
```

4. Copy the Sheet ID from the URL:
   `https://docs.google.com/spreadsheets/d/[THIS_IS_YOUR_SHEET_ID]/edit`

### Step 3: Connect Credentials (20 min)

**Required:**
- **Google Sheets** - OAuth2 (for invoice logging)
- **Gmail** - OAuth2 (for sending emails)

**Optional (choose one invoicing provider):**
- **QuickBooks Online** - OAuth2
- **Stripe** - API Key

**Optional (for CRM triggers):**
- **HubSpot** - OAuth2
- **Airtable** - API Key (or use webhook)

### Step 4: Configure Settings (15 min)

In each workflow, find the **"Load Config"** node and update:

```javascript
// REQUIRED - Update these values
config.sheetId = "YOUR_GOOGLE_SHEET_ID_HERE"
config.ownerNotificationEmail = "your-email@company.com"
config.companyName = "Your Company Name"
config.companyAddress = "123 Business St, City, State 12345"
config.replyToEmail = "billing@yourcompany.com"
config.fromEmailName = "Your Company Billing"

// PROVIDER SELECTION
config.invoicingProvider = "quickbooks"  // or "stripe" or "pdf_only"
config.crmProvider = "hubspot"           // or "airtable" or "manual"

// OPTIONAL - Customize as needed
config.currency = "USD"
config.paymentTermsDays = 14
config.reminderCadenceDays = [3, 7, 14]  // days after invoice sent
config.testMode = true                    // SET TO FALSE FOR PRODUCTION
config.lateFeeEnabled = false
config.lateFeeType = "percent"
config.lateFeeAmount = 0
config.timezone = "America/New_York"
config.escalationDaysOverdue = 14
```

### Step 5: Activate Workflows (5 min)

1. Test with `testMode = true` first
2. Send a test webhook (see below)
3. Verify email arrives at your ownerNotificationEmail
4. Set `testMode = false` when ready
5. Toggle workflows to **Active**

---

## How to Send a Test Invoice via Webhook

### Get Your Webhook URL

1. Open the **Main Workflow**
2. Click the **Webhook Trigger** node
3. Copy the **Test URL** or **Production URL**

### Send Test Request

```bash
curl -X POST "YOUR_WEBHOOK_URL" \
  -H "Content-Type: application/json" \
  -d '{
    "client_name": "John Smith",
    "client_email": "john@example.com",
    "client_company": "Acme Corp",
    "client_id": "client-001",
    "billable_event_id": "project-milestone-001",
    "items": [
      {
        "description": "Website Development - Phase 1",
        "quantity": 1,
        "unit_price": 2500.00,
        "amount": 2500.00
      },
      {
        "description": "Hosting Setup",
        "quantity": 1,
        "unit_price": 500.00,
        "amount": 500.00
      }
    ],
    "subtotal": 3000.00,
    "tax": 0,
    "discount": 0,
    "total": 3000.00,
    "notes": "Thank you for your business!",
    "po_number": "PO-12345"
  }'
```

---

## How to Map CRM Trigger (HubSpot)

### Configure HubSpot Deal Stage

1. In the **Main Workflow**, find the **"Is Closed Won?"** node
2. Change `closedwon` to your HubSpot stage ID
3. Common stage IDs:
   - `closedwon` - Closed Won
   - `qualifiedtobuy` - Qualified
   - Check HubSpot Settings > Objects > Deals > Pipelines for custom IDs

### Map Additional HubSpot Fields

Edit the **"Normalize HubSpot Data"** node to map custom properties:

```javascript
// Example: Map custom HubSpot properties
invoice.po_number = $json.properties?.po_number?.value || ''
invoice.service_period_start = $json.properties?.service_start?.value || ''
```

---

## How to Edit Reminder Cadence

In the **Payment Reminders** workflow, edit the **"Load Config"** node:

```javascript
// Days after invoice sent to send reminders
config.reminderCadenceDays = [3, 7, 14]  // Default: 3, 7, 14 days

// After this many days, switch to weekly reminders
config.weeklyAfterDays = 21

// Days overdue to send final notice
config.finalNoticeDaysOverdue = 30

// Maximum reminders per invoice
config.maxReminders = 10
```

---

## Email Templates

### Invoice Email (Sent on Creation)

**Subject:** `Invoice INV-12345 from Your Company due 2024-02-15`

```html
<h2>Invoice from Your Company</h2>

<p>Dear John Smith,</p>

<p>Please find your invoice details below:</p>

<table>
  <tr>
    <td><strong>Invoice Number</strong></td>
    <td>INV-12345</td>
  </tr>
  <tr>
    <td><strong>Amount Due</strong></td>
    <td><strong>$3,000.00 USD</strong></td>
  </tr>
  <tr>
    <td><strong>Due Date</strong></td>
    <td>2024-02-15</td>
  </tr>
  <tr>
    <td><strong>Description</strong></td>
    <td>Website Development - Phase 1, Hosting Setup</td>
  </tr>
</table>

<p><a href="[PAYMENT_LINK]">View and Pay Invoice</a></p>

<p>If you have any questions, please reply to this email.</p>

<p>Thank you for your business!</p>

<hr>
<p>Your Company<br>123 Business St, City, State 12345</p>
```

### Friendly Reminder (Before/Shortly After Due Date)

**Subject:** `Friendly Reminder: Invoice INV-12345 due 2024-02-15`

```html
<h2>Friendly Payment Reminder</h2>

<p>Dear John Smith,</p>

<p>This is a friendly reminder that the following invoice is approaching
its due date:</p>

<table style="background:#f9f9f9;">
  <tr>
    <td><strong>Invoice Number</strong></td>
    <td>INV-12345</td>
  </tr>
  <tr>
    <td><strong>Amount Due</strong></td>
    <td><strong>$3,000.00 USD</strong></td>
  </tr>
  <tr>
    <td><strong>Due Date</strong></td>
    <td>2024-02-15</td>
  </tr>
</table>

<p><a href="[PAYMENT_LINK]">View and Pay Invoice</a></p>

<p>If you have already submitted payment, please disregard this reminder.
Payments may take a few days to process.</p>

<p>Thank you for your business!</p>
```

### Past Due Notice (After Due Date)

**Subject:** `Past Due Notice: Invoice INV-12345 - Payment Required`

```html
<div style="background:#fff3cd;border-left:4px solid #ffc107;padding:15px;">
  <strong>Payment Past Due</strong>
</div>

<p>Dear John Smith,</p>

<p>Our records indicate that the following invoice is now
<strong>7 days past due</strong>:</p>

<table style="background:#fff3cd;">
  <tr>
    <td><strong>Invoice Number</strong></td>
    <td>INV-12345</td>
  </tr>
  <tr>
    <td><strong>Amount Due</strong></td>
    <td><strong style="color:#d9534f;">$3,000.00 USD</strong></td>
  </tr>
  <tr>
    <td><strong>Due Date</strong></td>
    <td>2024-02-15</td>
  </tr>
  <tr>
    <td><strong>Days Overdue</strong></td>
    <td><strong style="color:#d9534f;">7 days</strong></td>
  </tr>
</table>

<p><a href="[PAYMENT_LINK]" style="background:#d9534f;color:white;padding:12px 24px;">Pay Now</a></p>

<p>Please submit payment at your earliest convenience to avoid any service
interruptions or additional fees.</p>
```

### Final Notice (30+ Days Overdue)

**Subject:** `FINAL NOTICE: Invoice INV-12345 - Immediate Payment Required`

```html
<div style="background:#f8d7da;border-left:4px solid #d9534f;padding:15px;">
  <strong style="color:#d9534f;">FINAL NOTICE - Immediate Action Required</strong>
</div>

<p>Dear John Smith,</p>

<p>Despite our previous reminders, the following invoice remains unpaid
and is now <strong>35 days past due</strong>:</p>

<table style="background:#f8d7da;">
  <tr>
    <td><strong>Invoice Number</strong></td>
    <td>INV-12345</td>
  </tr>
  <tr>
    <td><strong>Amount Due</strong></td>
    <td><strong style="color:#d9534f;font-size:18px;">$3,000.00 USD</strong></td>
  </tr>
  <tr>
    <td><strong>Days Overdue</strong></td>
    <td><strong style="color:#d9534f;">35 days</strong></td>
  </tr>
</table>

<p style="background:#f8d7da;padding:15px;">
  <strong>This is your final notice before we escalate this matter.</strong>
  Failure to remit payment within the next 7 days may result in:
  <ul>
    <li>Suspension of services</li>
    <li>Reporting to credit agencies</li>
    <li>Referral to collections</li>
  </ul>
</p>

<p><a href="[PAYMENT_LINK]" style="background:#d9534f;color:white;padding:15px 30px;font-weight:bold;">PAY NOW</a></p>

<p>If you are experiencing financial difficulties, please contact us
immediately to discuss payment arrangements.</p>
```

---

## Test Plan

### Test 1: Webhook Invoice Creation

**Steps:**
1. Set `testMode = true` in config
2. Send the curl command above to your webhook URL
3. Check your Google Sheet for new row
4. Check your email for invoice email (sent to ownerNotificationEmail in test mode)

**Expected Results:**
- New row in Google Sheet with all fields populated
- Email received with "[TEST MODE]" banner
- Response includes `success: true, invoice_number: "INV-..."`

### Test 2: Dedupe Prevention

**Steps:**
1. Send the exact same webhook payload twice
2. Check Google Sheet

**Expected Results:**
- Only ONE row in sheet (not two)
- Second request returns `message: "Duplicate invoice detected - skipped creation"`

### Test 3: Payment Status Update to Paid

**Steps:**
1. Create a test invoice (Test 1)
2. Manually update the invoice status to "Paid" in QuickBooks/Stripe
3. Wait for payment tracking workflow to run (or trigger manually)
4. Check Google Sheet

**Expected Results:**
- Sheet row updated: status = "Paid", amount_paid = total, balance_due = 0
- Payment received notification email sent to owner

### Test 4: Overdue Reminder Run

**Steps:**
1. Create a test invoice
2. Manually edit the Google Sheet row:
   - Set `due_date` to 10 days ago
   - Set `timestamp_created` to 10 days ago
   - Clear `last_reminder_ts`
3. Run the reminder workflow manually

**Expected Results:**
- Reminder email sent (to owner in test mode)
- Sheet updated: last_reminder_type = "past_due", last_reminder_ts = now, reminder_count = 1

### Test 5: Provider Switching

**Steps:**
1. Change `config.invoicingProvider = "stripe"` in config
2. Send a test webhook
3. Verify invoice created in Stripe (or change to "pdf_only" for HTML invoice)

**Expected Results:**
- Invoice created in selected provider
- Sheet shows correct provider value
- Email includes correct payment link (or manual payment instructions for pdf_only)

---

## Workflow Files

| File | Description |
|------|-------------|
| `invoice-billing-automation-complete.json` | **Combined export** - Import this for all 3 workflows |
| `invoice-billing-automation.json` | Main workflow only (invoice creation) |
| `invoice-payment-tracking.json` | Payment status polling (every 6 hours) |
| `invoice-reminder-system.json` | Payment reminders (daily at 9 AM) |

---

## Troubleshooting

### "Duplicate invoice detected" for new invoices
- Check dedupe_key format: `client_id_billable_event_id_total_due_date`
- Ensure billable_event_id is unique for each invoice

### Emails not sending
- Verify Gmail credential is connected and authorized
- Check spam folder
- In test mode, emails go to ownerNotificationEmail only

### QuickBooks/Stripe invoice creation fails
- Verify OAuth credentials are valid and not expired
- For QuickBooks: Replace YOUR_COMPANY_ID in the HTTP Request node
- For Stripe: Ensure customer exists in Stripe

### Reminders not sending on schedule
- Verify workflow is set to Active
- Check server timezone matches config.timezone
- Verify days_since_sent calculation matches your cadence

---

## Support

For issues or feature requests, check the n8n community forum or documentation.
