# BRD — Member Loan Payoff Quote (SAMPLE for testing the agent)

**Version** 0.3 · **Sponsor** Consumer Lending Ops · **Date** 2026-08-10

## 1. Business objective
Members frequently call to request a payoff quote on auto and personal loans. Today agents calculate quotes in the
core banking system and email a PDF manually (avg 11 min/call). We want members and agents to generate a payoff quote
inside Salesforce in under 1 minute, with an audit trail.

## 2. Scope
### 2.1 In scope
- FR-1 Agent can generate a payoff quote from the Loan record (good-through date selectable, max 30 days out).
- FR-2 Quote pulls current principal, accrued interest and fees from core banking in real time.
- FR-3 Quote is stored as a related record with status (Draft/Issued/Expired) and a generated PDF.
- FR-4 Member can request the same quote from the member portal (Experience Cloud) for eligible loans only.
- FR-5 Issued quotes appear on the Loan record timeline and are searchable by quote number.
- FR-6 Ops dashboard: quotes issued per day, expired without payoff, average generation time.
### 2.2 Out of scope
Payment execution; mortgage products; loans in bankruptcy or charge-off status.

## 3. Actors
Member Service Agent, Member (portal), Lending Ops Analyst, Compliance Reviewer.

## 4. Business rules
- BR-1 A quote is valid through the selected good-through date; after that status = Expired (nightly).
- BR-2 Loans with a past-due > 60 days or legal hold cannot be quoted from the portal (agent may, with a warning).
- BR-3 Every quote must record who generated it, channel, and the core-banking response ID.

## 5. Non-functional requirements
- NFR-1 Quote generated end-to-end < 5 s (P95). NFR-2 Core-banking outage → graceful message, no partial records.
- NFR-3 PII (SSN, full account number) never stored on the quote; last-4 only. Field-level security enforced.
- NFR-4 Audit: Field History on Quote status; 7-year retention. NFR-5 WCAG 2.1 AA for the portal component.

## 6. Dependencies / assumptions
- Core banking exposes `GET /loans/{id}/payoff?date=` via the existing `CoreBanking` Named Credential.
- PDF generation may reuse the existing statement-PDF Visualforce/Apex utility if one exists.

## 7. Open questions
- Q-1 Should agents be able to override fees? Q-2 Email/SMS delivery of the PDF — phase 1 or 2?
