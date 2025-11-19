# 🚀 QUICK START GUIDE
## Get Your First Automated Product in 2 Hours

---

## ⏱️ Time Investment
- **Minimum Setup:** 2 hours (Workflows 1-3 only)
- **First Product:** 10-15 minutes (after setup)
- **Full System:** 8-12 hours (all 8 workflows)

---

## 📋 What You'll Need

### Accounts (Sign up before starting)
1. ✅ **Google Account** - FREE
   - For Google Sheets & Drive
   - Sign up: https://accounts.google.com

2. ✅ **Make.com Account** - FREE (1000 ops/month)
   - For automation scenarios
   - Sign up: https://www.make.com/en/register

3. ✅ **OpenAI Account** - $10-20/month
   - For AI content generation
   - Sign up: https://platform.openai.com/signup
   - Add credit: $10 to start

4. ✅ **Canva Pro** - $12.99/month
   - For design templates
   - Sign up: https://www.canva.com/pro
   - Apply for API access: https://www.canva.dev

5. ✅ **Etsy Shop** - FREE + $0.20/listing
   - To sell products
   - Sign up: https://www.etsy.com/sell

### Total Initial Cost
- **Setup:** ~$25-35/month
- **Per Product:** ~$0.50-1.00
- **ROI:** Break even at 10-15 sales/month

---

## 🎯 2-Hour Sprint: First Automated Product

### Hour 1: Setup Foundation

**0:00-0:15 - Google Sheet Setup**
1. Open Google Sheets
2. Create new spreadsheet: "Etsy Ideas Research"
3. Copy this header row:
   ```
   Date Generated | Product Name | Keywords | Search Volume | Competition | Niche Category | Target Audience | Content Generated | Etsy Title | Product Description | Tags | Product Features | Customer Persona | Usage Ideas | Generated Date | Design Created | Canva Design URL | Canva Design ID | PNG File URL | PDF File URL | Files Created Date
   ```
4. Save and share with yourself (copy URL)

**0:15-0:25 - OpenAI API Key**
1. Go to https://platform.openai.com/api-keys
2. Click "Create new secret key"
3. Name: "Make.com Etsy Automation"
4. Copy key (starts with `sk-proj-...`)
5. Save in safe place (can't view again!)
6. Add $10 credit to account (Billing section)

**0:25-0:45 - Make.com Account**
1. Sign up at https://www.make.com/en/register
2. Verify email
3. Start free trial if available
4. Familiarize with interface:
   - Click "Scenarios"
   - Click "Create a new scenario"
   - See the canvas

**0:45-1:00 - Canva Setup**
1. Upgrade to Canva Pro (if not already)
2. Go to https://www.canva.dev
3. Sign in with Canva account
4. Click "Get started" or "Create app"
5. **Note:** API approval may take 1-3 days
6. For now, note your Canva login for template creation

---

### Hour 2: Build First Workflow

**⚠️ START WITH WORKFLOW 1 ONLY**

**1:00-1:15 - Create Make.com Scenario**
1. In Make.com, click "+ Create a new scenario"
2. Name it: "01 - Idea & Keyword Research"
3. Click in canvas → Search "Schedule"
4. Add "Schedule" module
5. Configure:
   - Run: Daily
   - Time: 6:00 AM
   - Click OK

**1:15-1:30 - Add OpenAI Module**
1. Click "+" after Schedule module
2. Search "OpenAI"
3. Select "Create a Chat Completion"
4. Click "Add" for connection
5. Paste your OpenAI API key
6. Name connection: "OpenAI Production"
7. Save

**1:30-1:45 - Configure OpenAI Prompt**
1. Model: Select "gpt-4o-mini"
2. Messages → Add item:
   - Role: `user`
   - Content: (Copy from Workflow 1 README, Module 2)
3. Temperature: `0.8`
4. Max tokens: `3000`
5. Response format: "JSON object"
6. Click OK

**1:45-2:00 - Connect to Google Sheets**
1. Click "+" after OpenAI
2. Search "Google Sheets"
3. Select "Add Multiple Rows"
4. Create connection (sign in with Google)
5. Select your "Etsy Ideas Research" spreadsheet
6. Map output (basic version):
   - Just paste OpenAI output for now
7. Click "Run once"
8. **SUCCESS!** Check your Google Sheet - ideas appeared?

---

## ✅ Success Checklist

After 2 hours, you should have:
- [ ] Google Sheet with headers
- [ ] OpenAI API key working
- [ ] Make.com account set up
- [ ] Workflow 1 built and tested
- [ ] 50 product ideas in your sheet
- [ ] Ready for Workflow 2

---

## 🎓 Next Steps

### Day 2: Add Content Generation (Workflow 2)
**Time:** 1-2 hours
**Goal:** Generate titles, descriptions, tags

**Steps:**
1. Open `workflows/workflow-02-product-content/README.md`
2. Follow setup instructions
3. Test with 1-2 products
4. Verify quality

---

### Day 3: Create Designs (Workflow 3) ⭐ CRITICAL
**Time:** 2-3 hours
**Goal:** Automate Canva design creation

**Pre-requisites:**
- Canva API access approved (may need to wait)
- Brand Template created in Canva
- Template ID obtained

**Steps:**
1. Create simple Canva template
2. Add data fields
3. Get template ID
4. Follow Workflow 3 README
5. Test with 1 product (takes 5-10 min)
6. Celebrate your first automated design! 🎉

---

### Day 4: Create Listings (Workflow 4)
**Time:** 2-3 hours
**Goal:** Auto-create Etsy draft listings

**Pre-requisites:**
- Etsy shop created
- Etsy API access approved

**Steps:**
1. Apply for Etsy API access
2. Wait for approval
3. Follow Workflow 4 README
4. Test with 1 product
5. Review draft listing on Etsy
6. Publish manually

---

### Week 2+: Advanced Features
**Time:** 4-6 hours total
**Goal:** Complete automation system

**Add:**
- Workflow 6: File organization
- Workflow 7: Listing updates
- Workflow 8: Social media posting
- Workflow 5: Mockups (optional)

---

## 🆘 Stuck? Quick Troubleshooting

### OpenAI Not Working
**Error:** "Invalid API key"
**Fix:**
1. Check key copied correctly (no spaces)
2. Verify billing enabled in OpenAI account
3. Try generating new key

### Google Sheets Not Connecting
**Error:** "Permission denied"
**Fix:**
1. Re-authenticate connection
2. Check sheet is shared properly
3. Try with different Google account

### Scenario Not Running
**Error:** Nothing happens
**Fix:**
1. Click "Run once" (manual test)
2. Check scenario is ON (toggle)
3. Verify all modules connected

### JSON Parse Error
**Error:** "Unexpected token"
**Fix:**
1. OpenAI didn't return valid JSON
2. Add "Return ONLY JSON" to prompt
3. Use "JSON object" response format

---

## 💡 Pro Tips

**Tip 1:** Start small
- Don't build all 8 workflows at once
- Perfect Workflow 1 before moving to 2
- Test thoroughly at each step

**Tip 2:** Use "Run once" liberally
- Don't wait for schedule
- Test immediately
- Fix errors faster

**Tip 3:** Monitor operations
- Free tier: 1,000 ops/month
- Workflow 1 uses ~10 ops per run
- Upgrade when needed

**Tip 4:** Save your work
- Export scenarios regularly
- Backup Google Sheet
- Document customizations

**Tip 5:** Join communities
- Make.com community forum
- Etsy seller groups
- Learn from others

---

## 📞 Need Help?

**Can't figure it out?**
1. Check workflow-specific README
2. Search error message in docs
3. Ask in Make.com community
4. Review YouTube tutorials on Make.com basics

**Still stuck?**
- Create GitHub issue in this repo
- Include error screenshots
- Describe what you tried

---

## 🎉 You're Ready!

**Set a timer for 2 hours and GO!**

By the end, you'll have:
- ✅ Foundation set up
- ✅ First workflow running
- ✅ Product ideas generated
- ✅ Momentum to continue

**The hardest part is starting. You've got this! 💪**

---

**Quick Links:**
- [Full README](README.md)
- [Workflow 1 Guide](workflows/workflow-01-idea-research/README.md)
- [Workflow 2 Guide](workflows/workflow-02-product-content/README.md)
- [Workflow 3 Guide](workflows/workflow-03-canva-autofill/README.md)

**Questions?** Open an issue: [GitHub Issues](../../issues)

**Ready?** ⏰ **START YOUR 2-HOUR TIMER NOW!**
