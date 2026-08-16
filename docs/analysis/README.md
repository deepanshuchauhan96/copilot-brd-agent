# docs/analysis — BRD analyses produced by the `brd-analyst` agent

Each BRD gets its own folder: `docs/analysis/<slug>/analysis.md` (human-readable) + `backlog.json` (machine-readable,
consumed by `jira-publisher`). These files are the Gate 1 evidence: requirements register, impact map, rubric-cited
estimates, stories with acceptance criteria, test cases, RTM and the Definition-of-Ready evaluation.

## Approval (human-in-the-loop gate)
1. Review `analysis.md` — especially §4 impact map, §7 estimate roll-up, §9 DoR result, §11 RTM gaps.
2. Edit anything that is wrong (both files, or `analysis.md` and ask the analyst to regenerate `backlog.json`).
3. In `backlog.json` set `"approved": true`, `"approvedBy": "<name>"`, `"approvedAt": "<YYYY-MM-DD>"`.
   If DoR failed and you still want to publish, also set `"dorOverrideReason"`.
4. Commit via a pull request so approval is a git record. Suggested `CODEOWNERS` entry (replace the `<<…>>` slugs
   with your GitHub org and real team slugs):
   ```
   docs/analysis/**   @<<github-org>>/<<salesforce-leads-team>> @<<github-org>>/<<salesforce-ba-team>>
   ```
   Both teams need explicit **write** access to the repo — GitHub assigns no owner to an unknown/under-privileged
   team and then any write-access reviewer can merge. Verify with `gh api repos/<<github-org>>/<<repo>>/codeowners/errors`
   (or open the CODEOWNERS file on github.com — errors are highlighted) and enable branch protection / a ruleset with
   *Require a pull request before merging* → *Require review from Code Owners*.
5. Run `/publish-to-jira backlogPath=docs/analysis/<slug>/backlog.json` in the VS Code Chat view (Session Target
   **Local**) with the `jira-publisher` agent selected.
   It writes the created Jira keys back into `backlog.json` and stamps the status line in `analysis.md`; commit that too.

Never hand-edit `jiraKey` values — they are what makes re-publishing idempotent.
