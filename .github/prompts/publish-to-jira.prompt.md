---
description: Publish an approved backlog.json to Jira (epics → stories → test cases). Asks for one confirmation first.
agent: jira-publisher
---
Publish `${input:backlogPath:docs/analysis/<slug>/backlog.json}` to Jira.

Pre-flight: verify `approved: true`, `approvedBy`, `approvedAt` and the DoR result, resolve issue types and custom
field IDs from the project metadata, check for existing issues with the `brd-<slug>` label to stay idempotent, then
show me the plan table and wait for my "yes" before creating anything. Write returned keys back into backlog.json
and update the analysis.md status line when done.
