# n8n-builder: Quick-Start & Credential Setup

Welcome! This project helps you build high-quality n8n automation workflows with Claude's assistance. The first workflow is a **remote job-discovery assistant** that finds cloud/DevOps entry-level roles and notifies you via WhatsApp.

## Quick Start (5 minutes)

### 1. Import the Workflow

1. Download or copy `workflows/remote-cloud-job-finder.json` from this repo.
2. In your n8n instance, go to **Workflows** → **Create** → **Import from file** and upload the JSON.
3. Click **Save** (don't activate yet).

### 2. Set Up Twilio WhatsApp Sandbox (3 minutes)

This workflow uses Twilio's WhatsApp Sandbox to send job notifications. It's free, quick to set up, and requires no business verification.

**Steps:**
1. Go to https://www.twilio.com/console/sms/whatsapp/sandbox
2. Note the **Sandbox Number** (e.g., `+14155238886`) and your personal **Join Code** (e.g., `join xxxx-xxxx`).
3. Open WhatsApp on your phone and send `join <CODE>` to the sandbox number (you'll get a reply confirming the join).
4. In the Twilio Console Dashboard, copy your **Account SID** and **Auth Token**.

### 3. Create the Twilio Credential in n8n

1. In your n8n instance, go to **Credentials** (bottom left) → **Create** (or **+**) → search for and select **Twilio API**.
2. Paste your **Account SID** and **Auth Token** (from step 2.4 above).
3. Click **Create** and name it something clear like `Twilio - WhatsApp Sandbox`.

### 4. Set Up the Data Table for Deduplication

1. In your n8n instance, go to the **Data** tab (left sidebar).
2. Click **Create** → **Data Table** and name it `remote_job_finder_seen_jobs`.
3. Add these columns:
   - `jobId` (string) — unique job identifier from the API.
   - `source` (string) — which API found it (remotive / remoteok / arbeitnow).
   - `title` (string) — job title.
   - `company` (string) — company name.
   - `url` (string) — link to apply.
   - `firstSeenAt` (datetime) — when this job was first notified.

### 5. Configure Environment Variables

Add these to your n8n instance's environment or `.env` file:

```bash
N8N_USER_WHATSAPP_NUMBER=+1234567890  # Your own phone number in E.164 format (e.g. +15551234567)
```

(Replace `+1234567890` with your actual WhatsApp number in E.164 international format.)

### 6. Update the Workflow & Test

1. Open `remote-cloud-job-finder.json` in n8n.
2. On the **Twilio** node, check that:
   - The credential is set to your newly-created Twilio credential.
   - `From` is set to the sandbox number from step 2 (e.g., `whatsapp:+14155238886`).
   - `To` references `whatsapp:{{$env.N8N_USER_WHATSAPP_NUMBER}}`.
3. Execute the workflow manually (press **Execute Workflow** button at the top).
4. Check your WhatsApp — you should receive one or more job notifications.

### 7. Seed the Dedup Table (Avoid First-Run Flood)

The very first run will treat all currently-matching jobs as "new" and send a notification for each. To avoid a burst of messages, do a seeding run:

1. In the workflow, go to the **Twilio** node.
2. Temporarily **disable** it (right-click → Disable).
3. Execute the workflow; let it run to completion. This populates `remote_job_finder_seen_jobs` with all current jobs without sending WhatsApp messages.
4. Re-enable the **Twilio** node.
5. Execute once more to confirm it finds no new jobs (seeding worked).

### 8. Activate & Monitor

1. Click **Activate** (top right) to turn on the schedule.
2. The workflow will now run every 6 hours automatically.
3. Monitor runs via the **Executions** tab to confirm each run completes and messages arrive at the expected frequency.

---

## Troubleshooting

### "WhatsApp Sandbox: 403 Forbidden" or "Twilio returns invalid credentials"
- Confirm your **Account SID** and **Auth Token** were copied correctly from the Twilio Console.
- Confirm you sent the `join <CODE>` message to the sandbox number from your phone and got a confirmation reply.
- Confirm your phone number is in the correct format in `N8N_USER_WHATSAPP_NUMBER` (E.164 with leading `+`, e.g., `+15551234567`).

### "RemoteOK returns 403 (Forbidden) in the HTTP Request node"
- This is expected without a browser-like `User-Agent` header — the workflow already includes this header in the RemoteOK node. Confirm the node's **Headers** tab has:
  ```
  User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36
  ```
- If this header is missing, add it manually.

### "Workflow runs but finds no jobs / sends no messages"
- Execute each `Fetch -` node individually and inspect the output to confirm the APIs are returning data.
- Check the `Filter - Remote + Cloud/DevOps Keyword Match` node output — it may be filtering too aggressively. Review the keyword/seniority lists in the node's code and adjust if needed.
- Check `remote_job_finder_seen_jobs` Data Table — if all returned jobs already have rows in the table, they'll be filtered out as "already seen." This is expected after the first few runs.

### "Data Table node errors: 'remote_job_finder_seen_jobs not found'"
- Confirm the Data Table exists in n8n's Data tab and is named exactly `remote_job_finder_seen_jobs` (case-sensitive).
- If it doesn't exist, create it manually following steps in section 4 above.

### "Workflow fails with 'Continue On Fail' message"
- This is expected if one of the three job APIs is temporarily down or rate-limited. The workflow skips that source and continues with the other two. Check the execution log to see which source failed and why.

---

## Environment Variables Reference

| Variable | Purpose | Example | Where to Set |
|---|---|---|---|
| `N8N_USER_WHATSAPP_NUMBER` | Your personal WhatsApp number to receive job notifications | `+15551234567` | n8n `.env` or instance Environment Variables |
| `N8N_API_URL` | n8n instance API endpoint (needed once n8n-mcp is connected) | `http://localhost:5678/api` | System env or n8n config |
| `N8N_API_KEY` | n8n API authentication key (needed once n8n-mcp is connected) | `n8n_api_...` | System env or n8n config |

---

## Next Steps

- **Monitor runs** for a week; adjust keyword filters if you're getting too few or too many results.
- **Phase 2 preview:** Once your resume/profile is ready, the job-finder will extend to auto-fill and auto-submit applications via company career page APIs (Greenhouse, Lever, Workable, Ashby).
- **Feedback:** Open an issue or discussion in this project repo to suggest improvements, additional job sources, or changes to the filter logic.

---

## Credential Security Notes

- **Never commit** `Account SID`, `Auth Token`, phone numbers, or other secrets to this repo.
- Always store credentials in n8n's Credentials vault (encrypted at rest).
- Always reference credentials by ID in workflow JSON, not by their secret values.
- Environment variables that contain secrets should also not be committed; they belong in your n8n instance's `.env` or Environment Variables UI.

---

**Questions?** Refer to [CLAUDE.md](CLAUDE.md) for project conventions and architecture, or check the per-workflow README in [workflows/README.md](workflows/README.md).
