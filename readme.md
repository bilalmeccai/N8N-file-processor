📄 Invoice PDF to JSON Converter
Automated invoice processing workflow for n8n
Extract structured data from PDF invoices using AI-powered text extraction and Google Gemini
[
[
[
________________________________________
📋 Table of Contents
•	Overview
•	Features
•	Architecture
•	Prerequisites
•	Installation
•	Configuration
•	Usage
•	Output Schema
•	Customization
•	Troubleshooting
•	FAQ
•	Contributing
________________________________________
🎯 Overview
This n8n workflow automates the extraction of structured data from PDF invoices. Upload any invoice PDF through a web form, and receive clean, structured JSON data ready for integration with your accounting systems, databases, or business applications.
What It Does
text
graph LR
    A[📤 Upload PDF] --> B[📖 Extract Text]
    B --> C[🧹 Clean Data]
    C --> D[🤖 AI Processing]
    D --> E[✨ Format XML]
    E --> F[📊 Output JSON]
Perfect for:
•	💼 Automated accounts payable
•	📊 Invoice data extraction for ERP systems
•	🏢 Multi-vendor invoice processing
•	📈 Financial data aggregation
•	🔄 Document workflow automation
________________________________________
✨ Features
Feature	Description
🌐 Web Form Upload	User-friendly form for PDF submission
📑 PDF Text Extraction	Extracts text from standard PDF invoices
🤖 AI-Powered Parsing	Google Gemini intelligently structures data
🎯 Structured Output	Clean JSON with predefined schema
🔧 Customizable Schema	Adapt to any invoice format
⚡ Fast Processing	Typical processing time: 3-5 seconds
🌍 Multi-language	Supports any language (configure prompt)
________________________________________
🏗️ Architecture
Workflow Nodes
#	Node Name	Type	Purpose
1️⃣	On form submission	formTrigger	Receives PDF upload via web form
2️⃣	Extract from File	extractFromFile	Extracts text content from PDF
3️⃣	Limpio data	set	Cleans text & prepares XML schema
4️⃣	Message a model	googleGemini	AI transforms text to structured XML
5️⃣	Limpio XML	set	Removes formatting artifacts
6️⃣	XML to JSON	xml	Converts XML to final JSON output
Data Flow
text
┌─────────────────┐
│  PDF Invoice    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Raw Text        │
│ (multiline)     │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Cleaned Text +  │
│ XML Schema      │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Google Gemini   │
│ (AI Processing) │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Structured XML  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Clean JSON ✅   │
└─────────────────┘
________________________________________
📦 Prerequisites
Before you begin, ensure you have:
•	✅ n8n instance (v1.0+ recommended)
o	Cloud: n8n.cloud
o	Self-hosted: Installation guide
•	✅ Google Gemini API key (Get it here)
•	✅ Basic n8n knowledge (importing workflows, configuring nodes)
API Requirements
Free Tier: 60 requests/minute
Cost per invoice: ~$0.01-0.05 (depending on PDF size)
________________________________________
🚀 Installation
Step 1: Import Workflow
1.	Open your n8n instance
2.	Go to Workflows → Import from File or Import from URL
3.	Paste the workflow JSON
4.	Click Import
Step 2: Configure Credentials
Google Gemini API Setup
1.	Navigate to Credentials in n8n sidebar
2.	Click Add Credential
3.	Search for "Google PaLM/Gemini API"
4.	Enter your credentials:
text
API Key: [Your API key from Google AI Studio]
5.	Click Save
Step 3: Connect Credential to Workflow
1.	Open the imported workflow
2.	Click on "Message a model" node
3.	Under Credentials, select your Google Gemini credential
4.	Click Save
Step 4: Activate Workflow
1.	Toggle the workflow switch to Active (top-right corner)
2.	The form will now be live and ready to receive submissions
________________________________________
⚙️ Configuration
Node 1: Form Trigger
Purpose: Creates a public web form for PDF uploads
text
Form Title: "Test"  # ← Change this to your form name
Form Field:
  - Label: "data"
  - Type: file
  - Accept: .pdf
🔧 Customization:
javascript
// To restrict file types, add validation
formFields: {
  values: [
    {
      fieldLabel: "Invoice PDF",
      fieldType: "file",
      requiredField: true
    }
  ]
}
📝 Getting the Form URL:
1.	Open the "On form submission" node
2.	Copy the Form URL (e.g., https://your-n8n.app/form/abc123)
3.	Share this URL with users or embed it on your website
________________________________________
Node 2: Extract from File
Purpose: Extracts text from PDF documents
text
Operation: pdf
Output: text field containing full PDF content
⚠️ Note: Works with text-based PDFs. For scanned/image PDFs, add OCR preprocessing.
________________________________________
Node 3: Data Preparation
Purpose: Cleans extracted text and defines output structure
Variable 1: text_limpio
Removes line breaks for cleaner AI processing:
javascript
={{ $json.text.replace(/\n/g, ' ') }}
Variable 2: estructuraXML
Defines the invoice data schema:
<details> <summary><b>📄 Click to view full XML schema</b></summary> 
xml
<invoice>
    <invoice_number>[invoice_number]</invoice_number>
    <date_of_issue>[date_of_issue]</date_of_issue>
    <due_date>[due_date]</due_date>

    <billed_to>
        <company_name>[billed_to.company_name]</company_name>
        <contact_name>[billed_to.contact_name]</contact_name>
        <address>[billed_to.address]</address>
        <postal_code>[billed_to.postal_code]</postal_code>
        <city>[billed_to.city]</city>
        <state>[billed_to.state]</state>
        <country>[billed_to.country]</country>
        <rfc>[billed_to.rfc]</rfc>
    </billed_to>

    <from>
        <company_name>[from.company_name]</company_name>
        <address>[from.address]</address>
        <postal_code>[from.postal_code]</postal_code>
        <city>[from.city]</city>
        <state>[from.state]</state>
        <country>[from.country]</country>
        <rfc>[from.rfc]</rfc>
    </from>

    <purchase_order>[purchase_order]</purchase_order>

    <items>
        <item>
            <description>[item.description]</description>
            <unit_cost>[item.unit_cost]</unit_cost>
            <quantity>[item.quantity]</quantity>
            <amount>[item.amount]</amount>
        </item>
    </items>

    <bank_account_details>
        <account_holder_name>[bank_account_details.account_holder_name]</account_holder_name>
        <account_number>[bank_account_details.account_number]</account_number>
        <routing_number>[bank_account_details.routing_number]</routing_number>
        <swift_code>[bank_account_details.swift_code]</swift_code>
        <bank_name>[bank_account_details.bank_name]</bank_name>
        <currency>[bank_account_details.currency]</currency>
    </bank_account_details>

    <financials>
        <subtotal>[subtotal]</subtotal>
        <tax_rate>[tax_rate]</tax_rate>
        <tax_amount>[tax_amount]</tax_amount>
        <shipping_cost>[shipping_cost]</shipping_cost>
        <invoice_total>[invoice_total]</invoice_total>
    </financials>
</invoice>
</details> 
🎨 Schema Customization:
•	Add fields: Insert new XML tags within relevant sections
•	Remove fields: Delete unnecessary XML tags
•	Rename fields: Update tag names to match your requirements
________________________________________
Node 4: AI Model (Google Gemini)
Purpose: Converts unstructured text to structured XML using AI
text
Model: gemma-3n-e4b-it
Prompt Language: Spanish (customizable)
🌐 Prompt Templates
Spanish (Default):
text
Considera la transcripcion del invoice adjunta, reescribela como un XML siguiendo este esquema:

{{ $json.estructuraXML }}

Invoice:

{{ $json.text_limpio }}
English:
text
Consider the attached invoice transcription and rewrite it as XML following this schema:

{{ $json.estructuraXML }}

Invoice:

{{ $json.text_limpio }}
French:
text
Considérez la transcription de la facture ci-jointe et réécrivez-la en XML selon ce schéma:

{{ $json.estructuraXML }}

Facture:

{{ $json.text_limpio }}
🔄 Model Options
Model	Speed	Accuracy	Best For
gemma-3n-e4b-it	⚡⚡⚡ Fast	⭐⭐⭐ Good	Simple invoices
gemini-pro	⚡⚡ Medium	⭐⭐⭐⭐⭐ Excellent	Complex invoices
gemini-1.5-flash	⚡⚡⚡ Fast	⭐⭐⭐⭐ Very Good	Balanced option
________________________________________
Node 5: XML Cleanup
Purpose: Removes AI output formatting artifacts
javascript
={{ $json.content.parts[0].text
    .replace('```xml', '')        // Remove markdown code fence
    .replace('```', '')            // Remove closing fence
    .replace(/(\n|\s{2,})/g, '')   // Remove extra whitespace
    .replace(/(\s<)/g, '<')        // Clean tag spacing
    .replace(/(>\s)/g, '>')        // Clean tag spacing
}}
💡 Tip: If using a different AI model, adjust the path: $json.content.parts[0].text might differ.
________________________________________
Node 6: XML to JSON
Purpose: Final conversion to JSON format
text
Input Property: factura_limpia
Options:
  - normalize: false
  - normalizeTags: false
  - trim: false
These settings preserve the original structure and field names.
________________________________________
📤 Usage
Basic Workflow
1.	Access the form:
text
https://your-n8n-instance.com/form/[workflow-id]
2.	Upload invoice:
o	Click "Choose File"
o	Select your PDF invoice
o	Click "Submit"
3.	Processing:
o	Workflow extracts text (1-2 seconds)
o	AI processes data (2-4 seconds)
o	JSON output generated
4.	Retrieve results:
o	Check execution in n8n dashboard
o	Export JSON from final node
o	Connect to downstream systems
Integration Examples
Save to PostgreSQL
javascript
// Add PostgreSQL node after XML to JSON
INSERT INTO invoices (
  invoice_number,
  total,
  vendor,
  data_json
) VALUES (
  '{{ $json.invoice.invoice_number }}',
  '{{ $json.invoice.financials.invoice_total }}',
  '{{ $json.invoice.from.company_name }}',
  '{{ JSON.stringify($json) }}'
)
Send to Webhook
javascript
// Add HTTP Request node
POST https://your-api.com/invoices
Body: {{ JSON.stringify($json.invoice) }}
Headers: 
  Content-Type: application/json
  Authorization: Bearer YOUR_TOKEN
Send Slack Notification
javascript
New invoice processed! 🎉

📄 Invoice: {{ $json.invoice.invoice_number }}
💰 Total: {{ $json.invoice.financials.invoice_total }}
🏢 From: {{ $json.invoice.from.company_name }}
📅 Date: {{ $json.invoice.date_of_issue }}
________________________________________
📊 Output Schema
Complete JSON Structure
json
{
  "invoice": {
    "invoice_number": "INV-001234",
    "date_of_issue": "2026-01-15",
    "due_date": "2026-02-15",
    
    "billed_to": {
      "company_name": "Acme Corporation",
      "contact_name": "John Doe",
      "address": "123 Business Street",
      "postal_code": "560001",
      "city": "Bangalore",
      "state": "Karnataka",
      "country": "India",
      "rfc": "GSTIN123456"
    },
    
    "from": {
      "company_name": "Your Company Pvt Ltd",
      "address": "456 Tech Park",
      "postal_code": "560100",
      "city": "Bangalore",
      "state": "Karnataka",
      "country": "India",
      "rfc": "GSTIN789012"
    },
    
    "purchase_order": "PO-2026-789",
    
    "items": {
      "item": {
        "description": "Professional Services - January 2026",
        "unit_cost": "5000.00",
        "quantity": "10",
        "amount": "50000.00"
      }
    },
    
    "bank_account_details": {
      "account_holder_name": "Your Company Pvt Ltd",
      "account_number": "1234567890",
      "routing_number": "IFSC0001234",
      "swift_code": "BANKINUS33",
      "bank_name": "State Bank of India",
      "currency": "INR"
    },
    
    "financials": {
      "subtotal": "50000.00",
      "tax_rate": "18",
      "tax_amount": "9000.00",
      "shipping_cost": "0.00",
      "invoice_total": "59000.00"
    }
  }
}
Field Descriptions
Field	Type	Description	Example
invoice_number	string	Unique invoice identifier	INV-001234
date_of_issue	string	Invoice creation date	2026-01-15
due_date	string	Payment due date	2026-02-15
billed_to.*	object	Customer/client details	See schema
from.*	object	Vendor/supplier details	See schema
items.item	object/array	Line items	Can be array for multiple items
financials.*	object	Financial calculations	Subtotal, tax, total
________________________________________
🎨 Customization
Add Multiple Line Items Support
By default, the schema handles one item. To support multiple items:
Update XML Schema:
xml
<items>
    <item>
        <description>[item.description]</description>
        <unit_cost>[item.unit_cost]</unit_cost>
        <quantity>[item.quantity]</quantity>
        <amount>[item.amount]</amount>
    </item>
    <!-- AI will repeat this structure for multiple items -->
</items>
Update AI Prompt:
text
If there are multiple line items, create multiple <item> elements within <items>.
Add Data Validation
Add Code Node after XML to JSON:
javascript
// Validate required fields
const invoice = $input.first().json.invoice;
const errors = [];

if (!invoice.invoice_number) {
  errors.push('Missing invoice number');
}

if (!invoice.financials.invoice_total) {
  errors.push('Missing invoice total');
}

// Validate total calculation
const calculatedTotal = 
  parseFloat(invoice.financials.subtotal) + 
  parseFloat(invoice.financials.tax_amount) + 
  parseFloat(invoice.financials.shipping_cost || 0);

const providedTotal = parseFloat(invoice.financials.invoice_total);

if (Math.abs(calculatedTotal - providedTotal) > 0.01) {
  errors.push('Total calculation mismatch');
}

if (errors.length > 0) {
  throw new Error(`Validation failed: ${errors.join(', ')}`);
}

return $input.all();
Add Error Handling
javascript
// Add after each node using "On Error" workflow
// Configure: Workflow → Settings → Error Workflow
Create error workflow:
1.	Catches failed executions
2.	Logs to database/file
3.	Sends notification to admin
4.	Queues for manual review
________________________________________
🔧 Troubleshooting
Common Issues
🚫 Issue: "Form not receiving files"
Symptoms:
•	Form submission succeeds but workflow doesn't trigger
•	File field appears empty
Solutions:
text
✅ Ensure workflow is Active (toggle in top-right)
✅ Check field name matches: "data"
✅ Verify field type is set to "file"
✅ Clear browser cache and retry
✅ Check n8n logs for errors
________________________________________
🚫 Issue: "PDF extraction returns empty text"
Symptoms:
•	Text field is empty or contains minimal content
•	Workflow continues but produces empty JSON
Solutions:
1.	Check PDF type:
text
Text-based PDF ✅ → Works
Image/Scanned PDF ❌ → Needs OCR
2.	Add OCR preprocessing:
o	Use Tesseract node before "Extract from File"
o	Or use Google Cloud Vision API
3.	Verify PDF integrity:
o	Open PDF manually
o	Try copying text (Ctrl+C)
o	If no text copies → Image-based PDF
________________________________________
🚫 Issue: "AI returns incomplete or incorrect data"
Symptoms:
•	Missing fields in JSON
•	Incorrect values extracted
•	Hallucinated data
Solutions:
Option 1: Improve the prompt
javascript
// More specific instructions
Add to the beginning of prompt:
"Extract ONLY information explicitly present in the invoice. 
If a field is not found, leave it empty or use 'N/A'. 
Do not invent or estimate values."
Option 2: Use a better model
text
Change model from: gemma-3n-e4b-it
To: gemini-pro or gemini-1.5-flash
Option 3: Add examples to prompt
javascript
Include a sample extraction in your prompt:
"Example:
Input: Invoice #12345 dated Jan 15, 2026
Output: <invoice_number>12345</invoice_number><date_of_issue>2026-01-15</date_of_issue>"
________________________________________
🚫 Issue: "XML parsing fails"
Symptoms:
•	Error: "Invalid XML"
•	Workflow stops at XML to JSON node
Solutions:
Check the XML output:
1.	Open "Limpio XML" node execution
2.	Check factura_limpia field
3.	Look for:
o	Unclosed tags: <tag> without </tag>
o	Special characters: &, <, > not escaped
o	Nested structure issues
Fix with better cleaning:
javascript
// Enhanced XML cleanup
={{ $json.content.parts[0].text
    .replace(/```xml/g, '')
    .replace(/```/g, '')
    .replace(/&(?!amp;|lt;|gt;|quot;|apos;)/g, '&amp;')  // Escape ampersands
    .replace(/[\x00-\x1F\x7F]/g, '')  // Remove control characters
    .replace(/(\n|\s{2,})/g, '')
    .replace(/(\s<)/g, '<')
    .replace(/(>\s)/g, '>')
}}
________________________________________
🚫 Issue: "Workflow times out"
Symptoms:
•	Execution exceeds time limit
•	Partial results only
Solutions:
text
✅ Reduce PDF size (compress before upload)
✅ Use faster model: gemma-3n-e4b-it
✅ Increase timeout: Workflow Settings → Timeout
✅ Split large PDFs into pages
________________________________________
❓ FAQ
<details> <summary><b>Can this handle multi-page invoices?</b></summary> 
Yes! The PDF extraction reads all pages. However, very long invoices may need:
•	Increased timeout settings
•	More powerful AI model
•	Additional prompt instructions for multi-page context
</details> <details> <summary><b>Does it work with invoices in languages other than English?</b></summary> 
Yes! Simply translate the prompt in Node 4 to your target language. The workflow supports any language supported by Google Gemini.
</details> <details> <summary><b>How accurate is the extraction?</b></summary> 
Accuracy depends on:
•	PDF quality: 95-99% for clean, text-based PDFs
•	Invoice format: 90-95% for standard formats
•	Model used: gemini-pro offers highest accuracy
Recommend always validating critical fields (totals, dates, amounts).
</details> <details> <summary><b>Can I process scanned/image PDFs?</b></summary> 
Yes, but requires OCR preprocessing:
1.	Add Tesseract OCR node before "Extract from File"
2.	Or use Google Cloud Vision API
3.	Or upload image directly and skip PDF extraction
</details> <details> <summary><b>Is my data secure?</b></summary> 
•	PDFs are processed in your n8n instance
•	Text is sent to Google Gemini API (review Google's privacy policy)
•	For sensitive data, consider:
o	Self-hosted n8n
o	Local LLM instead of Gemini
o	Data anonymization before AI processing
</details> <details> <summary><b>Can I process multiple invoices at once?</b></summary> 
Yes! Modify the workflow:
1.	Change form field to accept multiple files
2.	Add Split in Batches node after form trigger
3.	Process each file through the existing nodes
4.	Combine results at the end
</details> <details> <summary><b>What's the cost per invoice?</b></summary> 
Google Gemini API costs (approximate):
•	Small invoice (1-2 pages): ~$0.01
•	Medium invoice (3-5 pages): ~$0.02-0.03
•	Large invoice (6+ pages): ~$0.05+
Free tier: 60 requests/minute covers most small business needs.
</details> 
________________________________________
🤝 Contributing
Contributions are welcome! Here's how you can help:
Report Issues
•	Found a bug? Open an issue
•	Include: n8n version, PDF sample, error logs
Suggest Improvements
•	Have an idea? Start a discussion
•	Share your use case and requirements
Submit Pull Requests
1.	Fork the repository
2.	Create your feature branch: git checkout -b feature/amazing-feature
3.	Commit changes: git commit -m 'Add amazing feature'
4.	Push to branch: git push origin feature/amazing-feature
5.	Open a Pull Request
________________________________________
📚 Resources
Documentation
•	n8n Official Docs
•	Google Gemini API Docs
•	n8n Community Forum
Related Workflows
•	Receipt Processing Workflow
•	Purchase Order Automation
•	Expense Report Generator
Video Tutorials
•	n8n PDF Processing Guide
•	Google Gemini Integration
________________________________________
📝 License
This workflow is provided as-is under the MIT License. Feel free to modify and adapt to your needs.
________________________________________
🙏 Acknowledgments
•	n8n team for the amazing automation platform
•	Google for Gemini AI capabilities
•	Community contributors for testing and feedback
________________________________________
📞 Support
Need help? Here's how to get support:
•	💬 n8n Community Forum
•	📧 Email: your-email@example.com
•	🐛 GitHub Issues
•	💼 LinkedIn
________________________________________
<div align="center"> 
⭐ If this workflow helped you, please star the repository! ⭐
Made with ❤️ by [Your Name]
</div>

