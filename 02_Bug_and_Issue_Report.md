# Bug & Issue Report — Cynbit Website

**Application:** https://www.cynbit.in  
**Audit date:** 17 November 2026  
**Environment:** Unauthenticated public user, Windows 10, HTTPS crawler + HTML inspection  
**Related:** `01_QA_Audit_Report.md`

Legend: **S1** Critical · **S2** High · **S3** Medium · **S4** Low  
Status of all items: **New** (open)

---

## Summary

| Severity | Count |
| ---: | ---: |
| S1 Critical | 2 |
| S2 High | 11 |
| S3 Medium | 16 |
| S4 Low | 8 |
| **Total** | **37** |

| Type | Count |
| --- | ---: |
| Functional / broken | 10 |
| Usability | 8 |
| Accessibility | 8 |
| Responsiveness / visual | 3 |
| Performance | 2 |
| Security / privacy | 6 |

---

## S1 — Critical

### BUG-001 — Footer “Join the Newsletter” does not subscribe anyone

| | |
| --- | --- |
| **Type** | Functional |
| **Severity** | S1 |
| **Pages** | Global footer (home, contact, careers, legal, …) |
| **Steps** | 1. Open any marketing page. 2. Scroll to “Stay Connected” / “Subscribe to our Briefing”. 3. Click **Join the Newsletter**. |
| **Expected** | Email field + submit, or navigation to a subscribe provider. |
| **Actual** | Control is a `<button>` **outside any `<form>`**, with no `type` action, no `href`, and no email input on most pages. Copy promises “Join 25k+ architects”. |
| **Impact** | Primary footer conversion is a dead control. Users believe they subscribed. |
| **Evidence** | `scripts/home.html` — button markup adjacent to the 25k+ copy; `home_form_tags` on homepage = 0. |

### BUG-002 — Public BitElevate enrollment charges money with no bot protection

| | |
| --- | --- |
| **Type** | Security / functional risk |
| **Severity** | S1 |
| **Page** | https://www.cynbit.in/bitelevate |
| **Steps** | Inspect enrollment form (name, university, email, phone, track, duration) and submit CTA **Pay ₹1,499 & Enroll via Cashfree**. |
| **Expected** | CAPTCHA / rate limit / Terms acceptance before payment gateway; clear refund and credential-provisioning copy. |
| **Actual** | No reCAPTCHA / hCaptcha / Turnstile in page HTML or marketing JS chunk. Payment keywords: Cashfree, UPI, PhonePe. No file in this audit submitted a payment. |
| **Impact** | Spam enrollments and payment abuse against a live gateway; operations load on “instant credential generation”. |
| **Evidence** | `scripts/deep_probe.json` → `hints_bitelevate.html.captcha = false`; chunk `app/(marketing)/bitelevate/page-*.js` contains Cashfree. |

---

## S2 — High

### BUG-003 — Related blog URL 404s (slug mismatch)

| | |
| --- | --- |
| **Type** | Broken link |
| **Severity** | S2 |
| **Steps** | Open in-page link `https://www.cynbit.in/blog/ai-agents-transforming-business` |
| **Expected** | Article, or redirect to the live slug. |
| **Actual** | **HTTP 404**, custom “Node Not Found” page. Sitemap / live article is `https://www.cynbit.in/blog/ai-agents-business-operations`. |
| **Impact** | Readers and Googlebot hit a dead related-post. |
| **Evidence** | `audit_raw.json` `broken_links`; retry 17 Nov 2026 still 404. |

### BUG-004 — `support@cynbit.in` is shown but not mailed

| | |
| --- | --- |
| **Type** | Functional / usability |
| **Severity** | S2 |
| **Page** | https://www.cynbit.in/contact |
| **Steps** | Under “Mail us at:”, observe `opportunities@cynbit.in` and `support@cynbit.in`. Click the block. |
| **Expected** | Each address is its own `mailto:`; support inquiries go to support. |
| **Actual** | Both lines sit inside **one** `<a href="mailto:opportunities@cynbit.in">`. There is **no** `mailto:support@cynbit.in` in the document. Footer “Contact Support” also does not target that inbox. |
| **Impact** | Support traffic is misrouted or users copy the wrong mental model. |
| **Evidence** | `contact_mailto` = only `mailto:opportunities@cynbit.in`; HTML snippet in `scripts/extract_ui.json` → `support_html_around`. |

### BUG-005 — Career “Score Value” is capped at 10 even for percentage scoring

| | |
| --- | --- |
| **Type** | Functional |
| **Severity** | S2 |
| **Page** | https://www.cynbit.in/careers/qa-testing-intern (and other role templates) |
| **Steps** | 1. Open application. 2. Set Score System to percentage (label exists). 3. Enter `84`. |
| **Expected** | Max 100 for %; max 10 for CGPA/GPA. |
| **Actual** | Score input is `type="number" step="0.01" min="0" max="10"` with **no** `name` on that input in the SSR HTML, and max does not change with score system. |
| **Impact** | Applicants with % marks cannot enter a valid score; some score fields may not POST. |
| **Evidence** | `scripts/extract_ui.json` → `career_input_types`. |

### BUG-006 — Phone / WhatsApp fields are plain text, not telephone controls

| | |
| --- | --- |
| **Type** | Functional / mobile usability |
| **Severity** | S2 |
| **Pages** | Career apply; BitElevate enroll |
| **Expected** | `type="tel"`, `autocomplete="tel"`, and a simple pattern (`+91…` or 10 digits). Mobile shows numeric keypad. |
| **Actual** | `placeholder="+91..."` but `type` omitted (defaults to text). No `pattern`, no `autocomplete`. |
| **Impact** | Typos, harder mobile entry, weaker server validation. |

### BUG-007 — Form labels are visible but not programmatically associated

| | |
| --- | --- |
| **Type** | Accessibility |
| **Severity** | S2 |
| **WCAG** | 1.3.1, 4.1.2 |
| **Pages** | Contact, careers apply, BitElevate, products inquiry |
| **Actual** | `<label>` text exists, but inputs have **no `id`**, labels have **no `for`**, and inputs have **no `aria-label`**. Placeholder is the only hint for AT in many fields. |
| **Impact** | Screen readers may announce “edit text, blank” instead of “Email Address, required”. |
| **Evidence** | Parser: contact 5 unlabeled inputs; career apply 22; BitElevate 8. |

### BUG-008 — Material icon ligatures leak into accessible names

| | |
| --- | --- |
| **Type** | Accessibility |
| **Severity** | S2 |
| **WCAG** | 2.5.3, 4.1.2 |
| **Actual** | Icon glyphs are text nodes (`expand_more`, `arrow_forward`, `east`, `menu`, `alternate_email`, `west`). Buttons become “Companyexpand_more”, “Join the Newslettereast”, “Start discovery arrow_forward”. |
| **Impact** | Every page header/footer is noisy or unintelligible with a screen reader. If the icon font fails, sighted users also see raw names. |
| **Fix hint** | `aria-hidden="true"` on icon spans; keep visible text only. |

### BUG-009 — Social links and logo links have no accessible name

| | |
| --- | --- |
| **Type** | Accessibility |
| **Severity** | S2 |
| **WCAG** | 2.4.4, 4.1.2 |
| **Actual** | Empty `<a href="/">` (logo, twice) and empty social anchors to LinkedIn, Instagram, Facebook, X — icon-only, no `aria-label`. Repeated on every page. |
| **Evidence** | `empty_links` in `audit_raw.json` for essentially all HTML pages. |

### BUG-010 — Missing standard security headers on HTML responses

| | |
| --- | --- |
| **Type** | Security |
| **Severity** | S2 |
| **Actual** | Present: `Strict-Transport-Security: max-age=63072000`. **Absent** on homepage: `Content-Security-Policy`, `X-Frame-Options` / `frame-ancestors`, `X-Content-Type-Options`, `Referrer-Policy`, `Permissions-Policy`. |
| **Impact** | Clickjacking of contact/apply/pay pages; MIME sniffing; referrer leakage of portal URLs. |
| **Evidence** | `audit_raw.json` → `security.homepage`. |

### BUG-011 — `Access-Control-Allow-Origin: *` on document responses

| | |
| --- | --- |
| **Type** | Security |
| **Severity** | S2 |
| **Actual** | Homepage HTML (and several other routes, including 404 HTML) send `Access-Control-Allow-Origin: *`. |
| **Impact** | Unnecessary CORS relaxation on a credentialed-product marketing site; often a mis-set Vercel/Next header. |

### BUG-012 — Intern portal is public, robots-disallowed, and username-less

| | |
| --- | --- |
| **Type** | Security / usability |
| **Severity** | S2 |
| **URL** | https://www.cynbit.in/portal (**200**) |
| **robots.txt** | `Disallow: /portal/` (hides from honest crawlers; does not hide from people) |
| **UI** | Title “Cynbit Portal”. Copy: “Enter your credentials to continue”. Single field **Security Key** + **Access Portal**. No email/username, no “forgot key”, no visible MFA. |
| **Impact** | Shared-secret design is easy to leak in intern WhatsApp groups; no account lockout UX. **Not exploited in this audit.** |

### BUG-013 — No skip-to-content link; keyboard users repeat the mega-nav every page

| | |
| --- | --- |
| **Type** | Accessibility |
| **Severity** | S2 |
| **WCAG** | 2.4.1 |
| **Actual** | `skip_link = false` on parsed pages. Nav is duplicated (desktop + mobile drawer) before main content. |

---

## S3 — Medium

### BUG-014 — Duplicate document titles (`… | Cynbit Technologies | Cynbit Technologies`)

**SEO / usability.** Home, Contact, Careers, Blog, Privacy, Terms, Cookies, Story. Browser tabs and SERP titles are redundant. OG titles inherit the same string.

### BUG-015 — Sitemap and JSON-LD use apex host; live site is `www`

- `robots.txt` → `Sitemap: https://cynbit.in/sitemap.xml`
- `<loc>` values are `https://cynbit.in/...`
- JSON-LD `@id` / `url` = `https://cynbit.in`
- Users and curl land on `https://www.cynbit.in`

Apex HTTPS uses **307 Temporary Redirect**, which is a weak signal for canonicalization (prefer **301**).

### BUG-016 — Blog category URLs contain unencoded spaces

Example href: `/blog?category=AI Agents` (also AI Automation, Web Development, …). Spec-wise invalid. Browsers often encode on click; some tools, emails, and crawlers will not. Encoded form `?category=AI%20Agents` returns 200.

### BUG-017 — Custom 404 uses the homepage `<title>`

`https://www.cynbit.in/this-page-should-404-xyz` → body is a proper 404 (“Node Not Found”, links **Return to Base** `/` and **Report Broken Node** `/contact`) but `<title>` is “Cynbit Technologies | AI Automation & Software Solutions Jaipur”. Bookmarks and analytics cannot distinguish errors.

### BUG-018 — Multiple `<main>` landmarks on the same page

Contact, BitElevate, Cookies, Privacy, Blog index report **2 × `<main>`**. Landmark navigation becomes ambiguous (WCAG 1.3.1 / 2.4.1).

### BUG-019 — Heading outline is flattened / skipped

H1 nodes concatenate the visual H1 plus the following paragraph (“Architecting the future withAI automation.” — **missing space** after “with”). Cookie policy goes H1 → H3 (skips H2). FAQ items on Contact are H4 under H2.

### BUG-020 — Career apply collects Date of Birth + Gender as required, with no resume file upload

For a QA intern role this is heavy PII. Resume is **URL-only** (`portfolioUrl`, Google Drive). Candidates without a public link are blocked. Gender is a required `<select>`. Privacy policy does not mention DPDP 2023, lawful basis, or retention.

### BUG-021 — No `autocomplete` on long application / payment identity fields

Name, email, tel, address, DOB would autofill if `autocomplete="name|email|tel|bday|street-address"` were set. 20+ fields increases drop-off.

### BUG-022 — Contact page omits the third public phone

Footer: `+91 89479 62667`, `+91 78912 09777`, `+91 83023 37367`. Contact “Contact us at:” lists only the first two.

### BUG-023 — `favicon.ico` 404s; Apple touch icon is an SVG

HTML correctly points to `/favicon.svg` (**200**). Browsers still request `/favicon.ico` → **404**. `rel="apple-touch-icon" href="/favicon.svg"` is poorly supported on iOS vs PNG 180×180.

### BUG-024 — Cookie policy describes analytics cookies; no consent UI in SSR

Policy (updated 20 May 2026) lists Google Analytics and functional cookies, and claims “session tokens, CSRF protection”. Saved homepage HTML has **no Accept / Reject cookie banner**. If GA loads client-side without consent, this is a DPDP / GDPR gap; if it does not load, the policy is inaccurate.

### BUG-025 — Privacy policy is generic vs actual data collected

Policy talks about name/email/professional context. Production forms also collect DOB, gender, WhatsApp, permanent address, academic scores, LinkedIn, and payment identity (BitElevate). No DPDP / GDPR / CCPA keywords in `privacy.html`.

### BUG-026 — Several HTML responses were slow (3–11 s) in this run

| URL | Fetch time |
| --- | ---: |
| `/case-studies/fintech-payments-evolution` | 25.3 s timeout (retry later 200 / 0.7 s class) |
| `/case-studies/evento-ticketing-scalability` | 11.5 s |
| `/diaries/attendance-chase` | 9.0 s |
| `/team` | 5.6 s |
| Several career + blog pages | 3.5–4.7 s |

Homepage can be fast (`~345 ms`, Vercel HIT). Inconsistent TTFB hurts mobile users on 4G.

### BUG-027 — JSON-LD geo coordinates look rounded placeholders

`"latitude":"26.8000","longitude":"75.8000"` for 32/115 Pratap Nagar, Jaipur. Map deep-links and rich results may pin the wrong spot.

### BUG-028 — Mobile menu control relies on the word `menu` as icon text

Hamburger is a `<button>` whose text is the ligature `menu`, without `aria-label="Open menu"` / `aria-expanded`. Combined with a duplicated nav, mobile AT and keyboard behavior is uncertain without a device pass.

### BUG-029 — Tiny type (`text-[9px]`, `text-[10px]`) on primary CTAs and labels

Home: 11× `text-[9px]`, 44× `text-[10px]`. Footer newsletter button uses `text-[10px]` uppercase. Below comfortable reading size (~12 px) and likely WCAG 1.4.4 / 1.4.8 issues on 360 px screens.

---

## S4 — Low

### BUG-030 — `/admin`, `/login`, `/apply` are 404s while copy refers to an Intern Portal

Not a leak (good), but marketing says “private Intern Portal” without a discoverable “Intern login” link in the main nav. Users search `/login` and get the generic 404.

### BUG-031 — No `/.well-known/security.txt`

Missed chance for a vulnerability disclosure contact (`opportunities@` is the only public mail).

### BUG-032 — HSTS missing `includeSubDomains` and `preload`

Fine for a single www host; weaker if `api.` or `portal.` subdomains appear later.

### BUG-033 — Apple / PWA: `/manifest.json` 404

No installable PWA; only report if that was intended.

### BUG-034 — FAQ heading “Common Inquiries.” uses H2 while answers are H4

Minor outline gap (see BUG-019).

### BUG-035 — `Crawl-delay: 1` in robots.txt

Ignored by Google; may slow polite crawlers. Low impact.

### BUG-036 — Testimonials and ISO / “25k+” / “1,000+ interns” claims are unverifiable from the UI

Marketing risk if a user or college coordinator asks for proof. Not a code defect.

### BUG-037 — Third-party LinkedIn URLs returned 403/999 to the crawler

Likely bot protection. **Do not treat as a confirmed broken social profile** without a logged-in browser check.

---

## Issues checked and **not** filed as bugs

| Check | Outcome |
| --- | --- |
| HTTP → HTTPS | 308 permanent-style redirect; OK |
| Trailing slash on job URL | `/careers/qa-testing-intern/` canonicalizes to non-slash 200 |
| `lang="en"` + viewport | Present on 200 HTML pages |
| Image `alt` in SSR | No missing `alt` attributes in parsed sitemap HTML |
| `target=_blank` without `noopener` | Not observed on sampled pages |
| GitHub lab repo | `github.com/Cynbit-Technologies/bitelevate-fullstack-lab` **200** |
| Instagram / Facebook / X | **200** |
| `/admin` exposed admin UI | **404** (good) |
| Custom 404 vs blank Vercel error | Custom page exists (title still wrong — BUG-017) |

---

## Suggested ticket fields (for Linear / Jira import)

`ID | Title | Severity | Type | URL | Repro | Expected | Actual | Evidence`
