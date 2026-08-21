# BRD → Backlog agent for GitHub Copilot (Salesforce · Jira MCP · Salesforce MCP)

Merge the **`.github/`, `.vscode/` and `docs/` folders** into the root of the Salesforce repo (keep this README too
if you like — e.g. as `docs/brd-agent/README.md` — so the runbook is versioned with the config). GitHub Copilot users
then get two custom agents:

| Agent | What it does | Writes to |
|---|---|---|
| **`brd-analyst`** | Reads a BRD, scans the codebase (+ optionally the sandbox org via Salesforce MCP), builds an impact map, estimates with the team rubric **including the cost of complying with your Apex/Flow review instructions**, drafts epics → stories → acceptance criteria → test cases, fills a requirement traceability matrix and evaluates Definition-of-Ready | `docs/analysis/<slug>/analysis.md` + `backlog.json` — **never Jira, never the org** |
| **`jira-publisher`** | After a human approves the analysis, creates the Epics/Stories (with story points + AC) and test cases in Jira, idempotently, after one "yes" | Jira, via the Atlassian MCP (VS Code only) |

The human review between the two agents is deliberate — it is the "Requirement readiness sign-off / human-in-the-loop"
control from Gate 1 of the Agentic Engineering Framework, and it stops the model from spraying wrong tickets into Jira.

```
BRD ──▶ @brd-analyst ──▶ analysis.md + backlog.json ──▶ 👤 review/edit, set approved ──▶ /publish-to-jira ──▶ Jira
              │  reads: repo, CODEBASE_MAP, instruction files, rubric, (sandbox org), (Jira history)
```

## Related: Story Gap Analyst (separate repo)
The `story-gap-analyst` agent — auditing existing Jira stories against a requirement (5-column gap table with
completeness percentages) — lives in its own repo with its samples and examples:
**https://github.com/deepanshuchauhan96/story-gap-analyst**. This repo keeps the BRD→backlog agents only.

## What's in here
```
.github/agents/brd-analyst.agent.md          the analyst agent (read-only toward Jira/org, no shell, no web)
.github/agents/jira-publisher.agent.md       the publisher agent (explicit Jira create/edit/link allow-list, target: vscode)
.github/prompts/analyze-brd.prompt.md        /analyze-brd brdPath=<brd path>
.github/prompts/publish-to-jira.prompt.md    /publish-to-jira backlogPath=<backlog.json>
.github/brd-agent/config.json                Jira keys, field IDs, test-case strategy, standards file list, rubric knobs
.github/brd-agent/estimation-rubric.md       component baselines × complexity + standards tax + DoD tax + risk buffer
.github/brd-agent/analysis-template.md       output structure (RTM, DoR evaluation) + backlog.json schema
.github/instructions/*.instructions.md       PLACEHOLDERS — replace with your real Apex / Flow review instructions
.vscode/mcp.json                             Atlassian (Rovo) MCP + Salesforce DX MCP (sandbox-locked, read-oriented)
docs/analysis/README.md                      reviewer instructions: how to approve backlog.json (HITL gate, CODEOWNERS)
docs/brds/SAMPLE-BRD-loan-payoff-quote.md    a sample BRD to test the flow
```

## Prerequisites (each developer)
- **VS Code 1.107 or later** (1.130+ if your enterprise uses the MCP allow-list below). Copilot Chat is bundled with
  VS Code; sign in via the status-bar Copilot icon with your Copilot Business account.
- Open the **repo root** as the workspace folder and choose **Trust** when prompted — `.github/agents`,
  `.github/prompts` and `.vscode/mcp.json` are resolved from the workspace root, and Restricted Mode disables agents.
- **Verify** after setup: type `/` in the chat input → `analyze-brd` and `publish-to-jira` are listed, and the agents
  dropdown shows **brd-analyst** and **jira-publisher**. If not, right-click in the Chat view → **Diagnostics** to see
  the loaded agents/prompts and any load errors.

## Setup (one time, ~45 min, in this order)
1. **GitHub Copilot admin (Business/Enterprise only)** — the org/enterprise policy **MCP servers in Copilot** must be
   *Enabled* (already true if the Jira MCP works for you today). If your admin restricts MCP to registry servers or
   uses the enterprise **MCP allow-list** (`allowedMcpServers` / policy `ChatAllowedMcpServers`, VS Code ≥ 1.130),
   both servers in `.vscode/mcp.json` must be allow-listed. Simplest is by name — `{"serverName": "atlassian"}` and
   `{"serverName": "salesforce"}` (the keys this scaffold mandates in step 3) — or `{"serverUrl":
   "https://mcp.atlassian.com/*"}` for Atlassian. A `serverCommand` entry for the local Salesforce server only
   matches the **exact, complete** `[command, ...args]` array from `mcp.json`:
   `["npx","-y","@salesforce/mcp@0.30.15","--orgs","dev-sandbox","--toolsets","data,scale-products","--no-telemetry"]`
   — a shortened entry such as `["npx","@salesforce/mcp"]` does not match and the server is blocked; if you change
   the alias, version or toolsets, update the allow-list too. The Salesforce server is not a registry install, so it
   is also blocked when `chat.mcp.access` (policy `ChatMCP`) is `registry` instead of `all`. Repo-level custom agents
   in `.github/agents/` need no separate policy; agent mode just must not be disabled by the `chat.agent.enabled` policy.
2. **Atlassian admin (Rovo MCP server)** — Atlassian Administration → Rovo → *Rovo MCP server* → **Permissions**:
   Jira **Read**, **Write** and **Search** allowed (Write blocked ⇒ analyst works, publisher fails). A site admin
   completes the first OAuth consent once (the app installs just-in-time). **Domain settings** (same page): the
   default *Allow Atlassian supported domains* list lets every listed AI product (claude.ai, chatgpt.com, Cursor,
   Perplexity, Manus, …) OAuth into your Jira as any user — not just VS Code. Least privilege for a bank: deselect
   *Allow Atlassian supported domains* and *Add domain* only VS Code's redirect URLs: `http://127.0.0.1:*/**` and
   `https://vscode.dev/redirect` (plus `https://insiders.vscode.dev/redirect` if anyone uses Insiders). If your
   tenant already has the list off, add those entries or the consent screen fails with "Your organization admin must
   authorize access from this redirect URL". Leave **Authentication → API token** off (the default): `.vscode/mcp.json`
   connects with OAuth, and API-token connections bypass the domain allowlist entirely. These are tenant-wide
   Atlassian settings, not specific to this scaffold — agree them with your Atlassian admin.
3. **Get the files into the repo** — merge this branch, or copy `.github/`, `.vscode/` and `docs/` into the repo
   root. Note they are dot-folders: hidden in Finder/Explorer by default (`Cmd+Shift+.` on macOS, *Show ▸ Hidden
   items* on Windows), so copy from a terminal rather than dragging. Verify with `ls -a` at the repo root — both
   `.github` and `.vscode` must be listed. If the repo already has
   `.vscode/mcp.json`, **merge** the `atlassian` and `salesforce` entries into its `servers` map (VS Code reads one
   file per workspace); keep the server keys exactly `atlassian` / `salesforce` — the agents' tool allow-lists
   reference them. Do not overwrite an existing `.github/copilot-instructions.md` (this scaffold does not ship one).
4. **Point at your real review instructions** — move your Apex/Flow review instruction files into
   `.github/instructions/` (keep an `applyTo:` glob in the frontmatter) or list their paths in
   `config.json → standardsFiles`. Files still containing `PLACEHOLDER` make the analyst fall back to rubric defaults
   and cap confidence at M — you'll see the warning in chat.
5. **Salesforce** — on each analyst's machine (Node.js **20+**, `sf` CLI):
   `sf org login web -a dev-sandbox --instance-url https://<MyDomain>--<sandbox>.sandbox.my.salesforce.com`
   (the sandbox's My Domain URL; `https://test.salesforce.com` also works unless the sandbox sets *Setup ▸ My Domain ▸
   Login Policy ▸ Prevent login from https://test.salesforce.com*). The user needs the *Approve Uninstalled Connected
   Apps* permission, or a Salesforce admin must pre-authorise the Salesforce CLI connected app. Verify with
   `sf org display -o dev-sandbox` — *Instance Url* must show the `.sandbox.my.salesforce.com` domain. No
   `--set-default` is needed: the analyst passes the alias as `usernameOrAlias` on every call.
   > **The org alias must match in three places.** `dev-sandbox` is a placeholder name — use whatever you like, but
   > the same string has to appear in all three, or the analyst silently falls back to "repo only":
   > &nbsp;&nbsp;1. `.vscode/mcp.json` → `"--orgs", "<alias>"`
   > &nbsp;&nbsp;2. `.github/brd-agent/config.json` → `"targetOrgAlias": "<alias>"`
   > &nbsp;&nbsp;3. the login command → `sf org login web -a <alias> …`
   > If they drift apart the MCP server answers "No org found with the provided username/alias", the analyst writes
   > **repo only** in the analysis header and continues without org context — safe, but easy to miss.

   `.vscode/mcp.json` starts `@salesforce/mcp@0.30.15` locked to `--orgs dev-sandbox` with toolsets `data` (`run_soql_query`) and `scale-products`
   (`scan_apex_class_for_antipatterns`), telemetry off. `deploy_metadata`, `retrieve_metadata`, `run_apex_test`,
   `assign_permission_set` and scratch-org tools are intentionally not enabled there, and the analyst's frontmatter
   allow-list names only `salesforce/get_username`, `salesforce/run_soql_query` and
   `salesforce/scan_apex_class_for_antipatterns` — so widening `--toolsets` later (for other Copilot agents that
   share this `mcp.json`) grants the analyst nothing new. It additionally checks `Organization.IsSandbox` before
   using any org tool. Optional: add `code-analysis` to `--toolsets` for a PMD baseline (needs JDK 11+) and then add
   `salesforce/run_code_analyzer` and `salesforce/query_code_analyzer_results` to the analyst's `tools` list.
   - **Machine pre-flight (once, in a terminal).** VS Code must see the same `node`/`npx` your terminal does — having
     `sf` does not guarantee `npx` is on your PATH; install Node 20+ LTS system-wide (Windows: not only via nvm-windows
     in a shell profile; macOS/Linux nvm/fnm users: launch VS Code with `code .` from that shell). Then pre-download and
     smoke-test the server:
     `npx -y @salesforce/mcp@0.30.15 --orgs dev-sandbox --toolsets data,scale-products --no-telemetry` — the first run
     installs the package (minutes on a slow link); when it prints that the MCP server is running on stdio and waits,
     press Ctrl+C. Behind a corporate proxy / private registry: `npm config set registry <mirror>` and
     `npm config set https-proxy <proxy>` first (the mirror must carry `@salesforce/mcp@0.30.15`) and start VS Code from a
     shell that has the same proxy variables. **Windows:** if *MCP: List Servers ▸ salesforce ▸ Show Output* shows
     `spawn npx ENOENT` / `spawn node ENOENT`, VS Code could not launch `npx.cmd` — in your local `.vscode/mcp.json` set
     `"command": "cmd", "args": ["/c", "npx", "-y", "@salesforce/mcp@0.30.15", "--orgs", "dev-sandbox",
     "--toolsets", "data,scale-products", "--no-telemetry"]` (keep the key `salesforce`; don't commit it in a mixed
     macOS/Windows team; an enterprise MCP allow-list must then also match this command).
6. **Start the servers** — VS Code: *MCP: List Servers* → start `atlassian` (browser OAuth to your site — approve the
   Rovo MCP consent once) and `salesforce` (accept the trust prompt). Never point `atlassian` at `/v1/mcp/preview` —
   different tool surface.
7. **Fill `config.json`** — open the Chat view (Session Target **Local** — see *Where to run* under Run) → agents
   dropdown → **brd-analyst** and ask:
   *"Using cloudId https://<site>.atlassian.net, call getJiraProjectIssueTypesMetadata for project <KEY> and
   getJiraIssueTypeMetaWithFields for the Story type; tell me the issue type names and the story-points field id."*
   Then replace every `<<placeholder>>` (site, projectKey, storyPoints field, calibrationJql), set the exact
   issue-type names (team-managed projects say "Subtask", company-managed "Sub-task") and confirm `testCaseStrategy`
   (default `subtask`). Both agents refuse to call tools while a `<<…>>` value remains.
   - `testCaseStrategy`: `subtask` (default, works everywhere) · `issue` (Xray or Zephyr Squad "Test" issue type
     linked with `createIssueLink`; **Zephyr Scale is not supported** — its tests aren't Jira issues) ·
     `description` (test-case table in the story). The Atlassian MCP has **no attachment upload**, so "attach test
     cases" is implemented as linked issues/sub-tasks — better for traceability anyway. (`createIssueLink` shipped in
     March 2026 — atlassian-mcp-server #45 — but is not yet spelled out on the supported-tools page; the publisher
     falls back to `subtask` automatically if it is missing.)

## Run
**Where to run:** in the main VS Code window open the Chat view (⌃⌘I / Ctrl+Alt+I), select **New Chat (+)** and, if
a *Session Target* control is shown, choose **Local**. Do not use the *Agents window* or the Copilot / cloud targets
for this workflow: sessions on the Agent Host ignore `.github/prompts/*.prompt.md` (so `/analyze-brd` and
`/publish-to-jira` are not offered), and cloud sessions cannot use the OAuth `atlassian` server, so the agents would
have no Jira tools. Select the agent in the dropdown **before** running its prompt; approve tool calls when VS Code
asks (or *Allow in this session*); if it pauses with "Continue to iterate?", click Continue; when it finishes, click
**Keep** on the pending file edits.

**BRD format.** `brdPath` must point at a text file (`.md` preferred, `.txt` works). The analyst's `read` tool reads
workspace files as text and it has no shell, so a `.docx` or `.pdf` path yields no usable content. Real BRDs arrive
as Word/PDF: export or copy the text into `docs/brds/<name>.md` (Word: *File ▸ Save As ▸ Plain Text* or paste; PDF:
copy/export the text), keep the section numbering (§2.1, FR-n …) — the requirements register and RTM cite it — and
commit that `.md`: `backlog.json → brd` records it as the BRD of record. A pasted BRD also works: the analyst saves it
to `docs/brds/<slug>.md` first.

```
/analyze-brd brdPath=docs/brds/SAMPLE-BRD-loan-payoff-quote.md
```
The analysis folder is the BRD file name lower-cased with non-alphanumerics collapsed to `-`, so the sample lands in
`docs/analysis/sample-brd-loan-payoff-quote/`. Success = the agent's closing summary (estimate range, counts, DoR
result, RTM gaps) and both files present. Review its `analysis.md` (estimate, RTM §11, DoR §9), edit anything, then in
`backlog.json` set `"approved": true`, `"approvedBy": "<name>"`, `"approvedAt": "<ISO date>"` — ideally via a PR on
`docs/analysis/**` with CODEOWNERS so approval is a git record (see `docs/analysis/README.md`). Then switch to
**jira-publisher**:
```
/publish-to-jira backlogPath=docs/analysis/sample-brd-loan-payoff-quote/backlog.json
```
The publisher pre-flights permissions/field IDs, shows a plan table (with existing keys), waits for "yes", writes
sequentially, verifies, writes keys back into `backlog.json` (re-runs update instead of duplicating, keyed by the
`brd-<slug>` label) and stamps the analysis status line. Success = the final table of Jira keys.

## Data flows & controls (for security review)
- **What leaves the machine:** BRD text, repo excerpts the analyst reads, SOQL *metadata* results (object/field/flow/
  class names, counts, coverage numbers — never member records) → the GitHub Copilot model endpoint under your
  Copilot Business data terms; Jira issue text ↔ Atlassian Rovo MCP (`mcp.atlassian.com`, OAuth 2.1, your site only).
  Nothing goes to any other URL: the analyst has no web tool and no shell; telemetry of the Salesforce server is off.
- **Salesforce:** local `sf` CLI auth to the sandbox alias only (`--orgs dev-sandbox`); tools registered are
  `run_soql_query` and `scan_apex_class_for_antipatterns` (+ core `get_username`); no deploy/retrieve/test/user tools;
  the analyst verifies `Organization.IsSandbox` before any other org call and selects no member data (COUNT()/metadata
  objects only, `LIMIT` always).
- **Jira:** analyst = read tools only; publisher = create/edit/link only (no transition/comment/worklog/delete), scoped
  to `projectKey`, label `brd-<slug>`, one explicit "yes" per batch, every write recorded back into `backlog.json`,
  Source line (analysis path @ commit, approver, date) in every epic.
- **Audit trail:** `docs/analysis/<slug>/` in git (PR + CODEOWNERS recommended), `approvedBy/approvedAt` in
  `backlog.json`, Jira keys written back, analysis status line stamped on publish.
- **Secrets:** none in this scaffold; OAuth tokens live in VS Code's secret storage, `sf` auth in the CLI keychain.

## Design decisions worth knowing
- **Rubric-driven estimation.** LLM estimates without a rubric drift badly. Baselines + complexity multiplier +
  itemized standards tax (from *your* instruction files; the % values are only a sanity floor) + DoD tax + role
  hours + risk buffer, calibrated against recent Done stories in Jira. Every number cites a rubric line, so a tech
  lead can argue with a specific line, not a vibe.
- **`docs/CODEBASE_MAP.md`.** Large SFDX repos exceed context. The analyst maintains a ~400-line map, date/commit-
  stamped on line 1, and refreshes it when the stamp is older than 30 days (it has no file-timestamp tool) — commit it.
- **Repo vs org.** Git is truth for custom code; the org is truth for managed packages, org-only config, coverage
  and data volumes. Read-only SOQL (incl. Tooling API for `InstalledSubscriberPackage` / `ApexCodeCoverageAggregate` /
  `ValidationRule`) closes that gap; the server is locked to the sandbox alias.
- **Separation of duties.** Analyst: read-only Jira tools, no shell, no web. Publisher: only the Jira metadata/create/
  edit/link tools it needs (no transition/comment/worklog/Confluence), no Salesforce tools, `target: vscode`. Tool
  allow-lists in the agent frontmatter enforce it, not just prose.
- **Traceability.** Requirement IDs flow BRD → story → test → Jira description; the RTM and the evaluated DoR are the
  Gate 1 evidence artifacts; the publisher refuses a failed DoR unless an override reason is recorded.
- **Instruction files do double duty.** Copilot auto-applies `.github/instructions/*.instructions.md` when *editing*
  matching files; the analyst additionally *reads* them to price compliance into estimates.

## Phase 2 (optional): run the analyst in the cloud Copilot coding agent
`brd-analyst.agent.md` is not targeted, so it also works with the Copilot coding agent on GitHub.com (assign an issue
containing the BRD, get a PR with `docs/analysis/...`). MCP servers for the cloud agent are configured in the repo's
Copilot → coding agent settings (or the agent's `mcp-servers:` frontmatter) with secrets as `COPILOT_MCP_*`
environment variables; for Salesforce you'd store an `SFDX_AUTH_URL` for a **read-only, sandbox-only** integration
user (never a personal or production credential) and run `sf org login sfdx-url` in `copilot-setup-steps.yml` — treat
that as a separate InfoSec review. `jira-publisher` stays `target: vscode` on purpose — its "yes" gate needs an
interactive session. Do the VS Code version first — the interactive HITL loop is the point.

## References
- GitHub custom agents config: https://docs.github.com/en/copilot/reference/custom-agents-configuration
- Creating custom agents (repo `.github/agents/`): https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/create-custom-agents
- VS Code custom agents / prompt files / MCP servers: https://code.visualstudio.com/docs/copilot/customization/custom-agents
- Salesforce DX MCP Server (`@salesforce/mcp` 0.30.15, Node ≥ 20): https://www.npmjs.com/package/@salesforce/mcp
- Atlassian Rovo MCP Server tools: https://support.atlassian.com/atlassian-rovo-mcp-server/docs/supported-tools/
- Atlassian MCP repo: https://github.com/atlassian/atlassian-mcp-server
