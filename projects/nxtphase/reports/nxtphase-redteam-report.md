# Red Team & Security Assessment, nxtphase.ai

- Target: https://nxtphase.ai
- Date: 2 June 2026
- Authorization: Owner-requested self-assessment
- Method: Passive, non-destructive analysis (no exploitation, fuzzing, brute forcing or DoS)
- Stack: Webflow + Cloudflare + Microsoft 365

This markdown is the editable source. The reader-facing version is `nxtphase-redteam-report.html`.

## Executive summary

The site is a well built, lightweight Webflow marketing site behind Cloudflare. Transport security is strong, no secrets or trackers are exposed, and the single custom script is clean. The meaningful gaps are in hardening and compliance, not in exploitable code: no Content Security Policy, no DMARC or DKIM (domain spoofable in email), and no working privacy or cookie policy even though a contact form collects personal data.

No critical or directly exploitable vulnerability was found. Risk profile: moderate, dominated by low-effort fixes that mostly live in the Cloudflare and Microsoft 365 dashboards.

Severity counts: High 3, Medium 5, Low 7, Critical 0.

### Fix first

1. Publish DMARC and enable DKIM (H2).
2. Create and wire up privacy + cookie policy; footer links are dead and hidden (H3).
3. Add CSP and security headers at Cloudflare (H1, M1).
4. Add anti-automation to the contact form; make consent a real, logged checkbox linking to the policy (M3, L5).
5. Restore visible keyboard focus; stop using light brand green for normal text (M4, M5).

## Findings register

| ID | Finding | Severity | Category |
|----|---------|----------|----------|
| H1 | No Content Security Policy | High | Hardening |
| H2 | Email spoofing possible: no DMARC, no DKIM | High | Email |
| H3 | No working privacy or cookie policy (GDPR) | High | Compliance |
| M1 | Clickjacking: no X-Frame-Options / frame-ancestors | Medium | Hardening |
| M2 | No Subresource Integrity on GSAP and Slater scripts | Medium | Supply chain |
| M3 | Contact form has no CAPTCHA or rate limiting | Medium | Abuse |
| M4 | Keyboard focus indicators removed | Medium | WCAG 2.4.7 |
| M5 | Brand green text fails contrast (3.22:1) | Medium | WCAG 1.4.3 |
| L1 | HSTS without includeSubDomains or preload | Low | Hardening |
| L2 | Missing nosniff, Referrer-Policy, Permissions-Policy | Low | Hardening |
| L3 | No CAA DNS record | Low | DNS |
| L4 | Contact form declared method="get" (PII in URL fallback) | Low | Privacy |
| L5 | Consent text is not a recorded checkbox and links nowhere | Low | Compliance |
| L6 | lang="nl-NL" with English homepage content | Low | WCAG 3.1 |
| L7 | Aging front-end libraries (jQuery 3.5.1, GSAP) | Low | Hygiene |

## High severity

### H1, No Content Security Policy
No `Content-Security-Policy` on any page (HEAD and GET, no meta CSP). The site loads executable JS from four external origins with no allowlist. Without CSP, a compromise of any origin or a stored XSS via CMS content runs unconstrained against the DOM and contact form. Fix: add a CSP at Cloudflare (Transform Rules), start report-only then enforce. Webflow needs `'unsafe-inline'` for script/style; even so, CSP still blocks unknown external origins and framing.

### H2, Email spoofing: no DMARC and no DKIM
SPF is correct with hard fail (`-all`). But `_dmarc.nxtphase.ai` is empty and the M365 DKIM selectors (`selector1/2._domainkey`) are not published. SPF alone does not protect the visible From address. An attacker can phish as nxtphase.ai. Fix: enable DKIM in Microsoft 365 Defender and publish the CNAMEs; publish DMARC starting `p=none` with `rua`, then tighten to `quarantine`/`reject`.

### H3, No working privacy or cookie policy
Footer legal links are placeholders, both non-functional and hidden: `<a href="#" class="footer_legal-link hide">Privacy</a>` and `Cookies`. All probed policy URLs return 404. The contact form collects name, company, email, message and states "Ik ga akkoord met de verwerking van mijn gegevens". A Dutch company processing personal data must provide an accessible privacy statement (GDPR). Fix: publish privacyverklaring + cookie statement as Webflow pages, point the footer links at them, remove `hide`, and link the privacy statement from the consent line.

## Medium severity

### M1, Clickjacking
No `X-Frame-Options` and no CSP `frame-ancestors`; the site can be framed anywhere. Limited impact (no authenticated actions) but enables UI-redress and brand-impersonation. Fix: `X-Frame-Options: DENY` and `frame-ancestors 'none'`.

### M2, No SRI on GSAP and Slater
jQuery and Webflow chunks carry `integrity`; the five GSAP scripts and the custom Slater script do not. The Slater file sits on a third-party S3 bucket. If Slater is compromised, attacker JS runs on every page with no SRI and no CSP to stop it. Fix: move Slater code into Webflow custom code (first-party), or pin with SRI, plus the H1 CSP.

### M3, Contact form has no anti-automation
No reCAPTCHA, Turnstile, hCaptcha or honeypot. Open to spam and automated abuse; sustained submissions can exhaust the Webflow monthly form quota and silently drop real enquiries. Fix: enable Cloudflare Turnstile (privacy-friendly, no cookie banner needed).

### M4, Focus indicators suppressed (WCAG 2.4.7)
CSS has 8x `outline: 0` plus `outline: none`, and zero `:focus-visible` rules; only one `outline: 2px solid #2895f7` (Webflow default). Buttons, links and the custom journey tabs likely lose their focus ring. Fails Focus Visible (AA). Fix: add a global `:focus-visible` outline. Confirm on the live page.

### M5, Brand green fails contrast (WCAG 1.4.3)
Computed sRGB contrast: green `#3E9B5D` on cream `#F9F6F1` = 3.22:1 (fail normal, pass large); white on `#3E9B5D` = 3.47:1 (fail normal); dark green `#2D6F44` on cream = 5.62:1 (pass); white on `#2D6F44` = 6.05:1 (pass). Use `#2D6F44` for normal-size green text, links and button labels; reserve `#3E9B5D` for large display and decoration.

## Low severity

- L1, HSTS present but lacks `includeSubDomains` and `preload`. After verifying subdomains use HTTPS, set `max-age=63072000; includeSubDomains; preload` and submit to the preload list.
- L2, Missing `X-Content-Type-Options: nosniff`, `Referrer-Policy` (use `strict-origin-when-cross-origin`), `Permissions-Policy` (disable unused features). Add at Cloudflare.
- L3, No CAA record; any CA may issue. Add `0 issue "pki.goog"` plus your Cloudflare CAs.
- L4, Contact form markup is `method="get"` with no action. Webflow usually AJAX-submits, but if JS fails the native GET puts PII in the URL/logs/history. Set method to POST.
- L5, Consent is a static text line, not a checkbox (zero checkboxes on the page) and links to no policy. Cannot evidence consent (GDPR Art. 7). Prefer a clear notice + privacy link (legitimate-interest basis) and pair with H3.
- L6, `lang="nl-NL"` but the homepage is English; other pages Dutch. Set `lang` per page; mark inline language switches. (WCAG 3.1.1/3.1.2)
- L7, jQuery 3.5.1 (2020) and GSAP 3.15.0 are dictated by Webflow. No active vuln in this static context (3.5.0 XSS fixes included); keep current as Webflow updates. Hygiene only.

## Cookie and tracking analysis

Good on privacy, weak on documentation.

| Cookie | Set by | Flags | Purpose | Consent |
|--------|--------|-------|---------|---------|
| `_cfuvid` | Cloudflare (server) | HttpOnly; Secure; SameSite=None; session | Cloudflare rate limiting / bot management | Generally classed strictly necessary |

- No analytics, tag manager or marketing pixels anywhere (no GA, GTM, Meta/LinkedIn pixel, Hotjar, HubSpot). Confirmed across home, contact, case-study pages and the custom script.
- No consent platform present, and none is needed at the current cookie level.
- The only cookie is Cloudflare `_cfuvid`, documented as strictly necessary and not used for cross-site tracking.

The mistake is documentation, not over-tracking: even a strictly-necessary cookie should be disclosed in a cookie statement, and that link is currently dead (H3, L5). Keep the privacy-friendly posture; if analytics are added later, use a cookieless option (Cloudflare Web Analytics or Plausible).

## Custom code reverse engineering

The only developer-authored JS is the Slater file `61392.js` (22 KB), fully decompiled and read. It is decorative animation only: `initHeroGlow()` (WebGL shader glow), plus GSAP scroll animations, tabbed cards and SVG illustrations.

Security review of the custom code:

- No dangerous sinks (`eval`, `Function()`, `innerHTML`, `document.write`): none.
- No network calls, beacons, cookies, storage or fingerprinting: none.
- No hard-coded secrets/keys/tokens: none.
- Reads only developer-set `data-*` attributes (no untrusted input).
- Accessibility-aware: respects `prefers-reduced-motion`, sets `aria-selected`, arrow-key tab navigation.

The code is clean; the only residual risk is hosting (third-party Slater origin, no SRI/CSP, see M2/H1). Wider checks found no exposed keys, no `.env` or source-map leakage, no admin/staging endpoints in the sitemap. No server-side application exists (fully managed by Webflow).

## Accessibility (WCAG 2.1 AA) summary

Pass: page language present, single h1 with logical nesting, 16/16 images have alt, all form fields labelled, reduced-motion respected.
Fail: Focus Visible 2.4.7 (M4), Contrast 1.4.3 (M5).
Minor: language of parts 3.1 (L6).

Recommend a full audit to also verify rendered contrast on real components, focus order through sticky tabs, and `aria-hidden` on decorative SVGs.

## What is done well

TLS 1.3 with strong cert, HTTP to HTTPS 301, HSTS enabled, SPF hard fail, SRI on jQuery + Webflow, no exposed secrets, no third-party trackers, clean custom code, Cloudflare WAF, good heading/alt structure, reduced-motion aware. The privacy-by-default posture (zero analytics, zero marketing cookies) is above average; preserve it.

## Remediation roadmap

Quick wins (dashboard, hours): DKIM + DMARC (H2); privacy/cookie pages + footer links (H3, L5); Cloudflare Turnstile (M3); CAA record (L3).

Short term (Cloudflare header rules, days): CSP report-only then enforce, plus X-Frame-Options, nosniff, Referrer-Policy, Permissions-Policy, stronger HSTS (H1, M1, L1, L2); focus styles and green-text colour fix (M4, M5); form to POST and privacy link on consent (L4, L5).

Hardening (project decisions): in-house or SRI-pin the Slater script then tighten CSP (M2); re-check library versions on Webflow updates (L7); per-page lang and inline language marking (L6).

## Appendix, evidence

Response headers (home, GET): `HTTP/2 200`; `strict-transport-security: max-age=31536000` (no includeSubDomains/preload); `set-cookie: _cfuvid=...; HttpOnly; SameSite=None; Secure`; `server: cloudflare`. Absent: CSP, X-Frame-Options, X-Content-Type-Options, Referrer-Policy, Permissions-Policy.

DNS/email: A `198.202.211.1` (Cloudflare); no AAAA; NS `*.ns.cloudflare.com`; MX `nxtphase-ai.mail.protection.outlook.com`; SPF `v=spf1 include:spf.protection.outlook.com -all` OK; DMARC empty (MISSING); DKIM selector1/2 empty (MISSING); CAA none (MISSING).

External script origins: `cdn.prod.website-files.com` (Webflow chunks SRI, GSAP 3.15.0 no SRI); `d3e54v103j8qbb.cloudfront.net` (jQuery 3.5.1 SRI); `assets.slater.app` (custom 61392.js, no SRI, third-party S3).

Contrast (sRGB, WCAG 2.x): `#3E9B5D`/`#F9F6F1` 3.22:1; `#FFFFFF`/`#3E9B5D` 3.47:1; `#2D6F44`/`#F9F6F1` 5.62:1; `#FFFFFF`/`#2D6F44` 6.05:1.

Scope/method: passive analysis of public content only; no exploitation, fuzzing, brute forcing or DoS. Tools: curl, openssl, dig, static HTML/CSS/JS analysis, custom-script reverse engineering, deterministic WCAG contrast calculation, well-known-path probing. Raw artifacts under `nxtphase-redteam/raw/`. M4 and M5 are high-confidence from source and should be confirmed on the live rendered page.
