# WORKFLOW 8: Post to Social Media

## 📋 Metadata
- **Status**: Ready for production ⚠️ VERIFY BEFORE DEPLOYMENT
- **Last Verified**: 2025-01-19
- **Make.com API Version**: v2.0+
- **Modules Used**: 10-15 (varies by platform)
- **Estimated Credits/Run**: 15-25 operations per post
- **Difficulty**: Medium-Hard
- **Runtime**: ~1-2 minutes per post across platforms

## 🎯 What This Workflow Does

Automatically promotes new products on social media:

**Input:** New Etsy listing created (from Workflow 4)
**Process:**
1. Generate social media caption (AI or template)
2. Download product image (mockup or design)
3. Optionally resize for platform requirements
4. Post to Pinterest (direct API)
5. Post to Instagram (via Buffer/Later)
6. Post to Facebook (Graph API)
7. Track post URLs for analytics
8. Update sheet with social links

**Output:**
- Posts across 3+ platforms
- Consistent branding and messaging
- Traffic to Etsy listings
- Increased visibility
- Analytics tracking

**Business Value:**
- Consistent posting schedule
- Multi-platform presence
- No manual copy-paste
- Traffic generation
- **ROI:** Each social post can drive 10-50 visits

## 📱 Platform Strategy

### Pinterest (Priority 1) ⭐
**Why:** Direct to Etsy, visual discovery, long post lifespan
**Best For:** Printables, planners, wall art
**Post Type:** Pin with product image + listing link
**Frequency:** Every new product + repins

### Instagram (Priority 2)
**Why:** Brand building, engagement, story features
**Best For:** Lifestyle shots, behind-scenes, announcements
**Post Type:** Feed post + story
**Frequency:** 3-5x per week

### Facebook (Priority 3)
**Why:** Older demographic, groups, marketplace
**Best For:** Targeted ads, groups, page posts
**Post Type:** Page post with image + link
**Frequency:** 2-3x per week

### TikTok/Twitter/LinkedIn (Optional)
**When:** If audience present on platform
**Effort:** Additional integrations needed

## ⚙️ Modules Used

### Module 1: Google Sheets - Watch Rows
**Filter:** Listing Created = Yes AND Social Posted = No

**New Columns:**
- AI: Social Posted (Yes/No)
- AJ: Pinterest Pin URL
- AK: Instagram Post URL
- AL: Facebook Post URL
- AM: Total Clicks (updated manually or via analytics)

### Module 2: OpenAI - Generate Caption
**Purpose:** Create engaging social media copy

**Prompt:**
```
Create a social media caption for this product:

Product: {{1.Product Name}}
Description: {{1.Product Description}}
Target Audience: {{1.Target Audience}}

Requirements:
- 150-200 characters (short, punchy)
- Include relevant hashtags (5-10)
- Call-to-action (check it out, shop now, etc.)
- Emphasize benefits
- Friendly, enthusiastic tone
- Platform: Pinterest (adapt for others)

Example:
"🎉 New! Wedding Planner Printable to organize your big day! ✨ Digital download, instant access. Perfect for brides-to-be! 💍
#WeddingPlanner #BridalOrganizer #PrintablePlanner #WeddingPrep #BrideToBe #DigitalDownload #EtsyShop #WeddingPlanning"

Return ONLY the caption, nothing else.
```

**Output:** `{{2.choices[1].message.content}}`

### Module 3: Text Parser - Extract Hashtags
**Purpose:** Separate caption from hashtags for Instagram

**Pattern:** Match all `#tag` patterns
**Output:**
- Caption without hashtags
- Hashtags array

### Module 4: HTTP - Download Image
**Purpose:** Get product mockup or design PNG

**URL:** `{{1.PNG File URL}}` or `{{1.Mockup URL}}`
**Method:** GET
**Output:** Binary image data

### Module 5: Image Transformer (Optional)
**Purpose:** Resize/optimize for each platform

**Pinterest:** 1000px × 1500px (2:3 ratio)
**Instagram:** 1080px × 1080px (1:1 ratio)
**Facebook:** 1200px × 630px (landscape)

**If Make.com has image resize:**
- Use it here
- Otherwise: Upload as-is

### Module 6: HTTP - Post to Pinterest
**Endpoint:** `POST https://api.pinterest.com/v5/pins`

**Request:**
```json
{
  "board_id": "YOUR_BOARD_ID",
  "media_source": {
    "source_type": "image_url",
    "url": "{{image_url}}"
  },
  "title": "{{1.Product Name}}",
  "description": "{{2.caption}}",
  "link": "{{1.Etsy Listing URL}}"
}
```

**Authentication:** OAuth2 Bearer token
**Response:** Pin ID and URL

### Module 7: HTTP - Post to Instagram (via Buffer)
**Why Buffer:** Instagram API has limitations, Buffer easier

**Endpoint:** `POST https://api.bufferapp.com/1/updates/create.json`

**Request:**
```json
{
  "profile_ids": ["YOUR_INSTAGRAM_PROFILE_ID"],
  "text": "{{caption_without_hashtags}}",
  "media": {
    "photo": "{{image_url}}"
  },
  "shorten": false
}
```

**Alternative:** Later.com API, Hootsuite API, or direct Instagram Graph API

### Module 8: HTTP - Post to Facebook
**Endpoint:** `POST https://graph.facebook.com/v18.0/{page-id}/photos`

**Request:**
```json
{
  "url": "{{image_url}}",
  "caption": "{{2.caption}}\n\nShop now: {{1.Etsy Listing URL}}",
  "published": true
}
```

**Authentication:** Facebook Page Access Token
**Response:** Post ID and URL

### Module 9: Google Sheets - Update Row
**Columns:**
- AI: Social Posted = Yes
- AJ: Pinterest Pin URL
- AK: Instagram Post URL
- AL: Facebook Post URL
- AM: Posted Date

### Module 10: Sleep (Rate Limiting)
**Between platforms:** 2-3 seconds
**Prevents:** API rate limits

## 📝 Setup Instructions

### Step 1: Pinterest Setup

1. **Create Pinterest Business Account**
   - Convert personal to business (free)
   - Go to https://developers.pinterest.com

2. **Create App**
   - Name: "Etsy Product Promotions"
   - Get Client ID and Secret

3. **OAuth2 in Make.com**
   - Authorization URL: `https://www.pinterest.com/oauth/`
   - Token URL: `https://api.pinterest.com/v5/oauth/token`
   - Scope: `boards:read boards:write pins:read pins:write`

4. **Get Board ID**
   ```bash
   curl https://api.pinterest.com/v5/boards \
     -H "Authorization: Bearer YOUR_TOKEN"
   ```
   Find board, copy ID

### Step 2: Instagram Setup (via Buffer)

1. **Sign up for Buffer**
   - https://buffer.com
   - Connect Instagram account
   - Get API access (paid plan may be required)

2. **Get Access Token**
   - Buffer dashboard → Settings → API
   - Generate token

3. **Get Profile ID**
   ```bash
   curl https://api.bufferapp.com/1/profiles.json?access_token=YOUR_TOKEN
   ```
   Find Instagram profile, copy ID

4. **Configure in Make.com**
   - HTTP module with Bearer token

### Step 3: Facebook Setup

1. **Create Facebook App**
   - https://developers.facebook.com
   - Create app → Type: Business

2. **Get Page Access Token**
   - Graph API Explorer
   - Select your page
   - Get token with `pages_read_engagement`, `pages_manage_posts`

3. **Test Post**
   ```bash
   curl -X POST "https://graph.facebook.com/v18.0/PAGE_ID/photos" \
     -d "url=IMAGE_URL" \
     -d "caption=Test post" \
     -d "access_token=YOUR_TOKEN"
   ```

### Step 4: Build Scenario

1. Add all modules 1-10
2. Configure each platform's API
3. Test with one product
4. Verify posts appear on each platform
5. Activate scenario

## 💡 Advanced Features

### Feature 1: Scheduled Posting
**Instead of immediate:**
- Post at optimal times (evenings, weekends)
- Use Buffer's scheduling features
- Or delay in Make.com with Sleep module

### Feature 2: Content Variations
**Different caption per platform:**
- Pinterest: SEO-focused, keyword-rich
- Instagram: Casual, emoji-heavy
- Facebook: Longer, storytelling

**Implementation:**
- 3 different OpenAI calls with different prompts
- Or use template variations

### Feature 3: Story Posts (Instagram)
**In addition to feed:**
- Post to story with countdown sticker
- Link to Etsy (if eligible)
- 24-hour urgency

### Feature 4: Pinterest Boards by Category
**Organization:**
- Wedding products → Wedding Board
- Budget products → Finance Board
- Auto-select board based on niche

**Implementation:**
- Map niche category to board ID in variables
- Router to select correct board

### Feature 5: Analytics Tracking
**Track performance:**
- Use UTM parameters in links
- Track clicks per platform
- Update sheet with analytics data
- Optimize posting strategy

**Example Link:**
```
https://etsy.com/listing/123?utm_source=pinterest&utm_medium=social&utm_campaign=product_launch
```

## ⚠️ Error Handling

**Error 1: "Pinterest: Board not found"**
- Verify board ID
- Check if board deleted
- Use different board

**Error 2: "Instagram: Image size invalid"**
- Resize to 1080x1080
- Check aspect ratio (1:1 or 4:5)
- Compress if too large

**Error 3: "Facebook: OAuthException"**
- Token expired
- Re-authorize app
- Check page permissions

**Error 4: "Rate limit exceeded"**
- Add delays between posts
- Reduce posting frequency
- Spread across hours

**Error 5: "Image URL not accessible"**
- Make Google Drive file public
- Or upload to image hosting
- Check permissions

## ✅ Testing Checklist

### Test Case 1: Single Platform (Pinterest)
**Setup:** 1 product, Pinterest only
**Expected:** Pin created, link works, image shows
**Verify:** Go to Pinterest, find pin

### Test Case 2: All Platforms
**Setup:** 1 product, all 3 platforms
**Expected:** Posts on Pinterest, Instagram, Facebook
**Verify:** Check each platform, all present?

### Test Case 3: Caption Quality
**Setup:** Generate 5 captions for different products
**Expected:** Varied, engaging, on-brand
**Verify:** Read captions, sound good?

### Test Case 4: Link Tracking
**Setup:** Post with UTM parameters
**Expected:** Can track clicks in Google Analytics
**Verify:** Click link, appears in analytics?

## ✅ Deployment Checklist

**Platform Setup:**
- [ ] Pinterest business account created
- [ ] Pinterest API access configured
- [ ] Pinterest board ID obtained
- [ ] Instagram connected via Buffer/Later
- [ ] Instagram profile ID obtained
- [ ] Facebook page access token obtained
- [ ] All API authentications tested

**Make.com Configuration:**
- [ ] All 10 modules added
- [ ] OpenAI caption generation tested
- [ ] Image download working
- [ ] Pinterest posting successful
- [ ] Instagram posting successful
- [ ] Facebook posting successful
- [ ] Sheet updated with post URLs

**Content Quality:**
- [ ] Captions engaging and on-brand
- [ ] Hashtags relevant and varied
- [ ] Images high quality
- [ ] Links working and tracked
- [ ] CTA clear

**Production:**
- [ ] Scenario named: "08 - Post to Social Media"
- [ ] Scheduling: Every 2 hours (check for new listings)
- [ ] Or: Trigger immediately after Workflow 4
- [ ] Error notifications enabled
- [ ] Analytics tracking set up
- [ ] Performance monitoring plan

---

**🎉 Workflow 8 Complete!**

**All 8 Workflows Done!** Next: Create Master README for the entire system.
