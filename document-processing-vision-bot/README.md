# Document Processing Bot (Vision AI Edition)

An n8n workflow that automatically processes email attachments (invoices, receipts, contracts) using AI vision, organizes files in Google Drive, and logs everything to Google Sheets.

---

## What This Bot Does

1. **Watches your Gmail** for emails with a specific label and attachments
2. **Uses AI Vision (GPT-4o)** to "look at" each document like a human would
3. **Extracts structured data**: vendor, date, amount, line items, category
4. **Saves files to Google Drive** in organized folders: `/Processed Documents/Category/Year/`
5. **Logs everything to Google Sheets** for easy searching and reporting
6. **Sends alerts** only when something needs your attention

---

## Setup Time: 20-30 Minutes

### Prerequisites

- An n8n instance (cloud or self-hosted)
- A Gmail account
- A Google Drive account (usually same as Gmail)
- A Google Sheets account (usually same as Gmail)
- An OpenAI API account with credits ($5-10 is plenty to start)

---

## Step-by-Step Setup

### Step 1: Import the Workflow

1. Open n8n
2. Click the **+** button or go to **Workflows > Add Workflow**
3. Click the **three dots menu** (top right) > **Import from File**
4. Select the `workflow.json` file
5. Click **Save**

### Step 2: Create Gmail Labels

In Gmail (not n8n):

1. Go to gmail.com
2. On the left sidebar, scroll down and click **+ Create new label**
3. Create a label called: `DocBot/Inbox`
4. Create another label called: `DocBot/Processed`

**Optional but recommended**: Set up a Gmail filter:
1. Click the gear icon > **See all settings** > **Filters and Blocked Addresses**
2. Click **Create a new filter**
3. Set criteria (e.g., from specific senders, has attachment)
4. Click **Create filter** > Check **Apply the label** > Select `DocBot/Inbox`

### Step 3: Create Google Drive Folder

1. Go to drive.google.com
2. Click **+ New** > **Folder**
3. Name it: `Processed Documents`
4. Open the folder
5. **Copy the folder ID from the URL**:
   - URL looks like: `https://drive.google.com/drive/folders/1ABC123xyz...`
   - The ID is: `1ABC123xyz...` (everything after `/folders/`)
6. Save this ID - you will need it soon

### Step 4: Create Google Sheet

1. Go to sheets.google.com
2. Click **+ Blank** to create a new spreadsheet
3. Name it: `Document Processing Log` (or any name you like)
4. Rename the first tab (bottom of screen) to: `Document Log`
5. In row 1, add these column headers (copy exactly):

```
Received Timestamp | Email Subject | From Address | Original Filename | Category | Vendor | Document Date | Total Amount | Currency | Line Items Count | Confidence | Missing Fields | Drive File URL | Status | Notes | Dedupe Key
```

6. **Copy the spreadsheet ID from the URL**:
   - URL looks like: `https://docs.google.com/spreadsheets/d/1XYZ789abc.../edit`
   - The ID is: `1XYZ789abc...` (between `/d/` and `/edit`)
7. Save this ID - you will need it soon

### Step 5: Configure the Workflow

1. In n8n, open the workflow you imported
2. **Double-click the node called "Set Config - EDIT THIS FIRST"**
3. Update these values:
   - `rootFolderId`: Paste your Google Drive folder ID from Step 3
   - `spreadsheetId`: Paste your Google Sheets ID from Step 4
   - `notificationEmail`: Your email address for alerts
4. Click **Save**

### Step 6: Connect Credentials

You need to connect 3 services. Click each node and add credentials:

**Gmail (click "Gmail Trigger - Watch DocBot/Inbox"):**
1. Click **Credential to connect with**
2. Click **Create new credential**
3. Click **Sign in with Google**
4. Grant access to Gmail
5. Save

**Google Drive (click "Search Category Folder"):**
1. Click **Credential to connect with**
2. Select your existing Google credential OR create new
3. Grant access to Google Drive
4. Save

**Google Sheets (click "Log to Google Sheets"):**
1. Click **Credential to connect with**
2. Select your existing Google credential OR create new
3. Grant access to Google Sheets
4. Save

**OpenAI (click "OpenAI Vision - Extract Document Data"):**
1. Click **Credential to connect with**
2. Click **Create new credential**
3. Go to platform.openai.com > API Keys
4. Create a new API key
5. Paste it in n8n
6. Save

### Step 7: Test the Workflow

1. Send yourself a test email with a receipt or invoice attached
2. In Gmail, apply the label `DocBot/Inbox` to that email
3. In n8n, click **Test Workflow** (or wait for the trigger to fire)
4. Watch the execution - each node should turn green
5. Check your Google Drive - the file should appear in the right folder
6. Check your Google Sheet - a new row should be logged

### Step 8: Activate the Workflow

1. Toggle the **Active** switch in the top right
2. The workflow now runs automatically every minute

---

## Configuration Reference

All settings are in the **"Set Config - EDIT THIS FIRST"** node:

| Setting | Default | Description |
|---------|---------|-------------|
| `rootFolderId` | (required) | Google Drive folder ID for "Processed Documents" |
| `spreadsheetId` | (required) | Google Sheets spreadsheet ID |
| `sheetName` | Document Log | Name of the sheet tab |
| `notificationEmail` | (required) | Email for alerts |
| `confidenceThreshold` | 0.70 | AI confidence required (0.70 = 70%) |
| `inboxLabel` | DocBot/Inbox | Gmail label to watch |
| `processedLabel` | DocBot/Processed | Gmail label to apply after processing |
| `categories` | Invoice,Receipt,Contract,Other | Document categories |

---

## Customization

### Change Document Categories

1. Open **"Set Config - EDIT THIS FIRST"**
2. Edit the `categories` field
3. Also update the AI prompt in **"OpenAI Vision - Extract Document Data"** to include your new categories

### Change AI Model

1. Open **"OpenAI Vision - Extract Document Data"**
2. Change the Model dropdown:
   - `gpt-4o` - Best accuracy, ~$0.01-0.03 per document
   - `gpt-4o-mini` - Good accuracy, ~$0.001-0.005 per document

### Add More Extracted Fields

1. Open **"OpenAI Vision - Extract Document Data"**
2. Edit the system prompt to request additional fields
3. Update **"Parse and Validate Extraction"** to handle new fields
4. Update **"Build Log Row"** to include new fields
5. Add columns to your Google Sheet

---

## Troubleshooting

### "No emails found" or trigger not firing
- Make sure emails have the `DocBot/Inbox` label applied
- Make sure emails have attachments
- Check the Gmail trigger polling interval

### "Invalid folder ID" error
- Double-check you copied the correct ID from the URL
- Make sure there are no extra spaces

### "Unauthorized" or credential errors
- Re-authorize the credential
- Make sure you granted all required permissions

### AI returning wrong data
- Try a clearer document (better scan quality)
- Check the raw AI response in the execution log
- Adjust the confidence threshold lower if too many false alerts

### Files not appearing in Drive
- Check the execution log for errors
- Verify the folder ID is correct
- Make sure Drive has available storage

---

## Costs

- **n8n**: Free tier or paid plan depending on usage
- **OpenAI GPT-4o**: ~$0.01-0.03 per document
- **OpenAI GPT-4o-mini**: ~$0.001-0.005 per document
- **Google**: Free (within normal quotas)

Processing 100 documents/month with GPT-4o costs approximately $1-3.

---

## File Naming Convention

Files are saved as:
```
YYYY-MM-DD_Category_Vendor_Amount_OriginalFilename.ext
```

Example:
```
2024-03-15_Invoice_Acme_Corp_1250.00_scan001.pdf
```

---

## Folder Structure

```
Processed Documents/
  Invoice/
    2024/
      2024-03-15_Invoice_Acme_Corp_1250.00_scan001.pdf
    2023/
      ...
  Receipt/
    2024/
      ...
  Contract/
    2024/
      ...
  Other/
    2024/
      ...
```

---

## Support

- n8n Documentation: https://docs.n8n.io
- OpenAI Documentation: https://platform.openai.com/docs
- Community: https://community.n8n.io

---

## Version History

- v1.0.0 - Initial release with Gmail, Drive, Sheets, and OpenAI Vision integration
