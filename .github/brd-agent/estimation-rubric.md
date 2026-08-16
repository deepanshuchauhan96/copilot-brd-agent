# Estimation rubric (Salesforce delivery)

The agent MUST estimate with this rubric, not from intuition. Every number in `analysis.md` cites a line here.
All baselines are **starting points** — recalibrate quarterly against Jira actuals (see `calibrationJql`).

## 1. Component baselines (hours, "medium" complexity, one component)

| Component | New | Modify existing | Notes |
|---|---|---|---|
| Custom object + fields + page layout | 6 | 3 | + 1h per 5 fields beyond 10 |
| Field-level security / permission set / profile change | 2 | 1 | per perm set |
| Validation rule / formula field | 1.5 | 1 | |
| Record-triggered Flow (single object, ≤ 3 decision branches) | 8 | 4 | + 2h per additional branch/subflow |
| Screen Flow (≤ 4 screens) | 12 | 6 | + 2h per screen |
| Scheduled / Autolaunched Flow | 6 | 3 | |
| Apex trigger + handler (new object) | 12 | — | trigger framework assumed |
| Apex service / domain class (≤ 300 LOC) | 10 | 5 | |
| Apex Batch / Queueable / Schedulable | 12 | 6 | |
| Apex REST / Callout integration (one endpoint) | 16 | 8 | + Named Credential 2h |
| Platform Event / CDC subscriber | 8 | 4 | |
| LWC (simple form/list) | 12 | 6 | + Apex controller from above |
| LWC (complex: datatable, wizard, LDS/wire, events) | 24 | 12 | |
| Aura component (modify only; new Aura is discouraged) | — | 8 | |
| OmniStudio (OmniScript / FlexCard / Integration Procedure) | 12 | 6 | per artifact |
| Report / Dashboard | 2 | 1 | |
| Approval process / Assignment rule / Email template | 4 | 2 | |
| Data migration / backfill script (one object) | 8 | — | + 4h per dependent object |
| Experience Cloud page/config | 8 | 4 | |
| Managed-package config (e.g., nCino, FSC, DocuSign, MuleSoft connector) | 6 | 3 | + vendor dependency risk |
| Marketing Cloud / Data Cloud / Einstein–Agentforce configuration | 8 | 4 | treat as Complex unless a pattern exists |

**Role of a row.** These rows are **Admin/Config** hours (not Dev; they do not drive the QA/Review percentages):
Field-level security / permission set / profile change; Validation rule / formula field; Report / Dashboard; Approval
process / Assignment rule / Email template; Experience Cloud page/config; Managed-package config; Marketing Cloud /
Data Cloud / Einstein–Agentforce configuration. Every other row is **Dev** hours. A component type not listed here:
pick the closest row, say so in the rationale, and set confidence ≤ M.

## 2. Complexity multiplier (per component)
- **Simple** ×0.6 — isolated, no existing automation on the object, no integration, well-known pattern.
- **Medium** ×1.0 — touches an object with existing automation; moderate branching; standard security.
- **Complex** ×1.6 — order-of-execution / recursion concerns, large data volume (>1M rows), cross-object rollups,
  async chaining, external system dependency, sharing/territory implications, PII/compliance handling.
- **Very complex** ×2.2 — anything requiring architectural change, new integration pattern, or a managed-package upgrade.

## 3. Standards-compliance line items (mandatory — this is the "code review instructions" tax)
Read the instruction files listed in `config.json → standardsFiles` and derive concrete tasks. **Precedence:** the
itemized tasks derived from the team's real instruction files are authoritative; `standardsOverheadPct` is only a
sanity floor — flag the story if the itemized total is below the percentage or above 2× it. If the instruction
files are placeholders/missing, use the typical items below, label the story "STANDARDS TAX = RUBRIC DEFAULTS" and
cap confidence at M.

**Bases.** `Dev_component` = Σ(§1 baseline × §2 multiplier) over the story's Dev rows, all technologies together
(§9: 6.4h + 5h = 11.4h). The apex / flow / lwc floors are each `pct × Dev_component`, and their itemized line items are
Dev hours: **story Dev hours = `Dev_component` + Σ standards line items** (§9: 11.4h + 8.5h = 19.9h → 20h). The
config floor is `pct ×` the story's Admin/Config row hours and its items are booked as Admin/Config hours.

Typical items:

**Apex** (+`standardsOverheadPct.apex`, default 30% of `Dev_component`, itemized):
- Test class(es) meeting the coverage threshold in the Apex review instructions (org-wide ≥ 75% deploy minimum;
  team standard is usually ≥ 85–90%), including bulk (200+ records), negative, and permission/FLS test scenarios.
- Bulkification, no SOQL/DML in loops, selector/service pattern, `with sharing` / stripInaccessible / FLS checks.
- Error handling & logging per team pattern; custom exceptions; no hard-coded IDs/URLs (Custom Metadata / Labels).
- Salesforce Code Analyzer / PMD clean (zero high-severity); naming & ApexDoc; review-comment turnaround (1–2 cycles).

**Flow** (+`standardsOverheadPct.flow`, default 20%):
- Fault paths on every DML/Get/callout element; naming convention; description on every element; no hard-coded IDs.
- One record-triggered flow per object/context or entry criteria per team standard; before-save vs after-save choice.
- Bulk-safe (no Get/DML inside loops); Flow tests / test-run evidence; version documentation & activation plan.

**LWC** (+`standardsOverheadPct.lwc`, default 25%): Jest tests, accessibility (SLDS/a11y), LWS security, error UX.
**Config** (+`standardsOverheadPct.config`, default 10%): change-set / DevOps Center or AutoRABIT metadata, docs.

## 4. Non-Dev roles (per story unless the story says otherwise)
- **QA**: 40% of story Dev hours (`Dev_component` + standards, §3) (functional + regression + negative), +8h per story
  with integration or data migration.
- **Admin/Config**: sum of the story's Admin/Config rows (§1 "Role of a row") + Config standards items (§3).
- **BA**: 2h per story (refinement, AC finalization) + 4h per epic (walkthrough, UAT support).
- **Review/Lead**: 10% of story Dev hours (design review, PR review, TRB if architectural).
- **Rounding**: keep §1/§3 line items unrounded; round each role subtotal (Dev, QA, Admin/Config, Review) to the
  nearest 0.5h before summing `story_hours` (§9: 19.9 → 20h Dev, 7.96 → 8h QA, 1.99 → 2h Review).

## 5. Definition-of-Done tax
`definitionOfDoneTaxHoursPerStory` (default 3h) per story: PR hygiene, release notes, deployment steps, sandbox
validation evidence, DoD checklist. This is what makes "First Time Right" real — do not omit.

## 6. Risk buffer and confidence (per epic, on the total)
By confidence: **H** +10%, **M** +20%, **L** +35%. Confidence is L when requirements are ambiguous, the org has
unknown managed-package interactions, or an external system's API is undocumented.
Confidence is assessed per story (§9). **Epic confidence = the lowest confidence among its stories** (so the
placeholder cap of M raises the buffer to ≥ 20%); **overall confidence in the analysis header = the lowest epic
confidence.**

## 7. Story points
`story_hours = Dev + QA + Admin/Config + BA + Review + DoD` (unbuffered, uncalibrated, role subtotals rounded per §4).
`points = round_to_scale( story_hours / hoursPerStoryPoint )` using `storyPointScale`: take the scale value nearest to
the quotient; on an exact tie take the larger value (e.g. 36 h / 6 = 6.0 → 5; 39 h / 6 = 6.5 → 8; 45 h / 6 = 7.5 → 8).
Show both. The epic risk buffer (§6) and the calibration factor (§8) apply to the roll-up hours only, never to points.
If the quotient exceeds the largest value in `storyPointScale` (13 → more than 78 h with the defaults), do not clamp
it — split the story and re-estimate the parts.

## 8. Calibration (do this when Jira read access is available and configured)
Pull up to 20 Done stories via `calibrationJql` (the analyst uses `maxResults` 20). For each, infer its components from
summary/labels/description and compute the points this rubric would give. `factor = Σ actual points ÷ Σ rubric
points`, rounded to 2 decimals (if the points field is empty, use `timespent` ÷ `hoursPerStoryPoint` as actual). If
fewer than 5 stories are usable — or Jira is unavailable / not configured — `factor = 1.0`, `sample = <n>` and state
"insufficient sample". Apply the factor only to the §7 roll-up hours in the template (`Cal. h = Total h × factor`) and
to the header range; per-story hours and story points stay uncalibrated. Record it in `backlog.json → calibration`.

## 9. Output format for every story
```
Estimate: 20h Dev + 8h QA + 2h Admin + 2h BA + 2h Review + 3h DoD = 37h  → 37/6 = 6.2 → 5 pts (H confidence)
  ├─ Rubric §1: Record-triggered Flow (modify) 4h ×1.6 complex = 6.4h  [existing 3 flows on Loan__c]
  ├─ Rubric §1: Apex service (modify) 5h ×1.0 = 5h
  ├─ Rubric §1: Permission set FLS change 2h (Admin/Config)
  ├─ Rubric §3 Apex standards (from apex-code-review.instructions.md): test class ≥85% incl. bulk/negative 4h;
  │    PMD/FLS 1.5h; ApexDoc 0.5h   → 6h vs 30% floor of 11.4h Dev_component = 3.4h (2× = 6.8h) ✔
  ├─ Rubric §3 Flow standards (from flow-code-review.instructions.md): fault paths + descriptions 1.5h;
  │    test evidence 1h   → 2.5h vs 20% floor = 2.3h ✔
  ├─ Dev = 11.4h components + 8.5h standards = 19.9h → 20h; QA 40% = 8h; Review 10% = 2h
  └─ Assumptions: A-3, A-7. Risks: R-2. Requirements: FR-1, NFR-3.
```
