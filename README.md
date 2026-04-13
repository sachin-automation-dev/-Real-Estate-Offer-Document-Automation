# 🏠 Real Estate Offer Document Automation
### Google Sheets → PandaDoc → GoHighLevel (GHL) Pipeline Automation

> Fully automated offer signing workflow for real estate acquisitions.
> A checkbox click in Google Sheets triggers the entire process — generating a branded purchase agreement in PandaDoc, sending it to the seller for e-signature, and automatically updating the deal stage in GoHighLevel CRM. Zero manual effort after setup.

## 📌 The Problem This Solves

In a typical real estate acquisitions workflow, the team has to:
- Manually create a purchase agreement document for each lead
- Copy-paste property details and seller info into the document
- Email it to the seller and wait
- Remember to update the CRM pipeline stage after sending
- Check back and manually update the stage again once the seller signs

This automation eliminates every one of those steps. One checkbox click handles it all — end to end.

## ⚡ Full Workflow — Two Make.com Scenarios

This automation is split into two connected scenarios:

### 🔵 Scenario 1 — "Create Document" (Checkbox → PandaDoc → GHL: Offer Sent)

Google Sheets — Agent checks "Offer Sign" checkbox
        │
        ▼
Webhook Trigger (Make.com Custom Webhook)
        │
        ▼
Filter: Does the lead have an Email address? → Yes
        │
        ▼
PandaDoc — Create Document from Template
  • Template: AssetOps Standard Real Estate Purchase Agreement
  • Document Name: Full Property Address
  • Email Subject: "Cash Offer: [Full Property Address]"
  • Email Message: "Please review this offer - if you agree please sign the agreement"
  • Auto-filled fields from Google Sheet:
      - Offer Sent Date
      - Client Email
      - Closing Office
      - Legal Description
      - Full Property Address
      - Additional Information
  • Document sent immediately upon creation
        │
        ▼
Google Sheets — Search "SEND OFFERS" sheet
  • Find matching row by GHL Opportunity Name
        │
        ▼
GoHighLevel — Update Opportunity Stage → "Offer Sent"
  • Pipeline: OFFERS
  • Stage: Offer Sent
  • Opportunity matched by ID from Google Sheet

### 🟢 Scenario 2 — "When Doc Signed" (PandaDoc Webhook → GHL: Offer Signed)

PandaDoc Webhook — fires when document status changes
        │
        ▼
Filter: Is document status = "document.completed"? → Yes
        │
        ▼
Google Sheets — Search "SEND OFFERS" sheet
  • Filter rows where Column BM (Doc ID) matches PandaDoc document ID
        │
        ▼
GoHighLevel — Update Opportunity Stage → "Offer Signed"
  • Pipeline: OFFERS
  • Stage: Offer Signed
  • Filter: GHL Opportunity name matches Sheet data

## 🛠️ Tools & Platforms Used

 Tool             Role in Automation
Google Sheets      Master database — stores all lead and property records
Make.com           Core automation engine connecting all platforms
PandaDoc           Document generation and e-signature collection
GoHighLevel (GHL)  CRM — OFFERS pipeline stage management
Custom Webhook     Receives checkbox trigger from Google Sheets
PandaDoc Webhook   Fires when all parties complete signing



## 📊 Google Sheet Structure — Key Columns Used

The automation reads from a Google Sheet with the following important columns:

| Column | Field | Used In |

| M | Phone | Lead record |
| N | Email | PandaDoc recipient |
| O | Full Property Address | Document name + PandaDoc token |
| AT | Offer Sent Date | PandaDoc token |
| BJ | Legal Description | PandaDoc token |
| BK | Closing Office | PandaDoc token |
| BL | Additional Information | PandaDoc token |
| BM | Doc ID | Links PandaDoc document back to GHL opportunity |

> **Sheet name:** `SEND OFFERS`

## 📄 PandaDoc Template Details

| Setting | Value |
| Template Name | AssetOps Standard Real Estate Purchase Agreement |
| Send on Creation| Yes — document sent immediately after creation |

**Dynamic Tokens auto-filled by automation:**

| Token Name | Data Source |
|------------|-------------|
| `OFFERED` | Offer Sent date from Sheet |
| `Client.Email` | Email column (N) from Sheet |
| `Closing Office` | Column BK from Sheet |
| `Legal Description` | Column BJ from Sheet |
| `AssetOps LLC.Email` | Sender email (hardcoded) |
| `Full Property Address` | Column O from Sheet |
| `Additional Information` | Column BL from Sheet |

## 🔄 GoHighLevel Pipeline Configuration

| Setting | Value |
|---------|-------|
| **Pipeline Name** | OFFERS |
| **Stage updated on document send** | Offer Sent |
| **Stage updated on document signed** | Offer Signed |


## 📁 Repository Structure

real-estate-offer-automation/
│
├── README.md
├── blueprints/
│   ├── scenario1_create_document_blueprint.json     ← Checkbox → PandaDoc → GHL Offer Sent
│   └── scenario2_when_doc_signed_blueprint.json     ← PandaDoc Signed → GHL Offer Signed
└── screenshots/
    ├── 01-google-sheet-send-offers.png
    ├── 02-scenario1-make-overview.png
    ├── 03-pandadoc-template.png
    ├── 04-ghl-offer-sent-stage.png
    └── 05-ghl-offer-signed-stage.png
    
## 🚀 Setup Instructions

### Prerequisites
- Make.com account
- PandaDoc account with API access and purchase agreement template ready
- GoHighLevel account with OFFERS pipeline configured
- Google Sheet set up with the column structure described above

### Step 1 — Import Both Make.com Blueprints
1. In Make.com → **Scenarios → Create a new scenario**
2. Click the **three dots** → **Import Blueprint**
3. Import `scenario1_create_document_blueprint.json`
4. Repeat for `scenario2_when_doc_signed_blueprint.json`

### Step 2 — Reconnect All Credentials
After importing, reconnect your own accounts in each module:
- ✅ **Google Sheets** → connect your Google account
- ✅ **PandaDoc** → add your PandaDoc API connection
- ✅ **GoHighLevel** → add your GHL API key and sub-account

### Step 3 — Configure the Google Sheets Webhook (Scenario 1)
1. In Scenario 1, copy the **Custom Webhook URL** from the trigger module
2. In Google Sheets → **Extensions → Apps Script**
3. Add an `onEdit` trigger that fires the webhook when the Offer Sign checkbox is checked
4. Deploy and authorize the Apps Script

javascript
function onEdit(e) {
  var sheet = e.source.getActiveSheet();
  var range = e.range;
  
  // Adjust column number to match your "Offer Sign" checkbox column
  var checkboxCol = 65; // Example: Column BM
  
  if (range.getColumn() === checkboxCol && e.value === "TRUE") {
    var row = sheet.getRange(range.getRow(), 1, 1, sheet.getLastColumn()).getValues()[0];
    var headers = sheet.getRange(1, 1, 1, sheet.getLastColumn()).getValues()[0];
    
    var payload = {};
    headers.forEach(function(header, i) {
      payload[header] = row[i];
    });
    
    var webhookUrl = "YOUR_MAKE_WEBHOOK_URL_HERE";
    
    UrlFetchApp.fetch(webhookUrl, {
      method: "POST",
      contentType: "application/json",
      payload: JSON.stringify(payload)
    });
  }
}

### Step 4 — Configure PandaDoc Webhook (Scenario 2)
1. In Make.com Scenario 2, open the **PandaDoc Watch Documents** trigger
2. Copy the generated webhook URL
3. In PandaDoc → **Settings → Webhooks** → paste the URL
4. Subscribe to the `document_state_changed` event

### Step 5 — Update Your Template and Pipeline IDs
In the Make.com modules, update:
- PandaDoc Template ID → your own template
- GHL Pipeline ID and Stage IDs → match your GHL account
- Google Sheet name and column references → match your Sheet

### Step 6 — Test End to End
1. Add a test lead row in the Google Sheet with a valid email address
2. Check the **Offer Sign** checkbox
3. Verify in PandaDoc — document created and sent ✅
4. Verify in GHL — opportunity stage shows **Offer Sent** ✅
5. Open the document link and complete signing
6. Verify in GHL — opportunity stage updates to **Offer Signed** ✅


## 💡 Key Benefits & Results

- ⏱️ **Saves 20–30 minutes per offer** — no manual document creation or CRM updates
- ✅ **Zero data entry errors** — all fields pulled directly from the master spreadsheet  
- 📄 **Consistent, professional documents** — same branded template every time
- 🔄 **Real-time CRM accuracy** — GHL pipeline always reflects true deal status
- 📲 **Works from anywhere** — agent checks a box, automation handles the rest
- 🔔 **Full audit trail** — Doc ID stored in Column BM ties every document to its sheet row


## 👤 Built By

Sachin Gupta — Automation Specialist | AI & LLM Workflow Engineer

- 📧 sachindg411@gmail.com
- 🔗 [LinkedIn](https://www.linkedin.com/in/sachin-gupta-061a87209)
- 📍 Mumbai, India
## 📄 License

Open for learning and reference. Feel free to adapt for your own real estate automation use case.
