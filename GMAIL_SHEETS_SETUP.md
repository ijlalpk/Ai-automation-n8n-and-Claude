# Gmail & Google Sheets Setup for n8n Workflows

This guide walks you through setting up Gmail and Google Sheets credentials for the job-finder and job-tracker workflows.

## Part 1: Gmail Setup (for notifications + application confirmations)

### 1. Enable Gmail API in Google Cloud

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project (or select an existing one)
3. Search for **"Gmail API"** and click **Enable**
4. Go to **Credentials** (left sidebar) → **+ Create Credentials** → **OAuth 2.0 Client IDs**
5. Choose **Desktop application** as the application type
6. Click **Create**
7. Download the credentials as JSON (you'll need this later)

### 2. Create Gmail Credential in n8n

1. In your n8n instance, go to **Credentials** (bottom left) → **+** → Search for **Gmail**
2. Select **Gmail OAuth2**
3. You'll see a prompt to authenticate. Click the link to authorize n8n to access your Gmail account
4. Follow the Google login flow and grant permission to n8n
5. Once authorized, click **Create** and name it `Gmail - Job Notifications`

### 3. Configure Environment Variable

Add to your n8n `.env` file:

```bash
N8N_USER_EMAIL=your-email@gmail.com  # The email address that will receive job notifications
```

---

## Part 2: Google Sheets Setup (for job application tracking)

### 1. Enable Google Sheets API in Google Cloud

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Make sure you're in the same project as above
3. Search for **"Google Sheets API"** and click **Enable**
4. (No additional credentials needed — you'll use the same OAuth token as Gmail)

### 2. Create Your Job Applications Google Sheet

1. Go to [Google Sheets](https://sheets.google.com)
2. Create a new blank spreadsheet
3. Name it `Job Applications` (exactly)
4. In **Sheet1**, create these column headers (row 1):
   - A: `jobId`
   - B: `title`
   - C: `company`
   - D: `source`
   - E: `appliedAt`
   - F: `status`
   - G: `notes`
   - H: `url`

5. Copy the **Sheet ID** from the URL (the long alphanumeric string between `/d/` and `/edit`)
   - Example: `https://docs.google.com/spreadsheets/d/SHEET_ID_HERE/edit#gid=0`

### 3. Create Google Sheets Credential in n8n

1. In your n8n instance, go to **Credentials** → **+** → Search for **Google Sheets**
2. Select **Google Sheets OAuth2**
3. Click to authorize (this will use your existing Google Cloud OAuth token)
4. Once authorized, click **Create** and name it `Google Sheets - Job Tracker`

### 4. Configure Environment Variable

Add to your n8n `.env` file:

```bash
N8N_GOOGLE_SHEET_ID=your-sheet-id-here  # Copy from step 2.5 above
```

---

## Part 3: Update Workflows with Credentials

### For `remote-cloud-job-finder.json` (Gmail digest + WhatsApp)

1. Open the workflow in n8n
2. Find the **Gmail - Send Job Digest** node
3. In the **Credentials** dropdown, select `Gmail - Job Notifications`
4. Verify `N8N_USER_EMAIL` environment variable is set (step 1.3)

### For `job-application-tracker.json` (Log applications to Excel)

1. Open the workflow in n8n
2. Find the **Google Sheet - Append Job Application** node
3. In the **Credentials** dropdown, select `Google Sheets - Job Tracker`
4. Verify `N8N_GOOGLE_SHEET_ID` environment variable is set (step 2.4)
5. Find the **Gmail - Confirm Application Logged** node
6. In the **Credentials** dropdown, select `Gmail - Job Notifications`

---

## Part 4: Testing

### Test Gmail Digest (in remote-cloud-job-finder.json)

1. Open `remote-cloud-job-finder.json` in n8n
2. Temporarily **disable** the Twilio WhatsApp node
3. Execute the workflow manually
4. Check your Gmail inbox for the job digest email
5. Re-enable Twilio when done

### Test Job Application Logger (in job-application-tracker.json)

1. Open `job-application-tracker.json` in n8n
2. Click **Execute Workflow**
3. When prompted, fill in a test job entry:
   ```json
   {
     "jobId": "test_001",
     "title": "Test Cloud Engineer",
     "company": "Test Company",
     "source": "Remotive",
     "status": "Applied",
     "url": "https://example.com/job/123"
   }
   ```
4. Check your Google Sheet — a new row should appear
5. Check your Gmail inbox for the confirmation email

---

## Troubleshooting

### "Gmail returns 401 Unauthorized"
- Your Gmail OAuth token may have expired
- Go to **Credentials** in n8n, find `Gmail - Job Notifications`, and re-authorize

### "Google Sheets API: Spreadsheet not found"
- Verify `N8N_GOOGLE_SHEET_ID` env var is set correctly (no extra spaces/quotes)
- Confirm the Google Sheet exists and is named `Job Applications`
- Confirm the sheet has the correct column headers in row 1

### "Gmail sends but no email arrives"
- Check that `N8N_USER_EMAIL` is correct and matches your Gmail account
- Check Gmail spam/promotions folder
- Verify the Gmail credential has permission to send emails

### "Error: Invalid email in N8N_USER_EMAIL"
- Format must be a valid email address (e.g., `your.email@gmail.com`)
- No quotes or extra spaces

---

## Security Notes

- **Never commit** your `N8N_GOOGLE_SHEET_ID` or `N8N_USER_EMAIL` to version control if they're sensitive
- Store these as environment variables in your n8n instance (`.env` file or Environment Variables UI)
- OAuth tokens are stored encrypted in n8n's database; never share them
- The workflows reference credentials by ID only, not by actual secret values

---

**Ready to test?** Run through Part 4 above, then activate the workflows!
