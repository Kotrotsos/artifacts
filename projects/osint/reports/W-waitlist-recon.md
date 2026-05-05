# Waitlist / signup recon — `unomundi.com/signup`

**Engagement:** unomundi.com pentest, 2026-05-05.
**Scope:** marketing site waitlist as the only live production-data path (product is pre-launch).
**Method:** static HTML analysis + headless Chromium with network capture + privacy-policy review + cookie inventory. One bounded form-submission attempt with a clearly-marked test email, gated by reCAPTCHA before any data left the browser.

## Summary

The waitlist itself is **technically well-defended against abuse** — reCAPTCHA Enterprise gates submission, no rate-limit-bypass observed, Wix's first-party form telemetry handles delivery. However, the surrounding **privacy / consent / data-controller story has multiple GDPR transparency gaps** that matter specifically because the product collects waitlist emails on behalf of a children's app:

| ID | Severity | Finding |
|----|----------|---------|
| **W-1** | **MEDIUM** | Data-controller identity is ambiguous: privacy policy names `Una's World Holding B.V` (NL, KvK #98204637); footer claims `Una's World Holding Limited` (UK) |
| **W-2** | **MEDIUM** | Privacy policy does not state the legal basis for processing (GDPR Art. 13(1)(c) violation) |
| **W-3** | **MEDIUM** | Privacy policy does not name sub-processors (Wix, Mailjet, Google reCAPTCHA Enterprise, ElevenLabs) — GDPR Art. 13(1)(e) |
| **W-4** | **MEDIUM** | Privacy policy does not state data retention period — GDPR Art. 13(2)(a) |
| **W-5** | **MEDIUM** | Privacy policy contains no children's-data section — neither COPPA nor GDPR-K addressed despite the product's audience |
| **W-6** | **MEDIUM** | No cookie consent banner; 5 cookies (including 2 with SameSite=None) set before any user interaction |
| **W-7** | **LOW** | Form has no consent checkbox / opt-in for marketing communications (e-Privacy / PECR expectation when consent is the only valid basis for marketing) |
| **W-8** | **LOW** | Form mismatch with stated policy: form collects only `email`; policy says "name, email, and role" are collected |
| **W-9** | **INFO** | Public ElevenLabs voice agent ("Start a call" → ElevenAgents) is embedded on the marketing site — separate attack surface (cost abuse, prompt injection); no captcha gate observed for voice session start |
| **W-10** | **POSITIVE** | reCAPTCHA Enterprise gates the actual form submission; abuse / mass-stuffing is mitigated |

## Form anatomy

```html
<form id="comp-mkqj8dpc2" class="JVi7i2 wixui-form">
  <input name="email"
         type="email"
         placeholder="Email for Updates"
         pattern="^.+@.+\.[a-zA-Z]{2,63}$"
         required>
  <button>Join the waitlist</button>
</form>
```

- **One field only:** email
- **Validation:** HTML5 + regex (server-side validation occurs at the Wix backend)
- **No name field, no consent checkbox, no marketing opt-in, no captcha checkbox visible until after click**
- The captcha is shown *after* the user clicks "Join the waitlist" — not before — so the captcha pre-loaded scripts (Google reCAPTCHA Enterprise) are loaded without explicit consent

## Submission flow (captured live)

1. User clicks `Join the waitlist`
2. Browser POSTs to `https://frog.wix.com/form-builder` (Wix form-builder telemetry collector) — returns 204
3. Browser POSTs to `https://panorama.wixapps.net/api/v1/bulklog` (Wix global telemetry) — returns 204
4. Wix injects reCAPTCHA Enterprise widget (site key `6Ld0J8IcAAAAANyrnxzrRlX1xrrdXsOmsepUYosy`)
5. UI changes to "Verification - Please confirm you're human"
6. Submission of the email is paused until captcha is solved
7. (Not exercised — captcha bypass is exactly what the captcha is designed to stop, and the engagement question was answered)

Screenshot of the captcha state: `web/waitlist/captcha-dialog.png` (469 KB).

## Cookies set on first visit (no consent gate)

```
server-session-bind   httpOnly Lax   www.unomundi.com         (session)
XSRF-TOKEN            -        None  .www.unomundi.com        (session)
hs                    httpOnly Lax   .www.unomundi.com        (session)
svSession             httpOnly None  .www.unomundi.com        (~3 years)
bSession              -        None  .www.unomundi.com        (session)
```

`svSession` is the long-lived Wix server-vault session cookie; `bSession` is the Wix browser session. Both are set with `SameSite=None`, which is required for cross-site embedding but means they are sent on every cross-site request the browser makes. Under GDPR / e-Privacy, only strictly-necessary cookies may be set without consent. Wix classifies these as "necessary for site operation," but Dutch and UK regulators have historically taken a stricter view, especially when the same cookies are used for analytics or behavioral profiling.

The page also injects:
- `frog.wix.com` (Wix first-party telemetry pixel)
- `panorama.wixapps.net/api/v1/bulklog` (Wix global telemetry)
- `Sentry CDN` (`browser.sentry-cdn.com/7.120.3/...`) — error monitoring loaded by Wix
- `bo.wix.com/suricate/` (Wix performance monitoring)
- Google reCAPTCHA Enterprise (loaded after submit click, but pre-loaded scripts may be cached)
- ElevenLabs ConvAI / ElevenAgents (loaded for the "Start a call" widget)

None of these are named in the privacy policy.

## Privacy-policy compliance gaps (against GDPR Art. 13)

GDPR Article 13 requires that a data controller, when collecting personal data directly from a data subject, provide specific information at the time of collection. The current policy at `https://www.unomundi.com/english-privacy-policy` does not satisfy several mandatory elements:

| GDPR Art. 13 requirement | Status in current policy |
|---|---|
| Identity and contact details of the controller (Art. 13(1)(a)) | **Ambiguous** — policy text names B.V., footer says Ltd. |
| Contact details of the DPO (Art. 13(1)(b)) | Not stated; for a kids'-tech controller the DPO question matters |
| Purposes of processing AND legal basis (Art. 13(1)(c)) | Purposes stated (loosely); **legal basis missing** |
| Recipients / categories of recipients (Art. 13(1)(e)) | **Not named** — Wix, Mailjet, Google, ElevenLabs are all processors |
| Storage period (Art. 13(2)(a)) | **Not stated** |
| Right to lodge a complaint with a supervisory authority (Art. 13(2)(d)) | Generic GDPR rights mentioned; specific supervisory authority (AP for NL, ICO for UK) not named |
| Existence of automated decision-making (Art. 13(2)(f)) | Not addressed; the AI agent ("Una") on the site arguably falls under this once the kids' product launches |

Additional issues for the children's-product context (GDPR Art. 8 / Recital 38; UK Children's Code; COPPA where applicable):
- No explicit children's-privacy notice
- No statement about parental consent processes for the upcoming app
- No age verification / age-appropriate language

Also worth noting: although the waitlist signup is collecting *parents'* emails (not children's), the entire data flow exists to enroll the parent in onboarding for a children's product. UK Information Commissioner's Office has been clear that "the privacy notice for the parent must explain how the child's data will be handled in the eventual product" — this is missing.

## Two entities, one site

The page footer literally renders both entity strings:
```
Copyright © 2025 Una's World Holding BV. All Rights Reserved.
Copyright © 2026 Una's World Holding Limited. All Rights Reserved.
```

Privacy policy text references: `Una's World Holding B.V, based in the Netherlands (Chamber of Commerce #98204637)`.

OSINT confirmed:
- KvK (Dutch CoC) number `98204637` is the NL entity
- LinkedIn states "recently moved to UK incorporation" — explains the appearance of `Limited`

For the engagement disclosure, this matters because:
- The breach controller obligations under Art. 33 fall on whichever entity is the controller
- The data-subject contracts (consent, ToS, privacy policy) must clearly identify the counterparty
- Two entities + one privacy policy + one cookie banner (none) = compliance fog

The asset owner should pick one controller, name it unambiguously, document the data-transfer between B.V. and Ltd. (if both process), and put a `data-protection-officer@…` or equivalent address in the policy.

## Public ElevenLabs voice agent — separate attack surface

While walking the signup page, captured: `@e10 [button] "Start a call"` and `@e12 [link] "ElevenAgents"`. The bottom of every page ("Have a chat with Una") embeds an ElevenLabs ConvAI widget — a voice AI any visitor can converse with. Loaded scripts include the ElevenAgents widget bundle.

This is not a waitlist concern but worth flagging because:
- Voice AI agents are a known prompt-injection / brand-jailbreak surface
- Each session costs the company per minute on ElevenLabs (cost-abuse risk)
- For a kids-aware brand, an unauthenticated voice agent that any child visiting the marketing site can interact with raises a separate Art. 8 / age-appropriate-design question
- The agent ID and configuration are extractable from the page bundle (would let an attacker impersonate the brand voice on their own site)

Recommend a separate engagement phase to evaluate the voice agent if the asset owner is interested.

## Files

- `web/waitlist/signup.html` — captured page HTML (570 KB)
- `web/waitlist/signup-headers.txt` — response headers
- `web/waitlist/captcha-dialog.png` — screenshot of post-submit captcha state
- `web/waitlist/results.txt` — earlier endpoint-discovery probe
- (no row-level data captured; no email actually delivered)

## Test account housekeeping

Test email used in the bounded probe: `maildmz+unomundi-pentest-20260505@gmail.com` (operator's own gmail with a `+` alias).

The captcha was never solved, so the email was not registered with Wix Contacts and no confirmation email was triggered. **No cleanup needed on the asset owner's side.**
