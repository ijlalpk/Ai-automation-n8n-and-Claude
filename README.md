# n8n-builder: Quick-Start & Credential Setup

Welcome! This project helps you build high-quality n8n automation workflows with Claude's assistance.

**Current workflows:**
- **Phase 1B:** Remote job discovery with dual notifications (WhatsApp + Gmail) + job application tracking to Excel

## What's Included (Phase 1B)

### 1. Job Finder - Remote Cloud Discovery
- 📱 Discovers remote cloud/DevOps entry-level roles every 6 hours
- 📲 Sends individual WhatsApp alerts per job (Twilio Sandbox)
- 📧 Sends daily Gmail digest with all new jobs
- 💾 Tracks seen jobs to prevent duplicates
- 🔍 Filters by keywords and entry-level seniority

### 2. Job Application Tracker
- 📊 Logs applications to Google Sheet (Excel)
- 📝 Auto-generates confirmation emails for each logged application
- 🔗 Maintains permanent record of where you've applied
- 🎯 Tracks application status (Applied → Interviewed → Offer → etc.)

---

## Quick Setup (15 minutes)

### Step 1: Set Up Twilio WhatsApp Sandbox (3 min)

1. Go to https://www.twilio.com/console/sms/whatsapp/sandbox
2. Note the **Sandbox Number** (e.g., `+14155238886`)
3. Send `join <CODE>` from your phone's WhatsApp to sandbox number
4. Copy **Account SID** and **Auth Token**
5. In n8n: **Credentials** → **+** → **Twilio API** → paste SID/Token
6. Name it `Twilio - WhatsApp Sandbox`

### Step 2: Set Up Gmail (4 min)

See [GMAIL_SHEETS_SETUP.md](./GMAIL_SHEETS_SETUP.md) **Part 1** for:
- Enabling Gmail API in Google Cloud
- Creating Gmail credential in n8n
- Setting `N8N_USER_EMAIL` environment variable

### Step 3: Set Up Google Sheets (5 min)

See [GMAIL_SHEETS_SETUP.md](./GMAIL_SHEETS_SETUP.md) **Part 2** for:
- Creating `Job Applications` Google Sheet
- Setting up Google Sheets credential
- Setting `N8N_GOOGLE_SHEET_ID` environment variable

### Step 4: Configure Environment Variables

Add to your n8n `.env` file or Environment Variables UI:

```bash
N8N_USER_WHATSAPP_NUMBER=+1234567890     # Your WhatsApp number (E.164 format)
N8N_USER_EMAIL=your.email@gmail.com      # Your email for job digest + confirmations
N8N_GOOGLE_SHEET_ID=ABC123XYZ...         # ID from your Job Applications sheet URL
```

### Step 5: Create n8n Data Table for Dedup

1. In n8n, go to **Data** tab (left sidebar)
2. Click **Create** → **Data Table**
3. Name: `remote_job_finder_seen_jobs`
4. Add columns:
   - jobId (string)
   - source (string)
   - title (string)
   - company (string)
   - url (string)
   - firstSeenAt (datetime)

### Step 6: Import Workflows

1. Download both workflow JSON files:
   - `workflows/remote-cloud-job-finder.json`
   - `workflows/job-application-tracker.json`

2. In n8n: **Workflows** → **Create** → **Import from file** → upload each

3. For each workflow:
   - Verify credentials are linked (Twilio, Gmail, Google Sheets)
   - Verify environment variables are set
   - Manually execute once to test
   - Check WhatsApp, Gmail, and Google Sheet for results

### Step 7: Activate & Monitor

1. **Job Finder:** Click **Activate** to turn on 6-hour schedule
2. **Job Tracker:** Keep **Inactive** (trigger manually when you apply to jobs)
3. Monitor executions via **Executions** tab

---

## Workflows at a Glance

### remote-cloud-job-finder.json
| Aspect | Details |
|--------|---------|
| **Trigger** | Every 6 hours |
| **Sources** | Remotive, RemoteOK, Arbeitnow APIs |
| **Notifications** | WhatsApp (per-job) + Gmail (digest) |
| **Dedup** | n8n Data Table |
| **Status** | Ready to activate |

### job-application-tracker.json
| Aspect | Details |
|--------|---------|
| **Trigger** | Manual (click Execute) or Webhook |
| **Destination** | Google Sheet `Job Applications` |
| **Confirmation** | Gmail email per logged application |
| **Status** | Ready to use (trigger manually) |

---

## Troubleshooting

### "WhatsApp/Gmail don't arrive"
- Verify credentials are linked in each workflow node
- Check that env vars are set: `N8N_USER_WHATSAPP_NUMBER`, `N8N_USER_EMAIL`
- Check spam/promotions folder in Gmail
- Check WhatsApp didn't go to a different chat

### "Google Sheet not found"
- Verify `N8N_GOOGLE_SHEET_ID` is set correctly (no quotes/spaces)
- Confirm Google Sheet exists and is named `Job Applications` (case-sensitive)
- Confirm column headers in row 1: jobId, title, company, source, appliedAt, status, notes, url

### "RemoteOK returns 403"
- The User-Agent header is already in the workflow; should work
- If it fails, the workflow will skip that source and continue with other two

### "First run: too many WhatsApp messages"
- Expected! Seed the Data Table before activating (see workflows/README.md)
- Disable Twilio + Gmail nodes, run workflow once, then re-enable

---

## How to Use Job Tracker

**When you apply to a job:**

1. Get the job details (from job-finder WhatsApp, or manually from a job board)
2. Open `job-application-tracker` workflow in n8n
3. Click **Execute Workflow**
4. Fill in the job details (jobId, title, company, etc.)
5. Workflow logs to Google Sheet + sends confirmation email

**Tracking applications:**
- View/edit your Google Sheet to mark status: "Applied" → "Interviewed" → "Offer" → "Rejected"
- Keep notes column updated (e.g., "Follow up on 9/20", "Referral from John")

---

## Environment Variables Reference

| Variable | Purpose | Example | Required For |
|---|---|---|---|
| `N8N_USER_WHATSAPP_NUMBER` | Your WhatsApp number to receive job alerts | `+15551234567` | Job Finder |
| `N8N_USER_EMAIL` | Your email for job digest + confirmations | `your.email@gmail.com` | Job Finder + Job Tracker |
| `N8N_GOOGLE_SHEET_ID` | Google Sheet ID for application tracking | `1a2b3c4d5e...` | Job Tracker |

---

## Credential Security

- **Never commit** credentials, API keys, or phone numbers to version control
- Store credentials in n8n's **Credentials** vault (encrypted)
- Store env vars in n8n's **.env** file (local) or **Environment Variables** UI (cloud)
- Workflows reference credentials by ID only, never by secret values

---

## Full Documentation

- **CLAUDE.md** — Project conventions, MCP tooling, n8n-skills reference
- **GMAIL_SHEETS_SETUP.md** — Step-by-step Gmail + Google Sheets credential setup
- **workflows/README.md** — Detailed workflow documentation, testing, troubleshooting

---

## Next Steps (Phase 2 - Future)

- **Auto-apply via ATS APIs** — Automatically fill and submit applications on Greenhouse, Lever, Workable, Ashby
- **(NOT planned)** LinkedIn/Indeed auto-submit — ToS risk; manually apply only

---

**Questions?** Check the relevant documentation file above.

**Ready to launch?** Follow Quick Setup steps 1-7, then activate the Job Finder workflow!
