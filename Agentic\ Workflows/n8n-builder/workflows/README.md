# Workflows Reference

## Remote Cloud/DevOps Job Finder

**File:** `remote-cloud-job-finder.json`

### Purpose

Discovers new remote cloud/DevOps entry-level job postings from three public job-board APIs (Remotive, RemoteOK, Arbeitnow), filters for relevant keywords and seniority levels, deduplicates against a persistent store, and sends a WhatsApp notification for each new matching job.

### Trigger & Schedule

- **Type:** Cron schedule (n8n Schedule/Cron trigger)
- **Interval:** Every 6 hours
- **Rationale:** Remotive's published API guidance recommends a maximum of ~4 requests per day (24h / 4 = 6h). This schedule respects that rate limit while providing reasonable freshness for job discovery.
- **Offset:** Scheduled at minute 15 of each 6-hour interval (avoids on-the-hour thundering-herd issues).

### Data Sources

| Source | API Endpoint | Format | Remote Filtering | Notes |
|--------|---|---|---|---|
| **Remotive** | `https://remotive.com/api/remote-jobs` | JSON object with `.jobs[]` array | Always remote by definition | Rate-limit guidance embedded in response headers. Most reliable source. |
| **RemoteOK** | `https://remoteok.com/api` | JSON array; element `[0]` is legal notice (skip) | `true` (all remote) | Returns HTTP 403 without a browser `User-Agent` header; workflow includes this. |
| **Arbeitnow** | `https://arbeitnow.com/api/job-board-api` | JSON object with `.data[]` array | Field `remote` must be checked explicitly (`true` = remote) | Mixes remote and non-remote; filtering is mandatory. |

### Workflow Stages

1. **Schedule Trigger** — Fires every 6 hours.
2. **Fetch (×3, parallel)** — HTTP GET each API endpoint with appropriate headers (User-Agent for RemoteOK).
3. **Normalize (×3, parallel)** — Transform each API's response format into a common shape: `{jobId, source, title, company, url, description, isRemote, postedAt}`.
4. **Merge** — Combine all three normalized job lists into one.
5. **Filter** — Keep only jobs where:
   - `isRemote === true`
   - Title or description contains at least one cloud/DevOps entry-level keyword (see keyword list below)
   - Title does NOT contain any senior/manager/director titles
6. **Data Table — Get** — Fetch all previously-seen job IDs from the dedup store.
7. **Code — Dedup** — Remove any jobs already in the seen store (prevent duplicate notifications).
8. **Twilio — Send WhatsApp** — Send a WhatsApp message via Twilio Sandbox to the user's personal number for each new job.
9. **Data Table — Insert** — Mark each notified job as seen in the dedup store.

### Filter Keywords

**Include List (must match at least one):**
- Cloud engineer, Associate cloud engineer, Cloud support engineer
- DevOps engineer, Site reliability engineer, Cloud infrastructure engineer
- Cloud engineer intern, Entry level, Junior, Associate, Junior cloud, Junior DevOps
- SRE, Associate DevOps, Associate infrastructure
- AWS, Azure, GCP, Kubernetes, Docker

**Exclude List (seniority filter — must NOT match any):**
- Senior, Staff, Principal, Lead, Manager, Director, Head of, Chief, VP, Vice president, Architect

(Keywords are matched case-insensitively against title + description.)

### Deduplication & Storage

**Store:** n8n built-in Data Table named `remote_job_finder_seen_jobs`

**Columns:**
- `jobId` (string) — Unique identifier in format `<source>_<nativeId>`, e.g., `remotive_12345`, `remoteok_67890`, `arbeitnow_slug-name`.
- `source` (string) — Job board name (Remotive / RemoteOK / Arbeitnow).
- `title` (string) — Job title as posted.
- `company` (string) — Company name.
- `url` (string) — Direct link to the job posting.
- `firstSeenAt` (datetime) — ISO timestamp when this job was first notified to the user.

**Why n8n Data Table?**
- Zero external credentials/OAuth — no dependency on Google Sheets API tokens expiring, Airtable API rate limits, or file system access.
- Native to n8n; survives instance restarts and backups.
- Sufficient capacity (200 MiB per instance) for millions of job records.
- Simple row-level operations (Insert, Get, Upsert) map directly to the dedup logic.

### Required Credentials

**Twilio API**

- **n8n Credential Type:** `Twilio API`
- **Fields needed:**
  - Account SID (from https://www.twilio.com/console)
  - Auth Token (from https://www.twilio.com/console)
- **Credential Name (example):** `Twilio - WhatsApp Sandbox`
- **Setup:** See the main [README.md](../README.md) section "Set Up Twilio WhatsApp Sandbox" for step-by-step instructions.

### Required Environment Variables

| Variable | Purpose | Example | Where to Set |
|---|---|---|---|
| `N8N_USER_WHATSAPP_NUMBER` | User's own WhatsApp number to receive notifications | `+15551234567` | n8n `.env` or instance Environment Variables UI |

**Format:** E.164 international format with leading `+`, e.g., `+1` (USA), `+44` (UK), `+91` (India).

### Error Handling

- **HTTP Request nodes** (`Fetch -` nodes) have `Continue On Fail = true`, so a single API outage doesn't abort the entire workflow. If one source fails, the other two still execute and send notifications.
- **Twilio node** — if the send fails, that job is **not** marked seen, so it will retry on the next run.
- **Workflow-level error workflow:** Currently not set. For production, connect a companion `error-notifier.json` workflow that WhatsApps the user a failure message.

### Testing Checklist

Before activating the schedule:

- [ ] Twilio credential is created and linked to the workflow.
- [ ] `N8N_USER_WHATSAPP_NUMBER` environment variable is set correctly in n8n.
- [ ] Data Table `remote_job_finder_seen_jobs` exists with the columns listed above.
- [ ] Execute each `Fetch -` node individually; confirm 200 status and expected JSON shape. (RemoteOK: confirm 200, not 403, with the User-Agent header.)
- [ ] Execute each `Normalize -` node; spot-check output for correct jobId, title, company, url, isRemote.
- [ ] Execute the `Filter` node; confirm plausible output (a few dozen jobs max, no empty result unless truly no matching jobs posted recently).
- [ ] Execute `Data Table — Get` and `Code — Dedup`; confirm dedup logic works (run twice with no new postings between runs; second run should yield zero new jobs).
- [ ] Execute the full workflow manually once; check that at least one WhatsApp message arrives with correct title/company/link.
- [ ] **Seeding step (critical):** Disable the `Twilio` node, run the full workflow once to populate the Data Table, then re-enable the node and run again to confirm zero duplicate notifications.
- [ ] Temporarily break one API URL (e.g., typo in Arbeitnow URL) and run again; confirm the workflow completes and still sends notifications from the other two sources.

### Known Limitations & Gotchas

1. **RemoteOK HTTP 403 without User-Agent:** RemoteOK blocks requests that don't include a browser-like `User-Agent` header. The workflow includes this header; if it ever stops working, this is the first thing to check.

2. **First-run notification flood:** If you import this workflow and run it without first seeding the Data Table, you'll receive a WhatsApp message for every currently-matching job (could be 50+ messages). Always follow the seeding step in the main [README.md](../README.md) before activating the schedule.

3. **Keyword filter too strict/loose:** The keyword lists are a starting point. If you're getting too few job matches, loosen the include list or remove exclusions from the exclude list. If too many, tighten the keywords. Edit the `Filter - Remote + Cloud/DevOps Keyword Match` node's JavaScript code to adjust.

4. **Job board API downtime:** If any of the three APIs goes down for an extended period, that source won't contribute jobs to that run. The other two will still work (`Continue On Fail`). No manual intervention needed; the workflow will resume once the API is back up.

5. **Timezone in `postedAt`:** The workflow stores job posting timestamps in ISO format (UTC). Displayed timestamps in the WhatsApp message will be in the user's phone's timezone (automatic).

### Future Enhancements

- **Phase 2:** Extend with auto-fill + auto-submit functionality for company career pages running on Greenhouse, Lever, Workable, or Ashby (ATS platforms with public job-submission APIs).
- **Email digest:** Instead of (or in addition to) WhatsApp, send a daily email summary of all new jobs discovered.
- **Per-company filtering:** Allow the user to flag favorite companies or exclude certain companies entirely.
- **Salary filtering:** Add salary range thresholds to the filter logic.
- **LinkedIn/Indeed monitoring:** Add read-only tracking of these platforms (discover new postings but don't auto-submit, per user's ToS concerns). Requires alternative scraping/monitoring approaches (not covered in Phase 1).

---

**Last updated:** 2026-09-14  
**Phase:** 1 (Discovery + Notify)  
**Status:** Ready for import and testing
