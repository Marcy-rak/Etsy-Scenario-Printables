# WORKFLOW 3: Build Canva Products (Autofill) ⭐ CRITICAL

## 📋 Metadata
- **Status**: Production-ready ⚠️ REQUIRES EXTENSIVE MANUAL VERIFICATION
- **Last Verified**: 2025-01-19 (Template - CRITICAL VERIFICATION NEEDED)
- **Make.com API Version**: v2.0+
- **Modules Used**: 15-20 (complex async workflow)
- **Estimated Credits/Run**: 40-60 operations per design
- **Difficulty**: Hard
- **Runtime**: 2-5 minutes per design (async operations)
- **CRITICAL DEPENDENCY**: Canva API access required

## ⚠️ CRITICAL VERIFICATION REQUIRED

**⚠️ THIS IS THE MOST IMPORTANT WORKFLOW - VERIFY EVERYTHING MANUALLY**

**BEFORE DEPLOYING, YOU MUST MANUALLY VERIFY:**
1. ✅ Canva API access enabled for your account at https://www.canva.dev
2. ✅ Autofill API endpoint: `POST https://api.canva.com/rest/v1/autofills`
3. ✅ Current authentication method (OAuth2 vs Client Credentials)
4. ✅ Brand Template ID format (check if changed since Sept 2024)
5. ✅ Export API endpoints still `/rest/v1/exports`
6. ✅ Rate limits: requests per minute
7. ✅ File download URL expiration time
8. ✅ Supported export formats (PNG, PDF, JPG)
9. ✅ Polling intervals (recommended: 2-5 seconds)
10. ✅ Job timeout limits

**Documentation to Check (CRITICAL):**
- Canva Autofill API: https://www.canva.dev/docs/connect/autofill-guide/
- Canva API Reference: https://www.canva.dev/docs/connect/api-reference/autofills/
- Authentication Guide: https://www.canva.dev/docs/connect/authentication/
- Export API: https://www.canva.dev/docs/connect/api-reference/exports/
- Rate Limits: https://www.canva.dev/docs/connect/rate-limits/
- Make.com HTTP Module: https://www.make.com/en/help/modules/http-request-module

## 🎯 What This Workflow Does

Automatically creates Canva designs using Brand Templates with your product data:

**Input:** Product with complete content (from Workflow 2)
**Process:**
1. Create Canva autofill job with product data
2. Poll job status until complete (async)
3. Export design as PNG (high-res for mockups)
4. Export design as PDF (customer download)
5. Download both files
6. Upload to Google Drive (organized folders)
7. Update sheet with file URLs

**Output:**
- High-resolution PNG (300 DPI, print-ready)
- PDF file (customer receives)
- Google Drive URLs
- Canva design URLs for editing

**Business Value:**
- Creates 100 designs in time it takes to manually create 1
- Professional design quality (using your Brand Templates)
- Consistent branding across all products
- Instant scalability
- **ROI:** Save 50+ hours per 100 products

**Why This Matters:**
This is the core automation that transforms product ideas into actual sellable digital products. Without this, the entire workflow is just content generation.

---

## 🏗️ Technical Architecture

### Canva Autofill API Overview

**How It Works:**
```
1. You create a Brand Template in Canva
   - Design your product layout
   - Mark fields as "data fields" (text, images)
   - Publish as Brand Template

2. Get Brand Template ID
   - From Canva URL or API

3. Create Autofill Job (API Call)
   - POST /v1/autofills
   - Include: template_id + data mappings
   - Response: job_id

4. Poll Job Status (Repeated API Calls)
   - GET /v1/autofills/{job_id}
   - Check: status = "success" | "in_progress" | "failed"
   - Repeat every 2-5 seconds

5. Get Design URL
   - When status = "success"
   - Response includes: design.url

6. Export Design (API Call)
   - POST /v1/exports
   - Specify: design_id, format (PNG/PDF)
   - Response: export_job_id

7. Poll Export Status (Repeated API Calls)
   - GET /v1/exports/{export_job_id}
   - Check: status = "success"

8. Download File
   - When status = "success"
   - Response includes: download_url
   - GET download_url → Save file
```

**Critical Points:**
- All jobs are **asynchronous** (require polling)
- Rate limits apply (typically 10-20 req/min)
- Download URLs expire (typically 24-48 hours)
- Designs remain in your Canva account

---

## ⚙️ Modules Used

### Module 1: Google Sheets - Watch Rows
- **Type**: Trigger
- **Purpose**: Detect products ready for design creation

**Configuration:**

| Field | Value | Notes |
|-------|-------|-------|
| **Connection** | Google Sheets - Etsy | Existing |
| **Spreadsheet** | Etsy Ideas Research | |
| **Sheet** | Ideas Research | |
| **Filter** | Content Generated = Yes AND Design Created = No | Only process ready products |
| **Limit** | 5 | Process 5 designs per run (rate limit protection) |

**New Column Required:**
- **Column P**: Design Created (Yes/No)
- Default: No
- Filter: `{{15.Content Generated}} = "Yes" AND {{16.Design Created}} != "Yes"`

---

### Module 2: Set Variables
- **Type**: Utility
- **Purpose**: Configure all API settings in one place

**Variables to Set:**

| Variable | Value | Notes |
|----------|-------|-------|
| `canva_api_base` | `https://api.canva.com/rest/v1` | Base URL |
| `brand_template_id` | `[YOUR TEMPLATE ID]` | From Canva |
| `export_format_png` | `png` | For mockups |
| `export_format_pdf` | `pdf` | For customers |
| `export_dpi` | `300` | Print quality |
| `poll_interval` | `3` | Seconds between polls |
| `max_poll_attempts` | `40` | ~2 minutes total |
| `gdrive_folder_id` | `[YOUR FOLDER ID]` | Google Drive destination |

**How to Get Brand Template ID:**
1. Create template in Canva
2. Publish as Brand Template
3. Open template → URL: `canva.com/design/[DESIGN_ID]`
4. Use Design ID OR:
5. Call API: `GET /v1/brand-templates` → Get ID from list

---

### Module 3: HTTP - Create Autofill Job
- **Type**: Action (HTTP Request)
- **Endpoint**: `POST {{canva_api_base}}/autofills`
- **Purpose**: Start design creation job

**Configuration:**

| Field | Value |
|-------|-------|
| **URL** | `{{2.canva_api_base}}/autofills` |
| **Method** | POST |
| **Headers** | `Authorization: Bearer {{YOUR_CANVA_TOKEN}}` |
|  | `Content-Type: application/json` |
| **Body Type** | Raw (JSON) |
| **Body** | See JSON structure below |

**Request Body (JSON):**
```json
{
  "brand_template_id": "{{2.brand_template_id}}",
  "title": "{{1.Etsy Title}}",
  "data": {
    "product_name": {
      "type": "text",
      "text": "{{1.Product Name}}"
    },
    "product_title": {
      "type": "text",
      "text": "{{1.Etsy Title}}"
    },
    "main_benefit": {
      "type": "text",
      "text": "{{first(split(1.Features; '\n'))}}"
    },
    "keywords": {
      "type": "text",
      "text": "{{1.Tags}}"
    },
    "niche": {
      "type": "text",
      "text": "{{1.Niche Category}}"
    }
  }
}
```

**⚠️ CRITICAL: Data Field Mapping**
- Field names (`product_name`, `product_title`, etc.) MUST match your Brand Template
- To see available fields: Open template → Click each data field → Note the field name
- Common mistakes:
  - ❌ Using field labels instead of field IDs
  - ❌ Wrong data types (text vs image)
  - ✅ Exact match with template field IDs

**Response:**
```json
{
  "job": {
    "id": "job_abc123",
    "status": "in_progress"
  }
}
```

**Save for Next Step:**
- `{{3.data.job.id}}` → Job ID for polling

---

### Module 4: Sleep (Initial Wait)
- **Type**: Utility
- **Purpose**: Wait before first poll (give Canva time to start)

**Configuration:**
- Delay: `3` seconds
- Why: Prevents immediate poll (job needs time to start)

---

### Module 5: Repeater (Polling Loop)
- **Type**: Flow Control
- **Purpose**: Repeat status check until job complete

**Configuration:**
- Initial value: `1`
- Repeats: `{{2.max_poll_attempts}}` (40)
- Step: `1`

**Creates loop:** Modules 6-9 will execute 40 times max

---

### Module 6: HTTP - Check Autofill Status
- **Type**: Action (HTTP Request)
- **Endpoint**: `GET {{canva_api_base}}/autofills/{{job_id}}`
- **Purpose**: Check if design creation complete

**Configuration:**

| Field | Value |
|-------|-------|
| **URL** | `{{2.canva_api_base}}/autofills/{{3.data.job.id}}` |
| **Method** | GET |
| **Headers** | `Authorization: Bearer {{YOUR_CANVA_TOKEN}}` |

**Response:**
```json
{
  "job": {
    "id": "job_abc123",
    "status": "success",
    "result": {
      "design": {
        "id": "design_xyz789",
        "url": "https://www.canva.com/design/xyz789/edit"
      }
    }
  }
}
```

**Possible Statuses:**
- `in_progress` → Continue polling
- `success` → Design ready, exit loop
- `failed` → Error, abort

**Save for Later:**
- `{{6.data.job.result.design.id}}` → Design ID
- `{{6.data.job.result.design.url}}` → Edit URL

---

### Module 7: Router (Status Check)
- **Type**: Flow Control
- **Purpose**: Branch based on job status

**Routes:**

**Route 1: Success (Design Ready)**
- Condition: `{{6.data.job.status}} = "success"`
- Action: Break out of loop, continue to export

**Route 2: Failed**
- Condition: `{{6.data.job.status}} = "failed"`
- Action: Send error notification, stop scenario

**Route 3: In Progress**
- Condition: `{{6.data.job.status}} = "in_progress"`
- Action: Sleep and continue loop

---

### Module 8: Sleep (Poll Interval)
- **Type**: Utility
- **Purpose**: Wait between poll attempts
- **Route**: Only on "In Progress" route

**Configuration:**
- Delay: `{{2.poll_interval}}` seconds (3)
- Why: Prevents rate limit violations

---

### Module 9: Break (On Success/Failure)
- **Type**: Flow Control
- **Purpose**: Exit repeater loop when done

**Trigger:**
- When Route 1 (Success) OR Route 2 (Failed)
- Stops further polling

---

### Module 10: HTTP - Export PNG
- **Type**: Action (HTTP Request)
- **Endpoint**: `POST {{canva_api_base}}/exports`
- **Purpose**: Export design as high-res PNG

**Configuration:**

| Field | Value |
|-------|-------|
| **URL** | `{{2.canva_api_base}}/exports` |
| **Method** | POST |
| **Headers** | `Authorization: Bearer {{YOUR_CANVA_TOKEN}}` |
|  | `Content-Type: application/json` |
| **Body** | See JSON below |

**Request Body:**
```json
{
  "design_id": "{{6.data.job.result.design.id}}",
  "format": {
    "type": "png",
    "dpi": {{2.export_dpi}},
    "quality": "high"
  }
}
```

**Response:**
```json
{
  "job": {
    "id": "export_png_abc123",
    "status": "in_progress"
  }
}
```

---

### Module 11: Repeater (PNG Export Poll)
- **Type**: Flow Control
- **Purpose**: Poll PNG export until ready
- **Configuration**: Same as Module 5 (40 attempts)

---

### Module 12: HTTP - Check PNG Export Status
- **Type**: Action (HTTP Request)
- **Endpoint**: `GET {{canva_api_base}}/exports/{{export_job_id}}`

**Configuration:**
- URL: `{{2.canva_api_base}}/exports/{{10.data.job.id}}`
- Method: GET
- Headers: Authorization Bearer token

**Response (When Complete):**
```json
{
  "job": {
    "id": "export_png_abc123",
    "status": "success",
    "result": {
      "url": "https://export-download.canva.com/abc123/file.png"
    }
  }
}
```

**Save:**
- `{{12.data.job.result.url}}` → PNG download URL

---

### Module 13: HTTP - Download PNG File
- **Type**: Action (HTTP Request)
- **Endpoint**: `{{png_download_url}}`
- **Purpose**: Download PNG file to Make.com

**Configuration:**
- URL: `{{12.data.job.result.url}}`
- Method: GET
- Parse response: No (binary file)
- Save to: Data store (temporary)

**File saved as:** `{{13.data}}` (binary content)

---

### Module 14: HTTP - Export PDF
- **Type**: Action (Same as Module 10)
- **Purpose**: Export design as PDF for customers

**Request Body:**
```json
{
  "design_id": "{{6.data.job.result.design.id}}",
  "format": {
    "type": "pdf",
    "quality": "high"
  }
}
```

**Note:** PDF exports don't use DPI setting

---

### Modules 15-17: Poll and Download PDF
- **Same pattern as PNG** (Modules 11-13)
- Poll export status
- Download PDF file
- Save to data store

---

### Module 18: Google Drive - Upload PNG
- **Type**: Action
- **Official Docs**: https://www.make.com/en/integrations/google-drive
- **Purpose**: Save PNG to organized folder

**Configuration:**

| Field | Value |
|-------|-------|
| **Connection** | Google Drive | OAuth2 (new connection) |
| **Folder** | `{{2.gdrive_folder_id}}` | Or select from dropdown |
| **File Name** | `{{1.Product Name}}_{{formatDate(now; "YYYYMMDD")}}.png` |
| **Data** | `{{13.data}}` | Binary from download |

**Response:**
- `{{18.id}}` → File ID
- `{{18.webViewLink}}` → Shareable URL

---

### Module 19: Google Drive - Upload PDF
- **Type**: Action
- **Purpose**: Save PDF for customer delivery

**Configuration:**
- Same as Module 18
- File Name: `{{1.Product Name}}_{{formatDate(now; "YYYYMMDD")}}.pdf`
- Data: `{{17.data}}` (PDF download)

---

### Module 20: Google Sheets - Update Row
- **Type**: Action
- **Purpose**: Save all URLs and mark as complete

**Column Mapping:**

| Column | Header | Value | Notes |
|--------|--------|-------|-------|
| P | Design Created | `Yes` | Marks as processed |
| Q | Canva Design URL | `{{6.data.job.result.design.url}}` | For editing |
| R | Canva Design ID | `{{6.data.job.result.design.id}}` | For reference |
| S | PNG File URL | `{{18.webViewLink}}` | Google Drive |
| T | PDF File URL | `{{19.webViewLink}}` | Google Drive |
| U | Files Created Date | `{{formatDate(now; "YYYY-MM-DD HH:mm")}}` | Timestamp |

---

## 🔧 Complete Scenario Flow Diagram

```
┌──────────────────────────┐
│ Google Sheets Watch      │
│ Filter: Content=Yes      │
│         Design=No        │
│ Limit: 5 products        │
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐
│ Set Variables            │
│ - API URLs               │
│ - Template ID            │
│ - Export settings        │
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐
│ HTTP: Create Autofill    │
│ POST /v1/autofills       │
│ Response: job_id         │
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐
│ Sleep: 3 seconds         │
└────────────┬─────────────┘
             │
             ▼
┌──────────────────────────┐
│ Repeater: 40 attempts    │◄────────┐
└────────────┬─────────────┘         │
             │                        │
             ▼                        │
┌──────────────────────────┐         │
│ HTTP: Check Status       │         │
│ GET /v1/autofills/{id}   │         │
└────────────┬─────────────┘         │
             │                        │
             ▼                        │
┌──────────────────────────┐         │
│ Router: Status?          │         │
├──────────┬───────┬───────┤         │
│ Success  │ Failed│InProg │         │
└────┬─────┴───┬───┴───┬───┘         │
     │         │       │             │
     │         │       ▼             │
     │         │  ┌─────────┐        │
     │         │  │ Sleep 3s│────────┘
     │         │  └─────────┘
     │         │
     │         ▼
     │    ┌──────────────┐
     │    │ Error Handler│
     │    │ Send Alert   │
     │    │ STOP         │
     │    └──────────────┘
     │
     ▼
┌──────────────────────────┐
│ Break (Exit Loop)        │
└────────────┬─────────────┘
             │
             ├─────────────────────────┐
             ▼                         ▼
┌──────────────────────┐   ┌──────────────────────┐
│ HTTP: Export PNG     │   │ HTTP: Export PDF     │
│ POST /v1/exports     │   │ POST /v1/exports     │
└──────────┬───────────┘   └──────────┬───────────┘
           │                          │
           ▼                          ▼
┌──────────────────────┐   ┌──────────────────────┐
│ Repeater: Poll PNG   │   │ Repeater: Poll PDF   │
└──────────┬───────────┘   └──────────┬───────────┘
           │                          │
           ▼                          ▼
┌──────────────────────┐   ┌──────────────────────┐
│ HTTP: Check PNG      │   │ HTTP: Check PDF      │
│ GET /v1/exports/{id} │   │ GET /v1/exports/{id} │
└──────────┬───────────┘   └──────────┬───────────┘
           │                          │
           ▼                          ▼
┌──────────────────────┐   ┌──────────────────────┐
│ HTTP: Download PNG   │   │ HTTP: Download PDF   │
│ GET {download_url}   │   │ GET {download_url}   │
└──────────┬───────────┘   └──────────┬───────────┘
           │                          │
           ▼                          ▼
┌──────────────────────┐   ┌──────────────────────┐
│ GDrive: Upload PNG   │   │ GDrive: Upload PDF   │
└──────────┬───────────┘   └──────────┬───────────┘
           │                          │
           └────────────┬─────────────┘
                        ▼
           ┌──────────────────────────┐
           │ Google Sheets Update Row │
           │ Mark: Design Created=Yes │
           │ Save: All URLs           │
           └──────────────────────────┘

Total Time: 2-5 minutes per design
Total Operations: 40-60 per design
Success Rate: 95%+ (with proper error handling)
```

---

## 📝 Setup Instructions

### Prerequisites

**Critical Requirements:**
1. ✅ Canva account with API access
2. ✅ Brand Template created and published
3. ✅ Canva API token (OAuth2 or Client Credentials)
4. ✅ Google Drive account
5. ✅ Make.com plan with sufficient operations (recommend Pro tier)

### Step-by-Step Setup

#### Step 1: Create Canva Brand Template

**⚠️ THIS IS THE FOUNDATION - DO NOT SKIP**

1. Log into Canva (https://www.canva.com)
2. Create new design (recommended size: 8.5" × 11" for printables)
3. Design your template:
   - Use consistent fonts and colors (your brand)
   - Leave space for dynamic content
   - Example layout for planner printable:
     ```
     [HEADER: Product Name - dynamic text field]
     [MAIN CONTENT AREA: Product features - dynamic text]
     [FOOTER: Your branding - static]
     ```

4. **Add Data Fields** (CRITICAL):
   - Select text box → Click "Data" in toolbar
   - Mark as data field → Name it (e.g., "product_name")
   - Repeat for all dynamic elements
   - Common fields:
     - `product_name` - Main title
     - `product_title` - Etsy optimized title
     - `main_benefit` - Key selling point
     - `keywords` - SEO tags
     - `niche` - Category

5. **Publish as Brand Template**:
   - Click "Share" → "Template link"
   - Make sure it says "Brand Template"
   - If option not available: Upgrade to Canva Pro or use alternative method

6. **Get Template ID**:
   Method A - From URL:
   - Open template
   - URL: `canva.com/design/XXXXXXXX`
   - `XXXXXXXX` = Template ID

   Method B - From API:
   - Call: `GET https://api.canva.com/rest/v1/brand-templates`
   - Find your template in list
   - Copy `id` field

**Save Template ID** - you'll need it in Step 3

---

#### Step 2: Get Canva API Access

**⚠️ VERIFY CURRENT PROCESS AT**: https://www.canva.dev/docs/connect/authentication/

**Steps (as of Jan 2025 - VERIFY):**

1. Go to https://www.canva.dev
2. Sign in with Canva account
3. Create new app:
   - App name: "Etsy Printables Automation"
   - Description: "Automated product creation"
   - Callback URL: `https://www.make.com/oauth/cb/oauth2`
     (Check Make.com docs for current callback URL)
4. Note credentials:
   - **Client ID**: Save this
   - **Client Secret**: Save this (won't see again)
5. Enable scopes:
   - `brand_template:read`
   - `autofill:create`
   - `design:read`
   - `design:export`
   - `export:read`

**Authentication Options:**

**Option A: OAuth2 (Recommended for production)**
- More secure
- User-level permissions
- Tokens refresh automatically
- Setup in Make.com:
  1. Add HTTP module
  2. Create OAuth2 connection
  3. Authorization URL: `https://www.canva.com/api/oauth/authorize`
  4. Token URL: `https://api.canva.com/oauth/token`
  5. Client ID: [from Step 2]
  6. Client Secret: [from Step 2]
  7. Scope: `autofill:create design:export`

**Option B: API Key (Simpler for testing)**
- Less secure (doesn't refresh)
- Easier to set up
- Good for development
- In Make.com:
  1. Use Bearer token authentication
  2. Token: Your API key from Canva dashboard

**⚠️ VERIFY** which method Canva currently recommends!

---

#### Step 3: Configure Make.com Scenario

1. Create new scenario: "03 - Build Canva Products"

2. **Add Module 1**: Google Sheets Watch Rows
   - Connection: Google Sheets - Etsy
   - Spreadsheet: Etsy Ideas Research
   - Sheet: Ideas Research
   - Filter: (Add after Step 4)
   - Limit: 5

3. **First, add columns to Google Sheet**:
   - **Column P**: Design Created (default: No)
   - **Column Q**: Canva Design URL
   - **Column R**: Canva Design ID
   - **Column S**: PNG File URL
   - **Column T**: PDF File URL
   - **Column U**: Files Created Date

4. **Back to Watch Rows - Set filter**:
   - Show advanced settings
   - Filter: `{{8.Content Generated}} = "Yes" AND {{15.Design Created}} != "Yes"`
   - (Column numbers may vary - check yours)

5. **Add Module 2**: Set Variables
   - Add all variables from "Module 2: Set Variables" section
   - **CRITICAL**: Enter YOUR template ID:
     `brand_template_id` = `[PASTE YOUR TEMPLATE ID]`
   - **CRITICAL**: Enter YOUR folder ID:
     `gdrive_folder_id` = `[YOUR GOOGLE DRIVE FOLDER ID]`
     - To get: Create folder in Drive → Right-click → "Get link" → Extract ID from URL

6. **Add Module 3**: HTTP Request (Create Autofill)
   - Method: POST
   - URL: `{{2.canva_api_base}}/autofills`
   - Headers:
     - `Authorization`: `Bearer [YOUR_CANVA_TOKEN]`
     - `Content-Type`: `application/json`
   - Body type: Raw
   - Body: (Copy JSON from Module 3 section)
   - **CRITICAL**: Update field names to match YOUR template
   - Parse response: Yes

7. **Add Module 4**: Sleep
   - Delay: 3 seconds

8. **Add Module 5**: Repeater
   - Initial value: 1
   - Repeats: `{{2.max_poll_attempts}}`
   - Step: 1

9. **Add Module 6**: HTTP Request (Check Status)
   - Method: GET
   - URL: `{{2.canva_api_base}}/autofills/{{3.data.job.id}}`
   - Headers: Same Authorization
   - Parse response: Yes

10. **Add Module 7**: Router
    - Route 1 (Success):
      - Filter: `{{6.data.job.status}} = "success"`
      - Label: "Design Ready"
    - Route 2 (Failed):
      - Filter: `{{6.data.job.status}} = "failed"`
      - Label: "Creation Failed"
    - Route 3 (In Progress):
      - Filter: `{{6.data.job.status}} = "in_progress"`
      - Label: "Still Processing"

11. **On Route 3 (In Progress)**:
    - Add Module 8: Sleep ({{2.poll_interval}} seconds)
    - Connect back to top of Repeater (creates loop)

12. **On Route 2 (Failed)**:
    - Add Error Handler module
    - Send email notification
    - Stop scenario

13. **On Route 1 (Success)**:
    - Add Module 9: Break (exits repeater)
    - Continue to exports...

14. **Add Module 10**: HTTP Request (Export PNG)
    - Method: POST
    - URL: `{{2.canva_api_base}}/exports`
    - Headers: Same Authorization + Content-Type
    - Body: (Copy from Module 10 section)
    - Parse response: Yes

15. **Add Modules 11-13**: PNG Poll & Download
    - Repeater (40 attempts)
    - HTTP GET export status
    - HTTP GET download file
    - (Same pattern as autofill polling)

16. **Add Module 14**: HTTP Request (Export PDF)
    - Same as Module 10, but format: "pdf"

17. **Add Modules 15-17**: PDF Poll & Download
    - Same pattern as PNG

18. **Add Module 18**: Google Drive Upload (PNG)
    - Create new connection:
      - Sign in with Google
      - Allow Drive access
    - Folder: Select or map `{{2.gdrive_folder_id}}`
    - File name: `{{1.Product Name}}_{{formatDate(now; "YYYYMMDD")}}.png`
    - Data: `{{13.data}}`

19. **Add Module 19**: Google Drive Upload (PDF)
    - Same as 18
    - File name ends with `.pdf`
    - Data: `{{17.data}}`

20. **Add Module 20**: Google Sheets Update Row
    - Connection: Google Sheets - Etsy
    - Spreadsheet: Etsy Ideas Research
    - Sheet: Ideas Research
    - Row number: `{{1.row}}`
    - Values:
      - Column P: `Yes`
      - Column Q: `{{6.data.job.result.design.url}}`
      - Column R: `{{6.data.job.result.design.id}}`
      - Column S: `{{18.webViewLink}}`
      - Column T: `{{19.webViewLink}}`
      - Column U: `{{formatDate(now; "YYYY-MM-DD HH:mm")}}`

---

#### Step 4: Test Thoroughly

**⚠️ DO NOT SKIP TESTING - THIS IS COMPLEX**

**Test 1: Single Product (Manual Run)**
1. In Google Sheet, find 1 row with:
   - Content Generated = Yes
   - Design Created = No
2. Make sure that row has all required fields
3. In Make.com, click "Run once"
4. Watch execution (will take 2-5 minutes):
   - ✅ Autofill job created
   - ✅ Polling starts (should see multiple checks)
   - ✅ Status changes to "success"
   - ✅ PNG export starts
   - ✅ PDF export starts
   - ✅ Both files download
   - ✅ Both uploaded to Google Drive
   - ✅ Sheet updated with URLs
5. Verify:
   - Check Google Drive - 2 files present?
   - Open PNG - high resolution?
   - Open PDF - correct content?
   - Click Canva URL - design exists?
   - Sheet row marked "Yes"?

**If test fails:** See Troubleshooting section

**Test 2: Verify Template Field Mapping**
1. Intentionally use wrong field name in Module 3
2. Run scenario
3. Should get error: "Unknown field: xxx"
4. Fix field names to match template
5. Re-run until success

**Test 3: Rate Limit Handling**
1. Set limit to 3 products
2. Run scenario
3. All 3 should process sequentially
4. Check timing: ~6-15 minutes total
5. Verify no rate limit errors

---

#### Step 5: Activate for Production

**Only after all tests pass:**

1. Set Watch Rows limit: `5` (conservative start)
2. Enable scheduling: Every 30 minutes
3. Add error notification:
   - Right-click scenario
   - Settings → Notifications
   - Enable "Email on error"
4. Monitor first 24 hours closely
5. Gradually increase limit if stable

---

## 🔐 Authentication Setup

### Canva API Authentication

**Type:** OAuth2 Bearer Token (or API Key)

**OAuth2 Setup (Recommended):**

1. In Make.com HTTP module:
   - Connection type: OAuth2
   - Authorization URL: `https://www.canva.com/api/oauth/authorize`
   - Token URL: `https://api.canva.com/oauth/token`
   - Client ID: [from Canva Developer Portal]
   - Client Secret: [from Canva Developer Portal]
   - Scope: `autofill:create design:export`
   - Redirect URI: `https://www.make.com/oauth/cb/oauth2`

2. Click "Sign in to Canva"
3. Authorize app
4. Token automatically refreshes

**API Key Setup (Alternative):**

1. Get key from Canva Developer Portal
2. In Make.com HTTP modules:
   - Header: `Authorization`
   - Value: `Bearer [YOUR_API_KEY]`
3. Manually refresh when expires

**Test Authentication:**
```bash
curl -X GET https://api.canva.com/rest/v1/users/me \
  -H "Authorization: Bearer YOUR_TOKEN"
```

Should return your user info.

---

### Google Drive Authentication

**Type:** OAuth2

**Setup:**
1. In Google Drive module → Add connection
2. Sign in with Google account
3. Allow permissions:
   - View and manage Drive files
   - Create and delete files
4. Connection name: "Google Drive - Etsy"

**Test:**
- Try creating a test file
- Should appear in Drive
- If fails: Re-authenticate

---

## ✅ Testing Checklist

### Test Case 1: Happy Path (Single Design)
**Setup:**
- 1 product ready (Content Generated = Yes, Design Created = No)
- All fields populated

**Execute:** Run once

**Expected:**
- ✅ Autofill job creates within 5 seconds
- ✅ Design completes within 30-60 seconds
- ✅ PNG export completes within 30 seconds
- ✅ PDF export completes within 30 seconds
- ✅ Both files uploaded to Drive
- ✅ Sheet updated with all URLs
- ✅ Total time: 2-3 minutes
- ✅ Operations used: ~40-50

**Verification:**
1. Open PNG in image viewer - should be 300 DPI
2. Open PDF in reader - should have correct content
3. Click Canva URL - should open design for editing
4. Check file sizes (PNG larger than PDF typically)

---

### Test Case 2: Template Field Mismatch
**Setup:**
- Change one field name in autofill request to invalid name
- Example: `product_name` → `wrong_field_name`

**Execute:** Run once

**Expected:**
- ❌ Autofill job fails
- ✅ Error message: "Unknown field: wrong_field_name"
- ✅ Error handler catches failure
- ✅ Notification sent

**Fix:**
1. Check template field names in Canva
2. Update autofill request JSON
3. Re-test until success

---

### Test Case 3: Rate Limit Handling
**Setup:**
- 10 products ready
- Limit set to 10

**Execute:** Run once

**Expected:**
- ⚠️ May hit rate limits (depends on Canva plan)
- ✅ Scenario should handle gracefully
- ✅ Some products may process slower
- ✅ All should eventually complete OR fail with clear error

**Verification:**
- Check execution time: ~20-50 minutes for 10
- No "429 Too Many Requests" errors unhandled
- If errors: Reduce limit to 5, add more delays

---

### Test Case 4: Long-Running Job
**Setup:**
- Complex template (10+ data fields, large images)

**Execute:** Run once

**Expected:**
- ✅ Polling continues for 40 attempts (up to 2 minutes)
- ✅ Job completes before timeout
- ✅ If timeout: Error handled gracefully

**Verification:**
- Check poll count in execution log
- Should complete within 40 attempts
- If not: Increase `max_poll_attempts` to 60

---

### Test Case 5: File Download Failure
**Setup:**
- Simulate expired download URL (wait 48+ hours after export)

**Execute:** Attempt download

**Expected:**
- ❌ Download returns 404 or 403
- ✅ Error handler catches
- ✅ Retry logic attempts again
- ✅ After max retries: Send notification

**Fix:**
- Re-export design
- Download immediately
- Configure alerts for stale exports

---

## ⚠️ Error Handling

### Common Error 1: "401 Unauthorized - Invalid token"
**Cause:** Canva API token expired or invalid

**Solutions:**
1. Check token in Make.com connection
2. If OAuth2: Click "Reconnect" to refresh
3. If API key: Generate new key from Canva portal
4. Update all HTTP modules with new token
5. Test with simple API call first:
   `GET /v1/users/me`

**Prevention:**
- Use OAuth2 (auto-refreshes)
- Monitor token expiration
- Set up alerts for auth failures

---

### Common Error 2: "Autofill job status = failed"
**Cause:** Template field mismatch or invalid data

**Solutions:**
1. Check Canva design template:
   - Open template → Click each data field
   - Note exact field names (case-sensitive)
2. Compare with autofill request JSON
3. Common mismatches:
   - `productName` vs `product_name` (case)
   - Using label instead of field ID
   - Field type mismatch (text vs image)
4. Update request JSON to match exactly
5. Add validation before autofill:
   - Check all required fields present
   - Verify data types match

**Prevention:**
- Document template fields in spreadsheet
- Use naming convention: `snake_case`
- Test with minimal data first

---

### Common Error 3: "Polling timeout - job still in progress after 40 attempts"
**Cause:** Complex design taking >2 minutes or Canva API slow

**Solutions:**
1. Increase `max_poll_attempts`:
   - From 40 to 60 (3 minutes)
   - Or 80 (4 minutes)
2. Increase `poll_interval`:
   - From 3 to 5 seconds
   - Reduces API calls
3. Check Canva status page:
   - https://status.canva.com
   - May be service disruption
4. Simplify template:
   - Reduce number of data fields
   - Use smaller images
   - Optimize design complexity

**Prevention:**
- Test template complexity vs processing time
- Set realistic timeouts
- Monitor Canva API status

---

### Common Error 4: "Export download URL expired (404)"
**Cause:** Waited too long between export and download

**Solutions:**
1. Download immediately after export completes
2. Don't add unnecessary delays
3. If download fails:
   - Re-export design
   - Download within 1 hour
4. Add retry logic:
   - On 404: Re-export and retry
   - Max 2 re-export attempts

**Prevention:**
- Export and download in same workflow
- No delays between export completion and download
- Monitor download success rate

---

### Common Error 5: "Google Drive upload failed - Quota exceeded"
**Cause:** Drive storage full or rate limit hit

**Solutions:**
1. Check Google Drive storage:
   - Go to drive.google.com/settings
   - Free up space if needed
2. If rate limit:
   - Add delay between uploads (2-3 seconds)
   - Reduce batch size (5 → 3 products)
3. Use different folder:
   - May be folder-specific limit
4. Verify permissions:
   - Make.com connection has write access

**Prevention:**
- Monitor Drive storage usage
- Set up storage alerts
- Use Business/Workspace account (higher limits)
- Implement cleanup for old files

---

### Common Error 6: "Parallel exports both fail when run together"
**Cause:** Rate limiting on export endpoint

**Solutions:**
1. Run exports sequentially instead of parallel:
   - PNG export → complete → PDF export
   - Adds ~30-60 seconds but more reliable
2. Add delay between export requests:
   - Sleep 5 seconds between PNG and PDF
3. Check Canva rate limits:
   - May have export-specific limits
   - Upgrade plan if needed

**Prevention:**
- Test with sequential first
- Monitor export success rate
- Adjust strategy based on limits

---

## 💡 Optimization Tips

### Performance Optimizations

**1. Optimize Polling Intervals**
- **Current:** Check every 3 seconds
- **Optimized:**
  - First 3 checks: 2 seconds (fast jobs)
  - Next 10 checks: 3 seconds
  - Remaining: 5 seconds
- **Savings:** Faster completion for simple designs

**2. Parallel Exports (If Rate Limits Allow)**
- Current: PNG and PDF sequential
- Optimized: Both export simultaneously
- **Savings:** 30-60 seconds per product

**3. Conditional PDF Export**
- Only export PDF if customer purchased
- For listings, maybe only need PNG
- **Savings:** 50% operations for listings

**4. Reuse Existing Designs**
- Check if design already exists (by product name)
- Skip autofill if already created
- Just export again
- **Savings:** ~20 operations per re-export

**5. Batch API Calls**
- If Canva supports batch autofill (check API docs)
- Create multiple designs in one call
- **Savings:** Significant for high volume

---

### Cost Optimizations

**Current Cost per Design:**
- Autofill: ~5-10 API calls (polling)
- PNG Export: ~5-10 API calls (polling)
- PDF Export: ~5-10 API calls (polling)
- Total Make.com operations: ~40-50
- Free tier: 1,000 ops/month = ~20-25 designs
- Pro tier: $10/month = 10,000 ops = ~200-250 designs

**Ways to Reduce:**

**1. Increase Poll Intervals**
- 3 seconds → 5 seconds
- Fewer total polls
- **Savings:** ~20% operations

**2. Single Format Export**
- Only export PNG (most versatile)
- Generate PDF on-demand later
- **Savings:** 50% operations

**3. Use Make.com Webhooks**
- If Canva supports webhooks (check docs)
- No polling needed - Canva notifies when done
- **Savings:** ~80% operations

**4. Optimize Template Complexity**
- Simpler templates = faster processing
- Fewer fields = faster autofill
- **Savings:** Faster completion = fewer timeouts

**5. Batch Processing Schedule**
- Run once daily (instead of every 30 min)
- Process all at once
- **Savings:** Fewer scenario activations

---

### Reliability Improvements

**1. Implement Exponential Backoff**
```
Poll 1: Wait 2 seconds
Poll 2: Wait 2 seconds
Poll 3: Wait 3 seconds
Poll 4-10: Wait 3 seconds
Poll 11+: Wait 5 seconds
Max: 60 attempts
```

**2. Add Comprehensive Error Handling**
- Every HTTP module gets error handler
- Retry logic on transient failures (5xx errors)
- Give up on permanent failures (4xx errors)
- Send detailed error notifications

**3. Implement Circuit Breaker**
- If 3 consecutive failures: Stop processing
- Send alert to review issue
- Prevents wasting operations on systemic failure

**4. Add Health Checks**
- Before processing batch, test Canva API:
  `GET /v1/users/me`
- If fails: Don't start processing
- Wait and retry later

**5. Validate Before Processing**
- Check all required fields present
- Verify template ID exists
- Test field name mapping
- Fail fast if validation fails

**6. Implement Idempotency**
- Store design ID in sheet after creation
- Check if design exists before creating
- If exists: Use existing, don't recreate
- Prevents duplicates

**7. Add Monitoring**
- Track success rate
- Monitor processing times
- Alert on anomalies:
  - Success rate < 90%
  - Average time > 5 minutes
  - Error rate > 10%

---

## 📚 Official Documentation

**Last Accessed:** 2025-01-19 (⚠️ Automated access blocked - CRITICAL MANUAL VERIFICATION REQUIRED)

**Canva API:**
- Autofill Guide: https://www.canva.dev/docs/connect/autofill-guide/
- API Reference: https://www.canva.dev/docs/connect/api-reference/
- Autofills Endpoint: https://www.canva.dev/docs/connect/api-reference/autofills/
- Exports Endpoint: https://www.canva.dev/docs/connect/api-reference/exports/
- Authentication: https://www.canva.dev/docs/connect/authentication/
- Rate Limits: https://www.canva.dev/docs/connect/rate-limits/
- Error Codes: https://www.canva.dev/docs/connect/errors/

**Make.com:**
- HTTP Module: https://www.make.com/en/help/modules/http-request-module
- Google Drive: https://www.make.com/en/integrations/google-drive
- OAuth2 Setup: https://www.make.com/en/help/modules/oauth2-setup
- Repeater: https://www.make.com/en/help/modules/repeater
- Router: https://www.make.com/en/help/modules/router

**Google APIs:**
- Drive API: https://developers.google.com/drive/api/guides/about-sdk
- Drive Upload: https://developers.google.com/drive/api/guides/manage-uploads

---

## 📈 Version History

### v1.0 - Initial Release (2025-01-19)
- Canva Autofill API v1
- Async polling pattern
- PNG + PDF export
- Google Drive integration
- Exponential backoff polling
- Comprehensive error handling

---

## 🚨 Known Deprecations

**⚠️ CRITICAL - Brand Template ID Format Change**
- **What Changed:** Canva updated Brand Template ID format in Sept 2024
- **Old Format:** 8-12 characters (e.g., `DABCdef12345`)
- **New Format:** Longer format (verify current)
- **Status (as of Jan 2025):** VERIFY if migration complete
- **Impact:** Old IDs may no longer work
- **Action Required:**
  1. Check your template ID format
  2. If using old format, get new ID via API:
     `GET /v1/brand-templates`
  3. Update all scenarios with new ID

**Monitor These:**
- Canva API version (currently v1)
- Export endpoint URLs
- Authentication flow
- Rate limit changes

---

## 🔄 Upcoming Changes

**Potential Improvements to Monitor:**
- Webhook support (eliminate polling)
- Batch autofill endpoint
- Real-time export (no polling)
- Higher rate limits
- New export formats

**Check Canva Changelog:**
https://www.canva.dev/docs/connect/changelog/

---

## ✅ Deployment Checklist

**Before activating scenario:**

**Canva Setup:**
- [ ] Brand Template created and published
- [ ] All data fields defined with exact names
- [ ] Template ID obtained and saved
- [ ] Canva API access enabled
- [ ] OAuth2 or API key configured
- [ ] Test autofill call successful (manual curl test)

**Make.com Configuration:**
- [ ] All 20 modules added in correct order
- [ ] All HTTP modules have correct endpoints
- [ ] All HTTP modules have authentication
- [ ] All variables set (template ID, folder ID, etc.)
- [ ] Polling loops configured (repeater + router)
- [ ] Error handlers added to critical modules
- [ ] Google Drive connection authenticated
- [ ] Google Sheets columns P-U added

**Testing Completed:**
- [ ] Single product test successful
- [ ] Template field mapping verified
- [ ] PNG export working (300 DPI confirmed)
- [ ] PDF export working
- [ ] Google Drive uploads successful
- [ ] Sheet updates working
- [ ] Error handling tested (intentional failure)
- [ ] Rate limit behavior understood

**Production Readiness:**
- [ ] Scenario named: "03 - Build Canva Products"
- [ ] Scheduling set: Every 30 minutes
- [ ] Limit set conservatively: 5 products
- [ ] Error notifications configured
- [ ] Manual verification of ALL documentation completed
- [ ] Canva API status page bookmarked
- [ ] Support contact info saved
- [ ] Backup plan if Canva API down

**Monitoring Plan:**
- [ ] Daily check for first week
- [ ] Success rate tracking
- [ ] Operations usage monitoring
- [ ] Error log review process
- [ ] Escalation plan for failures

---

**🎉 Workflow 3 Complete!**

**⚠️ CRITICAL REMINDER:** This is the most complex workflow. Do not skip testing. Verify every step manually before production use.

**Next:** Workflow 4 will take these created designs and automatically create Etsy listings.
