# Gap analysis — Member Address Change (FINAL, after story updates)

| | |
|---|---|
| Requirement source | [Confluence page 655361](https://deepanshuchauhan96.atlassian.net/wiki/spaces/SD/pages/655361/GAP-TEST+Requirement+Member+Address+Change) · read live via MCP |
| Story source | JQL `labels = gap-test` → SCRUM-7, SCRUM-8, SCRUM-9, SCRUM-10, SCRUM-11 · read live via MCP |
| Analyzed | 2026-08-20 · by `story-gap-analyst` procedure (re-audit after corrections applied) |
| Checkpoints | 18 (same CP-1..CP-18 as the first run) |
| **Requirement coverage** | **100%** (18.0 of 18 checkpoint points; first run was 28%) |

## Gap table

| Existing story | Existing AC | Corrected story | Complete acceptance criteria | Completeness |
|---|---|---|---|---|
| **SCRUM-7** · [GAP-TEST] Update member mailing address<br>*As a member service agent with the "Profile Update" permission, I want to update a member's mailing address from the member profile page, so that statements go to the right place. Only users with the "Profile Update" permission can edit the address; all other users see the address fields read-only.* | <br>- Given an agent with the "Profile Update" permission, when they edit the address fields on the member profile and save, then the new address is saved on the member record and a success message is shown<br>- Given a user without the "Profile Update" permission, when they open the member profile, then the address fields are read-only and no edit action is available | No change needed | AC as written is complete for its checkpoints | **100% — all checkpoints covered (CP-1, CP-2, CP-3, CP-17, CP-18)** |
| **SCRUM-8** · [GAP-TEST] Validate address with USPS before save<br>*As a member service agent, I want the entered address validated against USPS before save - accepting the standardized suggestion or correcting my input - so that we store deliverable addresses. A P.O. Box is rejected as a residential (physical) address but allowed as a mailing address. If the USPS service is unavailable, a supervisor override permits save and the address is flagged for later re-validation.* | <br>- On save, the address is sent to the USPS validation service<br>- If USPS returns a standardized address, the agent can accept it and it is stored on the member record<br>- If the agent rejects the suggestion and corrects the input, the corrected address is re-validated and stored<br>- If a P.O. Box is entered as the residential (physical) address, save is blocked with a message<br>- If a P.O. Box is entered as the mailing address, it is allowed<br>- If the USPS service is unavailable, a supervisor override permits save and the record is flagged for later re-validation | No change needed | AC as written is complete for its checkpoints | **100% — all checkpoints covered (CP-4, CP-5, CP-6, CP-7, CP-8, CP-11, CP-12)** |
| **SCRUM-9** · [GAP-TEST] Send address-change email confirmation<br>*As a member, I receive an email confirmation after my address is changed, so that I can detect unauthorized changes.* | <br>- When an address change is saved, a confirmation email is sent to the member's email on file within 15 minutes<br>- The email includes an old-to-new address summary and a "not you? contact us" path | No change needed | AC as written is complete for its checkpoints | **100% — all checkpoints covered (CP-9, CP-10)** |
| **SCRUM-10** · [GAP-TEST] Block address change under fraud hold<br>*As a fraud analyst, I want address changes blocked on accounts with an active fraud hold, so that account-takeover is prevented.* | <br>- If the member's account has an active fraud hold, saving an address change is blocked<br>- The agent sees a message directing them to the fraud team | No change needed | AC as written is complete for its checkpoints | **100% — all checkpoints covered (CP-13, CP-14)** |
| **SCRUM-11** · [GAP-TEST] Audit trail for address changes<br>*As a compliance officer, I need every address change audited, so that we meet retention obligations.* | <br>- When a change is saved, an audit record stores the old address, the new address, the user, the date/time, and the channel<br>- Audit records are retained for 7 years and are retrievable | No change needed | AC as written is complete for its checkpoints | **100% — all checkpoints covered (CP-15, CP-16)** |

## Checkpoint appendix (evidence)

| CP | Req | Story | Score | Reason |
|---|---|---|---|---|
| CP-1 | FR-1 | SCRUM-7 | 1.0 | AC/description states it explicitly: address fields editable on member profile page |
| CP-2 | FR-1 | SCRUM-7 | 1.0 | AC/description states it explicitly: new address saved on member record |
| CP-3 | FR-1 | SCRUM-7 | 1.0 | AC/description states it explicitly: success message shown |
| CP-17 | NFR-2 | SCRUM-7 | 1.0 | AC/description states it explicitly: only Profile Update permission can edit |
| CP-18 | NFR-2 | SCRUM-7 | 1.0 | AC/description states it explicitly: fields read-only for other users |
| CP-4 | FR-2 | SCRUM-8 | 1.0 | AC/description states it explicitly: address sent to USPS on save |
| CP-5 | FR-2 | SCRUM-8 | 1.0 | AC/description states it explicitly: agent can accept standardized address |
| CP-6 | FR-2 | SCRUM-8 | 1.0 | AC/description states it explicitly: agent can correct input; corrected address re-validated |
| CP-7 | NFR-3 | SCRUM-8 | 1.0 | AC/description states it explicitly: supervisor override when USPS unavailable |
| CP-8 | NFR-3 | SCRUM-8 | 1.0 | AC/description states it explicitly: override-saved address flagged for re-validation |
| CP-11 | BR-1 | SCRUM-8 | 1.0 | AC/description states it explicitly: P.O. Box rejected as residential address |
| CP-12 | BR-1 | SCRUM-8 | 1.0 | AC/description states it explicitly: P.O. Box allowed as mailing address |
| CP-9 | FR-3 | SCRUM-9 | 1.0 | AC/description states it explicitly: confirmation email to member after change |
| CP-10 | FR-3 | SCRUM-9 | 1.0 | AC/description states it explicitly: email within 15 minutes |
| CP-13 | BR-2 | SCRUM-10 | 1.0 | AC/description states it explicitly: active fraud hold blocks change |
| CP-14 | BR-2 | SCRUM-10 | 1.0 | AC/description states it explicitly: agent directed to fraud team |
| CP-15 | NFR-1 | SCRUM-11 | 1.0 | AC/description states it explicitly: audit stores old+new address, user, date/time, channel |
| CP-16 | NFR-1 | SCRUM-11 | 1.0 | AC/description states it explicitly: audit retained 7 years |

Story scores: SCRUM-7 = 5.0/5 = 100% · SCRUM-8 = 7.0/7 = 100% · SCRUM-9 = 2.0/2 = 100% · SCRUM-10 = 2.0/2 = 100% · SCRUM-11 = 2.0/2 = 100%
Coverage = 18.0/18 = **100%** (was 28% before corrections)

## Open questions (carried from first run — completeness of text, not of design)
- Q-1: FR-3 — which system sends the confirmation email (Marketing Cloud, Apex, Flow)?
- Q-2: BR-1 — are residential vs mailing address a field pair on the member record today, or new fields?
- Q-3: NFR-1 — Field History (18-month limit) with archive, or a custom audit object for 7 years?