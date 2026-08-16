---
description: Run the BRD Analyst on a BRD file and produce docs/analysis/<slug>/analysis.md + backlog.json (no Jira writes).
agent: brd-analyst
---
Analyze the BRD at `${input:brdPath:docs/brds/<file>.md}` end-to-end following your procedure:

1. Read config (apply the `<<placeholder>>` config guard), rubric, template and every standards/instruction file
   (apply the placeholder guard).
2. Refresh `docs/CODEBASE_MAP.md` only if it is missing or its `<!-- generated: YYYY-MM-DD @ <sha> by brd-analyst -->`
   header line is missing or dated more than 30 days ago.
3. If the `salesforce` MCP tools are available, run the org guard (always `usernameOrAlias` =
   `config.salesforce.targetOrgAlias`; must be a sandbox) and use them read-only to confirm org-side config on
   affected objects; otherwise say "repo only".
4. Produce the full analysis file (incl. RTM §11 and the DoR evaluation §9) and backlog.json under
   `docs/analysis/<slug>/` (slug rule in your Inputs).
5. End with the summary block (estimate range, counts, DoR result, RTM gaps, top risks, open questions) and the
   publish command.

Do not create Jira issues. Do not write solution code. Do not run Apex tests or change the org.
