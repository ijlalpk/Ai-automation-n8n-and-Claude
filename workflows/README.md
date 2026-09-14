# Workflows Reference

## 1. Remote Cloud/DevOps Job Finder

**File:** `remote-cloud-job-finder.json`

### Purpose

Discovers new remote cloud/DevOps entry-level job postings from three public job-board APIs (Remotive, RemoteOK, Arbeitnow), filters for relevant keywords, deduplicates, and sends **dual notifications**: WhatsApp individual alerts + Gmail digest summary.

### Trigger & Schedule

- **Type:** Cron schedule (every 6 hours)
- **Rationale:** Remotive's API guidance: ~4 requests/day max. Balances freshness with rate-limit respect.

### Data Sources

| Source | API | Remote Filtering |
|--------|---|---|
| **Remotive** | `https://remotive.com/api/remote-jobs` | Always remote |
| **RemoteOK** | `https://remoteok.com/api` | All remote (requires User-Agent header) |
| **Arbeitnow** | `https://arbeitnow.com/api/job-board-api` | Check `remote === true` |

### Workflow Stages

1. **Schedule Trigger** — Fires every 6 hours
2. **Fetch (×3, parallel)** — HTTP GET each API endpoint
3. **Normalize (×3, parallel)** — Transform to common shape
4. **Merge** — Combine all three lists
5. **Filter** — Keep only: remote + cloud/DevOps keywords + entry-level
6. **Data Table — Get** — Fetch all seen job IDs
7. **Code — Dedup** — Remove already-seen jobs
8. **Twilio — Send WhatsApp** — Individual alert per new job
9. **Gmail — Send Job Digest** — HTML email summary of all new jobs (parallel to Twilio)
10. **Data Table — Insert** — Mark job as seen

### Filter Keywords

**Include:** cloud engineer, associate cloud engineer, cloud support engineer, devops engineer, site reliability engineer, sre, aws, azure, gcp, kubernetes, docker, and more

**Exclude (seniority):** senior, staff, principal, lead, manager, director, architect, vp, chief

### Notifications

**WhatsApp (per-job):**
- Via Twilio Sandbox
- Individual message per new job: title + company + source + apply link
- Received on `N8N_USER_WHATSAPP_NUMBER`

**Gmail (digest):**
- HTML email summarizing ALL new jobs found in this run
- Subject: "🆕 New Remote Cloud/DevOps Jobs Found"
- Sent to `N8N_USER_EMAIL`
- Includes count and full job details with links

### Required Credentials

1. **Twilio API** — Account SID + Auth Token (https://www.twilio.com/console)
2. **Gmail OAuth2** — (see [GMAIL_SHEETS_SETUP.md](../GMAIL_SHEETS_SETUP.md))

### Required Environment Variables

- `N8N_USER_WHATSAPP_NUMBER` — WhatsApp number in E.164 format (e.g., `+15551234567`)
- `N8N_USER_EMAIL` — Email to receive job digest (e.g., `your.email@gmail.com`)

### Testing

- [ ] Twilio credential created and linked
- [ ] Gmail credential created and linked
- [ ] Both env vars (`N8N_USER_WHATSAPP_NUMBER`, `N8N_USER_EMAIL`) set
- [ ] Data Table `remote_job_finder_seen_jobs` exists
- [ ] Manually execute workflow; verify WhatsApp message arrives AND Gmail digest email arrives
- [ ] Seed the Data Table before activating (disable Twilio/Gmail, run once, re-enable)
- [ ] Run twice in a row; confirm zero duplicates on second run

---

## 2. Job Application Tracker

**File:** `job-application-tracker.json`

### Purpose

Logs job applications you've submitted to a **Google Sheet Excel file** for tracking and record-keeping. Sends a Gmail confirmation for each logged application. Can be triggered manually or via webhook.

### Trigger

- **Type:** Manual (Execute Workflow button) or Webhook (can be triggered from external tools)
- **Frequency:** Use manually each time you apply to a job, or bulk-log multiple applications

### Workflow Stages

1. **Trigger — Manual / Webhook** — User executes or external trigger
2. **Google Sheet — Append Job Application** — Adds a row to `Job Applications` sheet with all job details
3. **Gmail — Confirm Application Logged** — Sends confirmation email to user

### Logged Data

Appends a row to your Google Sheet with:
- `jobId` — Unique ID (from job-finder or manual)
- `title` — Job title
- `company` — Company name
- `source` — Job board source (Remotive / RemoteOK / Arbeitnow / Manual)
- `appliedAt` — Timestamp of when logged (auto-filled with current time)
- `status` — Application status (default: "Applied", can be updated manually to "Interviewed", "Rejected", etc.)
- `notes` — Personal notes (e.g., referral source, contact person, follow-up date)
- `url` — Direct link to job posting

### How to Use

**Via n8n UI (Manual):**
1. Open workflow in n8n
2. Click **Execute Workflow**
3. When prompted, provide the job details:
   ```json
   {
     "jobId": "remotive_12345",
     "title": "Cloud Engineer",
     "company": "TechCorp",
     "source": "Remotive",
     "status": "Applied",
     "notes": "Referred by John Smith",
     "url": "https://remotive.com/jobs/12345"
   }
   ```
4. Confirm application is logged in Google Sheet
5. Check email for confirmation message

**Via Webhook (Integration):**
1. Get the workflow's webhook URL (n8n shows it in the node)
2. Send a POST request with job details (JSON body)
3. Application is logged and confirmation email sent
4. Useful for integrating with external tools/buttons

### Required Credentials

1. **Google Sheets OAuth2** — (see [GMAIL_SHEETS_SETUP.md](../GMAIL_SHEETS_SETUP.md))
2. **Gmail OAuth2** — (see [GMAIL_SHEETS_SETUP.md](../GMAIL_SHEETS_SETUP.md))

### Required Environment Variables

- `N8N_GOOGLE_SHEET_ID` — Your Google Sheet ID (https://docs.google.com/spreadsheets/d/SHEET_ID_HERE/...)
- `N8N_USER_EMAIL` — Confirmation email recipient (e.g., `your.email@gmail.com`)

### Google Sheet Setup

1. Create a new Google Sheet named `Job Applications`
2. Add column headers in row 1: jobId, title, company, source, appliedAt, status, notes, url
3. Copy the Sheet ID from the URL
4. Set `N8N_GOOGLE_SHEET_ID` env var
5. (See [GMAIL_SHEETS_SETUP.md](../GMAIL_SHEETS_SETUP.md) for full instructions)

### Testing

- [ ] Google Sheets credential created and linked
- [ ] Gmail credential created and linked
- [ ] `N8N_GOOGLE_SHEET_ID` env var set correctly
- [ ] Google Sheet `Job Applications` exists with correct column headers
- [ ] Manually execute workflow with test job data
- [ ] Verify row appears in Google Sheet
- [ ] Verify confirmation email arrives in inbox
- [ ] Edit status in sheet manually (e.g., "Interviewed", "Offer", "Rejected") for tracking

---

## Integration: Auto-Log from Job Finder (Future Enhancement)

A future enhancement could automatically log jobs you apply to by:
1. Adding a button/flag in the job-finder WhatsApp message (e.g., "React ✅ to log this application")
2. Triggering the job-tracker workflow when you react
3. Auto-populating fields from the original job data

This would eliminate manual data entry and create a seamless apply → log workflow.

---

## Known Limitations & Gotchas

### Job Finder
1. **RemoteOK HTTP 403** — Requires User-Agent header (included in workflow)
2. **First-run flood** — Seed the Data Table before activating
3. **Gmail attachment limits** — Digest is HTML-only (no file attachments)
4. **Timezone** — Job posted dates are stored in UTC; your phone/email displays in local timezone

### Job Tracker
1. **Manual logging required** — Currently requires manual execution or webhook
2. **No deduplication** — Can log the same jobId multiple times; manage this manually
3. **Sheet column order** — If you rearrange columns in Google Sheet, update the workflow's column mapping
4. **Batch logging** — For bulk imports, use Google Sheet UI directly and manually append rows

---

## Future Enhancements

- **Auto-apply via ATS APIs** — Greenhouse, Lever, Workable, Ashby (Phase 2)
- **Salary filtering** — Filter by salary range thresholds
- **Per-company tracking** — Favorite/exclude specific companies
- **Interview reminders** — Calendar sync for scheduled interviews
- **Analytics dashboard** — Track application rate, response rate, offer rate
- **Webhook integration** — Button/automation to log applications from WhatsApp message

---

**Last updated:** 2026-09-14  
**Phase:** 1B (Discovery + Dual Notify + Application Tracking)  
**Status:** Ready for import and testing

---

## 3. Interview Tracker - Application Pipeline

**File:** `interview-tracker.json`

### Purpose

Tracks job applications through the complete interview pipeline from discovered job → application submitted → interview scheduled → interview completed → final outcome (offer/rejection). Sends Gmail notifications at each stage with reminders before scheduled interviews.

### Trigger

- **Type:** Manual (Execute Workflow) or Webhook
- **Frequency:** As needed, each time you have an interview update or need to log progress

### Application Status Pipeline

```
Discovered → Applied → Interview Scheduled → Interviewed → Offer/Rejected
```

### Workflow Stages

1. **Trigger — Manual / Webhook** — User provides application update
2. **Google Sheet — Read Interview Tracker** — Fetch existing tracker data
3. **Code — Find or Create Job Entry** — Check if job already tracked
4. **Google Sheet — Append Interview Entry** — Log/update application status
5. **Code — Check Interview Date** — Detect if interview scheduled
6. **Gmail — Notify Interview Scheduled** — Send alert with interview details
7. **Gmail — Interview Reminder (1 Day Before)** — Auto-reminder day before
8. **Gmail — Notify Interview Outcome** — Alert on final result (offer/rejection)

### Tracked Data

Logs to Google Sheet column with:
- `jobId` — Unique job identifier
- `title` — Job title
- `company` — Company name
- `source` — Where job was found (Remotive, etc.)
- `status` — Current stage (Applied, Interview Scheduled, Interviewed, etc.)
- `appliedDate` — When you applied (auto-filled)
- `interviewDate` — Scheduled interview date/time
- `interviewType` — Phone / Video / In-Person
- `interviewNotes` — Your notes (prep topics, contact info, etc.)
- `followUpDate` — When to follow up if no response
- `outcome` — Final result (Pending, Offer, Rejection)
- `offerDetails` — Offer terms if accepted
- `rejectionReason` — Why rejected (if provided)
- `lastUpdated` — Last timestamp updated

### How to Use

**When you get called for an interview:**
1. Open `interview-tracker` workflow in n8n
2. Click **Execute Workflow**
3. Provide update:
   ```json
   {
     "jobId": "remotive_12345",
     "title": "Cloud Engineer",
     "company": "TechCorp",
     "status": "Interview Scheduled",
     "interviewDate": "2026-09-25T14:00:00Z",
     "interviewType": "Video",
     "interviewNotes": "Be ready to discuss Kubernetes, prepare 3 questions about team structure"
   }
   ```
4. Workflow logs to Google Sheet
5. Gmail notifications sent: "Interview Scheduled" alert
6. Auto-reminder sent 1 day before interview

**After interview:**
1. Update with outcome:
   ```json
   {
     "jobId": "remotive_12345",
     "status": "Interviewed",
     "outcome": "Offer",
     "offerDetails": "Senior Cloud Engineer, $150k + benefits, start date Oct 1"
   }
   ```
2. Workflow logs outcome to sheet
3. Gmail notification sent with offer/rejection status

### Required Credentials

1. **Google Sheets OAuth2** — Read/write access to Interview Tracker sheet
2. **Gmail OAuth2** — Send notification emails

### Required Environment Variables

- `N8N_GOOGLE_SHEET_ID` — Your Google Sheet ID
- `N8N_USER_EMAIL` — Email to receive interview notifications
- `N8N_INTERVIEW_REMINDER_DAYS` — Days before interview to send reminder (default: 1)

### Required Google Sheet Setup

1. In same Google Sheet as Job Applications, create new sheet named `Interview Tracker`
2. Row 1 headers: jobId, title, company, source, status, appliedDate, interviewDate, interviewType, interviewNotes, followUpDate, outcome, offerDetails, rejectionReason, lastUpdated

### Testing

- [ ] Google Sheets credential linked
- [ ] Gmail credential linked
- [ ] Interview Tracker sheet exists with correct headers
- [ ] Manually execute with test interview data
- [ ] Verify row appears in Google Sheet
- [ ] Verify "Interview Scheduled" notification arrives in Gmail
- [ ] Update with final outcome (offer/rejection) and verify notification

### Known Limitations

1. **Reminders are manual** — Requires workflow execution. Future: add daily scheduled job to check for upcoming interviews and auto-send reminders
2. **Interview Type** — Currently free-text field; could be dropdown (Phone/Video/In-Person)
3. **Calendar integration** — Future: sync with Google Calendar API to auto-detect interview dates
4. **Multiple rounds** — Currently tracks one interview; multiple rounds would need enhanced schema

### Future Enhancements

- **Scheduled reminder job** — Daily workflow checks upcoming interviews, auto-sends reminders
- **Calendar sync** — Pull interview dates from Google Calendar, push outcomes back
- **Salary negotiation tracker** — Log offer details, counter-offer flow, final accepted package
- **Interview preparation** — Link to prep documents, study materials, company research
- **Analytics** — Dashboard: applications → interview rate, offer rate, time-to-offer
- **Follow-up automation** — Send follow-up emails after X days if no response


---

## 4. Job Finder - Expanded (8 Sources + Experience Filter)

**File:** `job-finder-expanded.json` (RECOMMENDED - replaces original job-finder)

### Purpose

Enhanced version of Job Finder with **8 job board APIs** + **configurable 1-3 year experience filtering**. Discovers remote cloud/DevOps roles and notifies via WhatsApp. Includes experience level matching for junior/mid-level candidates.

### Data Sources (8 APIs - All Free)

| Source | Experience Filter? | Coverage | Notes |
|--------|---|---|---|
| **Remotive** | Tags-based | 18-50+ jobs | Clean API, unlimited free |
| **RemoteOK** | Tags-based | 250+ jobs | European focus, large feed |
| **Arbeitnow** | Tags-based | 250+ jobs | Requires `remote=true` filter |
| **Himalayas** ⭐ | NATIVE! (seniority param) | 20-100+ jobs | **Best for experience filtering!** |
| **Jobicy** | Engineering category | 50-200+ jobs | Strong engineering focus |
| **The Muse** | Level field available | 1000+ (mixed) | Career development focused |
| **Working Nomads** | Parse needed | 100-150 jobs | Unofficial API access |
| **HNHIRING** | (Coming Soon) | 100-500+/month | Hacker News monthly jobs |

### Experience Level Filtering (NEW!)

**Environment Variable:** `N8N_EXPERIENCE_YEARS` (set to 1, 2, or 3)

```bash
# In your n8n .env:
N8N_EXPERIENCE_YEARS=2  # Will filter for 0-2 years experience roles
```

**Filter Logic:**
- Matches keywords: "entry level", "junior", "associate", "graduate", "0-2 years", "1-2 years"
- Excludes: "senior", "staff", "principal", "lead", "director", "architect"
- If no experience specified in job description: **INCLUDES IT** (assumes junior-friendly)
- Result: Only roles suitable for 1-3 year professionals

**Examples:**
- `N8N_EXPERIENCE_YEARS=1` → Finds "Entry-level Cloud Engineer" + "Graduate Program"
- `N8N_EXPERIENCE_YEARS=2` → Finds above + "Associate Cloud Engineer"
- `N8N_EXPERIENCE_YEARS=3` → Finds above + "Mid-level DevOps Engineer"

### Workflow Stages

1. **Schedule Trigger** — Every 6 hours
2. **Fetch (×8, parallel)** — HTTP GET from all 8 job board APIs
3. **Normalize (×8, parallel)** — Transform each API format to common schema
4. **Merge** — Combine all 8 normalized lists
5. **Filter** — Remote + Cloud/DevOps keywords + Experience level (via `N8N_EXPERIENCE_YEARS`)
6. **Data Table — Get** — Fetch all seen job IDs
7. **Code — Dedup** — Remove already-notified jobs
8. **Twilio — Send WhatsApp** — Individual alert per new job
9. **Data Table — Insert** — Mark job as seen

### Required Credentials

- **Twilio API** — Account SID + Auth Token

### Required Environment Variables

- `N8N_USER_WHATSAPP_NUMBER` — Your WhatsApp number (E.164 format)
- `N8N_EXPERIENCE_YEARS` — Target experience level (1, 2, or 3)

### Advantages Over Original Job Finder

| Feature | Original | Expanded |
|---------|----------|----------|
| Job sources | 3 APIs | **8 APIs** |
| Experience filtering | Manual keyword only | **Configurable (1-3 years)** |
| Himalayas (native seniority) | ✗ | **✓** |
| Jobicy (engineering category) | ✗ | **✓** |
| The Muse (level field) | ✗ | **✓** |
| Coverage | Limited | **Much larger** |
| Notification | WhatsApp only | WhatsApp only (same) |
| Setup time | 5 min | **5 min** (same) |

### Testing

- [ ] Set `N8N_EXPERIENCE_YEARS` env var (1, 2, or 3)
- [ ] Create Data Table `remote_job_finder_seen_jobs` if not already created
- [ ] Execute workflow manually
- [ ] Verify WhatsApp messages arrive with jobs matching experience level
- [ ] Check that "senior" roles are excluded, "junior" roles included
- [ ] Run second time to confirm dedup works (zero new jobs if no new postings)

### Known Limitations

1. **Working Nomads API** — Unofficial/undocumented; Apify scraper recommended as backup
2. **HNHIRING** — Monthly post (first business day of month); older posts still available
3. **The Muse** — Larger general job board; mostly non-remote, must filter remote=true strictly
4. **Experience matching** — Keyword-based; if job description doesn't mention years/level, it's included (false positive acceptable vs. false negative)

### Recommendation

**Use `job-finder-expanded.json` instead of `remote-cloud-job-finder.json`** — same cost (free), same schedule (6h), same notifications, but **8x more job sources** + **configurable experience filtering**.

