# n8n-builder: Project Setup & Conventions

## Project Purpose

This project uses **Claude** together with the **n8n-mcp MCP server** (https://github.com/czlonkowski/n8n-mcp) and **n8n-skills** (https://github.com/czlonkowski/n8n-skills) to design and build high-quality n8n automation workflows for personal job-search automation.

**Current status:** Phase 1B (Discovery + Dual Notifications + Application Tracking) is complete and ready to deploy.

## MCP Tooling: n8n-mcp Server

Once connected (via `N8N_API_URL`, `N8N_API_KEY`, and `N8N_MCP_ACCESS_TOKEN` environment variables), this project gains access to the n8n-mcp server's complete tool suite:

### Discovery & Validation Tools
- `search_nodes` — Search across 2,755 n8n nodes (832 core + 1,923 community)
- `get_node` — Retrieve node info (docs, properties, versions)
- `validate_node` — Syntax/schema validation for individual nodes
- `validate_workflow` — Full workflow validation + AI Agent rules
- `search_templates` — Find workflow templates by keyword/metadata
- `get_template` — Retrieve workflow templates as JSON

### Live n8n Instance Management Tools (21 total)
- Workflow operations: create, update, delete, list, retrieve, version history, deploy templates
- Node resources: `n8n_explore_node_resources` for dynamic dropdowns
- Execution: test workflows, list/get/delete execution logs
- Data tables: row operations (Insert, Get, Update, Delete, Upsert)
- Credentials: list, retrieve, create, update
- Security: `n8n_audit_instance` for compliance scanning
- Agents: manage persisted AI assistants (n8n 2.34+)
- System: health checks, catalog listing

## Claude Skills for n8n Automation

This project has access to **n8n-skills** — a suite of 14 Claude skills (https://github.com/czlonkowski/n8n-skills) that teach best practices for building n8n workflows. These skills are auto-activated when relevant:

### Core Skills (Used in Most Workflows)

1. **n8n Expression Syntax** — Correct `{{}}` expression patterns, `$json`/`$node` variable access, debugging
2. **n8n MCP Tools Expert** (HIGHEST PRIORITY) — Using n8n-mcp server tools: search_nodes, validate_node, validate_workflow
3. **n8n Workflow Patterns** — 5 proven architectural patterns for scalable automation
4. **n8n Node Configuration** — Operation-aware guidance for node properties
5. **n8n Code JavaScript** — Writing effective JavaScript in n8n Code nodes
6. **n8n Error Handling** — Designing failure paths: error outputs, retries, structured responses

### Specialized Skills (Use as Needed)

7. **n8n Validation Expert** — Interpreting and fixing validation errors
8. **n8n Code Python** — Python in Code nodes (sandbox-aware)
9. **n8n Code Tool** — Building reusable Code Tools for AI Agents
10. **n8n Binary & Data** — Handling files, images, PDFs, and binary data
11. **n8n Sub-workflows** — Extracting shared logic into reusable sub-workflows
12. **n8n AI Agents** — Designing AI agents, LLM chains, tool calling
13. **n8n Multi-Instance** — Managing workflows across multiple n8n instances
14. **n8n Self-Hosting** — Deploying production n8n to Linux VMs (Docker Compose + Caddy)

## Workflow-Building Conventions

### Naming
- **Workflows:** `<Domain> - <Purpose>` (e.g., "Job Finder - Remote Cloud Discovery")
- **Nodes:** `<Stage> - <Detail>` (e.g., "Fetch - Remotive", "Filter - Remote + Cloud/DevOps Keyword Match")
- **Canvas documentation:** Every workflow includes a top-left Sticky Note with: purpose one-liner, schedule/trigger, data sources, dedup mechanism, external dependencies

### Error Handling
- Every workflow must set an **Error Workflow** in Settings
- HTTP Request nodes accessing external APIs should set **`Continue On Fail = true`** with clear documentation
- Any node with external side effects must have explicit error handling downstream
- Silent failures are not acceptable

### Credentials & Secrets
- **Never hardcode** API keys, tokens, passwords, or personal identifiers in committed JSON
- Use **n8n Credentials** for all secrets (referenced by credential ID only)
- Use **environment variables** for personal-but-not-secret config (e.g., phone numbers, emails); prefix with `N8N_`
- Reference env vars as `{{$env.N8N_VARIABLE_NAME}}` in workflow nodes

### Testing Before Activation
- Always run a manual **Execute Workflow** test before activating
- Test each stage in isolation using n8n's "Execute step" feature
- For message-sending workflows, test with the send nodes **temporarily disabled** first
- For side-effect workflows (database writes, applications), do a full dry-run before activation

### Documentation
- Every workflow gets a Sticky Note (mentioned above under Naming)
- Every workflow gets an entry in `workflows/README.md` with: purpose, trigger/schedule, sources, notifications, dedup mechanism, credentials, env-vars, testing checklist, gotchas

## Repository Layout

```
n8n-builder/
├── CLAUDE.md                              # This file — conventions & MCP reference
├── README.md                              # Quick-start guide (15 min setup)
├── GMAIL_SHEETS_SETUP.md                  # Gmail + Google Sheets credential setup
├── .claude/
│   └── settings.local.json                # Per-user local settings
└── workflows/
    ├── README.md                          # Per-workflow documentation
    ├── remote-cloud-job-finder.json       # Phase 1B: discovery + WhatsApp + Gmail
    └── job-application-tracker.json       # Phase 1B: log applications to Excel
```

## Current Workflows (Phase 1B - Deployed)

### 1. Job Finder - Remote Cloud Discovery
**File:** `workflows/remote-cloud-job-finder.json`

Discovers new remote cloud/DevOps entry-level roles from Remotive, RemoteOK, Arbeitnow APIs every 6 hours. Sends dual notifications:
- **WhatsApp:** Individual alert per new job via Twilio Sandbox
- **Gmail:** Daily digest HTML email with all new jobs

**Key features:**
- Sources: Remotive, RemoteOK, Arbeitnow (public, free APIs)
- Schedule: Every 6 hours (respects Remotive rate guidance of ~4 req/day)
- Filter: Remote + cloud/DevOps keywords + entry-level seniority
- Dedup: n8n Data Table (no external storage needed)
- Notifications: Parallel sends to WhatsApp + Gmail (both required, either can fail independently)

**Setup:** See README.md "Quick Setup" steps 1-5, 7

### 2. Job Application Tracker
**File:** `workflows/job-application-tracker.json`

Logs job applications you've submitted to a Google Sheet for centralized tracking. Sends confirmation emails. Manually triggered when you apply to a job.

**Key features:**
- Logs to: Google Sheet `Job Applications`
- Tracked fields: jobId, title, company, source, appliedAt, status, notes, url
- Confirmation: Gmail email for each logged application
- Trigger: Manual (click Execute Workflow) or via Webhook

**Setup:** See README.md "Quick Setup" steps 2-4, 6; then use manually per workflow

---

## Project Roadmap

### Phase 1B: Discovery + Dual Notify + Application Tracking ✅ COMPLETE

Current deployed state:
- Remote cloud job discovery every 6 hours
- WhatsApp alerts (Twilio Sandbox)
- Gmail digest emails
- Excel/Google Sheets application tracking
- Manual application logging

### Phase 2: Auto-Apply via ATS APIs (Future — Not Started)

Once resume/profile is ready:
- Auto-fill + auto-submit applications on Greenhouse, Lever, Workable, Ashby
- Track submission status and responses
- Integrate with job-finder for end-to-end automation

**Explicit exclusion:**
- LinkedIn and Indeed will NOT have auto-submit automation (no public API; ToS risk; user's own decision)

---

## Contributing / Extending

To add or modify a workflow:

1. **Design phase:** Sketch the workflow on paper; identify data sources, transformations, external actions
2. **Build:** Use n8n UI, follow naming conventions, add canvas Sticky Note with purpose/dependencies
3. **Test:** Execute at each stage; test error conditions; use mock sends for external actions first
4. **Export:** Export as JSON from n8n → save to `workflows/<name>.json`
5. **Document:** Add entry to `workflows/README.md` with purpose, trigger, sources, credentials, env-vars, testing checklist
6. **Commit & PR:** Include workflow JSON + README updates (once git is enabled)

---

## Security & Privacy Notes

- **Credentials:** Stored encrypted in n8n's database; never hardcoded or committed
- **Environment variables:** Stored in n8n's `.env` or Environment Variables UI; never committed
- **Workflows:** Reference credentials by ID and env vars by name only
- **Personal data:** Phone numbers, emails, and API tokens never appear in committed JSON files
- **Rate limits:** Respect API rate limits (e.g., Remotive's ~4 req/day guidance built into 6-hour schedule)

---

**Last updated:** 2026-09-14  
**Current phase:** 1B (Discovery + Dual Notifications + Application Tracking)  
**Status:** Ready for deployment and testing
