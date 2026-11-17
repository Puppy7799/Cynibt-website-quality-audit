# QA Audit Report — Cynbit Website Quality Audit

**Product:** [cynbit.in](https://www.cynbit.in) (Cynbit Technologies marketing site, careers, BitElevate enrollment, intern portal)  
**Auditor:** QA Testing Intern candidate  
**Audit date:** 17 November 2026  
**Submission target:** 19 November 2026, 08:45 AM  
**Build / host:** Next.js on Vercel (`Server: Vercel`, `X-Nextjs-Prerender: 1`)  
**Primary host:** `https://www.cynbit.in` (apex `https://cynbit.in` redirects here)

---

## 1. Objective

Evaluate the public Cynbit website as a real user would: discover pages, complete high-value workflows, and record issues that block, confuse, or reduce trust. The audit covers functional correctness, broken assets, usability, responsiveness signals, accessibility, performance, and security hygiene — not source-code review of a private repo.

## 2. Scope

### In scope

| Area | URLs / surfaces |
| --- | --- |
| Marketing | Home, Story, Team, Services, Products, Case studies, Impact, Blog, Diaries |
| Conversion | Contact inquiry, careers listing + job application, BitElevate enrollment / payment CTA |
| Trust / legal | Privacy, Terms, Cookies, 404 |
| Auth-adjacent | `/portal` intern login (publicly reachable) |
| Discovery | `robots.txt`, `sitemap.xml`, canonical / Open Graph / JSON-LD |
| Cross-cutting | Nav, footer, social links, tel/mailto, newsletter CTA, security headers |

**Sitemap coverage:** 80+ indexed URLs under `https://cynbit.in/sitemap.xml` (core pages, 25 blog posts, 13 case studies, 19 career pages, 8 diaries). All sitemap HTML URLs were requested.

### Out of scope (with reason)

| Item | Reason |
| --- | --- |
| Completing a live job application | Would create production HR noise |
| Completing BitElevate payment (₹1,499 Cashfree) | Live money movement; observed UI/copy only |
| Brute-forcing or bypassing `/portal` | Unauthorized access testing |
| Private intern dashboard after login | No credentials |
| Native mobile apps | Web-only product |
| Email deliverability of inquiry/apply | No test inbox / no production submit |
| Visual pixel QA on every breakpoint | Dedicated browser automation was unavailable in this environment (see limitations) |

## 3. Test approach

Testing mixed **exploratory QA** with a **read-only HTTP crawl**.

1. **Reconnaissance** — Identify host, sitemap, robots, nav IA, and conversion funnels.
2. **Functional walkthroughs** — Trace user goals (contact sales, apply for QA intern, enroll in BitElevate, read a case study, recover from 404).
3. **Link & asset integrity** — Request internal links, images/scripts sample, and key external destinations.
4. **Form inspection** — Field types, labels, required rules, file upload, CAPTCHA, payment gateway, validation constraints (e.g. score `max`).
5. **Accessibility static pass** — `lang`, headings, landmarks, skip links, control names, icon fonts, label association, target size / font size classes.
6. **Security observation** — TLS redirect, security headers, CORS, robots-disallowed surfaces that still render, public login design.
7. **Performance observation** — TTFB / total fetch time from Jaipur-adjacent Vercel `bom1` edge, HTML weight, cache headers.
8. **SEO / content consistency** — Titles, canonical host, sitemap host, schema.org JSON-LD, duplicate branding, broken related-post slugs.

Severity uses **user impact**, not only WCAG clause:

| Severity | Meaning |
| --- | --- |
| **S1 – Critical** | Blocks a primary journey or creates a serious trust/security gap |
| **S2 – High** | Major friction, broken path, or clear a11y/security defect with frequent exposure |
| **S3 – Medium** | Incorrect or incomplete behavior; workaround exists |
| **S4 – Low** | Polish, SEO, or edge-case quality |

## 4. Test environment matrix

### 4.1 Lab environment (this audit)

| Dimension | Value |
| --- | --- |
| OS | Windows 10 (build 26300) |
| Shell | PowerShell |
| Client | Python 3.12 `urllib` crawler + HTML parser (`scripts/site_audit.py`) |
| Network | Residential / standard HTTPS, User-Agent `CynbitQAAudit/1.0` |
| Date / time | 17 Nov 2026, ~05:30–06:15 UTC |
| Edge | Vercel `bom1` (BOM) |
| Auth | Unauthenticated public user only |

### 4.2 Intended device / browser matrix

Recommended coverage for a full release sign-off. **Executed in this pass** vs **still needed**.

| Viewport | Size | Chrome 131+ | Edge | Firefox | Safari iOS | Chrome Android |
| --- | --- | ---: | :---: | :---: | :---: | :---: |
| Mobile small | 360 × 800 | Partial (DOM/CSS only) | Not run | Not run | Not run | Not run |
| Mobile common | 390 × 844 | Partial | Not run | Not run | Not run | Not run |
| Phablet | 430 × 932 | Partial | — | — | — | — |
| Tablet | 768 × 1024 | Partial | Not run | Not run | Not run | — |
| Laptop | 1366 × 768 | Partial | Not run | Not run | — | — |
| Desktop | 1440 × 900 | Partial | Not run | Not run | — | — |
| Large | 1920 × 1080 | Partial | Not run | — | — | — |

“Partial” = viewport meta, responsive class names (`sm:`, `hidden sm:inline`), and a mobile `menu` control were confirmed in HTML; visual overflow, sticky header, and touch targets were **not** screenshot-verified.

### 4.3 User scenario matrix

| ID | Persona | Goal | Result |
| --- | --- | --- | --- |
| U-01 | Founder / buyer | Understand services and contact Cynbit | Contact form present; support email is not independently clickable |
| U-02 | Recruiter / candidate | Find QA Testing Intern and apply | Application form present; several field/UX defects |
| U-03 | Student | Enroll in BitElevate | Enrollment + Cashfree CTA present; fee copy shows ₹1,499 (₹3,999 struck through) |
| U-04 | Intern | Open Intern Portal | `/portal` loads; single “Security Key” field, no username |
| U-05 | Reader | Open a blog / related article | Related slug `.../ai-agents-transforming-business` returns **404** |
| U-06 | Reader | Filter blog by category | Href values contain unencoded spaces |
| U-07 | Lost user | Unknown URL | Custom 404 with Home + Contact; title is the homepage title |
| U-08 | Subscriber | Join newsletter from footer | Footer control is a **`<button>` with no form and no `mailto`** |
| U-09 | Assistive-tech user | Navigate by headings / names | Icon ligature names leak into accessible names; labels not programmatically tied |
| U-10 | Security-conscious user | Check HTTPS, headers, privacy | HTTPS + HSTS present; CSP / frame / MIME / referrer headers missing |

### 4.4 Browser / protocol matrix

| Check | Observed |
| --- | --- |
| `http://www.cynbit.in` → HTTPS | **308** to `https://www.cynbit.in/` |
| `https://cynbit.in` → www | **307 Temporary Redirect** to `https://www.cynbit.in/` |
| `https://www.cynbit.in` | **200** |
| `https://cynbit.com` | DNS fail / not this product |
| `https://www.cynbit.com` | **403** (not the marketing site) |
| TLS | Valid HTTPS via Vercel |
| HSTS | `Strict-Transport-Security: max-age=63072000` (no `includeSubDomains` / `preload` visible) |

## 5. Information architecture (tested)

Primary nav groups (desktop + duplicated mobile drawer):

- **Company:** Story, Team, Careers, Social Impact  
- **Expertise:** Services, Products, Case Studies  
- **BitElevate:** Ecosystem, Learning Paths, Student Diaries  
- **Blog**, **Contact**, **Join Us / Get Started Now**

Footer: phones, `opportunities@cynbit.in`, legal (Terms, Cookies, Privacy), “Join the Newsletter”, social (LinkedIn, Instagram, Facebook, X).

## 6. Coverage summary

| Bucket | Covered | Notes |
| --- | --- | --- |
| Sitemap HTML URLs | 86 × HTTP 200 | 1 intermittent timeout on first fetch of `/case-studies/fintech-payments-evolution` (retry **200**) |
| Extra probes | `/portal` 200, `/admin` `/login` `/apply` 404 (expected) | `/portal` is disallowed in robots but publicly served |
| Broken in-page link | `/blog/ai-agents-transforming-business` | **404**; correct sitemap slug is `/blog/ai-agents-business-operations` |
| Forms inspected | Contact, Careers apply, BitElevate enroll, blog search, footer newsletter | Newsletter footer non-functional |
| Security headers | Homepage + sample inner pages | See Bug report SEC-* |
| Accessibility static | All parsed HTML pages | No skip link; duplicate `<main>` on several templates |
| Performance | Fetch timings for 95 URLs | Several HTML documents 3–11 s TTFB in this run |

## 7. Limitations and residual risk

1. **No headed browser** in this environment (Cursor browser MCP did not register). Layout overflow, hover menus, focus rings, and Cashfree modal behavior need a follow-up pass in Chrome DevTools device mode.
2. **No production form submit** — client `required` attributes were reviewed; server-side validation, success toasts, and email routing are unverified.
3. **LinkedIn returned 403/999** to the crawler; that is typical bot filtering, not evidence the company page is down.
4. **Single geography / single UA** — CDN cache (`X-Vercel-Cache: HIT`, homepage `Age` ~42 hours) may differ for other regions.

Residual risk is highest on: **BitElevate payment**, **career apply API**, **portal auth**, and **mobile nav / cookie banner**, because those are JS-interactive.

## 8. Evidence pack (working files)

| File | Contents |
| --- | --- |
| `audit_raw.json` | Per-URL status, timings, headings, forms, headers |
| `scripts/site_audit.py` | Repeatable crawler |
| `scripts/deep_probe.json` | Portal, 404, category encoding, JSON-LD, privacy keywords |
| `scripts/*.html` | Saved DOM snapshots of key pages |

## 9. Overall quality assessment

The site is a polished Next.js marketing surface with a complete sitemap, custom 404, HTTPS, and coherent IA. It is **not yet intern-portal / careers-grade in form accessibility or conversion reliability**. The issues most likely to hit a real user in the next 48 hours are:

1. Footer newsletter and support-email affordances that look clickable but do the wrong thing or nothing.
2. A related-blog 404.
3. Career score validation that cannot represent percentage marks.
4. Assistive-tech noise from Material Symbol ligature text.
5. Missing clickjacking / CSP / MIME-sniffing headers on a site that takes applications and payments.

Detailed defects: `02_Bug_and_Issue_Report.md`  
Fix order: `03_Improvement_Recommendations.md`
