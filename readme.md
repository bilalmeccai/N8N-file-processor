This workflow takes an uploaded invoice PDF from an n8n Form Trigger, extracts the text, converts it into a structured XML based on a predefined invoice schema using Google Gemini, cleans the XML, and finally converts it into JSON for downstream processing.

Overview
Input: Invoice PDF file uploaded via n8n Form Trigger.
​

Processing:

Extract text from the PDF.
​

Normalize text and prepare an invoice XML schema template.

Use Google Gemini to generate structured XML from the raw text.
​

Clean the returned XML string.

Convert XML to JSON.

Output: JSON representation of the invoice, ready to use in later nodes (e.g., accounting, storage, validation).

Workflow Structure
Nodes (in execution order):

On form submission (n8n-nodes-base.formTrigger)

Extract from File (n8n-nodes-base.extractFromFile)

Limpio data (n8n-nodes-base.set)

Message a model (@n8n/n8n-nodes-langchain.googleGemini)

Limpio XML (n8n-nodes-base.set)

XML to JSON (n8n-nodes-base.xml)

Node-by-node details
1. On form submission
Type: n8n-nodes-base.formTrigger.
​

Purpose: Starts the workflow when a user submits a form with an invoice file.

Key configuration:

formTitle: Test

formFields:

fieldLabel: data

fieldType: file (file upload field to receive the PDF).

The submitted file is available as a binary property that is passed to the next node.

2. Extract from File
Type: n8n-nodes-base.extractFromFile.
​

Operation: pdf

Purpose: Extracts text content from the uploaded PDF invoice.

Behavior:

Reads the binary file from the previous node.

Produces a JSON field (commonly text) containing the extracted PDF text.
​

3. Limpio data
Type: n8n-nodes-base.set

Purpose: Prepare clean text and define the XML schema template used for prompting the model.

Assignments:

text_limpio (string)

Value: ={{ $json.text.replace(/\n/g, ' ') }}

Description: Removes newlines from the extracted text to produce a single-line, cleaner version for the model prompt.

estructuraXML (string)

Value: Static XML template:

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

This schema defines the target XML structure for the invoice (header, parties, line items, bank details, and financial totals).

4. Message a model
Type: @n8n/n8n-nodes-langchain.googleGemini.
​

Purpose: Use a Gemini model to turn the raw invoice text into XML matching the estructuraXML schema.

Key parameters:

modelId: models/gemma-3n-e4b-it (selected from the Google Gemini models list).
​

messages.values[0].content:
=Considera la transcripcion del invoice adjunta, reescribela como un XML siguiendo este esquema:

{{ $json.estructuraXML }}

Invoice:

{{ $json.text_limpio }}

Explanation:

Spanish prompt that tells the model:

Consider the invoice transcription.

Rewrite it as XML following the provided schema.

Then includes the schema (estructuraXML) and the cleaned invoice text (text_limpio) as context.

The node returns the model output in content.parts[0].text (standard Gemini Chat response structure).
​

5. Limpio XML
Type: n8n-nodes-base.set

Purpose: Clean the XML returned by the model so it is valid and compact before parsing.

Assignment:

factura_limpia (string)
Value:
={{ $json.content.parts[0].text
    .replace('```xml', '')
    .replace('```', '')
    .replace(/(\n|\s{2,})/g, '')
    .replace(/(\s<)/g, '<')
    .replace(/(>\s)/g, '>')
}}

What this does:

Removes Markdown code fences xml and that models often add.

Removes newlines and extra whitespace.

Cleans extra spaces before and after tags so the result is a compact XML string suitable for parsing.

6. XML to JSON
Type: n8n-nodes-base.xml

Purpose: Parse the cleaned XML into JSON.

Configuration:

dataPropertyName: factura_limpia

options:

normalize: false

normalizeTags: false

trim: false

Behavior:

Takes the factura_limpia field as XML input.

Produces JSON output where each XML tag becomes a JSON property.

This JSON now contains a structured representation of the invoice (invoice header, billed_to, from, items, bank_account_details, financials).

Input and output
Expected input
A form submission with:

One file field labeled data, containing the invoice PDF.

Final output (high level)
The final JSON will follow the logical structure of the XML schema, for example:
{
  "invoice": {
    "invoice_number": "INV-1234",
    "date_of_issue": "2025-01-15",
    "due_date": "2025-02-15",
    "billed_to": {
      "company_name": "...",
      "contact_name": "...",
      "address": "...",
      "postal_code": "...",
      "city": "...",
      "state": "...",
      "country": "...",
      "rfc": "..."
    },
    "from": {
      "company_name": "...",
      "address": "...",
      "postal_code": "...",
      "city": "...",
      "state": "...",
      "country": "...",
      "rfc": "..."
    },
    "purchase_order": "...",
    "items": {
      "item": {
        "description": "...",
        "unit_cost": "...",
        "quantity": "...",
        "amount": "..."
      }
    },
    "bank_account_details": {
      "account_holder_name": "...",
      "account_number": "...",
      "routing_number": "...",
      "swift_code": "...",
      "bank_name": "...",
      "currency": "..."
    },
    "financials": {
      "subtotal": "...",
      "tax_rate": "...",
      "tax_amount": "...",
      "shipping_cost": "...",
      "invoice_total": "..."
    }
  }
}

Requirements and setup notes
n8n instance with:

Form Trigger node available and configured.
​

Extract From File node (core).
​

Google Gemini Chat Model node (@n8n/n8n-nodes-langchain.googleGemini).
​

Valid Google Gemini/PaLM API credentials configured in n8n:

Credential name: Google Gemini(PaLM) Api account 2 (as referenced in the workflow).

Ensure the form’s file field name and the binary property used by Extract from File match your instance configuration.

Customization ideas
Add validation logic after XML to JSON (e.g., check required fields, totals).

Route invoices based on invoice_total or currency to different systems.

Store the JSON in a database (Postgres, MongoDB) or send it to an external API.

Localize or refine the model prompt for different invoice formats or languages.