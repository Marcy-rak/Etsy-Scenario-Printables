# WORKFLOW 6: Download & Organize Files

## 📋 Metadata
- **Status**: Ready for production ⚠️ VERIFY BEFORE DEPLOYMENT
- **Last Verified**: 2025-01-19
- **Make.com API Version**: v2.0+
- **Modules Used**: 8-10
- **Estimated Credits/Run**: 10-15 operations per product
- **Difficulty**: Medium
- **Runtime**: ~30 seconds per product

## 🎯 What This Workflow Does

Creates organized file structure in Google Drive for all product assets:

**Input:** Products with files in Google Drive (from Workflow 3)
**Process:**
1. Create folder structure (by product name or category)
2. Download files from URLs (PNG, PDF, mockups)
3. Organize into folders
4. Create product index spreadsheet
5. Generate shareable links
6. Update master sheet with folder URLs

**Output:**
- Organized folder structure
- All files in one place
- Easy to find and share
- Backup of all assets
- Shareable links for team/customers

**Business Value:**
- Easy file management
- Quick access to product files
- Backup and disaster recovery
- Team collaboration
- Customer delivery preparation

## ⚙️ Folder Structure

```
📁 Etsy Printables Products/
  📁 Wedding Planner/
    📄 Wedding_Planner_Design.png
    📄 Wedding_Planner_Customer.pdf
    📄 Wedding_Planner_Mockup.png
    📄 Product_Info.txt
  📁 Budget Tracker/
    📄 Budget_Tracker_Design.png
    📄 Budget_Tracker_Customer.pdf
    📄 Budget_Tracker_Mockup.png
    📄 Product_Info.txt
  📁 _Archive/
    (Old products)
  📄 Product_Index.xlsx
```

## ⚙️ Modules Used

### Module 1: Google Sheets - Watch Rows
**Filter:** Listing Created = Yes AND Files Organized = No

### Module 2: Google Drive - Create Folder
**Name:** `{{1.Product Name}}`
**Parent Folder:** Main products folder

### Module 3-5: Google Drive - Move/Copy Files
- PNG file
- PDF file
- Mockup file (if exists)

### Module 6: Google Drive - Create Text File
**Purpose:** Product metadata in folder
**Content:**
```
Product: {{1.Product Name}}
Created: {{1.Generated Date}}
Etsy Listing: {{1.Etsy Listing URL}}
Canva Design: {{1.Canva Design URL}}

Keywords: {{1.Tags}}
Category: {{1.Niche Category}}
```

### Module 7: Google Sheets - Update Row
**Columns:**
- AC: Files Organized (Yes)
- AD: Product Folder URL
- AE: Organization Date

## 📝 Quick Setup

1. Create main folder in Google Drive: "Etsy Printables Products"
2. Get folder ID from URL
3. Add modules to watch sheet and create subfolders
4. Move files from scattered locations to organized structure
5. Update sheet with folder URLs

## 💡 Use Cases

**Use Case 1: Team Collaboration**
- Share product folders with designers
- Everyone has access to latest files
- Easy handoff between team members

**Use Case 2: Customer Delivery**
- Quickly find customer files
- Share folder for bulk orders
- Organized for fast fulfillment

**Use Case 3: Backup**
- All files in structured location
- Easy to backup entire folder
- Disaster recovery ready

**Use Case 4: Portfolio**
- Show clients your work
- Easy to browse products
- Professional presentation

## ⚠️ Error Handling

**Error 1: Folder already exists**
- Solution: Check if folder exists first, reuse if found

**Error 2: File not accessible**
- Solution: Verify sharing permissions, re-upload if needed

**Error 3: Quota exceeded**
- Solution: Monitor storage usage, archive old files

## ✅ Deployment Checklist

- [ ] Main folder created in Google Drive
- [ ] Folder ID obtained
- [ ] Test folder creation successful
- [ ] Files organized correctly
- [ ] Metadata file created
- [ ] Sheet updated with folder URLs
- [ ] Shareable links working

**🎉 Workflow 6 Complete!**

**Next:** Workflow 7 - Update Existing Listings
