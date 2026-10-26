# AI Financial Document Assistant
 
An n8n workflow that receives invoices on Telegram, extracts the data with Google Gemini, validates it, detects duplicates, and records everything in Google Sheets. A second workflow produces an expense summary by category and currency.
 
Built with **n8n**, **Google Gemini**, **Google Sheets** and **Telegram**. All data is fictional.
 
## Workflows
 
| File | Purpose |
|---|---|
| `Part1_Invoice_Assistant.json` | Telegram → file download → Gemini extraction → validation → duplicate check → Google Sheets → Telegram reply |
| `Part1B_Expense_Summary.json` | Reads the expense tracker and writes a summary grouped by category and currency |
 
## How it works
 
```
Telegram (invoice PDF)
  -> Get file
  -> Gemini (extract JSON + category)
  -> Validate (missing fields, amount, date, category)
  -> Duplicate check (reads tracker)
  -> Google Sheets "Expenses" (status: OK / NEEDS REVIEW / DUPLICATE)
  -> Telegram reply
 
Manual run -> read "Expenses" -> group by category + currency -> "Summary" sheet + Telegram
```
 
**Extracted fields:** date, vendor, invoice number, description, amount, currency, tax.
**Categories:** personnel, travel, equipment, software, events, operational.
 
**Validation (status `NEEDS REVIEW`):**
- Missing date, vendor, invoice number, description, amount or currency
- Amount that is not a positive number
- Invalid or future date
- Unknown category
**Duplicate detection (status `DUPLICATE`):** same vendor and invoice number, or same vendor, amount, date and currency (only when a date exists).
 
**Summary:** only invoices with status `OK` are counted. Totals are grouped by category and currency.
 
## Setup
 
1. In n8n, import both JSON files (**Workflow menu → Import from file**).
2. Create credentials: **Telegram API** (bot token from BotFather), **Google Sheets OAuth2**, **Google Gemini API** (key from Google AI Studio).
3. Create a Google Sheet with two tabs and these headers in row 1:
   - `Expenses`: `date, vendor, invoice_number, description, amount, currency, tax, category, status, notes, processed_at`
   - `Summary`: `generated_at, category, currency, invoice_count, total_amount, total_tax`
4. In every Google Sheets node choose your document and tab. In every Telegram node replace `YOUR_CHAT_ID` with your chat ID. If a model is unavailable, change the model name in the Gemini node.
5. Activate the workflow and send an invoice to the bot **as a file**.
## Error handling
 
- Gemini calls retry automatically on temporary failures (up to 4 tries).
- Unparseable AI output is flagged instead of failing silently.
## Limitations
 
- Only PDFs sent as Telegram documents were tested. Photo messages are not handled yet.
- Sending several files at the same moment can hit AI rate limits.
- The duplicate check reads the full tracker on each run, which is fine for small volumes. For larger volumes use a database.
*All invoices and data are fictional. No credentials are included.*
