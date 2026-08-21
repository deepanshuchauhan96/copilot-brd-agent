# Completeness scoring method (gap analysis)

The story-gap-analyst MUST score with this method, not from intuition. The percentage is only defensible if a
reviewer can argue with a specific checkpoint instead of the number.

## 1. Decompose the requirement into checkpoints
Break the requirement into **atomic, testable checkpoints** (CP-1..CP-n). One checkpoint = one verifiable statement
a tester could pass/fail (a rule, a field, a validation, a channel, an NFR, an actor restriction, an audit need).
- Business rules, NFRs (performance, security/PII, audit, accessibility) and negative paths are checkpoints too.
- Do not invent checkpoints the requirement does not state. Ambiguities become open questions, not checkpoints.
- Target granularity: a 1–2 page requirement typically yields 15–40 checkpoints.

## 2. Map checkpoints to stories
Assign every checkpoint to the story whose scope should cover it (by the story's evident intent). Checkpoints no
existing story should cover go to a **GAP** bucket and become proposed new stories.

## 3. Score each checkpoint against the story as written
- **1.0 covered** — the story description or AC states it explicitly (paraphrase is fine; implication is not)
- **0.5 partial** — mentioned but under-specified (no negative path, no limit value, no channel, no error case)
- **0.0 missing** — absent from the story text
Judge only what is WRITTEN in the story/AC. "The developer would obviously do it" scores 0.

## 4. Compute
- Story completeness % = round( Σ scores ÷ # applicable checkpoints × 100 ) to the nearest 5.
- Proposed NEW stories score **0% (missing)**.
- Requirement coverage % (headline) = Σ scores across ALL checkpoints ÷ total checkpoints × 100.

## 5. Evidence appendix (mandatory)
After the table, list every checkpoint: `CP-n | requirement ref | story | score | one-line reason`.
The percentages must be reproducible from this appendix alone.
