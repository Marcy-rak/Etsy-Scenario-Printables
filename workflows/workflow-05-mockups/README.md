# WORKFLOW 5: Create Mockups (Optional/Advanced)

## 📋 Metadata
- **Status**: Optional - Advanced Users ⚠️ REQUIRES THIRD-PARTY SERVICES
- **Last Verified**: 2025-01-19 (Template - VERIFY BEFORE DEPLOYMENT)
- **Make.com API Version**: v2.0+
- **Modules Used**: 10-15 (varies by approach)
- **Estimated Credits/Run**: 20-40 operations per mockup
- **Difficulty**: Hard (Third-party API integration required)
- **Runtime**: ~1-3 minutes per mockup
- **CRITICAL NOTE**: Canva doesn't have mockup API - requires external service

## ⚠️ Critical Verification Required

**BEFORE DEPLOYING, MANUALLY VERIFY:**
- ✅ Chosen mockup service API availability (Smartmockups, Placeit, etc.)
- ✅ API pricing and rate limits
- ✅ Image quality and format requirements
- ✅ Output image specifications (resolution, format)
- ✅ Commercial use licensing
- ✅ Integration method (REST API, webhook, etc.)

**Documentation Checked (Manual verification needed):**
- Smartmockups API: https://smartmockups.com/api (if still available)
- Placeit API: https://www.placeit.net/api (check current status)
- Mockup Generator alternatives
- Make.com HTTP module: https://www.make.com/en/help/modules/http-request-module

## 🎯 What This Workflow Does

Creates product mockups to showcase your printables in realistic scenarios:

**Input:** Product design PNG from Workflow 3
**Process:**
1. Choose mockup template (frame, desk scene, tablet, etc.)
2. Upload design to mockup service API
3. Generate mockup preview
4. Download high-res mockup
5. Upload to Google Drive
6. Add to Etsy listing (Workflow 4)

**Output:**
- Professional mockup images
- Multiple angles/scenarios
- Lifestyle photos
- Ready for marketing use

**Business Value:**
- Increases conversion rate (mockups vs plain designs)
- Professional appearance
- Shows product in context
- Builds trust with buyers
- **ROI:** Mockups can increase sales 30-50%

## 🚧 Why This Is Optional/Advanced

**Challenges:**
1. **No Canva Mockup API** (as of Jan 2025)
   - Canva has mockup templates
   - But no programmatic access to mockup generation
   - Must use third-party services

2. **Third-Party Service Required**
   - Additional cost (per mockup or subscription)
   - Another API to integrate
   - Dependency on external service
   - May have limitations

3. **Manual Alternative Easier**
   - Create mockup templates in Canva
   - Manually place designs
   - Export and add to listings
   - Good for low volume

**When to Use This Workflow:**
- High volume (50+ products/month)
- Consistent mockup style needed
- Budget for mockup service
- Comfortable with complex integrations

**When to Skip:**
- Low volume (<20 products/month)
- Prefer manual control
- Limited budget
- Simple product presentation sufficient

---

## 📊 Mockup Service Options

### Option 1: Smartmockups (Recommended)
- **Website**: https://smartmockups.com
- **API**: Check if available (was discontinued, may be back)
- **Pros**: High quality, many templates, commercial use
- **Cons**: May not have API anymore, subscription cost
- **Pricing**: ~$29/month (verify current)

**Use Case:** Best for volume production, professional results

---

### Option 2: Placeit by Envato
- **Website**: https://www.placeit.net
- **API**: Limited/unavailable (check current status)
- **Pros**: Huge template library, video mockups
- **Cons**: Expensive, may not have API
- **Pricing**: ~$14.95/month or pay-per-mockup

**Use Case:** High-end mockups, marketing materials

---

### Option 3: Mockup Generator
- **Website**: https://mockupgenerator.net
- **API**: Check availability
- **Pros**: Simple, focused on print products
- **Cons**: Limited templates
- **Pricing**: Varies

**Use Case:** Printable-specific mockups

---

### Option 4: Mediamodifier
- **Website**: https://mediamodifier.com
- **API**: Yes (as of 2024)
- **Pros**: API available, good templates
- **Cons**: Learning curve, quality varies
- **Pricing**: API pricing separate

**Use Case:** API-first approach, automation

---

### Option 5: Custom Solution (Advanced)
- **Method**: Use image manipulation libraries
- **Tools**: ImageMagick, Sharp (Node.js), Pillow (Python)
- **Pros**: Full control, no recurring cost
- **Cons**: Requires programming, maintain templates
- **Complexity**: Very high

**Use Case:** Developers with specific needs

---

## ⚙️ Implementation Approach

**⚠️ IMPORTANT:** This workflow is highly dependent on chosen service. Below is a generic framework. Adapt to your chosen API.

### Recommended Approach: Mediamodifier API

**Why Mediamodifier:**
- Has public API (as of 2024)
- Reasonable pricing
- Good quality
- REST API easy to integrate

**Basic Flow:**
1. Upload design image
2. Select template ID
3. Generate mockup
4. Download result
5. Add to product listing

---

## ⚙️ Modules Used (Mediamodifier Example)

### Module 1: Google Sheets - Watch Rows
- **Type**: Trigger
- **Purpose**: Detect designs ready for mockup creation

**Configuration:**

| Field | Value |
|-------|-------|
| **Filter** | Listing Created = Yes AND Mockup Created = No |
| **Limit** | 5 (mockup generation slow) |

**New Column:**
- **Column AA**: Mockup Created (Yes/No)
- **Column AB**: Mockup Image URLs (comma-separated)

---

### Module 2: Set Variables
- **Type**: Utility
- **Purpose**: Configure mockup settings

**Variables:**

| Variable | Value |
|----------|-------|
| `mockup_api_key` | `[YOUR MEDIAMODIFIER API KEY]` |
| `mockup_template_id` | `[TEMPLATE ID]` |
| `mockup_quality` | `high` |
| `output_format` | `png` |
| `output_width` | `2000` (pixels) |

---

### Module 3: HTTP - Upload Design to Mockup Service
- **Type**: Action
- **Endpoint**: Varies by service
- **Purpose**: Send design PNG to mockup generator

**Generic Configuration:**

```json
POST /api/v1/mockups

Headers:
  Authorization: Bearer YOUR_API_KEY
  Content-Type: multipart/form-data

Body:
  template_id: TEMPLATE_ID
  design_image: [binary PNG data]
  options:
    quality: high
    format: png
```

**Download PNG First:**
- HTTP GET: `{{1.PNG File URL}}`
- Save: `{{.data}}`

---

### Module 4: Poll Mockup Status (If Async)
- **Type**: Repeater + HTTP
- **Purpose**: Check if mockup generation complete

**Pattern:** Same as Canva polling (Workflow 3)

---

### Module 5: HTTP - Download Mockup
- **Type**: Action
- **Purpose**: Get generated mockup file

**Configuration:**
- URL: From mockup service response
- Method: GET
- Parse: No (binary)

---

### Module 6: Google Drive - Upload Mockup
- **Type**: Action
- **Purpose**: Store mockup with other product files

**Configuration:**
- Folder: Same as product files
- Filename: `{{1.Product Name}}_mockup.png`
- Data: `{{5.data}}`

---

### Module 7: HTTP - Add to Etsy Listing (Optional)
- **Type**: Action
- **Purpose**: Add mockup as additional image

**Configuration:**
- Upload to listing (same as Workflow 4, Module 8)
- Rank: 2 (second image after main design)

---

### Module 8: Google Sheets - Update Row
- **Type**: Action
- **Purpose**: Mark mockup created

**Columns:**
- AA: `Yes`
- AB: `{{6.webViewLink}}`

---

## 📝 Setup Instructions (Generic)

### Step 1: Choose Mockup Service

**Evaluate Options:**
1. Check if API available
2. Review pricing
3. Test quality with manual upload
4. Verify commercial use allowed
5. Check rate limits

**Decision Matrix:**

| Service | API | Quality | Price | Best For |
|---------|-----|---------|-------|----------|
| Smartmockups | ⚠️ Check | ⭐⭐⭐⭐⭐ | $$$ | Pro use |
| Mediamodifier | ✅ Yes | ⭐⭐⭐⭐ | $$ | Automation |
| Placeit | ⚠️ Limited | ⭐⭐⭐⭐⭐ | $$$ | Marketing |
| Custom | ✅ Yes | Varies | $ | Developers |

---

### Step 2: Get API Access

**For Mediamodifier (Example):**
1. Sign up at https://mediamodifier.com
2. Go to API section
3. Generate API key
4. Note rate limits
5. Browse template library
6. Get template IDs for your mockup styles

**For Other Services:**
- Follow their API documentation
- Get credentials
- Test with curl or Postman first

---

### Step 3: Create Mockup Templates

**Manual Setup:**
1. Create mockup template in chosen service
2. Upload sample design
3. Adjust positioning, scale, rotation
4. Save template
5. Get template ID
6. Test with different designs

**Template Recommendations:**
- Desktop/workspace scene (for planners)
- Tablet/iPad mockup (for digital products)
- Framed print (for wall art)
- Notebook cover (for journals)

**Create 2-3 Templates:**
- Main mockup (hero image)
- Lifestyle scene
- Close-up detail

---

### Step 4: Build Make.com Scenario

**If API Available:**
- Follow modules 1-8 above
- Adapt to specific API endpoints
- Test with one product first

**If NO API:**
- Skip this workflow
- OR use Zapier/Make.com alternatives
- OR manual mockup creation

---

### Step 5: Alternative - Manual Mockup Workflow

**For Non-Technical Users:**

1. **Create Canva Mockup Templates**
   - Design mockup scenes in Canva
   - Add placeholder for your design
   - Save as template

2. **Manual Process (Fast for Low Volume)**
   - Open template
   - Replace placeholder with design
   - Export as PNG
   - Upload to Google Drive
   - Add to Etsy listing manually

3. **Batch Processing**
   - Duplicate template 10x
   - Bulk replace designs
   - Bulk export
   - Upload all to Drive

**Time:** ~2-3 minutes per mockup manually
**Break-even:** Manual vs automated worth it at ~30+ products/month

---

## 💡 Alternative Approach: Canva Manual + Automation

**Hybrid Method:**

1. **Create mockups in Canva (Manual)**
   - User creates mockup templates
   - Manually places designs
   - Exports mockups

2. **Automate Upload & Listing (Make.com)**
   - Watch Google Drive folder for new mockups
   - Automatically add to Etsy listings
   - Update spreadsheet

**Workflow:**
```
Manual: Create mockup in Canva → Export PNG → Save to Drive folder
Automated: Watch Drive folder → Upload to Etsy → Update sheet
```

**Benefits:**
- No external API needed
- Full creative control
- Still saves time on admin tasks
- Lower complexity

---

## ✅ Testing Checklist

**If Using API:**

### Test Case 1: Single Mockup Generation
**Setup:**
- 1 product with design
- Valid template ID
- API key configured

**Execute:** Run once

**Expected:**
- ✅ Design uploaded to service
- ✅ Mockup generated
- ✅ High-res output (2000px+)
- ✅ Downloaded successfully
- ✅ Uploaded to Drive
- ✅ Sheet updated

**Verification:**
- Open mockup - quality good?
- Design positioned correctly?
- No watermarks (if paid plan)?

---

### Test Case 2: Multiple Templates
**Setup:**
- 3 different template IDs
- Same design

**Execute:** Generate 3 mockups

**Expected:**
- ✅ All 3 styles generated
- ✅ Design adapts to each
- ✅ Consistent quality

**Verification:**
- Compare all 3 - variety good?
- Choose best for listings

---

## ⚠️ Error Handling

### Common Error 1: "API key invalid"
**Solution:** Verify key, check service status, renew subscription

### Common Error 2: "Template not found"
**Solution:** Verify template ID, check if template deleted, use different template

### Common Error 3: "Image quality poor"
**Solution:** Increase output resolution, use higher quality template, check input image DPI

### Common Error 4: "Rate limit exceeded"
**Solution:** Reduce batch size, add delays, upgrade plan

---

## 📚 Official Documentation

**Mockup Services:**
- Smartmockups: https://smartmockups.com
- Mediamodifier API: https://mediamodifier.com/api
- Placeit: https://www.placeit.net

**Make.com:**
- HTTP Module: https://www.make.com/en/help/modules/http-request-module

---

## ✅ Deployment Checklist

**Before activating (if using API):**

- [ ] Mockup service chosen and tested
- [ ] API access configured
- [ ] Template(s) created and IDs obtained
- [ ] Single mockup test successful
- [ ] Quality verified (resolution, positioning)
- [ ] Cost per mockup calculated
- [ ] Rate limits understood
- [ ] Scenario built and tested
- [ ] Error handling added
- [ ] Manual fallback documented

**If Skipping This Workflow:**
- [ ] Manual mockup process documented
- [ ] Canva templates created
- [ ] Team trained on manual process
- [ ] Time estimate for manual creation
- [ ] Quality control process defined

---

**🎉 Workflow 5 Complete!**

**Recommendation:** Most users should **skip this workflow** initially and use manual mockup creation. Add automation later if volume justifies the complexity and cost.

**Next:** Workflow 6 - Download & Organize Files
