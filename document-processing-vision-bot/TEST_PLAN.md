# Test Plan for Document Processing Bot

This test plan covers three scenarios to validate the workflow is working correctly.

---

## Pre-Test Checklist

Before running tests, confirm:

- [ ] Workflow is imported into n8n
- [ ] All credentials are connected (Gmail, Drive, Sheets, OpenAI)
- [ ] Configuration is set in "Set Config" node
- [ ] Gmail labels exist: `DocBot/Inbox` and `DocBot/Processed`
- [ ] Google Drive folder exists with correct ID
- [ ] Google Sheet exists with correct headers and ID
- [ ] Workflow is saved (not necessarily activated yet)

---

## Test 1: Simple Receipt Image

**Purpose**: Verify basic image processing and data extraction works.

### Setup
1. Find or create a simple receipt image (JPG or PNG)
   - Can be a photo of a store receipt
   - Should have: store name, date, total amount
2. Send yourself an email with this receipt attached
3. In Gmail, apply the label `DocBot/Inbox` to the email

### Execute
1. In n8n, click **Test Workflow** (or wait for automatic trigger)
2. Watch the execution progress through each node

### Expected Results

| Check | Expected |
|-------|----------|
| All nodes complete | Green checkmarks, no red errors |
| Category extracted | "Receipt" |
| Vendor extracted | Store name from receipt |
| Date extracted | Date from receipt in YYYY-MM-DD format |
| Total extracted | Dollar amount as number |
| Confidence | 0.70 or higher (no notification sent) |
| Drive file | Appears in: Processed Documents > Receipt > [Year] |
| Filename | Formatted as: YYYY-MM-DD_Receipt_Vendor_Amount_original.jpg |
| Sheet row | New row added with all fields populated |
| Gmail labels | `DocBot/Inbox` removed, `DocBot/Processed` added |
| Notification | NO notification sent (confidence was high enough) |

### Troubleshooting
- If confidence is below threshold, you will get a notification - this is expected behavior
- If extraction is wrong, check the AI response in the "OpenAI Vision" node output

---

## Test 2: Multi-Page PDF Invoice

**Purpose**: Verify PDF handling and complex document extraction.

### Setup
1. Find or create a 2+ page PDF invoice
   - Should have: company name, invoice date, line items, total
   - Can be a sample invoice from online
2. Send yourself an email with this PDF attached
3. In Gmail, apply the label `DocBot/Inbox` to the email

### Execute
1. In n8n, click **Test Workflow**
2. Watch the execution progress through each node

### Expected Results

| Check | Expected |
|-------|----------|
| All nodes complete | Green checkmarks |
| Category extracted | "Invoice" |
| Vendor extracted | Company name issuing the invoice |
| Date extracted | Invoice date in YYYY-MM-DD format |
| Total extracted | Invoice total as number |
| Currency extracted | USD, EUR, or appropriate code |
| Line items | Array with at least some items extracted |
| Confidence | Should be reasonable (0.70+) for clear invoices |
| Drive file | Appears in: Processed Documents > Invoice > [Year] |
| Filename | Formatted as: YYYY-MM-DD_Invoice_Vendor_Amount_original.pdf |
| Sheet row | New row added with all fields, line_items_count > 0 |
| Gmail labels | `DocBot/Inbox` removed, `DocBot/Processed` added |

### Notes
- Multi-page PDFs may have slightly lower confidence scores
- The AI sees the entire document, so it can extract data from any page

---

## Test 3: Irrelevant Attachment (Should Trigger Notification)

**Purpose**: Verify the workflow correctly identifies non-document files and sends alerts.

### Setup
1. Create or find a non-financial document:
   - A photo of a landscape or pet
   - A personal letter or memo
   - A random screenshot
2. Send yourself an email with this file attached
3. In Gmail, apply the label `DocBot/Inbox` to the email

### Execute
1. In n8n, click **Test Workflow**
2. Watch the execution progress through each node

### Expected Results

| Check | Expected |
|-------|----------|
| All nodes complete | Green checkmarks (not errors) |
| Category extracted | "Other" |
| Vendor extracted | null or empty |
| Missing fields | Should list vendor, document_date, total_amount |
| Confidence | Low (below 0.50 typically) |
| Status | "Needs Review" |
| Drive file | Still uploaded to: Processed Documents > Other > [Year] |
| Sheet row | New row added with Status = "Needs Review" |
| Gmail labels | `DocBot/Inbox` removed, `DocBot/Processed` added |
| **Notification** | **YES - Alert email should be sent** |

### Notification Email Should Contain
- Subject: "DocBot Alert: Needs Review - [filename]"
- Reason: "Categorized as Other" and/or "Low confidence"
- Email details (subject, from, attachment name)
- Extracted JSON (showing mostly null/empty values)
- Drive file link
- Instructions on what to do next

### Why This Test Matters
This validates that:
1. The workflow does not crash on unexpected files
2. Files are still stored even when unrecognized
3. The notification system alerts users appropriately
4. The status correctly reflects "Needs Review"

---

## Bonus Test: Multiple Attachments

**Purpose**: Verify batch processing of multiple files in one email.

### Setup
1. Send yourself an email with 3 attachments:
   - One receipt image
   - One invoice PDF
   - One random image (non-document)
2. Apply the `DocBot/Inbox` label

### Expected Results
- 3 separate rows in Google Sheets
- 3 separate files in Google Drive (possibly in different category folders)
- 1 notification email (for the random image only)
- All processed from a single email

---

## Test Results Template

Copy and fill in for each test:

```
Test #: ___
Date: ___
Attachment: ___

Results:
- [ ] Nodes completed without error
- [ ] Category: ___
- [ ] Vendor: ___
- [ ] Date: ___
- [ ] Total: ___
- [ ] Confidence: ___
- [ ] Drive file location: ___
- [ ] Sheet row added: Yes/No
- [ ] Labels updated: Yes/No
- [ ] Notification sent: Yes/No (expected: ___)

Notes:
___
```

---

## Common Test Failures and Solutions

### "Credential not found" error
- Re-authorize the credential
- Make sure you selected the credential in the node

### "File not found" in Drive
- Check the folder ID is correct
- Verify the folder exists and you have write access

### "Sheet not found" error
- Check the spreadsheet ID is correct
- Verify the sheet tab is named exactly "Document Log"
- Make sure column headers exist

### Notification not sent when expected
- Check notificationEmail is set correctly in config
- Verify Gmail can send emails (check spam folder)
- Check the "Needs Notification?" node logic

### Low confidence on clear documents
- Some documents are harder to read (low resolution, unusual format)
- Consider lowering the threshold to 0.60
- Check if the document type is unusual

---

## After Testing

Once all tests pass:

1. **Activate the workflow** by toggling the Active switch
2. **Set up your Gmail filter** to auto-label incoming documents
3. **Monitor for a few days** to catch any edge cases
4. **Adjust confidence threshold** based on your preferences

The bot is now ready for production use.
