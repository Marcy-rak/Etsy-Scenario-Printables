# WORKFLOW 7: Update Existing Listings

## 📋 Metadata
- **Status**: Ready for production ⚠️ VERIFY BEFORE DEPLOYMENT
- **Last Verified**: 2025-01-19
- **Make.com API Version**: v2.0+
- **Modules Used**: 5-8
- **Estimated Credits/Run**: 8-12 operations per update
- **Difficulty**: Medium
- **Runtime**: ~15-30 seconds per listing

## 🎯 What This Workflow Does

Bulk update existing Etsy listings when you improve content:

**Input:** Products with listing ID + updated content
**Process:**
1. Detect which fields changed (title, description, tags, price)
2. Call Etsy API to update listing
3. Optionally update images
4. Log changes for tracking
5. Update sheet with revision date

**Output:**
- Updated listings on Etsy
- Change history logged
- Consistent updates across all products
- SEO improvements applied

**Business Value:**
- Improve underperforming listings
- Update seasonal keywords
- Fix typos at scale
- Price adjustments
- A/B testing different content

## 🎯 Common Update Scenarios

### Scenario 1: Seasonal Keyword Updates
**When:** Before holidays
**What:** Update tags with seasonal terms
**Example:** Add "valentines day" tag in January

### Scenario 2: Price Changes
**When:** Sales, promotions, market changes
**What:** Bulk price adjustment
**Example:** 20% off all products

### Scenario 3: Description Improvements
**When:** Better SEO insights available
**What:** Enhance descriptions with new keywords
**Example:** Add trending search terms

### Scenario 4: Image Replacements
**When:** Better mockups created
**What:** Replace listing images
**Example:** Update to new brand style

## ⚙️ Modules Used

### Module 1: Google Sheets - Watch Rows
**Filter:** Listing Created = Yes AND Update Pending = Yes

**New Columns:**
- AF: Update Pending (Yes/No/Scheduled)
- AG: Update Type (Tags/Price/Description/Images)
- AH: Last Updated Date

### Module 2: Router (By Update Type)
**Routes:**
- Route 1: Update tags only
- Route 2: Update price only
- Route 3: Update description only
- Route 4: Full update (all fields)

### Module 3: HTTP - Update Listing
**Endpoint:** `PATCH /v3/application/shops/{shop_id}/listings/{listing_id}`

**Request Body (Example - Tags Only):**
```json
{
  "tags": ["new tag 1", "new tag 2", ...]
}
```

**Request Body (Full Update):**
```json
{
  "title": "{{1.Etsy Title}}",
  "description": "{{1.Product Description}}",
  "tags": {{tags_array}},
  "price": {{1.Price}}
}
```

### Module 4: Google Sheets - Log Change
**Sheet:** "Update History"
**Columns:**
- Date/Time
- Product Name
- Listing ID
- Fields Updated
- Old Values
- New Values
- Updated By (Automation)

### Module 5: Google Sheets - Update Row
**Columns:**
- AF: Update Pending → No
- AH: Last Updated Date → Now

## 📝 Setup Instructions

### Step 1: Add Update Columns to Sheet
- Column AF: Update Pending
- Column AG: Update Type
- Column AH: Last Updated Date

### Step 2: Create "Update History" Sheet
Track all changes for analysis and rollback if needed.

### Step 3: Build Scenario
1. Watch for "Update Pending = Yes"
2. Route by update type
3. Call Etsy API with changes
4. Log to history sheet
5. Mark as updated

### Step 4: Mark Products for Update
When you want to update listings:
1. Make changes in Google Sheet (new title, tags, etc.)
2. Set "Update Pending" = Yes
3. Set "Update Type" = what changed
4. Scenario will process automatically

## 💡 Advanced Features

### Feature 1: Scheduled Updates
**Use Case:** Holiday keyword injection
**Setup:**
- Column: "Update Date" (future date)
- Filter: Update Date <= Today AND Update Pending = Yes
- Auto-updates on schedule

### Feature 2: A/B Testing
**Use Case:** Test different titles
**Setup:**
- Create 2 versions of product
- Test Version A for 7 days
- Switch to Version B
- Compare sales data
- Keep winner

### Feature 3: Bulk Price Changes
**Use Case:** Sale pricing
**Setup:**
- Formula in sheet: `=ORIGINAL_PRICE * 0.8` (20% off)
- Mark all for update
- Run scenario once
- All prices updated

### Feature 4: Rollback Changes
**Use Case:** Update didn't perform well
**Setup:**
- Check "Update History" sheet
- Copy old values back to main sheet
- Mark for update again
- Restore previous version

## ⚠️ Error Handling

**Error 1: "Listing not found (404)"**
- Listing may be deleted
- Check listing ID correct
- Skip and log error

**Error 2: "Invalid tags"**
- Tag validation failed
- Check tag length (<= 20 chars)
- Check tag count (<= 13)

**Error 3: "Price invalid"**
- Price too low (<$0.20 on Etsy)
- Check formula errors
- Verify number format

**Error 4: "Rate limit exceeded"**
- Too many updates too fast
- Add delays between updates
- Reduce batch size

## ✅ Testing Checklist

### Test Case 1: Update Tags Only
**Setup:** Change tags in 1 product, mark for update
**Expected:** Only tags updated, other fields unchanged
**Verify:** Check Etsy listing - tags changed?

### Test Case 2: Bulk Price Update
**Setup:** Change price for 10 products, mark all
**Expected:** All 10 prices updated
**Verify:** All listings show new price?

### Test Case 3: Rollback
**Setup:** Make update, then rollback using history
**Expected:** Original values restored
**Verify:** Listing back to original state?

## ✅ Deployment Checklist

- [ ] Update columns added to sheet
- [ ] Update History sheet created
- [ ] Router configured for update types
- [ ] Etsy API update endpoint tested
- [ ] Change logging working
- [ ] Test update successful (1 product)
- [ ] Batch update tested (3-5 products)
- [ ] Rollback procedure documented
- [ ] Team trained on update process

**🎉 Workflow 7 Complete!**

**Next:** Workflow 8 - Post to Social Media
