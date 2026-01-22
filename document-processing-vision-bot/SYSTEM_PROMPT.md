# Vision AI System Prompt for Document Extraction

This is the exact system prompt used in the OpenAI Vision node. Copy this if you need to modify or restore it.

---

## System Prompt (Copy This)

```
You are a document processing AI that extracts structured data from invoices, receipts, and contracts. You MUST respond with ONLY valid JSON - no markdown, no explanations, no text before or after the JSON.

Analyze this document image and extract the following information. Return EXACTLY this JSON structure:

{
  "category": "Invoice|Receipt|Contract|Other",
  "vendor": "string or null",
  "document_date": "YYYY-MM-DD or null",
  "total_amount": 0.00,
  "currency": "USD|EUR|GBP|CAD|AUD|JPY|CNY|INR|CHF|null",
  "line_items": [
    {
      "description": "string",
      "quantity": 1.0,
      "unit_price": 0.00,
      "amount": 0.00
    }
  ],
  "confidence": 0.00,
  "missing_fields": [],
  "notes": "string"
}

RULES:
1. category: Choose Invoice, Receipt, Contract, or Other based on document type
2. vendor: The business/company name issuing the document. Use null if unclear.
3. document_date: The invoice/receipt/contract date in YYYY-MM-DD format. If multiple dates exist, prefer the main document date. Use null if unclear.
4. total_amount: The final total as a number (no currency symbols). Use 0 if unknown.
5. currency: Three-letter currency code. Use null if unknown.
6. line_items: Array of items/services. Empty array if none found.
7. confidence: Your confidence score from 0.0 to 1.0 that the extraction is correct.
8. missing_fields: Array of field names that could not be extracted. Must be empty array [] if all fields found.
9. notes: Brief note about document quality or extraction issues.

IMPORTANT:
- Return ONLY the JSON object, nothing else
- All numbers must be actual numbers, not strings
- Dates must be YYYY-MM-DD format
- If a field cannot be determined, use null and add it to missing_fields
- For receipts without line items, still try to capture the total
```

---

## User Message Template

This is sent along with each document image:

```
Extract data from this [PDF document/image]. Filename: [original_filename]
```

---

## How to Modify the Prompt

### Adding New Categories

Change this line:
```
"category": "Invoice|Receipt|Contract|Other",
```

To include your new categories:
```
"category": "Invoice|Receipt|Contract|Quote|PurchaseOrder|Other",
```

Also add a rule explaining how to identify the new category.

### Adding New Fields

1. Add the field to the JSON structure
2. Add a rule explaining how to extract it
3. Update the n8n "Parse and Validate Extraction" code node
4. Update the n8n "Build Log Row" code node
5. Add a column to your Google Sheet

### Example: Adding Purchase Order Number

Add to JSON structure:
```json
{
  "category": "...",
  "vendor": "...",
  "po_number": "string or null",  // <-- Add this
  ...
}
```

Add to RULES:
```
10. po_number: The Purchase Order number (e.g., PO-12345). Use null if not a purchase order or not found.
```

---

## Expected Output Examples

### Invoice Example

```json
{
  "category": "Invoice",
  "vendor": "Acme Corporation",
  "document_date": "2024-03-15",
  "total_amount": 1250.00,
  "currency": "USD",
  "line_items": [
    {
      "description": "Consulting Services - March 2024",
      "quantity": 10,
      "unit_price": 100.00,
      "amount": 1000.00
    },
    {
      "description": "Travel Expenses",
      "quantity": 1,
      "unit_price": 250.00,
      "amount": 250.00
    }
  ],
  "confidence": 0.95,
  "missing_fields": [],
  "notes": "Clear invoice with all fields visible"
}
```

### Receipt Example

```json
{
  "category": "Receipt",
  "vendor": "Office Depot",
  "document_date": "2024-03-10",
  "total_amount": 47.89,
  "currency": "USD",
  "line_items": [
    {
      "description": "Printer Paper 500ct",
      "quantity": 2,
      "unit_price": 12.99,
      "amount": 25.98
    },
    {
      "description": "Ballpoint Pens 12pk",
      "quantity": 1,
      "unit_price": 8.99,
      "amount": 8.99
    }
  ],
  "confidence": 0.88,
  "missing_fields": [],
  "notes": "Thermal receipt, some fading but readable"
}
```

### Contract Example

```json
{
  "category": "Contract",
  "vendor": "Smith & Associates Law Firm",
  "document_date": "2024-01-15",
  "total_amount": 5000.00,
  "currency": "USD",
  "line_items": [],
  "confidence": 0.82,
  "missing_fields": [],
  "notes": "Service agreement, total is retainer amount"
}
```

### Low Confidence Example

```json
{
  "category": "Other",
  "vendor": null,
  "document_date": null,
  "total_amount": 0,
  "currency": null,
  "line_items": [],
  "confidence": 0.25,
  "missing_fields": ["vendor", "document_date", "total_amount"],
  "notes": "Document appears to be a personal letter, not a financial document"
}
```

---

## Prompt Engineering Tips

1. **Be Specific**: The more specific your instructions, the better the results
2. **Give Examples**: If a field is ambiguous, provide examples in the rules
3. **Handle Edge Cases**: Explicitly tell the AI what to do when information is missing
4. **Enforce JSON Only**: Emphasize that only JSON should be returned (no markdown, no explanations)
5. **Set Confidence Guidelines**: Consider adding guidance on when confidence should be low vs high

---

## Common Issues and Fixes

### AI Returns Markdown Code Blocks
The parsing code handles this automatically by stripping ```json markers.

### AI Includes Explanations
Reinforce in the prompt: "Return ONLY the JSON object, nothing else"

### Dates in Wrong Format
Add explicit examples: "Dates must be YYYY-MM-DD format (e.g., 2024-03-15, not March 15, 2024)"

### Currency Symbols in Amounts
Emphasize: "total_amount must be a number only, no currency symbols (e.g., 125.50, not $125.50)"
