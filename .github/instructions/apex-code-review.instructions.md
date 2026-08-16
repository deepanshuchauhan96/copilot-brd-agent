---
applyTo: "**/*.cls,**/*.trigger"
description: PLACEHOLDER — replace with your team's existing Apex code-review instructions (keep the applyTo frontmatter).
---
<!--
  Drop your existing Apex review instructions here (or point config.json → standardsFiles at wherever they already live).
  The brd-analyst agent reads this file to price "standards compliance" into every estimate; Copilot also auto-applies it
  when editing matching files. Typical content the estimator looks for:
  - minimum test coverage % and required test scenarios (bulk 200+, negative, FLS/permission)
  - bulkification / no SOQL-DML in loops / selector-service-domain layering
  - `with sharing`, stripInaccessible / FLS enforcement, no hard-coded IDs
  - error handling & logging pattern, custom exceptions
  - Salesforce Code Analyzer / PMD rule set & severity gate
  - naming, ApexDoc, PR review expectations
-->
