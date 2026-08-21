# Gap analysis — Member Address Change (LIVE TEST)

| | |
|---|---|
| Requirement source | [Confluence page 655361](https://deepanshuchauhan96.atlassian.net/wiki/spaces/SD/pages/655361/GAP-TEST+Requirement+Member+Address+Change) · space "Software Development" · read live via MCP |
| Story source | JQL `labels = gap-test` → [SCRUM-7](https://deepanshuchauhan96.atlassian.net/browse/SCRUM-7), [SCRUM-8](https://deepanshuchauhan96.atlassian.net/browse/SCRUM-8) · read live via MCP |
| Analyzed | 2026-08-20 · by `story-gap-analyst` procedure (LIVE run against Confluence + Jira) |
| Checkpoints | 18 (CP-1..CP-18, appendix below) |
| **Requirement coverage** | **28%** (5.0 of 18 checkpoint points) |

## Gap table

| Existing story | Existing AC | Corrected story | Complete acceptance criteria | Completeness |
|---|---|---|---|---|
| **SCRUM-7** · [GAP-TEST] Update member mailing address<br>*As a member service agent, I want to update a member's mailing address from the member profile page, so that statements go to the right place.* | - Agent opens the member profile and edits the address fields<br>- New address is saved on the member record<br>- A success message is shown | Title: **Update member mailing address (permission-controlled)**<br>Add to description: only users with the **"Profile Update"** permission may edit; all other users see the address fields **read-only** (NFR-2). | Scenario: agent with Profile Update permission edits and saves address → new address stored, success message shown<br>Scenario: user WITHOUT Profile Update permission opens profile → address fields are read-only, no edit action available<br>Scenario: saved change appears on the member record immediately | **60%** — missing: CP-17 (permission restriction), CP-18 (read-only for others) |
| **SCRUM-8** · [GAP-TEST] Validate address with USPS before save<br>*As a member service agent, I want the entered address to be checked against USPS, so that we store deliverable addresses.* | - On save, the address is sent to the USPS validation service<br>- If USPS returns a standardized address, the agent can accept it<br>- The accepted address is stored on the member record | Title: no change.<br>Add to description: agent may **correct the input** instead of accepting the suggestion (FR-2); **P.O. Box rejected as residential**, allowed as mailing (BR-1); USPS **unavailable → supervisor override** saves with a re-validation flag (NFR-3). | Scenario: USPS returns standardized address → agent accepts → stored<br>Scenario: agent rejects suggestion and corrects the input → corrected address re-validated and stored<br>Scenario: P.O. Box entered as residential address → save blocked with message<br>Scenario: P.O. Box entered as mailing address → allowed<br>Scenario: USPS service down → supervisor override permits save; record flagged for re-validation | **30%** — missing: CP-6 (correct input), CP-7 (override), CP-8 (re-validation flag), CP-11/12 (P.O. Box rules) |
| *(none — gap)* | — | **NEW-1: Send address-change email confirmation**<br>*As a member, I receive an email confirmation after my address is changed, so that I can detect unauthorized changes.* | Scenario: address change saved → confirmation email sent to the member's email on file **within 15 minutes**<br>Scenario: email includes old→new summary and a "not you? contact us" path | **0% — missing** (CP-9, CP-10) |
| *(none — gap)* | — | **NEW-2: Block address change under fraud hold**<br>*As a fraud analyst, I want address changes blocked on accounts with an active fraud hold, so that account-takeover is prevented.* | Scenario: account has active fraud hold → save blocked<br>Scenario: agent sees message directing them to the fraud team | **0% — missing** (CP-13, CP-14) |
| *(none — gap)* | — | **NEW-3: Audit trail for address changes**<br>*As a compliance officer, I need every address change audited, so that we meet retention obligations.* | Scenario: change saved → audit record stores old address, new address, user, date/time, channel<br>Scenario: audit records retained 7 years and retrievable | **0% — missing** (CP-15, CP-16) |

## Checkpoint appendix (evidence)

| CP | Req | Story | Score | Reason |
|---|---|---|---|---|
| CP-1 | FR-1 | SCRUM-7 | 1.0 | AC states agent edits address fields on profile |
| CP-2 | FR-1 | SCRUM-7 | 1.0 | AC states address saved on member record |
| CP-3 | FR-1 | SCRUM-7 | 1.0 | AC states success message |
| CP-4 | FR-2 | SCRUM-8 | 1.0 | AC states address sent to USPS on save |
| CP-5 | FR-2 | SCRUM-8 | 1.0 | AC states agent can accept standardized address |
| CP-6 | FR-2 | SCRUM-8 | 0.0 | Correcting the input not mentioned |
| CP-7 | NFR-3 | SCRUM-8 | 0.0 | Supervisor override on USPS outage absent |
| CP-8 | NFR-3 | SCRUM-8 | 0.0 | Re-validation flag absent |
| CP-9 | FR-3 | GAP→NEW-1 | 0.0 | No story sends confirmation email |
| CP-10 | FR-3 | GAP→NEW-1 | 0.0 | 15-minute SLA absent |
| CP-11 | BR-1 | SCRUM-8 | 0.0 | P.O. Box residential rejection absent |
| CP-12 | BR-1 | SCRUM-8 | 0.0 | P.O. Box mailing allowance absent |
| CP-13 | BR-2 | GAP→NEW-2 | 0.0 | Fraud-hold block absent |
| CP-14 | BR-2 | GAP→NEW-2 | 0.0 | Fraud-team message absent |
| CP-15 | NFR-1 | GAP→NEW-3 | 0.0 | Old/new/user/time/channel audit absent |
| CP-16 | NFR-1 | GAP→NEW-3 | 0.0 | 7-year retention absent |
| CP-17 | NFR-2 | SCRUM-7 | 0.0 | Permission restriction absent |
| CP-18 | NFR-2 | SCRUM-7 | 0.0 | Read-only for other users absent |

Story scores: SCRUM-7 = 3.0/5 = 60% · SCRUM-8 = 2.0/7 ≈ 30% · Coverage = 5.0/18 = **28%**

## Open questions
- Q-1: FR-3 — which template/system sends the confirmation email (Marketing Cloud, Apex, Flow)? Affects NEW-1 sizing.
- Q-2: BR-1 — is "residential vs mailing" a field pair on the member record today, or new fields?
- Q-3: NFR-1 — is Field History (18-month limit) acceptable with an archive, or is a custom audit object required for 7 years?
