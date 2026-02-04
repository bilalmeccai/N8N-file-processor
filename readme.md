Invoice PDF to JSON - n8n Workflow
A simple n8n workflow that converts PDF invoices into structured JSON data using AI-powered text extraction and Google Gemini.

What This Workflow Does
Upload a PDF invoice through a web form → Extract text → AI converts it to structured data → Output clean JSON

Use cases:

Automated invoice processing

Accounts payable automation

Invoice data extraction for ERP systems

Receipt management systems

How It Works

[Web Form] → [PDF Text Extraction] → [Data Cleanup] → [AI Processing] → [XML Cleanup] → [JSON Output]


Step-by-step Process
Form Submission: User uploads invoice PDF via web form

Text Extraction: PDF content is extracted as plain text

Data Preparation: Text is cleaned and an XML template is prepared

AI Conversion: Google Gemini AI transforms raw text into structured XML

XML Cleanup: Remove formatting artifacts from AI output

JSON Conversion: Final structured JSON ready for use

Quick Start
Prerequisites
n8n instance (cloud or self-hosted)

Google Gemini API key (get one here)

Basic familiarity with n8n workflows

Installation Steps
Import the workflow: Copy the workflow JSON and import it into your n8n instance

Configure Google Gemini credentials:

Go to Credentials in n8n

Add new credential: "Google Gemini/PaLM API"

Enter your API key

Activate the workflow: Toggle the workflow to "Active"

Get the form URL: Open the "On form submission" node and copy the Form URL

Test it: Visit the form URL, upload a sample invoice PDF, and submit

Workflow Configuration
Node 1: Form Trigger
What it does: Creates a web form to receive PDF uploads

Configuration:

Form title: Test (change this to your preferred name)

Field name: data

Field type: File upload

To customize:

Update the form title to match your use case (e.g., "Invoice Upload")

Add additional form fields if needed (invoice number, vendor name, etc.)

Node 2: Extract from File
What it does: Reads the PDF and extracts all text content

Configuration:

Operation: PDF

No additional setup required

Note: This works with most standard PDFs. Scanned images may require OCR preprocessing.

Node 3: Data Preparation
What it does: Cleans extracted text and defines the output structure

Key components:

Text cleaning: Removes line breaks and extra spaces

XML schema: Defines what data to extract from the invoice

To customize the schema: Edit the estructuraXML field to match your invoice format. The current schema includes:

Invoice header (number, dates)

Billing information (customer details)

Vendor information (your company details)

Line items (products/services)

Bank details

Financial totals (subtotal, tax, total)

Node 4: AI Model (Google Gemini)
What it does: Uses AI to extract structured data from unstructured text

Configuration:

Model: gemma-3n-e4b-it (you can change this to other Gemini models)

Prompt language: Spanish (modify the prompt for other languages)

To change the language: Update the prompt in messages.values[0].content:

English version:

Consider the invoice transcription below and rewrite it as XML following this schema:

{{ $json.estructuraXML }}

Invoice:

{{ $json.text_limpio }}


Other supported Gemini models:

gemini-pro - Best for complex invoices

gemini-pro-vision - If you need image analysis

gemma-3n-e4b-it - Faster, good for simple invoices

Node 5: XML Cleanup
What it does: Removes formatting artifacts from AI-generated XML

Configuration:

Removes markdown code fences (```xml)

Strips whitespace and line breaks

Prepares valid XML for parsing

No changes needed unless you modify the AI model output format.

Node 6: XML to JSON Converter
What it does: Converts the cleaned XML into JSON format

Configuration:

Input field: factura_limpia

Normalization: Disabled (preserves original structure)

Output: A nested JSON object with all invoice data structured and ready to use.

Expected Output
The workflow produces a JSON object with this structure:

{
  "invoice": {
    "invoice_number": "INV-1234",
    "date_of_issue": "2026-01-15",
    "due_date": "2026-02-15",
    "billed_to": {
      "company_name": "Acme Corp",
      "contact_name": "John Doe",
      "address": "123 Main St",
      "postal_code": "12345",
      "city": "Springfield",
      "state": "IL",
      "country": "USA",
      "rfc": "ABC123456"
    },
    "from": {
      "company_name": "Your Company Inc",
      "address": "456 Business Ave",
      "postal_code": "67890",
      "city": "Chicago",
      "state": "IL",
      "country": "USA",
      "rfc": "XYZ789012"
    },
    "purchase_order": "PO-5678",
    "items": {
      "item": {
        "description": "Consulting Services",
        "unit_cost": "150.00",
        "quantity": "10",
        "amount": "1500.00"
      }
    },
    "bank_account_details": {
      "account_holder_name": "Your Company Inc",
      "account_number": "1234567890",
      "routing_number": "987654321",
      "swift_code": "BANKUS33",
      "bank_name": "Example Bank",
      "currency": "USD"
    },
    "financials": {
      "subtotal": "1500.00",
      "tax_rate": "10",
      "tax_amount": "150.00",
      "shipping_cost": "25.00",
      "invoice_total": "1675.00"
    }
  }
}

Common Customizations
1. Add validation
After the JSON conversion, add a Code node or IF node to validate:

Required fields are present

Totals are calculated correctly

Dates are in valid format

2. Save to database
Connect a database node (PostgreSQL, MongoDB, MySQL) after the JSON output to store invoices automatically.

3. Send notifications
Add an Email or Slack node to notify your team when invoices are processed.

4. Multi-currency support
Modify the prompt to detect and convert currencies, or add a Currency Converter node.

5. Error handling
Add error handling nodes to catch failed extractions and send them for manual review.

Troubleshooting
Problem: Form not receiving files
Solution: Check that the field type is set to "file" and the form is published (workflow is active).

Problem: PDF extraction returns empty text
Solution: The PDF may be image-based. Add an OCR preprocessing step before extraction.

Problem: AI returns incomplete data
Solution:

Try a more powerful model (e.g., gemini-pro)

Refine the prompt with examples

Ensure the PDF text quality is good

Problem: XML parsing fails
Solution: Check the "Limpio XML" node output. The AI may have returned invalid XML. Adjust the cleanup regex patterns.

Problem: Missing invoice fields
Solution: Update the XML schema in Node 3 to match your invoice format exactly.

Extending This Workflow
Connect to other systems:

QuickBooks, Xero, or other accounting software

Google Sheets for simple tracking

Airtable for collaborative invoice management

Custom APIs for proprietary ERP systems

Process other documents:

Purchase orders

Receipts

Contracts

Bank statements

Add approval workflow:

Send extracted data for human review

Require manager approval before saving

Track approval status and history

API Credentials Setup
Getting Google Gemini API Key
Visit Google AI Studio

Sign in with your Google account

Click "Create API Key"

Copy the key

In n8n: Credentials → Add → Google PaLM/Gemini → Paste key

Cost Considerations
Google Gemini API pricing (as of 2026):

Free tier: 60 requests per minute

Paid tier: Pay per token usage

For invoice processing, expect minimal costs (typically $0.01-0.05 per invoice depending on PDF size).

License
This workflow is provided as-is for use with n8n. Modify and adapt it to your needs.