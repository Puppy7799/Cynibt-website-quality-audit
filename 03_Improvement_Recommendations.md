# Improvement Recommendations — Cynbit Website

**Audience:** Engineering, Design, and QA  
**Date:** 17 November 2026  
**Input:** `01_QA_Audit_Report.md`, `02_Bug_and_Issue_Report.md`  
**Horizon:** P0 this week (before more intern applications) · P1 next sprint · P2 backlog

Recommendations are written so a developer can pick them up without re-testing from scratch.

---

## 1. Priority map

| Priority | Goal | Bug IDs |
| --- | --- | --- |
| **P0** | Restore broken conversions and stop silent failures | BUG-001, BUG-003, BUG-004, BUG-005 |
| **P0** | Harden apply / pay / portal | BUG-002, BUG-010, BUG-011, BUG-012 |
| **P1** | Make the site usable with a keyboard and a screen reader | BUG-006–009, BUG-013, BUG-018, BUG-019, BUG-028 |
| **P1** | Trust, SEO, legal | BUG-014, BUG-015, BUG-020, BUG-024, BUG-025 |
| **P2** | Performance, polish, process | BUG-016, BUG-017, BUG-021–023, BUG-026, BUG-027, BUG-029–036 |

---

## 2. P0 — Ship this week

### R-01. Make newsletter subscription a real workflow

**Fixes BUG-001.**

- Replace the footer `<button>` with a short form: email + submit, or a link to Beehiiv / Loops / Mailchimp.
- Show success and duplicate-email states.
- If the product is not ready, **hide the CTA** rather than leave a dead control promising “25k+”.
- Add a Playwright test: footer control is a link or a form that reaches a 2xx subscribe API.

### R-02. Repair internal blog graph

**Fixes BUG-003, BUG-016.**

- Redirect `/blog/ai-agents-transforming-business` → `/blog/ai-agents-business-operations` (301).
- Encode category query values (`AI%20Agents`) or use slugs (`?category=ai-agents`).
- Add a CI step: crawl sitemap + extract `<a href>` on blog templates; fail the build on in-domain 404s.

### R-03. Split contact mailtos

**Fixes BUG-004, BUG-022.**

- `opportunities@` and `support@` as two links.
- Align the three `tel:` numbers across contact body and footer.
- Optional: `mailto:support@cynbit.in?subject=Website%20404` for the 404 “Report Broken Node” button (today it only goes to `/contact`).

### R-04. Fix academic score input

**Fixes BUG-005.**

- Bind `name="scoreValue"` (currently missing in SSR).
- If score system = CGPA → `max=10`; if percentage → `max=100`; if “Other” → no max, helper text.
- Client and server validation must match.

### R-05. Security headers at the edge

**Fixes BUG-010, BUG-011.**

Set in `vercel.json` or middleware (illustrative):

- `Content-Security-Policy` (start Report-Only for a week)
- `X-Frame-Options: DENY` plus CSP `frame-ancestors 'none'`
- `X-Content-Type-Options: nosniff`
- `Referrer-Policy: strict-origin-when-cross-origin`
- `Permissions-Policy: camera=(), microphone=(), geolocation=()`
- Remove `Access-Control-Allow-Origin: *` from HTML routes; allow it only on APIs that must be public
- Strengthen HSTS with `includeSubDomains` once all hosts are HTTPS

### R-06. Portal and payment hygiene

**Fixes BUG-002, BUG-012.**

- Portal: email **and** password (or magic link), lockout messaging, “forgot key”, generic errors.
- Do not rely on `robots.txt` Disallow as access control.
- BitElevate: Terms checkbox, bot protection (Turnstile), server-side rate limit, and a test mode flag so QA can exercise Cashfree without live charges.
- Log and alert on repeated `/portal` failures.

---

## 3. P1 — Next sprint (quality users can feel)

### R-07. Accessibility on forms and chrome

**Fixes BUG-006–009, BUG-013, BUG-018, BUG-028.**

| Change | Why |
| --- | --- |
| Every input: unique `id` + `<label for>` | WCAG 1.3.1 / 4.1.2 |
| `type="tel"` + `autocomplete` | Mobile and password managers |
| `aria-hidden="true"` on Material Symbols spans | Stops “arrow_forward” in names |
| `aria-label` on logo and social icons | Empty links |
| One `<main>` per page; skip link as first focusable | Keyboard |
| Mobile menu: `aria-expanded`, `aria-controls`, focus trap | Drawer pattern |
| Visible `:focus-visible` rings | Keyboard users on dark/light surfaces |

Target: axe-core / Playwright `@axe-core/playwright` **zero serious** on Home, Contact, Careers apply, BitElevate, Portal.

### R-08. Reduce PII on intern applications; allow resume upload

**Fixes BUG-020, BUG-021, BUG-025.**

- Make **Gender** and **DOB** optional or remove for intern roles unless HR has a documented need.
- Add optional PDF upload (size/type limits) **in addition to** URL.
- Update Privacy Policy with DPDP 2023: what you collect, why, retention, grievance officer, and payment processor (Cashfree).
- If Google Analytics is used, add a consent banner with Reject; if not, remove it from the Cookie Policy.

### R-09. Titles, canonicals, schema

**Fixes BUG-014, BUG-015, BUG-017, BUG-027.**

- Template: `{Page} | Cynbit Technologies` once.
- Canonical + sitemap + JSON-LD all `https://www.cynbit.in`.
- Apex `https://cynbit.in` → **301** (not 307).
- 404 title: `Page not found | Cynbit Technologies`.
- Replace 26.8000, 75.8000 with a geocoded Pratap Nagar coordinate.

### R-10. Content and heading hygiene

**Fixes BUG-019, BUG-029, BUG-036.**

- Do not wrap subtitle `<p>` inside `<h1>`.
- Fix “withAI” spacing.
- Raise footer/CTA font from 9–10 px to ≥12 px (14 px preferred).
- Keep struck-through ₹3,999 / live ₹1,499 consistent everywhere (search snippets still quoted ₹3,999 as the fee).

---

## 4. P2 — Hardening and platform QA

### R-11. Performance budget

**Fixes BUG-026.**

- Investigate 4–11 s HTML on case-study/diary routes (cold prerender? blocking fonts?).
- Homepage `Age` ~42 h with `must-revalidate` is OK for ISR; ensure error pages are `no-store`.
- Preload only above-the-fold fonts; `favicon.ico` fallback to stop console 404s (BUG-023).
- Lab: Lighthouse mobile on Home, BitElevate, Careers apply — budgets TBT < 200 ms, LCP < 2.5 s on 4G Fast.

### R-12. Favicon / touch icons

Add `/favicon.ico`, 180×180 PNG apple-touch-icon, and a real `manifest.json` **or** remove the manifest link.

### R-13. Disclosure and ops

- `/.well-known/security.txt` → `Contact: mailto:opportunities@cynbit.in` (or a security alias).
- Intern login link in footer (small) pointing to `/portal` so people stop guessing `/login`.

---

## 5. Recommended QA process upgrades (for the intern role)

These match the job description (test methodology, bug tracking, automation as a plus).

### 5.1 Test strategy

| Layer | Tool | Cadence |
| --- | --- | --- |
| Smoke | Playwright: home, contact validation, careers apply required fields, 404, portal renders | Every PR |
| Link check | Reuse `scripts/site_audit.py` or `lychee` on sitemap + extracted hrefs | Nightly |
| A11y | `@axe-core/playwright` on 5 templates | Every PR |
| Headers | Assert CSP, XCTO, HSTS on `https://www.cynbit.in` | Every PR |
| Payment | Cashfree sandbox + stub webhook | Staging only |
| Visual | Chromatic or Playwright screenshots 360 / 768 / 1440 | Main pages |

### 5.2 Device matrix to close this audit’s gap

Run exploratory + screenshot on:

1. Chrome 360×800 Android  
2. Safari iOS 390×844  
3. Chrome 1440×900 Windows  
4. Keyboard-only pass (Tab through header → contact form)  
5. NVDA or VoiceOver on header + apply form  

Record actual vs expected for: sticky header overlap, drawer scroll lock, Cashfree sheet, date picker on iOS.

### 5.3 Bug report template (use going forward)

```text
Title:
Environment: OS / Browser / Viewport / Incognito / Auth
Severity: S1–S4 (user impact)
Type: Functional | Usability | A11y | Responsive | Perf | Security
Preconditions:
Steps:
Expected:
Actual:
Workaround:
Evidence: URL, HTTP status, screenshot, HAR
```

### 5.4 Definition of done for a marketing/careers release

- [ ] No in-domain 404s from header, footer, sitemap, or related posts  
- [ ] Contact, apply, enroll: labels, required, validation, success, error  
- [ ] Newsletter either works or is removed  
- [ ] axe serious = 0 on templates  
- [ ] Security header baseline present  
- [ ] Payment not submittable in production from automated tests  
- [ ] Privacy copy matches fields actually collected  

### 5.5 What I would automate first as intern

1. Playwright spec `footer-newsletter.spec.ts` (currently fails — good first red test).  
2. `internal-links.spec.ts` driven by sitemap.xml.  
3. `apply-form-score.spec.ts` for CGPA vs percentage.  
4. `headers.spec.ts` for CSP/XCTO/HSTS.  
5. Soften later: visual snapshots of BitElevate pricing block so ₹1,499 vs ₹3,999 cannot drift.

---

## 6. Suggested 14-day fix sequence

| Day | Owner | Outcome |
| ---: | --- | --- |
| 1 | Frontend | R-01, R-02, R-03 (dead CTA, 404 slug, mailtos) |
| 2 | Frontend | R-04 score field; tel inputs |
| 2 | Platform | R-05 headers; drop ACAO `*` on HTML |
| 3–4 | Frontend | Labels, icon `aria-hidden`, skip link, single `<main>` |
| 5 | Product/HR | Trim intern PII; resume upload |
| 6 | Legal | Privacy + cookie consent vs actual processors |
| 7 | SEO | Titles, www canonical, 301 apex, 404 title |
| 8–9 | QA intern | Playwright smoke + axe on 5 templates |
| 10 | Platform | Portal identity model + rate limit |
| 11 | Perf | Trace slow case-study routes |
| 12–14 | QA | Device matrix; regression; close tickets |

---

## 7. Sign-off note for reviewers

This audit did **not** submit applications, did **not** complete Cashfree payment, and did **not** attempt to guess portal keys. Interactive JS (menu, payment modal, toast errors) still needs a headed-browser pass — that is the highest-value next session if I join as QA intern.
