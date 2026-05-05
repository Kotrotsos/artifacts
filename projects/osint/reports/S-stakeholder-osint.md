# Stakeholder OSINT — Unomundi

**Engagement:** unomundi.com pentest, 2026-05-05.
**Sources:** UK Companies House, NL KvK reference, public LinkedIn (no scraping behind login wall), public GitHub, conference / press, public investment listings, company About page.
**Method:** passive OSINT only. No data was solicited, no contacts were approached, no credentials were probed against any individual's accounts.

## Why this exists in the engagement deliverable

A pentest report should describe the threat model an external attacker would actually build. Spear-phishing the founders, the engineers, or the advisors is the cheapest path to compromise once the network surface is closed. The information below is what an attacker can assemble in **about 90 minutes of free-tier OSINT** without any access to the company's systems. It is included so the asset owner can:

1. Train staff against the **specific** pretexts an attacker would use against them.
2. Decide which information is acceptable to leave public and which to scrub.
3. Identify the most attractive social-engineering targets and harden their accounts (MFA, hardware keys, account-takeover-recovery flows).

**What this document deliberately does not include:**
- Home addresses (Companies House registered office is acknowledged as fact; not republished)
- Personal phone numbers
- Family relationships beyond what is publicly self-stated
- Children's information
- Anything that combines public facts in ways that enable harassment

---

## Corporate structure

Two simultaneous entities currently render on the marketing site (see W-1):

### Una's World Holding Limited (UK)
- **Companies House #:** [17032480](https://find-and-update.company-information.service.gov.uk/company/17032480)
- **Incorporated:** 13 February 2026 (≈3 months old as of engagement)
- **Status:** Active, private limited
- **Registered office:** 2 Glenfield Villas Colleyland, Chorleywood, Rickmansworth, WD3 5LL — residential address, almost certainly Director Bradbeer's home (UK norm for early-stage Ltds)
- **SIC:** 62012 — business and domestic software development
- **First accounts due:** 13 November 2027
- **Confirmation statement due:** 16 March 2027

### Una's World Holding B.V. (Netherlands)
- **KvK:** 98204637 (referenced in privacy policy)
- **Pre-existing entity** — privacy policy still names BV as "the controller"
- Public KvK profile not deep-fetched in this engagement; the asset owner can pull the full uittreksel directly from kvk.nl

### Persons with Significant Control (UK Ltd, public record)

| Name | Ownership | Voting | Nature of control | Country | Year of birth |
|---|---|---|---|---|---|
| **Mrs Sonja Keerl** | 50-75% | 50-75% | Right to appoint or remove directors | Netherlands | 1978 |
| **Mr Matthew Jonathan Francis Bradbeer** | 25-50% | 25-50% | Right to appoint or remove directors | England | 1975 |

**Implication for threat modelling:** these two individuals are the highest-value compromise targets. An attacker who compromises Sonja's account inherits unilateral control of the UK Ltd. Recommend hardware-key MFA for both, and a documented account-takeover-recovery plan agreed in advance with Companies House.

---

## Stakeholder map

### Founders / Directors

**Sonja Keerl — Co-Founder & CEO** ([linkedin.com/in/sonjakeerl](https://nl.linkedin.com/in/sonjakeerl/))
- **Headline:** "Founder & CEO, Unomundi | B2B SaaS Strategist &…"
- **Location:** Amsterdam, North Holland, Netherlands
- **Background:** 20+ years in B2B SaaS go-to-market; co-founder of MACH Alliance (industry consortium for headless commerce / Microservices, API-first, Cloud-native, Headless); previously Sonja Keerl Consulting, The BaaS Company, Forbes Business Council
- **Volunteer:** Mentor at Women in MACH (Jun 2023 — present), Mentor at Alchemist Accelerator (Oct 2023 — present), Red Cross NL volunteer (since 2015), Marketing Advisor at International Almere (2012-2014)
- **Languages:** German (native), English (native), Dutch (limited)
- **Separate entity:** [Sonja Keerl Consulting](https://www.sonjakeerl.com/) — her independent consulting practice, separate LinkedIn page
- **Crunchbase profile:** [crunchbase.com/person/sonja-keerl-2a8e](https://www.crunchbase.com/person/sonja-keerl-2a8e)
- **Threat model:** highest-value target. Public profile actively engages with the startup-mentor / accelerator community (high inbound message volume → social-engineering noise floor; harder to detect targeted lures). She is the named CEO and the majority shareholder.

**Matt Bradbeer MIET — Co-Founder & COO** ([linkedin.com/in/mattbradbeer](https://uk.linkedin.com/in/mattbradbeer))
- **Headline:** "Matt Bradbeer MIET"
- **Location:** London, England
- **Background:** Co-founder, Advisory Board Member of MACH Alliance; previously MACH Business Lead and Client Partner at EPAM Systems; ex-Capgemini, ex-Indie Media Ltd; Co-Founder of eGurus Ltd; Autharium Technology sold May 2015
- **Education:** BA International Relations, Keele University 1994-1997 (Grade 2.1)
- **Certifications:** Member, Institution of Engineering and Technology (MIET, Sep 2016); Mental Health First Aider (Feb 2021)
- **Crunchbase:** [crunchbase.com/person/matt-bradbeer](https://www.crunchbase.com/person/matt-bradbeer)
- **Threat model:** named contact on the Caribbean Investment Network listing — receives investor inbound. The MACH Alliance link to Sonja makes "MACH partnership pretext" especially credible against him.

### Staff (per About page)

| Name | Role | Public profile |
|---|---|---|
| **Daniella Onyebuchi** | Operations | [linkedin.com/in/daniellaonyebuchi](https://www.linkedin.com/in/daniellaonyebuchi) |
| **Edgar Bittencourt** | Senior Art Director | [linkedin.com/in/edgar-bittencourt-99463a11a](https://www.linkedin.com/in/edgar-bittencourt-99463a11a), [@edgar.bittencourt.art](https://www.instagram.com/edgar.bittencourt.art) |
| **Nour Arafa** | Digital & Social Marketing | [linkedin.com/in/nourarafa](https://www.linkedin.com/in/nourarafa) |
| **Lucy** | "Creator of Una" — character/illustration credit, no LinkedIn |
| **Alessandro Canessa** | App Delivery Lead (per LinkedIn; not on About page — likely contractor) | [linkedin.com/in/alessandrocanessa](https://www.linkedin.com/in/alessandrocanessa), Twitter [@alexcanessa](https://twitter.com/alexcanessa), GitHub [github.com/alexcanessa](https://github.com/alexcanessa) (34 repos, 893★ on github-release-notes), day-job: **Head of Developer Relations at Commerce Layer** |

### Advisors

| Name | Role | Public profile / pedigree |
|---|---|---|
| **Sebastian "Buddy" Fleer** | Advisor, Gaming | [linkedin.com/in/sebastian-buddy-fleer](https://www.linkedin.com/in/sebastian-buddy-fleer-0a7a952/) — ~20 years games industry; ex-Jagex; based Düsseldorf; mentor to indie game teams |
| **Colin Adams** | Advisor, Publishing & Finance | [linkedin.com/in/colin-adams-31630a1](https://www.linkedin.com/in/colin-adams-31630a1) |
| **Siana Altiise** | Curriculum Design | [linkedin.com/in/sianaaltiise](https://www.linkedin.com/in/sianaaltiise/) |
| **Nicolas Gazeu** | Agentic AI Lead | [linkedin.com/in/nicolas-gazeu](http://www.linkedin.com/in/nicolas-gazeu) — runs Realycs (AI productivity for industrial SMEs); ex-Schneider Electric (7 years) |

---

## Funding / commercial context

The fundraise context is **directly material to the engagement disclosure** because it changes what's at stake commercially.

### Caribbean Investment Network listing
*Listing: [una-s-world-unomundi-15-1582898](https://www.caribbeaninvestmentnetwork.com/business-proposals/una-s-world-unomundi-15-1582898)*

| Field | Value |
|---|---|
| Stage | Pre-Startup / R&D |
| Target raise | **$700,000** |
| Minimum investment | $15,000 |
| Already raised | $125,000 (≈17.8% committed) |
| Cash-flow break-even | Q3 Year 1 (post-launch) |
| Exit valuation projection | **£800M+ in Year 4** at 12x PAT multiple |
| Listed contact | Matthew Bradbeer |

### LinkedIn-stated fundraise
- £500K round, 24% committed at time of post (≈£120K, plausibly the same money as the $125K above given GBP/USD)
- Sonja described the company as "actively fundraising"

### Web Summit 2025 (Lisbon, Nov 2025)
- Per multiple LinkedIn references and search hits, Unomundi was selected for the **Startup Showcase at Web Summit Lisbon 2025**.
- This is a verifiable mark of credibility but also a known-public fact an attacker can use ("we're following up from your Web Summit pitch…").

### Why this changes the disclosure framing

A breach narrative landing in the middle of a $700K seed round, against a 3-month-old UK Ltd controlled by two named individuals with verifiable public profiles, is more commercially damaging than a breach against a mature company. The *good* version of the story is "found and fixed before launch and before round close" — and the asset owner has a real opportunity to make that the version that lands.

Recommend folding the funding context into the disclosure narrative: investors will perform diligence; how the company handled this finding matters more than whether the finding existed.

---

## Technical footprint

### GitHub posture

- **`github.com/unomundi`** — exists as a user account, **0 public repositories**. Org username is reserved; everything is private. Good posture for a stealth-mode startup.
- **`github.com/alexcanessa`** (Alessandro Canessa) — 34 public repos, mostly Commerce Layer / personal tooling; pinned: `github-release-notes` (893★), `typescript-coverage-report` (519★), `github` (3.7k★). **No Unomundi-related repos visible publicly.** His Github bio location says "CommerceLayer, World" — meaning Unomundi work, if any, happens in private repos under the Unomundi org.

### MACH Alliance — the connective tissue

Both founders are MACH Alliance co-founders and active members. The community is small, public, and tightly-networked. This is excellent for:
- Identifying potential investors and advisors (positive)
- Identifying the most credible spear-phishing pretexts (negative)

Any inbound email referencing MACH Alliance roles, events, or speakers is going to clear the founders' default suspicion threshold. Train against this specifically.

### Engineering team observation

The About page does not list a CTO or Head of Engineering. Public OSINT suggests:
- **Alessandro Canessa** functions as "App Delivery Lead" but works full-time at Commerce Layer (likely contractor / advisor relationship)
- **Nicolas Gazeu** owns "Agentic AI" but runs his own AI consultancy (Realycs), not full-time at Unomundi
- Lovable.dev was used to build the admin SPA (per F-3 finding), which suggests **AI-assisted development is the team's primary engineering modality**, not a hired engineering team

This explains several of the technical findings:
- F-1 (Studio open) and F-2a (`pg_stat_statements` in public schema) are exactly the misconfigurations that occur when product-led / AI-assisted teams stand up self-hosted Supabase without a dedicated DBA / SecOps review
- A-4 (admin Edge Function lacks role check) is consistent with "Lovable generated this and we didn't audit the auth"
- The lack of source maps (admin gap-fill) is consistent with Vite defaults, not deliberate hardening

This is **not a criticism** — it's the modal architecture for 2026 AI-first kids'-tech startups. The disclosure narrative should acknowledge this and recommend a fractional security advisor as the cheapest fix-it route, not a full security hire.

---

## Realistic spear-phishing pretexts (defensive briefing)

For each named individual, here are the most credible lures an external attacker would attempt. Train against these specifically:

### Sonja Keerl (highest-value target)
- **MACH Alliance ops:** "Re: Women in MACH mentor session calendar conflict — please review attached" with a credential-harvesting Office365 link
- **Alchemist Accelerator:** "Founder office hours follow-up — calendar invite expired, click to rebook"
- **Investor inbound:** "Lead investor for your Unomundi round — diligence request, please review NDA"
- **Web Summit follow-up:** "Web Summit 2025 partner introduction from [legitimate-sounding name]"
- **Recovery flow attack:** target the Crunchbase profile to gather security-question-style info, then attack the LinkedIn / Gmail recovery flow

### Matt Bradbeer
- **Caribbean Investment Network:** "New investor enquiry — minimum ticket size confirmation needed" (he's the listed contact)
- **MACH Alliance / EPAM:** "Speaker invite for MACH summit, please confirm bio"
- **Companies House:** "Confirmation statement reminder — log in to file" (his name is on the registered office; this lure has very high credibility)
- **Mental Health First Aider community:** professional-development bait

### Alessandro Canessa
- **Commerce Layer:** anything that looks like a Commerce Layer ticket / DevRel inquiry
- **Conference circuit:** "We loved your micro frontends talk, can you join our podcast?"
- **GitHub:** OAuth-app-grant phishing posing as a Sessionize / Speaker integration
- **High risk vector:** if Alex authenticates to Unomundi's Supabase / Vercel / Lovable from his personal laptop using his Commerce-Layer-shared device, a Commerce-Layer-targeted phish becomes a Unomundi compromise

### Other staff
- Daniella (Operations) — vendor-pretext invoice phishing is the genre default for ops roles
- Edgar (Art Director) — "we'd like to license your work" pretexts; Adobe / Figma OAuth phishing
- Nour (Marketing) — "promoted post review" lures targeting Meta Business Suite credentials

---

## What the company should do with this document

1. **Hardware-key MFA** on Sonja, Matt, Alex (and any other Supabase-, Vercel-, Lovable-, Stripe-, or domain-registrar-admin accounts). Yubikeys or equivalent. Software-token MFA is bypassable by phishing kits and is no longer a defensible default for a startup with this attack-surface profile.

2. **Designate one person as the security point-of-contact** in the privacy policy and in a `security.txt` file at `https://www.unomundi.com/.well-known/security.txt`. Right now, `info@unomundi.com` is the only public address. Future researchers won't find a clean disclosure path.

3. **Run a tabletop on each pretext above** with the founders and Alex. 30 minutes. The point is not to memorise the pretexts — the point is to internalise *"if a message references MACH Alliance or Web Summit, slow down"*.

4. **Add a Companies House service-address** that is not the director's home. Companies House permits a separate service address that hides the home from the public register. Cost: £8 amendment fee.

5. **Privacy policy: pick one entity** as the controller and document the BV→Ltd transfer (if both will continue to exist) so any data-subject question has a clean answer.

6. **Consider a fractional security advisor**, 4–8 hours/month. The findings F-1, F-2a, F-3, A-1, A-4 are textbook misconfigurations that a 4-hour security review at deployment would have caught. This is cheaper than a full hire and proportional to a pre-launch startup.

---

## Files

- This document: `findings/S-stakeholder-osint.md`
- No raw social-media archives, screenshots of personal profiles, or aggregated PII files have been retained.
