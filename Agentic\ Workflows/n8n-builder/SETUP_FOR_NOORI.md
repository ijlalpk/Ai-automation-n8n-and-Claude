# ⚡ n8n Job Automation Setup - Personalized for Noori

**Your Details:**
- WhatsApp: +923336102811
- Email: aghaijalal17@gmail.com
- Google Sheet: Job Applications (ID: 1fp4I2ASymFMtNL6I7VTpjGv03t02kwIM85kFmsRBzvM)

---

## 🚀 QUICK SETUP (15 minutes)

### STEP 1: Twilio WhatsApp Setup (3 min)

1. Open: https://www.twilio.com/console/sms/whatsapp/sandbox
2. Note the **Sandbox Number** shown (e.g., +14155238886)
3. Find your **Join Code** on the same page
4. **Open WhatsApp on your phone**
5. Send message to sandbox number: `join YOUR_CODE_HERE`
6. Wait for confirmation message from Twilio
7. Go to Twilio Console Dashboard
8. Copy **Account SID** (starts with `AC...`)
9. Copy **Auth Token** (long string)
10. ✅ Save these for Step 4

---

### STEP 2: Gmail Setup (4 min)

**Via GMAIL_SHEETS_SETUP.md - Part 1:**

1. Go to: https://console.cloud.google.com/
2. Create new project or select existing one
3. Search for **"Gmail API"** → Click **Enable**
4. Go to **Credentials** (left sidebar)
5. Click **+ Create Credentials** → **OAuth 2.0 Client IDs**
6. Choose **Desktop application** → Click **Create**
7. Download credentials JSON

**In n8n:**
1. Go to **Credentials** (bottom left)
2. Click **+** → Search **Gmail**
3. Select **Gmail OAuth2**
4. Click link to authorize
5. Log in with **aghaijalal17@gmail.com**
6. Grant permissions
7. Click **Create**
8. Name it: `Gmail - Job Notifications`
9. ✅ Done

---

### STEP 3: Google Sheets Setup (5 min)

**Via GMAIL_SHEETS_SETUP.md - Part 2:**

1. Go to: https://console.cloud.google.com/ (same project)
2. Search for **"Google Sheets API"** → Click **Enable**
3. Go to your Google Sheet: https://docs.google.com/spreadsheets/d/1fp4I2ASymFMtNL6I7VTpjGv03t02kwIM85kFmsRBzvM/
4. Make sure you have 2 sheets:
   - **Sheet1 (Job Applications)** with columns: jobId | title | company | source | appliedAt | status | notes | url
   - **Sheet2 (Interview Tracker)** with columns: jobId | title | company | source | status | appliedDate | interviewDate | interviewType | interviewNotes | followUpDate | outcome | offerDetails | rejectionReason | lastUpdated

**In n8n:**
1. Go to **Credentials** → **+** → Search **Google Sheets**
2. Select **Google Sheets OAuth2**
3. Authorize with same Google account
4. Click **Create**
5. Name it: `Google Sheets - Job Tracker`
6. ✅ Done

---

### STEP 4: Twilio Credential in n8n (1 min)

1. Go to **Credentials** → **+** → Search **Twilio**
2. Select **Twilio API**
3. Paste **Account SID** (from Step 1)
4. Paste **Auth Token** (from Step 1)
5. Click **Create**
6. Name it: `Twilio - WhatsApp Sandbox`
7. ✅ Done

---

### STEP 5: Create n8n Data Table (2 min)

1. In n8n, go to **Data** tab (left sidebar)
2. Click **Create** → **Data Table**
3. Name: `remote_job_finder_seen_jobs`
4. Add columns:
   - jobId (text)
   - source (text)
   - title (text)
   - company (text)
   - url (text)
   - firstSeenAt (datetime)
5. Click **Create**
6. ✅ Done

---

### STEP 6: Environment Variables (2 min)

**In your n8n instance**, add to `.env` file or Environment Variables UI:

```bash
N8N_USER_WHATSAPP_NUMBER=+923336102811
N8N_USER_EMAIL=aghaijalal17@gmail.com
N8N_GOOGLE_SHEET_ID=1fp4I2ASymFMtNL6I7VTpjGv03t02kwIM85kFmsRBzvM
N8N_EXPERIENCE_YEARS=2
```

Then **restart n8n** for variables to take effect.

✅ Done

---

### STEP 7: Import Workflows (1 min)

In your n8n instance:

1. Go to **Workflows** → **Create** → **Import from file**
2. Upload: `workflows/job-finder-expanded.json`
3. Click **Import**
4. Repeat for:
   - `workflows/job-application-tracker.json`
   - `workflows/interview-tracker.json`
5. ✅ Done

---

## ✅ TEST EVERYTHING

### Test 1: Job Finder

1. Open **job-finder-expanded** workflow
2. Click **Execute Workflow** button
3. Wait 10 seconds
4. **Check your WhatsApp** (+923336102811) for job notification
5. You should see: "🆕 Job Title @ Company | Source | Link"
6. ✅ If message arrives, everything works!

### Test 2: Job Application Tracker

1. Open **job-application-tracker** workflow
2. Click **Execute Workflow**
3. When prompted, paste this test data:
```json
{
  "jobId": "test_001",
  "title": "Cloud Engineer",
  "company": "Test Company",
  "source": "Remotive",
  "status": "Applied",
  "url": "https://example.com/job"
}
```
4. Check your **Google Sheet** - new row should appear
5. Check **Gmail** (aghaijalal17@gmail.com) for confirmation email
6. ✅ If row + email arrive, it works!

### Test 3: Interview Tracker

1. Open **interview-tracker** workflow
2. Click **Execute Workflow**
3. When prompted, paste:
```json
{
  "jobId": "test_001",
  "title": "Cloud Engineer",
  "company": "Test Company",
  "status": "Interview Scheduled",
  "interviewDate": "2026-09-25T14:00:00Z",
  "interviewType": "Video",
  "interviewNotes": "Test interview"
}
```
4. Check your **Google Sheet** Interview Tracker sheet - new row should appear
5. Check **Gmail** for "Interview Scheduled" notification
6. ✅ If row + email arrive, it works!

---

## 🎯 FINAL ACTIVATION

Once all 3 tests pass:

1. Open **job-finder-expanded** workflow
2. Click **Activate** (top right)
3. It will now run automatically every 6 hours
4. Keep other 2 workflows **inactive** - trigger manually when needed

---

## 📊 YOU NOW HAVE

✅ **Job Finder:** Discovers jobs every 6 hours from 8 job boards
✅ **Job Tracker:** Logs your applications to Excel
✅ **Interview Tracker:** Tracks interviews from scheduled → offer/rejection
✅ **Notifications:** WhatsApp alerts + Gmail confirmations at every stage
✅ **Zero Cost:** All free APIs

---

## 🆘 TROUBLESHOOTING

**"WhatsApp message doesn't arrive"**
- Confirm you sent `join <CODE>` from +923336102811 to Twilio sandbox
- Confirm Twilio credential is linked in workflow
- Confirm N8N_USER_WHATSAPP_NUMBER env var is set
- Check WhatsApp spam/business folder

**"Google Sheet not found"**
- Confirm N8N_GOOGLE_SHEET_ID is correct (copy from URL)
- Confirm Google Sheets credential is linked
- Confirm sheet names match exactly (Job Applications, Interview Tracker)

**"Gmail doesn't arrive"**
- Confirm Gmail credential is linked
- Confirm N8N_USER_EMAIL=aghaijalal17@gmail.com
- Check Gmail spam/promotions folder

**"Data Table error"**
- Confirm data table exists in n8n Data tab
- Confirm it's named exactly: `remote_job_finder_seen_jobs`
- Confirm all 6 columns exist

---

## 📞 SUPPORT

All documentation in:
- `/home/noorijlal/Documents/Agentic Workflows/n8n-builder/README.md`
- `/home/noorijlal/Documents/Agentic Workflows/n8n-builder/GMAIL_SHEETS_SETUP.md`
- `/home/noorijlal/Documents/Agentic Workflows/n8n-builder/workflows/README.md`

**Total setup time: 15 minutes**  
**You've got this! 🚀**

