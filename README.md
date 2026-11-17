# QA Testing Intern Task — Cynbit Website Quality Audit

Public quality audit of [https://www.cynbit.in](https://www.cynbit.in) for the Cynbit QA Testing Intern challenge.

## Deliverables

1. [QA Audit Report](./01_QA_Audit_Report.md) — methodology, scope, environment and scenario matrices  
2. [Bug & Issue Report](./02_Bug_and_Issue_Report.md) — 37 issues (2 critical, 11 high)  
3. [Improvement Recommendations](./03_Improvement_Recommendations.md) — P0–P2 engineering fixes and a QA process plan  

## How the site was tested

Read-only HTTPS crawl of the sitemap, HTML inspection of conversion pages (contact, careers apply, BitElevate, portal, 404), and header/security checks. No live applications or payments were submitted.

Repeatable crawler: `scripts/site_audit.py`  
Raw evidence: `audit_raw.json`
