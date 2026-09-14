# n8n-builder: Project Setup & Conventions

## Project Purpose

This project uses **Claude** together with the **n8n-mcp MCP server** (https://github.com/czlonkowski/n8n-mcp) and n8n-related Claude skills to design and build high-quality n8n automation workflows. The current focus is a personal remote job-discovery assistant that monitors cloud/DevOps entry-level roles and delivers new opportunities via WhatsApp notifications.

## MCP Tooling: n8n-mcp Server

Once connected (via `N8N_API_URL`, `N8N_API_KEY`, and `N8N_MCP_ACCESS_TOKEN` environment variables), this project gains access to the n8n-mcp server's complete tool suite:

### Discovery & Validation Tools
- `search_nodes` — Full-text search across 2,755 n8n nodes (832 core + 1,923 community) with community-node filtering.
- `get_node` — Retrieve node info (docs, properties, version history) in multiple formats.
- `validate_node` — Syntax/schema validation for individual nodes.
- `validate_workflow` — Full workflow validation against n8n's execution model + AI Agent rules.
- `search_templates` — Find workflow templates by keyword, node type, task, or metadata.
- `get_template` — Retrieve workflow templates as JSON or normalized export.

### Live n8n Instance Management Tools (21 total)
- Workflow operations: `n8n_create_workflow`, update, delete, list, retrieve, version history, deploy templates.
- Node resources: `n8n_explore_node_resources` (dynamic dropdowns using live credentials).
- Execution: test workflows, list/get/delete execution logs.
- Data tables: row operations (Insert, Get, Update, Delete, Upsert).
- Credentials: list, retrieve, create, update.
- Security: `n8n_audit_instance` for compliance scanning.
- Agents: manage persisted AI assistants (n8n 2.34+).
- System: health checks, catalog listing.

### Build Workflow: Until MCP is Connected

Until the n8n-mcp server is connected to this project, workflows are authored as exportable JSON files under `workflows/` and documented in `workflows/README.md`. The standard import/deployment process:

1. Export workflow JSON from this repo.
2. Import into your n8n instance via the UI or n8n CLI.
3. Populate required credentials and environment variables (documented per workflow).
4. Execute test runs; validate output at each stage.
5. Activate the workflow's schedule or trigger.

Once n8n-mcp is connected, the build loop becomes: `search_nodes` → `validate_node` → `validate_workflow` → `n8n_create_workflow` / `n8n_update_workflow` (using Claude to author and validate in real-time).

## Claude Skills for n8n Automation

This project has access to **n8n-skills** — a suite of 14 Claude skills (https://github.com/czlonkowski/n8n-skills) that teach best practices for building n8n workflows. These skills are auto-activated when relevant:

### Core Skills (Used in Most Workflows)

1. **n8n Expression Syntax** — Correct `{{}}` expression patterns, `$json`/`$node` variable access, and debugging expression errors.
2. **n8n MCP Tools Expert** (HIGHEST PRIORITY) — Using the n8n-mcp server tools: `search_nodes`, `validate_node`, `validate_workflow`, `search_templates`, `get_template`.
3. **n8n Workflow Patterns** — 5 proven architectural patterns for building scalable automation flows.
4. **n8n Node Configuration** — Operation-aware guidance for configuring node properties and dependencies.
5. **n8n Code JavaScript** — Writing effective JavaScript in n8n Code nodes, using `this.helpers`, date handling.
6. **n8n Error Handling** — Designing failure paths: per-node error outputs, retries, structured responses.

### Specialized Skills (Use as Needed)

7. **n8n Validation Expert** — Interpreting and fixing workflow validation errors.
8. **n8n Code Python** — Writing Python code in n8n Code nodes (sandbox-aware).
9. **n8n Code Tool** — Building reusable Code Tools callable by n8n AI Agents (distinct from Code nodes).
10. **n8n Binary & Data** — Handling files, images, PDFs, and binary data; managing `$binary` vs `$json`.
11. **n8n Sub-workflows** — Extracting shared logic into reusable sub-workflows with typed inputs.
12. **n8n AI Agents** — Designing AI agents, LLM chains, tool calling, and structured output.
13. **n8n Multi-Instance** — Managing workflows across multiple n8n instances/environments.
14. **n8n Self-Hosting** — Deploying production n8n to Linux VMs (Docker Compose + Caddy).

**How they're used:** When you ask me to build or fix a workflow, these skills load automatically to provide targeted guidance. For example, if you're writing JavaScript in a Code node, skill #5 will provide context. If you need to validate your workflow, skill #7 will guide you.

---

## Workflow-Building Conventions

### Naming
- **Workflows:** `<Domain> - <Purpose>` (e.g., "Job Finder - Remote Cloud Discovery").
- **Nodes:** `<Stage> - <Detail>` (e.g., "Fetch - Remotive", "Normalize - RemoteOK", "Filter - Remote + Cloud/DevOps Keyword Match").
- **Canvas documentation:** Every workflow includes a top-left Sticky Note (non-executable) with: purpose one-liner, schedule/trigger type, data sources, dedup mechanism, and any external dependencies (credentials, environment variables, data tables).

### Error Handling
- Every workflow must set an **Error Workflow** in Settings (even if it's just a simple "Error Notifier" that logs or sends a message to the user).
- HTTP Request nodes that depend on external APIs should set **`Continue On Fail = true`** to prevent a single source failure from aborting the entire workflow; document the reasoning in a node comment.
- Any node with external side effects (API calls, message sends, database writes) should have explicit error handling downstream or be documented as part of the error workflow.
- Silent failures are not acceptable: if something fails that matters to the user, they must be notified.

### Credentials & Secrets
- **Never hardcode** API keys, tokens, passwords, or personal identifiers (phone numbers, emails) in node parameters that get committed to this repo.
- Use **n8n Credentials** for all secrets (Twilio tokens, API keys, database passwords, etc.). Reference them by credential ID; the actual values live in your n8n instance.
- Use **environment variables** for personal-but-not-secret config (e.g., user's own phone number, which is personally identifying but not a credential). Prefix with `N8N_` for clarity. Reference as `{{$env.N8N_VARIABLE_NAME}}` in workflow nodes.
- Every required credential and environment variable must be documented in the workflow's README entry and in the project-level `README.md`.

### Testing Before Activation
- Always run a manual **Execute Workflow** test pass before toggling a workflow to Active.
- If the workflow has multiple stages, test each stage in isolation (using n8n's "Execute step" feature) to spot errors early.
- For workflows that send external messages (WhatsApp, email, Slack, etc.), test with the send node **temporarily disconnected or mocked** (e.g., a Code node that returns the message content without sending it), verify the payload looks correct, then re-enable and test end-to-end.
- For workflows with side effects (database writes, application submissions), run at least one full dry-run before activation to ensure the output format/behavior matches expectations.

### Documentation
- Every workflow gets a description in a canvas **Sticky Note** (the one mentioned above under Naming).
- Every workflow also gets an entry in `workflows/README.md` with: purpose, trigger/schedule, sources, dedup mechanism (if applicable), required credentials, required environment variables, and any gotchas or known limitations.

## Repository Layout

```
n8n-builder/
├── CLAUDE.md                              # This file — project conventions and MCP reference
├── README.md                              # Credential setup, env-var walkthrough, quick-start
├── .claude/
│   └── settings.local.json                # Per-user local settings (permissions, hooks)
└── workflows/
    ├── README.md                          # Per-workflow documentation and index
    ├── remote-cloud-job-finder.json       # Phase 1: job discovery + WhatsApp notify
    ├── error-notifier.json                # Companion error workflow for job-finder
    └── [future workflows TBD]
```

## Project Roadmap

### Phase 1: Discovery + Notify (Current — In Progress)

**Workflow:** `remote-cloud-job-finder.json`

Discovers new remote cloud/DevOps entry-level job postings from public job-board APIs (Remotive, RemoteOK, Arbeitnow), filters for keywords and seniority, deduplicates, and sends a WhatsApp notification for each new match.

- **Sources:** Remotive, RemoteOK, Arbeitnow APIs (public, free, no authentication required).
- **Schedule:** Every 6 hours (constrained by Remotive's published rate guidance: ~4 requests/day).
- **Notify channel:** WhatsApp (Twilio Sandbox for Phase 1; Meta Cloud API as a future upgrade option).
- **Dedup storage:** n8n Data Table `remote_job_finder_seen_jobs` (zero external credential, native persistence).
- **Filtering:** Remote roles only; keywords: cloud engineer, associate cloud engineer, cloud support engineer, devops engineer, site reliability engineer, cloud infrastructure engineer, cloud engineer intern (case-insensitive); exclude senior/staff/principal/lead/manager titles.

### Phase 2: Auto-Apply via ATS APIs (Future — Not Started)

Once the user's resume/profile is ready, extend the workflow to **auto-fill and submit applications** on company career pages built with Greenhouse, Lever, Workable, or Ashby (platforms with legitimate public application-submission APIs).

- **Explicit exclusion:** LinkedIn and Indeed will **not** have auto-submit automation (no public apply API; automating them violates their ToS and risks account bans).
- **Resume sourcing:** TBD — likely a Google Drive PDF or a stored profile JSON.
- **Workflow state:** track which jobs have been auto-applied, handle rate-limiting per company.

## Contributing / Next Steps

To add a new workflow to this project:

1. **Design phase:** Sketch the workflow on paper or in a tool like miro/excalidraw; identify data sources, transformation logic, and external actions (API calls, messages, database writes).
2. **Build:** Use n8n's UI to construct the workflow, follow the naming and error-handling conventions above, add a descriptive Sticky Note on the canvas.
3. **Test:** Execute at each stage, verify outputs, test error conditions. Create a test-only version of any external-action node (mock send, mock write) for a dry-run.
4. **Export:** Export the workflow as JSON from n8n.
5. **Document:** Save to `workflows/<name>.json`, add an entry to `workflows/README.md` with purpose, schedule, sources, credentials, env-vars, and gotchas.
6. **Commit & PR** (once git is enabled in this project): include the workflow JSON + README updates.

---

**Last updated:** 2026-09-14  
**Current focus:** Phase 1 job-finder launch and Twilio WhatsApp setup documentation.
