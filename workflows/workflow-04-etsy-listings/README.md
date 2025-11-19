# WORKFLOW 4: Create Etsy Listings (Draft)

## 📋 Metadata
- **Status**: Ready for production ⚠️ REQUIRES MANUAL VERIFICATION
- **Last Verified**: 2025-01-19 (Template - VERIFY BEFORE DEPLOYMENT)
- **Make.com API Version**: v2.0+
- **Modules Used**: 8-12
- **Estimated Credits/Run**: 15-25 operations per listing
- **Difficulty**: Medium-Hard
- **Runtime**: ~30-60 seconds per listing
- **CRITICAL DEPENDENCY**: Etsy API access + Shop connection

## ⚠️ Critical Verification Required

**BEFORE DEPLOYING, MANUALLY VERIFY:**
- ✅ Etsy API v3 current endpoints at https://developers.etsy.com/documentation/reference
- ✅ Etsy OAuth2 authentication process
- ✅ Create listing endpoint: `POST /v3/application/shops/{shop_id}/listings`
- ✅ Upload image endpoint: `POST /v3/application/shops/{shop_id}/listings/{listing_id}/images`
- ✅ Required fields for draft listings
- ✅ Digital listing specific requirements
- ✅ Tag format and limitations (13 max, 20 chars each)
- ✅ Current rate limits (requests per second)
- ✅ Make.com Etsy module status

**Documentation Checked (Manual verification needed):**
- Make.com Etsy Module: https://apps.make.com/etsy
- Etsy API v3 Reference: https://developers.etsy.com/documentation/reference
- Create Listing: https://developers.etsy.com/documentation/reference#operation/createDraftListing
- Upload Images: https://developers.etsy.com/documentation/reference#operation/uploadListingImage
- OAuth2: https://developers.etsy.com/documentation/essentials/authentication

## 🎯 What This Workflow Does

Automatically creates Etsy draft listings with all content and files from previous workflows:

**Input:** Products with designs created (from Workflow 3)
**Process:**
1. Retrieve product data from Google Sheet
2. Format tags (comma-separated → array)
3. Create draft listing via Etsy API
4. Upload product images (PNG from Google Drive)
5. Set digital file (PDF from Google Drive)
6. Configure listing settings (digital, price, quantity)
7. Update sheet with listing URL

**Output:**
- Draft Etsy listing (not published)
- All images attached
- Digital file configured
- Ready for manual review & publish

**Business Value:**
- Creates listings 10x faster than manual
- Zero copy-paste errors
- SEO-optimized from start
- Consistent formatting
- Bulk creation capability
- **Time Saved:** 20 minutes per listing

**Why Draft Mode:**
- Allows manual review before going live
- Verify pricing and settings
- Check image quality
- Make final tweaks
- Prevent accidental publishing

---

## ⚙️ Modules Used

### Module 1: Google Sheets - Watch Rows
- **Type**: Trigger
- **Purpose**: Detect products ready for listing creation

**Configuration:**

| Field | Value | Notes |
|-------|-------|-------|
| **Connection** | Google Sheets - Etsy | Existing |
| **Spreadsheet** | Etsy Ideas Research | |
| **Sheet** | Ideas Research | |
| **Filter** | Design Created = Yes AND Listing Created = No | |
| **Limit** | 10 | Process 10 listings per run |

**New Column Required:**
- **Column V**: Listing Created (Yes/No)
- **Column W**: Etsy Listing ID
- **Column X**: Etsy Listing URL
- **Column Y**: Listing Status (Draft/Active)
- **Column Z**: Listing Created Date

**Filter:**
```
{{16.Design Created}} = "Yes" AND {{22.Listing Created}} != "Yes"
```
(Verify column numbers match your sheet)

---

### Module 2: Set Variables
- **Type**: Utility
- **Purpose**: Configure Etsy shop and listing defaults

**Variables:**

| Variable | Value | Notes |
|----------|-------|-------|
| `etsy_shop_id` | `[YOUR SHOP ID]` | From Etsy API or URL |
| `listing_price` | `4.99` | Default price (USD) |
| `listing_quantity` | `999` | Digital = unlimited |
| `who_made` | `i_did` | Required field |
| `when_made` | `made_to_order` | Digital products |
| `is_digital` | `true` | CRITICAL for printables |
| `taxonomy_id` | `1464` | Digital downloads category |
| `processing_min` | `1` | Instant download |
| `processing_max` | `1` | Instant download |

**How to Get Shop ID:**

**Method 1 - From API:**
```bash
curl https://openapi.etsy.com/v3/application/users/me/shops \
  -H "Authorization: Bearer YOUR_TOKEN"
```
Response: `shop_id` field

**Method 2 - From URL:**
- Go to your shop page
- URL: `etsy.com/shop/YourShopName`
- Not direct shop ID, need API call

**Taxonomy ID for Digital:**
- Etsy category ID for digital downloads
- Current: `1464` (verify in Etsy docs)
- Full list: `GET /v3/application/seller-taxonomy/nodes`

---

### Module 3: Text Parser - Split Tags
- **Type**: Utility
- **Purpose**: Convert comma-separated tags to array

**Configuration:**

| Field | Value |
|-------|-------|
| **Text** | `{{1.Tags}}` |
| **Pattern** | `,` (comma) |
| **Output** | Array of strings |

**Why Needed:**
- Sheet stores: `"tag1,tag2,tag3"` (string)
- Etsy API needs: `["tag1", "tag2", "tag3"]` (array)

**Processing:**
1. Split by comma
2. Trim whitespace from each
3. Validate: max 13 tags, 20 chars each
4. Filter empty strings

---

### Module 4: Iterator (Tag Validation)
- **Type**: Flow Control
- **Purpose**: Process each tag individually

**Configuration:**
- Array: `{{split(1.Tags; ",")}}`
- Process: Trim and validate each

**Operations per tag:**
1. Trim whitespace: `{{trim(4.value)}}`
2. Check length: `{{length(trim(4.value))}} <= 20`
3. If too long: Truncate to 20 chars
4. Add to validated array

---

### Module 5: Array Aggregator (Tags)
- **Type**: Utility
- **Purpose**: Collect validated tags into array

**Configuration:**
- Source Module: Iterator (Module 4)
- Target Structure: Array of strings
- Max items: 13 (Etsy limit)

**Output:**
```json
[
  "wedding planner",
  "printable pdf",
  "bridal organizer",
  ...
]
```

---

### Module 6: HTTP - Create Draft Listing
- **Type**: Action (HTTP Request)
- **Endpoint**: `POST https://openapi.etsy.com/v3/application/shops/{shop_id}/listings`
- **Purpose**: Create listing on Etsy

**⚠️ CRITICAL:** Use Make.com Etsy module if available (easier). Use HTTP only if needed.

**Configuration (HTTP Method):**

| Field | Value |
|-------|-------|
| **URL** | `https://openapi.etsy.com/v3/application/shops/{{2.etsy_shop_id}}/listings` |
| **Method** | POST |
| **Headers** | `Authorization: Bearer {{YOUR_ETSY_TOKEN}}` |
|  | `Content-Type: application/json` |
|  | `x-api-key: {{YOUR_ETSY_API_KEY}}` |
| **Body** | See JSON below |

**Request Body:**
```json
{
  "quantity": {{2.listing_quantity}},
  "title": "{{substring(1.Etsy Title; 0; 140)}}",
  "description": "{{1.Product Description}}",
  "price": {{2.listing_price}},
  "who_made": "{{2.who_made}}",
  "when_made": "{{2.when_made}}",
  "taxonomy_id": {{2.taxonomy_id}},
  "shipping_profile_id": null,
  "return_policy_id": null,
  "materials": [],
  "shop_section_id": null,
  "processing_min": {{2.processing_min}},
  "processing_max": {{2.processing_max}},
  "tags": {{5.array}},
  "styles": [],
  "item_weight": null,
  "item_length": null,
  "item_width": null,
  "item_height": null,
  "item_weight_unit": null,
  "item_dimensions_unit": null,
  "is_personalizable": false,
  "personalization_is_required": false,
  "personalization_char_count_max": null,
  "personalization_instructions": null,
  "state": "draft",
  "is_supply": false,
  "production_partner_ids": [],
  "image_ids": [],
  "is_digital": {{2.is_digital}}
}
```

**Critical Fields:**
- `title`: Max 140 chars (truncate with `substring`)
- `description`: Max 5000 chars
- `tags`: Array of 13 max
- `state`: MUST be `"draft"` (not `"active"`)
- `is_digital`: MUST be `true` for printables
- `price`: Decimal number (not string)

**Response:**
```json
{
  "listing_id": 1234567890,
  "title": "Wedding Planner Printable...",
  "state": "draft",
  "url": "https://www.etsy.com/listing/1234567890",
  ...
}
```

**Save:**
- `{{6.listing_id}}` → For image upload
- `{{6.url}}` → Listing URL
- `{{6.state}}` → Status

---

### Module 7: HTTP - Download PNG from Google Drive
- **Type**: Action
- **Purpose**: Get image file for upload to Etsy

**Configuration:**

**Method A - Direct Download:**
If Google Drive URL is direct download link:
- URL: `{{1.PNG File URL}}`
- Method: GET
- Parse response: No (binary)

**Method B - Google Drive API:**
If using file ID:
```
GET https://www.googleapis.com/drive/v3/files/{{file_id}}?alt=media
Authorization: Bearer {{GOOGLE_TOKEN}}
```

**⚠️ CRITICAL:** Etsy image requirements:
- Format: PNG, JPG, or GIF
- Max size: 10 MB
- Min dimensions: 2000px × 1500px (recommended)
- Max dimensions: 4000px × 3000px
- Aspect ratio: 4:3 recommended

**Save:**
- `{{7.data}}` → Binary image data

---

### Module 8: HTTP - Upload Listing Image
- **Type**: Action (HTTP Request)
- **Endpoint**: `POST /v3/application/shops/{shop_id}/listings/{listing_id}/images`
- **Purpose**: Add product image to listing

**Configuration:**

| Field | Value |
|-------|-------|
| **URL** | `https://openapi.etsy.com/v3/application/shops/{{2.etsy_shop_id}}/listings/{{6.listing_id}}/images` |
| **Method** | POST |
| **Headers** | `Authorization: Bearer {{YOUR_ETSY_TOKEN}}` |
|  | `x-api-key: {{YOUR_ETSY_API_KEY}}` |
| **Body Type** | Multipart/form-data |
| **Fields** | See below |

**Multipart Fields:**
- `image`: `{{7.data}}` (binary file)
- `rank`: `1` (first image)
- `overwrite`: `false`
- `is_watermarked`: `false`

**⚠️ Make.com Specifics:**
In Make.com HTTP module:
1. Body type: `Multipart/form-data`
2. Click "Add item"
3. Key: `image`
4. Value: `{{7.data}}`
5. Type: `File`

**Response:**
```json
{
  "listing_image_id": 9876543210,
  "listing_id": 1234567890,
  "url_75x75": "...",
  "url_170x135": "...",
  "url_fullxfull": "...",
  "rank": 1
}
```

---

### Module 9: HTTP - Upload Digital File (PDF)
- **Type**: Action
- **Endpoint**: `POST /v3/application/shops/{shop_id}/listings/{listing_id}/files`
- **Purpose**: Attach PDF for customer download

**Configuration:**

Similar to image upload:
- URL: `.../listings/{{6.listing_id}}/files`
- Method: POST
- Body type: Multipart/form-data

**Fields:**
- `file`: `{{PDF_data}}` (from Drive download)
- `name`: `{{1.Product Name}}.pdf`
- `rank`: `1`

**⚠️ Etsy Digital File Requirements:**
- Max size: 20 MB
- Allowed types: PDF, PNG, JPG, ZIP, etc.
- Must be set for `is_digital: true` listings

**Download PDF First:**
Add module before this:
- HTTP GET: `{{1.PDF File URL}}`
- Save: `{{.data}}`
- Use in file upload

---

### Module 10: Google Sheets - Update Row
- **Type**: Action
- **Purpose**: Mark listing as created, save URLs

**Configuration:**

| Field | Value |
|-------|-------|
| **Connection** | Google Sheets - Etsy |
| **Spreadsheet** | Etsy Ideas Research |
| **Sheet** | Ideas Research |
| **Row** | `{{1.row}}` |

**Columns to Update:**

| Column | Header | Value |
|--------|--------|-------|
| V | Listing Created | `Yes` |
| W | Etsy Listing ID | `{{6.listing_id}}` |
| X | Etsy Listing URL | `{{6.url}}` |
| Y | Listing Status | `{{6.state}}` (Draft) |
| Z | Listing Created Date | `{{formatDate(now; "YYYY-MM-DD HH:mm")}}` |

---

### Module 11: Error Handler (Optional)
- **Type**: Error Handler
- **Purpose**: Catch API failures

**Add to Module 6 (Create Listing):**
1. Right-click → Add error handler
2. Add "Google Sheets - Update Row"
3. Set Column V to: `Error - See logs`
4. Prevents infinite retries

**Common Etsy Errors to Handle:**
- 400: Invalid parameters
- 401: Authentication failed
- 403: Insufficient permissions
- 429: Rate limit exceeded
- 500: Etsy server error

---

## 🔧 Complete Scenario Flow Diagram

```
┌─────────────────────────────────────┐
│ Google Sheets Watch Rows            │
│ Filter: Design Created = Yes        │
│         Listing Created = No        │
│ Returns: Full product data          │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ Set Variables                       │
│ - Etsy shop ID                      │
│ - Listing defaults                  │
│ - Price, quantity, category         │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ Text Parser: Split Tags             │
│ Input: "tag1,tag2,tag3"             │
│ Output: ["tag1", "tag2", "tag3"]    │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ Iterator: Process Each Tag          │
│ - Trim whitespace                   │
│ - Validate length <= 20             │
│ - Truncate if needed                │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ Array Aggregator: Collect Tags     │
│ Max 13 tags                         │
│ Output: Validated tag array         │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ HTTP: Create Draft Listing          │
│ POST /v3/.../listings               │
│ Response: listing_id, URL           │
└──────────────┬──────────────────────┘
               │
               ├─────────────────────────┐
               ▼                         ▼
┌────────────────────────┐  ┌────────────────────────┐
│ HTTP: Download PNG     │  │ HTTP: Download PDF     │
│ From Google Drive      │  │ From Google Drive      │
└──────────┬─────────────┘  └──────────┬─────────────┘
           │                           │
           ▼                           ▼
┌────────────────────────┐  ┌────────────────────────┐
│ HTTP: Upload Image     │  │ HTTP: Upload File      │
│ To Etsy listing        │  │ To Etsy listing        │
│ Multipart/form-data    │  │ Digital file           │
└──────────┬─────────────┘  └──────────┬─────────────┘
           │                           │
           └───────────┬───────────────┘
                       ▼
          ┌──────────────────────────┐
          │ Google Sheets Update Row │
          │ - Mark as Created        │
          │ - Save listing URL       │
          │ - Save listing ID        │
          └──────────────────────────┘

Total Time: ~30-60 seconds per listing
Total Operations: ~15-25 per listing
Etsy API Calls: 3-4 per listing
```

---

## 📝 Setup Instructions

### Prerequisites

**Critical Requirements:**
1. ✅ Etsy shop with selling enabled
2. ✅ Etsy API key and OAuth2 access
3. ✅ Products with designs created (Workflow 3 complete)
4. ✅ Google Drive files accessible
5. ✅ Make.com account with operations available

### Step-by-Step Setup

#### Step 1: Get Etsy API Access

**⚠️ VERIFY CURRENT PROCESS:** https://developers.etsy.com/documentation/essentials/authentication

**Steps (as of Jan 2025 - VERIFY):**

1. Go to https://www.etsy.com/developers/your-apps
2. Click "Create a New App"
3. Fill in app details:
   - App name: "Etsy Printables Automation"
   - Description: "Automated listing creation"
   - Callback URL: `https://www.make.com/oauth/cb/oauth2`
     (Check Make.com for current URL)
4. Submit for review (may take 1-3 days)
5. Once approved, get credentials:
   - **API Key (Keystring)**: Save this
   - **Shared Secret**: Save this
6. Enable scopes:
   - `listings_r` (read listings)
   - `listings_w` (write listings)
   - `listings_d` (delete listings - optional)
   - `shops_r` (read shop info)
   - `shops_w` (write shop info)

**Get Shop ID:**
```bash
curl https://openapi.etsy.com/v3/application/users/me/shops \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "x-api-key: YOUR_API_KEY"
```

Response: Extract `shop_id`

---

#### Step 2: Configure OAuth2 in Make.com

**If using Make.com Etsy Module:**
1. Add Etsy module to scenario
2. Click "Add" connection
3. Enter API Key
4. Click "Sign in to Etsy"
5. Authorize app
6. Done!

**If using HTTP Module (manual OAuth2):**
1. Add HTTP module
2. Create OAuth2 connection:
   - Authorization URL: `https://www.etsy.com/oauth/connect`
   - Token URL: `https://api.etsy.com/v3/public/oauth/token`
   - Client ID: Your API Key
   - Client Secret: Your Shared Secret
   - Scope: `listings_w listings_r shops_r`
   - Grant type: Authorization code
   - PKCE: Enabled (check Etsy docs)
3. Click "Authorize"
4. Etsy login and approve
5. Token automatically managed

---

#### Step 3: Prepare Google Sheet

1. Open "Etsy Ideas Research"
2. Add columns V-Z:
   - **V**: Listing Created (Yes/No)
   - **W**: Etsy Listing ID
   - **X**: Etsy Listing URL
   - **Y**: Listing Status
   - **Z**: Listing Created Date
3. Set formula in V2:
   ```
   =IF(W2="","No","Yes")
   ```
   (Auto-sets "Yes" when listing ID present)
4. Copy formula down

---

#### Step 4: Create Make.com Scenario

1. New scenario: "04 - Create Etsy Listings"

2. **Module 1: Google Sheets Watch Rows**
   - Connection: Google Sheets - Etsy
   - Spreadsheet: Etsy Ideas Research
   - Sheet: Ideas Research
   - Filter: `Design Created = Yes AND Listing Created = No`
   - Limit: 10

3. **Module 2: Set Variables**
   - Add all variables from Module 2 section
   - **CRITICAL:** Set your shop ID:
     `etsy_shop_id` = `[YOUR SHOP ID FROM STEP 1]`
   - Adjust defaults:
     - `listing_price` = Your preferred price
     - `taxonomy_id` = Verify current category ID

4. **Module 3: Text Parser**
   - Type: Split text
   - Text: `{{1.Tags}}`
   - Pattern: `,`

5. **Module 4: Iterator**
   - Array: `{{3.array}}`

6. **Module 5: Array Aggregator**
   - Source: Iterator
   - Add item: `{{trim(4.value)}}`
   - Limit: 13 items

7. **Module 6: HTTP - Create Listing**
   - Method: POST
   - URL: `https://openapi.etsy.com/v3/application/shops/{{2.etsy_shop_id}}/listings`
   - Headers:
     - `Authorization`: `Bearer {{YOUR_ETSY_TOKEN}}`
     - `x-api-key`: `{{YOUR_API_KEY}}`
     - `Content-Type`: `application/json`
   - Body: (Copy JSON from Module 6 section)
   - Parse response: Yes

8. **Module 7: HTTP - Download PNG**
   - Method: GET
   - URL: `{{1.PNG File URL}}`
   - Parse response: No

9. **Module 8: HTTP - Upload Image**
   - Method: POST
   - URL: `https://openapi.etsy.com/v3/application/shops/{{2.etsy_shop_id}}/listings/{{6.listing_id}}/images`
   - Headers: Same as Module 6
   - Body type: Multipart/form-data
   - Add field:
     - Key: `image`
     - Value: `{{7.data}}`
     - Type: File
   - Add field:
     - Key: `rank`
     - Value: `1`

10. **Module 9: HTTP - Download PDF**
    - Same as Module 7
    - URL: `{{1.PDF File URL}}`

11. **Module 10: HTTP - Upload Digital File**
    - Method: POST
    - URL: `https://openapi.etsy.com/v3/application/shops/{{2.etsy_shop_id}}/listings/{{6.listing_id}}/files`
    - Headers: Same
    - Body type: Multipart/form-data
    - Add field:
      - Key: `file`
      - Value: `{{9.data}}`
      - Type: File
    - Add field:
      - Key: `name`
      - Value: `{{1.Product Name}}.pdf`

12. **Module 11: Google Sheets Update**
    - Connection: Google Sheets - Etsy
    - Spreadsheet: Etsy Ideas Research
    - Sheet: Ideas Research
    - Row: `{{1.row}}`
    - Values:
      - Column V: `Yes`
      - Column W: `{{6.listing_id}}`
      - Column X: `{{6.url}}`
      - Column Y: `{{6.state}}`
      - Column Z: `{{formatDate(now; "YYYY-MM-DD HH:mm")}}`

---

#### Step 5: Test with One Product

1. In Google Sheet, find 1 product with:
   - Design Created = Yes
   - Listing Created = No
2. Verify it has:
   - Etsy Title (under 140 chars)
   - Description
   - Tags
   - PNG File URL
   - PDF File URL
3. Click "Run once" in Make.com
4. Watch execution (~30-60 seconds)
5. Check for success:
   - No errors in log
   - Listing ID returned
   - Image uploaded
   - File uploaded
   - Sheet updated

**Verify on Etsy:**
1. Go to your Etsy shop dashboard
2. Navigate to "Listings" → "Drafts"
3. Find new listing
4. Open it
5. Check:
   - Title correct?
   - Description formatted?
   - Tags all present (up to 13)?
   - Image showing?
   - Digital file attached?
   - State = Draft?

**If anything wrong:** See Troubleshooting section

---

#### Step 6: Activate for Production

**Only after successful test:**

1. Enable scheduling: Every 30 minutes
2. Adjust limit if needed (10 → 5 for slower)
3. Add error notifications
4. Monitor first day closely
5. Review draft listings before publishing

---

## 🔐 Authentication Setup

### Etsy API OAuth2

**Type:** OAuth2 with PKCE

**Setup in Make.com:**
1. HTTP module → Create OAuth2 connection
2. Configuration:
   - Authorization URL: `https://www.etsy.com/oauth/connect`
   - Token URL: `https://api.etsy.com/v3/public/oauth/token`
   - Client ID: [Etsy API Key]
   - Client Secret: [Etsy Shared Secret]
   - Scope: `listings_w listings_r shops_r`
   - Grant type: `authorization_code`
   - PKCE: Enable code challenge
   - Redirect URI: Make.com's OAuth callback
3. Click "Authorize"
4. Sign in to Etsy
5. Approve permissions
6. Token auto-refreshes

**Manual Test:**
```bash
curl https://openapi.etsy.com/v3/application/users/me \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "x-api-key: YOUR_API_KEY"
```

Should return your user data.

---

## ✅ Testing Checklist

### Test Case 1: Happy Path (Draft Listing)
**Setup:**
- 1 product with all required data
- PNG and PDF files accessible

**Execute:** Run once

**Expected:**
- ✅ Listing created in draft state
- ✅ Title under 140 chars
- ✅ Description formatted
- ✅ All 13 tags present
- ✅ Image uploaded and visible
- ✅ PDF file attached
- ✅ Sheet updated with listing URL
- ✅ Can open listing in Etsy dashboard

**Verification:**
1. Go to Etsy shop → Drafts
2. Find listing
3. Check all fields populated
4. Try editing - everything editable?
5. Try publishing - any errors?

---

### Test Case 2: Tag Validation
**Setup:**
- Product with 15 tags (over limit)
- Some tags over 20 chars

**Execute:** Run once

**Expected:**
- ✅ Only first 13 tags used
- ✅ Long tags truncated to 20 chars
- ✅ No API error
- ✅ Listing created successfully

**Verification:**
- Check listing - exactly 13 tags?
- All tags <= 20 characters?
- No truncation mid-word (ideally)

---

### Test Case 3: Image Upload Failure
**Setup:**
- Invalid image URL (404)
- Or oversized image (>10 MB)

**Execute:** Run once

**Expected:**
- ⚠️ Listing created (no image)
- ❌ Image upload fails
- ✅ Error logged
- ✅ PDF still uploaded
- ✅ Sheet updated with listing URL

**Verification:**
- Listing exists but no image?
- Error handler caught failure?
- Can manually add image later?

---

### Test Case 4: Batch Creation (10 Listings)
**Setup:**
- 10 products ready
- All with valid data

**Execute:** Run once

**Expected:**
- ✅ All 10 listings created
- ✅ No duplicate listings
- ✅ All marked in sheet
- ✅ Total time: ~5-10 minutes
- ✅ No rate limit errors

**Verification:**
- Etsy drafts shows 10 new listings?
- All have images and files?
- Sheet has all 10 URLs?

---

## ⚠️ Error Handling

### Common Error 1: "400 Bad Request - Invalid title"
**Cause:** Title exceeds 140 characters

**Solutions:**
1. Check title length in sheet
2. Add validation before API call:
   ```
   {{if(length(1.Etsy Title) > 140; substring(1.Etsy Title; 0; 140); 1.Etsy Title)}}
   ```
3. Or truncate in API request:
   ```
   "title": "{{substring(1.Etsy Title; 0; 140)}}"
   ```

**Prevention:**
- Workflow 2 should generate titles under 140 chars
- Add post-generation validation
- Truncate with "..." if needed

---

### Common Error 2: "403 Forbidden - Insufficient permissions"
**Cause:** OAuth token missing `listings_w` scope

**Solutions:**
1. Check OAuth scopes in Etsy app
2. Ensure `listings_w` enabled
3. Re-authorize connection in Make.com
4. Revoke and recreate OAuth token

**Prevention:**
- Document required scopes
- Test auth before bulk processing
- Monitor auth status

---

### Common Error 3: "422 Unprocessable - Invalid taxonomy_id"
**Cause:** Category ID invalid or changed

**Solutions:**
1. Get current taxonomy for digital:
   ```bash
   curl https://openapi.etsy.com/v3/application/seller-taxonomy/nodes \
     -H "x-api-key: YOUR_KEY"
   ```
2. Find "Digital Downloads" category
3. Update `taxonomy_id` variable
4. Re-run scenario

**Prevention:**
- Verify taxonomy IDs periodically
- Monitor Etsy API changelog
- Have backup category ID

---

### Common Error 4: "Image upload failed - File too large"
**Cause:** PNG from Canva over 10 MB

**Solutions:**
1. Compress image before upload:
   - Add image optimization module
   - Reduce quality or dimensions
   - Convert to JPG if needed
2. Check Canva export settings:
   - Lower DPI (300 → 150)
   - Smaller dimensions
3. Upload optimized version

**Prevention:**
- Configure Canva exports appropriately
- Add file size check before upload
- Compress large files automatically

---

### Common Error 5: "Rate limit exceeded (429)"
**Cause:** Too many API calls too quickly

**Solutions:**
1. Reduce batch size (10 → 5)
2. Add delay between listings:
   - Sleep module (3-5 seconds)
   - After each listing creation
3. Check Etsy rate limits:
   - Typically: 10 requests/second
   - Daily limits may apply
4. Spread out processing:
   - Run hourly instead of every 30 min
   - Process fewer at a time

**Prevention:**
- Stay within rate limits
- Monitor API usage
- Implement backoff strategy

---

## 💡 Optimization Tips

### Performance Optimizations

**1. Parallel File Downloads**
- Download PNG and PDF simultaneously
- Not sequentially
- **Savings:** ~5-10 seconds per listing

**2. Conditional Digital File Upload**
- Only upload if `is_digital = true`
- Skip for physical products (if any)
- **Savings:** Operations + time

**3. Batch Validation**
- Validate all data before API calls
- Fail fast if missing required fields
- **Savings:** Wasted API calls

**4. Reuse Existing Listings**
- Check if product already has listing
- Update instead of create
- **Savings:** Duplicate prevention

---

### Cost Optimizations

**Current Cost per Listing:**
- Make.com operations: ~15-25
- Free tier: 1,000 ops/month = ~40-60 listings
- Pro tier: 10,000 ops = ~400-600 listings

**Ways to Reduce:**

**1. Use Make.com Etsy Module**
- Native integration more efficient
- Handles auth automatically
- **Savings:** ~20% operations

**2. Reduce Polling/Checks**
- Don't check listing status after creation
- Trust API response
- **Savings:** 1-2 operations per listing

**3. Conditional Image Upload**
- Only if image URL present
- Skip if missing
- **Savings:** Failed upload operations

---

### Reliability Improvements

**1. Add Pre-Flight Validation**
Before creating listing:
```
- Title length <= 140
- Description length <= 5000
- Tags count <= 13
- Each tag length <= 20
- PNG URL accessible
- PDF URL accessible
- Price > 0
```

**2. Implement Idempotency**
- Store listing ID immediately after creation
- Check if already exists before creating
- Use SKU or product name as unique key
- Prevents duplicates on retry

**3. Error Recovery**
- Save partial progress
- If image upload fails: Log but continue
- Can manually fix later
- Don't fail entire batch

**4. Monitor Listing Quality**
- Random sample check
- Verify all fields populated
- Check image quality
- Review tags relevance

---

## 📚 Official Documentation

**Last Accessed:** 2025-01-19 (⚠️ Manual verification required)

**Etsy API:**
- API Reference: https://developers.etsy.com/documentation/reference
- Create Listing: https://developers.etsy.com/documentation/reference#operation/createDraftListing
- Upload Image: https://developers.etsy.com/documentation/reference#operation/uploadListingImage
- Upload File: https://developers.etsy.com/documentation/reference#operation/uploadListingFile
- OAuth2: https://developers.etsy.com/documentation/essentials/authentication
- Rate Limits: https://developers.etsy.com/documentation/essentials/rate-limits

**Make.com:**
- Etsy Module: https://apps.make.com/etsy
- HTTP Module: https://www.make.com/en/help/modules/http-request-module
- Multipart Upload: https://www.make.com/en/help/modules/http-multipart

---

## 📈 Version History

### v1.0 - Initial Release (2025-01-19)
- Etsy API v3
- Draft listing creation
- Image and file upload
- Tag validation
- OAuth2 authentication

---

## ✅ Deployment Checklist

Before activating:

**Etsy Setup:**
- [ ] Etsy shop active and in good standing
- [ ] Etsy API app created and approved
- [ ] API Key and Shared Secret obtained
- [ ] OAuth2 configured in Make.com
- [ ] Shop ID retrieved and saved
- [ ] Taxonomy ID verified for digital downloads
- [ ] Test API call successful

**Make.com Configuration:**
- [ ] All 11 modules added
- [ ] Variables set (shop ID, taxonomy, price)
- [ ] Tag validation implemented
- [ ] HTTP requests have correct endpoints
- [ ] Authentication headers present
- [ ] Multipart uploads configured correctly
- [ ] Google Sheets columns V-Z added

**Testing:**
- [ ] Single listing test successful
- [ ] Listing visible in Etsy drafts
- [ ] Image uploaded correctly
- [ ] Digital file attached
- [ ] Tags formatted properly
- [ ] Sheet updated with URL
- [ ] Batch test (3-5 listings) successful

**Production:**
- [ ] Scenario named: "04 - Create Etsy Listings"
- [ ] Scheduling: Every 30 minutes
- [ ] Limit: 10 (or conservative number)
- [ ] Error notifications enabled
- [ ] Manual review process established
- [ ] Publish workflow documented

---

**🎉 Workflow 4 Complete!**

Next: Workflow 5 (Mockups), Workflow 6 (File Organization), Workflow 7 (Update Listings), Workflow 8 (Social Media)
