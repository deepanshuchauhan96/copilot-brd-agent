# BRD Analysis — <BRD title>

| | |
|---|---|
| BRD source | `<path or attachment name>` (version/date) |
| Analyzed on | <date> · by `brd-analyst` (Copilot) |
| Repo commit | `<7-char sha>` |
| Org | `dev-sandbox` · `<org Name>` (or "repo only") |
| Status | DRAFT — needs human review before publishing to Jira |
| Estimate roll-up | **<low>–<high> h** (calibrated; low = unbuffered, high = buffered — see §7) · **<n> pts** · confidence <lowest epic H/M/L> · calibration ×<f> (n=<sample>) |
| DoR | PASS / FAIL (<n>/<N> items) — see §9 |

## 1. Executive summary
3–6 sentences: what the business wants, what it touches, biggest risks, headline estimate.

## 2. Requirements register
| ID | Type | Requirement (verbatim/condensed) | BRD ref | Priority | Ambiguity? |
|---|---|---|---|---|---|
| BO-1 | Business objective | … | §1 | — | — |
| FR-1 | Functional | … | §2.1 | Must | — |
| BR-1 | Business rule | … | §4 | Must | — |
| NFR-1 | Security/PII | … | §5 | Must | Q-2 |

## 3. Assumptions & open questions
- **A-1** …  (drives estimate of S-3)
- **Q-1** … → *owner: BA* · impact if answered "no": +8h on E-2

## 4. Current-state impact map
| Req | Component | Path / org evidence | Change type | Risk notes |
|---|---|---|---|---|
| FR-1 | `Loan__c.Status__c` picklist | `force-app/main/default/objects/Loan__c/fields/Status__c.field-meta.xml` | modify (+2 values) | 3 record-triggered flows on Loan__c already (see §4.1) |
| FR-1 | `LoanStatusHandler.cls` | `force-app/.../classes/LoanStatusHandler.cls:88` | modify | recursion guard present |
| FR-2 | Named Credential `CoreBanking` | org only (SOQL NamedCredential) — **not in repo** | reuse | |
| FR-3 | `PayoffQuote__c` | not found in repo or org — **new component** | new | |

(The rows above are illustrative examples only — replace them; never cite a path you did not read.)

### 4.1 Existing automation on affected objects (order-of-execution check)
### 4.2 Governor-limit / data-volume hotspots
### 4.3 Security, sharing, FLS & PII touchpoints
### 4.4 Test-coverage & code-quality baseline of touched classes (ApexCodeCoverageAggregate / antipattern scan)

## 5. Gap analysis & solution outline (no code)
Per requirement: reuse / extend / new; recommended pattern; alternatives rejected and why.

## 6. Backlog draft
### Epic E-1 — <name>   (serves BO-1)
Narrative · business value · dependencies · confidence & risk buffer

#### Story S-1 — As a <persona>, I want <capability>, so that <value>
- **Requirements covered:** FR-1, BR-1, NFR-3
- **Description / technical notes** (cite §4 rows)
- **Acceptance criteria**
  ```gherkin
  Scenario: <happy path>
    Given … When … Then …
  Scenario: <negative / validation>
  Scenario: <bulk 200+ records>
  Scenario: <security/FLS>
  ```
- **Standards compliance checklist** (from the instruction files listed in §10; if they were placeholders, prefix with
  "STANDARDS TAX = RUBRIC DEFAULTS"): ☐ test class ≥ <team threshold> incl. bulk/negative ☐ no SOQL/DML in loops
  ☐ FLS / `with sharing` ☐ fault paths on all Flow DML ☐ PMD / Code Analyzer clean ☐ no hard-coded IDs …
- **Estimate** (rubric-cited block — see rubric §9; include Admin/Config hours)
- **Dependencies**: S-0, external X
- **Test cases**
  | ID | Title | Type | Priority | Preconditions | Steps | Expected | Data | Reqs |
  |---|---|---|---|---|---|---|---|---|
  | TC-1.1 | … | Functional | P1 | … | 1. … 2. … | … | … | FR-1 |
  | TC-1.2 | … | Negative | P2 | | | | | BR-1 |
  | TC-1.3 | … | Bulk | P2 | | | | | NFR-2 |

## 7. Estimate roll-up
| Epic | Stories | Dev h | QA h | Admin h | BA h | Review h | DoD h | Conf | Buffer % | Total h | Cal. h | Pts |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
Total h = (Dev + QA + Admin + BA + Review + DoD) × (1 + Buffer %); Buffer % from rubric §6 by epic Conf (= lowest
story confidence); Cal. h = Total h × calibration factor (rubric §8; ×1.0 if n < 5). Pts = Σ story points, computed on
unbuffered, uncalibrated story hours. Header range: **low** = Σ unbuffered story hours × factor, **high** = Σ Cal. h;
header confidence = lowest epic Conf.

## 8. Risks & mitigations
| ID | Risk | Likelihood | Impact | Mitigation | Owner |

## 9. Definition-of-Ready evaluation (Gate 1 control)
| # | Item | Result | Evidence | Owner if ✘ |
|---|---|---|---|---|
| 1 | All FR/BR/NFR numbered & unambiguous (or ambiguity logged in §3) | ✔/✘ | §2 | |
| 2 | Every Must FR/BR/NFR maps to ≥1 story AND ≥1 test (NFRs: ≥1 test or a verifying AC); every other RTM §11 ✘ has an owner/question | ✔/✘ | §11 | |
| 3 | Every story has Gherkin AC incl. negative + bulk + security, or an explicit `N/A — <reason>` for a scenario type that cannot apply (e.g. bulk on a report/config-only story) | ✔/✘ | §6 | |
| 4 | Every story has a rubric-cited estimate and confidence | ✔/✘ | §6/§7 | |
| 5 | Dependencies & integrations identified | ✔/✘ | §4/§6 | |
| 6 | Open questions have owners | ✔/✘ | §3 | |
| 7 | Standards files were real (not placeholders) and priced in | ✔/✘ | §10 | |
Coverage: FRs/BRs with ≥1 story n/N · NFRs with ≥1 AC or test n/N · stories with AC+tests+estimate n/N · questions with owner n/N
**DoR result: PASS / FAIL** (FAIL if any of items 2–4 is ✘).

## 10. Standards files consulted
| File | Present? | Placeholder? | Rules priced in (short) |
|---|---|---|---|

## 11. Requirement traceability matrix (RTM) & test coverage
| Req ID | BRD ref | Objective | Story IDs | Test IDs | Covered? |
|---|---|---|---|---|---|
| FR-1 | §2.1 | BO-1 | S-1, S-3 | TC-1.1, TC-3.2 | ✔ |
| NFR-5 | §5 | BO-1 | — | — | ✘ GAP → Q-n |

Test coverage summary
| Req | #Functional | #Negative | #Bulk | #Security | #Integration | Gap? |
|---|---|---|---|---|---|---|

---
### backlog.json schema (written alongside this file)
```json
{
  "brd": "docs/brds/<file>", "slug": "<dir name; a-z0-9- only, ≤50 chars, derived per brd-analyst Inputs; also the Jira label suffix brd-<slug>>",
  "generatedAt": "ISO", "analysisCommit": "<7-char sha or unknown>",
  "approved": false, "approvedBy": null, "approvedAt": null, "dorOverrideReason": null,
  "calibration": {"factor": 1.0, "sample": 0},
  "dor": {"passed": false, "items": [{"id": 1, "result": "pass|fail", "evidence": "§2", "owner": null}]},
  "epics": [{
    "id": "E-1", "summary": "<epic name — one line, ≤ 120 chars; this is the Jira summary verbatim>", "description": "…",
    "businessObjective": "BO-1", "confidence": "M", "jiraKey": null,
    "stories": [{
      "id": "S-1", "summary": "<one line, ≤ 200 chars — the story sentence 'As a <persona>, I want <capability>, so that <value>', condensed if needed; this is the Jira summary verbatim (Jira rejects ≥ 255 chars or any newline)>",
      "description": "…", "technicalNotes": "…", "requirements": ["FR-1", "BR-1", "NFR-3"],
      "acceptanceCriteria": "gherkin text", "standardsChecklist": ["…"],
      "estimate": {"devH": 0, "qaH": 0, "adminH": 0, "baH": 0, "reviewH": 0, "dodH": 0, "totalH": 0, "points": 0, "confidence": "M", "rationale": "rubric lines"},
      "dependencies": ["S-0"], "labels": [], "jiraKey": null,
      "tests": [{"id": "TC-1.1", "title": "<one line, ≤ 100 chars — Jira summary becomes '<id>: <title>'>", "type": "Functional", "priority": "P1", "requirements": ["FR-1"], "preconditions": "…", "steps": ["…"], "expected": "…", "data": "…", "jiraKey": null}]
    }]
  }]
}
```
`dor.items` always has exactly the 7 items of §9 in that order. `estimate.totalH` = story_hours per rubric §7
(unbuffered, uncalibrated). `slug` must equal the analysis directory name.
