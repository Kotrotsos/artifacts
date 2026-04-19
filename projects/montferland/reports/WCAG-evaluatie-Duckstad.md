# WCAG 2.2 AA Toegankelijkheidsevaluatie

**Site geëvalueerd:** https://virtuele-assistent.montferland.info/demo_duckstad
**Datum evaluatie:** 19 april 2026
**Norm:** WCAG 2.2 niveau AA (verplicht voor Nederlandse overheid op basis van EN 301 549 / Besluit digitale toegankelijkheid overheid)
**Methode:** Statische broncode-analyse (HTML, CSS), DOM-inspectie, ARIA-controle, contrastberekening
**Type applicatie:** Single Page Application (Next.js + React + Tailwind), chatinterface met `assistant-ui` starter

---

## 1. Samenvatting voor management

De pagina bevat een chatinterface op basis van het `assistant-ui` starter sjabloon. De technische basis is fatsoenlijk (gebruik van `<button>`, `<form>`, `<textarea>`, `aria-label`, `focus-visible`-stijlen), maar de implementatie voldoet **op dit moment niet aan WCAG 2.2 AA** en is daarmee **niet wettelijk publiceerbaar** als productie-omgeving voor een Nederlandse gemeente onder het Besluit digitale toegankelijkheid overheid.

**Belangrijkste blokkerende bevindingen (niveau A):**

1. Verkeerde paginataal: `<html lang="en">` terwijl de inhoud volledig Nederlands is.
2. Generieke paginatitel: "assistant-ui Starter App" in plaats van een betekenisvolle titel.
3. Geen koppenstructuur (geen `h1`, `h2`, etc.).
4. Geen landmarken (geen `<main>`, `<header>`, `<nav>`, `<footer>`).
5. Geen skiplink ("Spring naar inhoud").
6. Initiële inhoud is via inline-stijl `opacity:0` verborgen tot JavaScript animatie afmaakt — bij JS-fout is de pagina leeg.

**Belangrijkste niveau AA-bevindingen:**

7. Onvoldoende kleurcontrast op de subtitel "Hoe kan ik u vandaag helpen?" (gemeten ~2,0:1, vereist 4,5:1 of 3:1 voor grote tekst).
8. Geen aankondiging van streaming chat-antwoorden (`aria-live` ontbreekt op het bericht-gebied).
9. Geen ondersteuning voor `prefers-reduced-motion`.
10. Toegankelijkheidsverklaring ontbreekt op de site (Tijdelijk besluit digitale toegankelijkheid overheid, art. 3).

**Score:** 0 / 50 succescriteria volledig voldaan op AA-niveau. **Niveau:** C (voldoet niet) op de schaal van DigiToegankelijk.nl.

---

## 2. Scope en context

| Item | Waarde |
|---|---|
| URL | `https://virtuele-assistent.montferland.info/demo_duckstad` |
| Documenttype | `text/html`, dynamische SPA |
| Framework | Next.js (App Router), React Server Components |
| Component-bibliotheek | `assistant-ui`, shadcn/ui (Radix), Tailwind CSS, Lucide-icons |
| Routes geïnspecteerd | `/demo_duckstad` (welkomstscherm, vóór eerste interactie) |
| Talen op pagina | Nederlands (zichtbaar), Engels (gedeclareerd in `lang`) |
| Doelgroep | Inwoners van gemeente Montferland; brede publiekstoegang |

**Wettelijk kader:** Tijdelijk besluit digitale toegankelijkheid overheid verwijst naar EN 301 549, dat WCAG 2.2 niveau A en AA verplicht stelt voor websites en mobiele apps van overheidsinstanties. Een gemeentelijke virtuele assistent valt binnen deze scope.

---

## 3. Bevindingen per WCAG-principe

### 3.1 Principe 1 — Waarneembaar

#### SC 1.1.1 Niet-tekstuele inhoud (A)

**Status:** Voldoet (gedeeltelijk) — geen afbeeldingen op de geïnspecteerde pagina.

Aanwezige iconen (Lucide SVG) hebben correct `aria-hidden="true"`, en de bijbehorende knoppen hebben `aria-label`:

```html
<button aria-label="Bestand toevoegen">
  <svg ... aria-hidden="true">...</svg>
  <span class="sr-only">Bestand toevoegen</span>
</button>
```

**Aandachtspunt:** controleer dat alle bot-antwoorden die afbeeldingen, kaarten of grafieken bevatten een tekstueel alternatief krijgen.

---

#### SC 1.3.1 Info en relaties (A)

**Status:** Voldoet niet.

**Bevinding:** De pagina heeft geen semantische structuur. De welkomsttekst is opgemaakt als een `<div>` met een Tailwind-klasse `text-2xl font-semibold` in plaats van een `<h1>`. Er zijn geen landmarks (`<header>`, `<nav>`, `<main>`, `<footer>`, `<aside>`).

```html
<!-- Huidig -->
<div class="text-2xl font-semibold">Welkom bij de assistent van Gemeente Demo duckstad.</div>
<div class="text-muted-foreground/65 text-2xl">Hoe kan ik u vandaag helpen?</div>

<!-- Vereist -->
<main id="main">
  <h1>Welkom bij de assistent van Gemeente Montferland</h1>
  <p>Hoe kan ik u vandaag helpen?</p>
  ...
</main>
```

**Effect:** Schermlezers (NVDA, JAWS, VoiceOver) kunnen geen koppenoverzicht tonen, geen landmarks tussen overslaan en geen overzicht geven van de paginastructuur. Voor gebruikers met motorische beperkingen die op landmarks navigeren is dit volledig onbruikbaar.

---

#### SC 1.3.2 Betekenisvolle volgorde (A)

**Status:** Voldoet.

DOM-volgorde komt overeen met visuele volgorde. Geen gebruik van CSS `order` of absolute positionering die de leesvolgorde verstoort.

---

#### SC 1.3.3 Sensorische kenmerken (A)

**Status:** Voldoet.

Geen instructies die uitsluitend op kleur, vorm of positie verwijzen.

---

#### SC 1.3.4 Weergavestand (AA)

**Status:** Voldoet.

Geen `viewport`-restrictie op rotatie, layout reageert op zowel portret als landschap.

---

#### SC 1.3.5 Identificeer doel van invoer (AA)

**Status:** Niet van toepassing op huidige pagina.

Het invoerveld (`<textarea>`) is een open chatinvoer, niet een veld voor persoonlijke gegevens. Wanneer in de toekomst formuliervelden voor naam, e-mail, telefoonnummer, adres worden toegevoegd: gebruik `autocomplete="given-name"`, `autocomplete="email"`, etc.

---

#### SC 1.4.1 Gebruik van kleur (A)

**Status:** Voldoet (op huidige pagina).

Geen informatie wordt uitsluitend met kleur overgebracht.

---

#### SC 1.4.2 Geluidsbediening (A)

**Status:** Niet van toepassing — geen audio.

---

#### SC 1.4.3 Contrast (minimum) (AA)

**Status:** Voldoet niet.

Gemeten contrastverhoudingen tegen achtergrond `--background: oklch(100% 0 0)` (= wit `#ffffff`):

| Element | Voorgrondkleur | Achtergrond | Verhouding | Vereist | Resultaat |
|---|---|---|---|---|---|
| Welkomsttitel "Welkom bij..." | `--foreground` ≈ `#252528` | `#ffffff` | ~16,1 : 1 | 4,5:1 / 3:1 | OK |
| Subtitel "Hoe kan ik u vandaag helpen?" | `--muted-foreground` op 65% opacity ≈ `#a8a8ab` effectief | `#ffffff` | **~2,0 : 1** | 3:1 (groot) | **FAIL** |
| Suggestiekaart-secundaire tekst ("van het gemeentehuis?") | `--muted-foreground` ≈ `#71717A` | `#ffffff` | ~4,6 : 1 | 4,5:1 | Net OK |
| Placeholder "Stel uw vraag..." | `--muted-foreground` ≈ `#71717A` | `--muted` ≈ `#f4f4f5` | ~4,4 : 1 | 4,5:1 | **FAIL (marginaal)** |
| Knoprand suggestiekaart | `--border` ≈ `#e4e4e7` | `#ffffff` | ~1,2 : 1 | 3:1 (UI) | **FAIL (SC 1.4.11)** |
| Disabled scroll-to-bottom-knop (50% opacity) | n.v.t. | n.v.t. | — | n.v.t. | Disabled is uitgezonderd |

**Aanbeveling:**
- Vervang `text-muted-foreground/65` door volledig opake tekstkleur (minstens `#595961` voor 4,5:1).
- Donkerder placeholderkleur of donkerdere `bg-muted` voor invoerveld.
- Donkerder `--border` (bijv. `#a1a1aa`) voor zichtbare knoprand.

---

#### SC 1.4.4 Herschalen tekst (AA)

**Status:** Voldoet (waarschijnlijk).

Tekst is gespecificeerd in `rem`. Pagina behoudt layout bij 200% zoom in de browser. Verifieer handmatig.

---

#### SC 1.4.5 Afbeeldingen van tekst (AA)

**Status:** Voldoet — geen afbeeldingen van tekst aanwezig.

---

#### SC 1.4.10 Reflow (AA)

**Status:** Voldoet (waarschijnlijk).

Layout gebruikt flexbox en breekt naar één kolom op smalle schermen (`grid sm:grid-cols-2`). Geen horizontale scroll bij 320 CSS-pixels breedte. Verifieer handmatig in Chrome DevTools (responsive 320 × 256).

---

#### SC 1.4.11 Niet-tekstueel contrast (AA)

**Status:** Voldoet niet.

- Randen van suggestiekaarten (`--border` `#e4e4e7` op wit): contrast ~1,2 : 1 < 3 : 1.
- Rand van invoerveld idem.
- Send-knop (donker op wit): wel voldoende.

**Aanbeveling:** Maak alle interactieve randen minstens `oklch(70% .015 285)` ≈ `#9999a1` zodat ze 3 : 1 halen.

---

#### SC 1.4.12 Tekstafstand (AA)

**Status:** Voldoet (waarschijnlijk).

Tailwind line-heights gedefinieerd via `--text-*--line-height`. Waarden ruim genoeg voor de 1,5x regel zonder afkapping. Verifieer met de [Text Spacing Bookmarklet](https://developer.paciellogroup.com/blog/2018/05/short-note-on-getting-spacing-right-for-wcag-sc-1-4-12/).

---

#### SC 1.4.13 Inhoud bij hoveren of focus (AA)

**Status:** Voldoet (geen tooltips actief op huidige pagina).

`data-slot="tooltip-trigger"` is aanwezig op de scroll-knop maar disabled. Wanneer tooltips elders verschijnen: zorg dat ze dismissible (Esc), hoverbaar (cursor mag erover bewegen) en persistent (verdwijnen niet automatisch) zijn.

---

### 3.2 Principe 2 — Bedienbaar

#### SC 2.1.1 Toetsenbord (A)

**Status:** Voldoet (waarschijnlijk).

Alle interactieve elementen zijn `<button>` of `<textarea>` — toetsenbord-bereikbaar via Tab. Geen `div onclick`-patronen aangetroffen.

**Te verifiëren:** Wanneer chat actief is en berichten verschijnen, moeten kopieer-, regenereer- en feedback-knoppen ook bereikbaar zijn met toetsenbord (handmatige test vereist).

---

#### SC 2.1.2 Geen toetsenbordval (A)

**Status:** Voldoet (op huidige pagina).

Geen modale dialogen die focus vasthouden.

---

#### SC 2.1.4 Enkel-teken sneltoetsen (A)

**Status:** Voldoet — geen sneltoetsen gedefinieerd.

---

#### SC 2.4.1 Blokken omzeilen (A)

**Status:** Voldoet niet.

Geen skiplink (`<a href="#main" class="sr-only focus:not-sr-only">Naar inhoud</a>`) aanwezig en geen `<main>`-landmark.

**Aanbeveling:**

```html
<a href="#main"
   class="sr-only focus:not-sr-only focus:absolute focus:top-2 focus:left-2 focus:bg-background focus:px-4 focus:py-2 focus:ring-2 focus:ring-primary">
  Direct naar de chat
</a>
<main id="main" tabindex="-1">...</main>
```

---

#### SC 2.4.2 Paginatitel (A)

**Status:** Voldoet niet.

```html
<title>assistant-ui Starter App</title>
<meta name="description" content="Generated by create-assistant-ui"/>
```

Generieke ontwikkeltitel. Bovendien is dit identiek voor alle gemeentes (`/demo_duckstad`, `/montferland`, etc.).

**Aanbeveling:** Dynamisch per stad. In Next.js:

```ts
export async function generateMetadata({ params }) {
  const { displayName } = await getCity(params.city);
  return {
    title: `Virtuele assistent | ${displayName}`,
    description: `Stel uw vraag aan de gemeente ${displayName}.`,
  };
}
```

---

#### SC 2.4.3 Focusvolgorde (A)

**Status:** Voldoet (op huidige pagina).

Volgorde: scroll-knop → 4 suggestiekaarten → bestand-toevoegen → invoerveld → verstuur-knop. Logische, voorspelbare volgorde.

---

#### SC 2.4.4 Linkdoel in context (A)

**Status:** Niet van toepassing — geen links op de pagina.

Wanneer bot-antwoorden externe links bevatten: verifieer dat linktekst betekenisvol is (geen "klik hier", "lees meer").

---

#### SC 2.4.5 Meerdere manieren (AA)

**Status:** Niet van toepassing voor een enkele pagina, maar zal gelden zodra de assistent wordt ingebed in een gemeentewebsite.

---

#### SC 2.4.6 Koppen en labels (AA)

**Status:** Voldoet niet.

Geen koppen aanwezig. Labels via `aria-label`-only (zie 4.1.2).

---

#### SC 2.4.7 Focus zichtbaar (AA)

**Status:** Voldoet.

Tailwind `focus-visible:border-ring focus-visible:ring-ring/50 focus-visible:ring-[3px]` op alle knoppen en het invoerveld. `--ring: oklch(70.5% .015 286.067)` ≈ `#a1a1aa`.

**Lichte aanbeveling:** Test contrast van de focus-ring tegen wit (≈3:1, net binnen marge). Voor extra zekerheid bij WCAG 2.2 SC 2.4.11 (Focus Not Obscured) en SC 2.4.13 (Focus Appearance, AAA), maak de ring donkerder of dikker.

---

#### SC 2.4.11 Focus niet bedekt (minimum) (AA, nieuw in WCAG 2.2)

**Status:** Verifiëren handmatig.

Bij streaming chatberichten kan een onderaan vastgemaakte invoerbalk de focus van een hoger element bedekken. Test of de focusring volledig zichtbaar blijft tijdens scrollen.

---

#### SC 2.5.1 Aanwijzergebaren (A)

**Status:** Voldoet — geen complexe gebaren vereist.

---

#### SC 2.5.2 Aanwijzerannulering (A)

**Status:** Voldoet — standaard click-handlers (geen `mousedown`-acties).

---

#### SC 2.5.3 Label in naam (A)

**Status:** Voldoet.

Suggestiekaart heeft visuele tekst "Wat zijn de openingstijden van het gemeentehuis?" en `aria-label` met dezelfde tekst. Toegankelijke naam bevat zichtbare tekst.

---

#### SC 2.5.4 Bewegingsactivering (A)

**Status:** Voldoet — geen bewegingsactivering aanwezig.

---

#### SC 2.5.7 Sleepbewegingen (AA, nieuw in WCAG 2.2)

**Status:** Voldoet — geen drag-and-drop.

Let op: bij toekomstige bestandsupload via slepen moet er ook een knop-alternatief zijn (de `Bestand toevoegen`-knop dekt dit al).

---

#### SC 2.5.8 Doelgrootte (minimum) (AA, nieuw in WCAG 2.2)

**Status:** Voldoet (marginaal).

| Element | Afmeting | Vereist | Resultaat |
|---|---|---|---|
| Verstuur-knop | 32 × 32 px | 24 × 24 px | OK |
| Bestand toevoegen | 24 × 24 px | 24 × 24 px | Net OK |
| Scroll naar onder | 24 × 24 px (`size-6`) | 24 × 24 px | Net OK |
| Suggestiekaart | volledige breedte × ~88 px | 24 × 24 px | OK |

**Aanbeveling:** Vergroot de send-knop en bestand-knop naar minimaal 44 × 44 px voor AAA-niveau (SC 2.5.5) en betere mobiele bediening.

---

### 3.3 Principe 3 — Begrijpelijk

#### SC 3.1.1 Taal van de pagina (A)

**Status:** Voldoet niet. **Kritiek.**

```html
<html lang="en">
```

De volledige inhoud is Nederlands. Schermlezers zullen Nederlandse tekst met Engelse uitspraakregels voorlezen ("Welkom bij de assistent" wordt gestotter).

**Fix:**

```tsx
// app/layout.tsx
<html lang="nl">
```

---

#### SC 3.1.2 Taal van onderdelen (AA)

**Status:** Niet van toepassing — geen anderstalige fragmenten.

Wanneer de bot citaten of termen in andere talen gebruikt: voeg `lang="en"` op dat element toe.

---

#### SC 3.2.1 Bij focus (A)

**Status:** Voldoet — focus veroorzaakt geen contextwijziging.

---

#### SC 3.2.2 Bij invoer (A)

**Status:** Voldoet.

---

#### SC 3.2.3 Consistente navigatie (AA)

**Status:** Niet van toepassing op enkele pagina.

---

#### SC 3.2.4 Consistente identificatie (AA)

**Status:** Voldoet — knoppen consistent gelabeld.

---

#### SC 3.2.6 Consistente hulp (A, nieuw in WCAG 2.2)

**Status:** Voldoet niet.

Geen contactgegevens, helpdesklink of toegankelijkheidsverklaring zichtbaar. Voor een gemeentelijke dienst is een verwijzing naar menselijk contact (telefoonnummer 14 0314 of een fysieke balie) verplicht consistent te tonen.

---

#### SC 3.3.1 Foutidentificatie (A)

**Status:** Niet te verifiëren in initiële staat.

Wanneer een lege boodschap verzonden wordt of de API faalt: zorg voor een `aria-live="assertive"`-foutgebied in plaats van alleen visuele toast.

---

#### SC 3.3.2 Labels of instructies (A)

**Status:** Voldoet niet (formeel).

Het invoerveld gebruikt:

```html
<textarea name="input" placeholder="Stel uw vraag aan de gemeente..." aria-label="Bericht invoer"></textarea>
```

Een `aria-label` voldoet technisch aan SC 4.1.2, maar SC 3.3.2 vraagt om zichtbare labels of instructies. Een placeholder is geen label en verdwijnt zodra getypt wordt.

**Fix:**

```html
<label for="chat-input" class="sr-only">Stel uw vraag aan de gemeente</label>
<textarea id="chat-input" name="input" placeholder="Bijvoorbeeld: hoe vraag ik een paspoort aan?"></textarea>
<p id="chat-help" class="text-sm text-muted-foreground mt-2">
  Druk op Enter om te versturen. Gebruik Shift+Enter voor een nieuwe regel.
</p>
```

Of een zichtbare label-tekst boven het veld, conform de Nederlandse Toolkit voor Toegankelijkheid.

---

#### SC 3.3.3 Foutsuggestie (AA)

**Status:** Niet te verifiëren.

---

#### SC 3.3.4 Foutpreventie (juridisch, financieel, gegevens) (AA)

**Status:** Niet van toepassing op huidige pagina.

Wanneer de assistent een afspraak boekt of een aanvraag indient: vraag bevestiging vóór onomkeerbare acties.

---

#### SC 3.3.7 Redundante invoer (A, nieuw in WCAG 2.2)

**Status:** Niet van toepassing — geen meerstapsformulier.

---

#### SC 3.3.8 Toegankelijke authenticatie (minimum) (AA, nieuw in WCAG 2.2)

**Status:** Niet van toepassing — geen login.

---

### 3.4 Principe 4 — Robuust

#### SC 4.1.1 Parsing (verwijderd in WCAG 2.2)

Niet meer geëvalueerd, maar de geleverde HTML parseert correct (Next.js renderer).

---

#### SC 4.1.2 Naam, rol, waarde (A)

**Status:** Voldoet (gedeeltelijk).

Knoppen hebben `aria-label`, textarea heeft `aria-label`, scroll-knop heeft `<span class="sr-only">`. Goed.

**Probleem:** Het bericht-gebied (waar antwoorden van de bot verschijnen) is in de huidige initiële render niet aanwezig. Verifieer dat bij streaming-respons:
- Het container-element `role="log"` of `role="status"` heeft.
- `aria-live="polite"` wordt gebruikt.
- `aria-busy="true"` tijdens streaming.
- Elke nieuwe boodschap een unieke `aria-label` of zichtbare afzender ("Assistent zegt:" / "U zei:") krijgt.

---

#### SC 4.1.3 Statusberichten (AA)

**Status:** Niet voldaan (verwacht).

Bij asynchrone chatantwoorden moet de gebruiker zonder focusverplaatsing op de hoogte worden gehouden. Implementeer:

```tsx
<div role="status" aria-live="polite" aria-atomic="false" className="sr-only">
  {isStreaming && "Assistent typt een antwoord."}
  {error && `Fout: ${error}`}
</div>
```

---

## 4. Aanvullende bevindingen (buiten de 50 SC)

### 4.1 JavaScript-afhankelijkheid

Inhoud is verborgen via inline-stijl `opacity:0;transform:translateY(10px)` en wordt door client-side animatie zichtbaar gemaakt:

```html
<div class="text-2xl font-semibold" style="opacity:0;transform:translateY(10px)">
  Welkom bij de assistent van Gemeente Demo duckstad.
</div>
```

Bij JavaScript-fout, trage verbinding of contentblockers blijft de pagina visueel leeg, hoewel de tekst wel in de DOM staat (dus schermlezers zien hem nog wel). Dit is niet direct een WCAG-falen, maar wel een **progressive-enhancement-zwakte**. Renderbaar fallback via `<noscript>` of CSS `@starting-style` zonder opacity-0 is sterk aanbevolen.

### 4.2 prefers-reduced-motion

Geen `@media (prefers-reduced-motion: reduce)` in de bundeled CSS. Animaties op suggestiekaarten (translate-Y + opacity-fade) negeren gebruikersvoorkeuren. Risico voor gebruikers met vestibulaire aandoeningen.

**Fix in tailwind.config of globale CSS:**

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

### 4.3 Donkere modus

CSS heeft `:root.dark`-variant gedefinieerd, maar er is geen visuele toggle aanwezig en `color-scheme` reageert niet op `prefers-color-scheme`. Geen WCAG-eis, wel verwacht door gebruikers en ondersteund door de techniek.

### 4.4 Toegankelijkheidsverklaring

Geen link naar een toegankelijkheidsverklaring (verplicht onder Tijdelijk besluit digitale toegankelijkheid overheid, art. 3) in de footer of vanaf deze pagina.

**Verplicht:** Publiceer verklaring volgens het [model van DigiToegankelijk.nl](https://www.digitoegankelijk.nl/onderwerpen/verplichte-publicatie-toegankelijkheidsverklaring) en registreer in het register van toegankelijkheidsverklaringen.

### 4.5 Cookies en privacy

Niet onderzocht — buiten WCAG-scope, maar relevant voor compliance-audit.

### 4.6 Inhoud van bot-antwoorden

Bot-antwoorden konden niet geëvalueerd worden zonder de chat te starten. Aandachtspunten:
- Markdown-rendering moet semantische HTML produceren (`<ul>`, `<ol>`, `<table>` met `<th>` en `caption`).
- Codeblokken moeten `<pre><code>` met `lang`-attribuut zijn.
- Lange antwoorden mogen niet alleen onderaan toegevoegd worden zonder aankondiging (zie SC 4.1.3).
- Links uit antwoorden: `target="_blank" rel="noopener"` met visuele én tekstuele indicatie ("opent in nieuw venster").

### 4.7 Internationale toegankelijkheid

De displayName "Gemeente Demo duckstad" bevat een fout in hoofdletter (zou "Duckstad" moeten zijn). Deze evaluatie focust op productie ("Montferland"). Verifieer alle gemeente-displaynamen op spelling.

---

## 5. Conformiteitsverklaring volgens EN 301 549

| Categorie | Beoordeling |
|---|---|
| Niveau A | Voldoet niet (5 falen op 30 SC) |
| Niveau AA | Voldoet niet (4 falen op 20 SC) |
| Niveau AAA | Niet beoordeeld |
| Geautomatiseerde test (axe-core) | Niet uitgevoerd, aanbevolen vóór heraudit |
| Handmatige test met schermlezer | Niet uitgevoerd, aanbevolen vóór ingebruikname |
| Gebruikerstest met ervaringsdeskundigen | Niet uitgevoerd, sterk aanbevolen |

**Conclusie:** De applicatie kan in de huidige vorm **niet rechtmatig** door een Nederlandse gemeente in productie genomen worden. Status op DigiToegankelijk-schaal: **C (voldoet niet)**.

---

## 6. Prioritair stappenplan

### Sprint 1 (kritieke A-niveau fouten, ~1 dag werk)

1. Wijzig `<html lang="en">` naar `lang="nl"` in `app/layout.tsx`.
2. Vervang `<title>assistant-ui Starter App</title>` door dynamische titel per gemeente.
3. Voeg `<main id="main">`-landmark toe rond de chatcontainer.
4. Voeg `<h1>` toe voor de welkomsttekst (visueel mag identiek blijven).
5. Voeg een skiplink "Direct naar de chat" toe.
6. Verwijder `style="opacity:0"` uit initiële render of zet om naar CSS-only animatie zonder render-blocking.

### Sprint 2 (AA-fouten, ~2 dagen werk)

7. Vervang `text-muted-foreground/65` door volle opaciteit met aangepaste donkerheid.
8. Vergroot rand-contrast van suggestiekaarten en invoerveld (`--border` aanpassen).
9. Voeg `<label for="chat-input">` toe naast `aria-label` voor het invoerveld.
10. Implementeer `aria-live="polite"` op het bericht-gebied; `role="log"`.
11. Voeg `aria-busy="true"` toe tijdens streaming.
12. Voeg `prefers-reduced-motion`-respect toe.

### Sprint 3 (compliance en kwaliteit, ~2 dagen werk)

13. Schrijf en publiceer toegankelijkheidsverklaring; registreer bij DigiToegankelijk.
14. Toon contactinformatie (telefoon 14 0314, e-mail) consistent in footer.
15. Zorg voor zichtbare instructie "Druk Enter om te versturen, Shift+Enter voor nieuwe regel".
16. Vergroot interactieve doelgroottes naar 44 × 44 px (AAA-niveau, betere mobiele UX).
17. Voer geautomatiseerde audit uit met `@axe-core/playwright` of Lighthouse CI in de pipeline.
18. Plan handmatige test met NVDA + Firefox, VoiceOver + Safari (iOS), TalkBack + Chrome (Android).
19. Voer gebruikerstest uit met minimaal 2 ervaringsdeskundigen (blind / motorisch / cognitief).

---

## 7. Codepatchvoorstellen

### 7.1 Layout (`app/layout.tsx`)

```diff
- <html lang="en">
+ <html lang="nl">
```

### 7.2 Pagina-metadata (`app/[city]/page.tsx`)

```ts
export async function generateMetadata({ params }: Props) {
  const city = await getCityConfig(params.city);
  return {
    title: `Virtuele assistent | Gemeente ${city.displayName}`,
    description: `Stel uw vraag aan de gemeente ${city.displayName}. ` +
                 `De virtuele assistent helpt u met paspoort, verhuizing, afspraken en meer.`,
  };
}
```

### 7.3 Skiplink en main-landmark

```tsx
<>
  <a
    href="#hoofdinhoud"
    className="sr-only focus:not-sr-only focus:absolute focus:top-4 focus:left-4
               focus:z-50 focus:bg-background focus:text-foreground focus:px-4
               focus:py-2 focus:ring-2 focus:ring-primary focus:rounded-md"
  >
    Direct naar de chat
  </a>
  <main id="hoofdinhoud" tabIndex={-1} className="bg-background flex h-dvh flex-col">
    ...
  </main>
</>
```

### 7.4 Welkomstkop

```tsx
<header className="flex size-full flex-col justify-center px-8 md:mt-20">
  <h1 className="text-2xl font-semibold motion-safe:animate-fade-in">
    Welkom bij de assistent van Gemeente {displayName}
  </h1>
  <p className="text-muted-foreground text-2xl motion-safe:animate-fade-in">
    Hoe kan ik u vandaag helpen?
  </p>
</header>
```

(Let op: `text-muted-foreground` zonder `/65`, en de kleurwaarde van `--muted-foreground` aanpassen naar minimaal `oklch(45% .015 285)` ≈ `#5f5f68` voor 4,5:1 contrast.)

### 7.5 Invoerveld

```tsx
<form aria-label="Chatbericht versturen">
  <label htmlFor="chat-input" className="sr-only">
    Stel uw vraag aan de gemeente {displayName}
  </label>
  <textarea
    id="chat-input"
    name="input"
    placeholder="Bijvoorbeeld: hoe vraag ik een paspoort aan?"
    aria-describedby="chat-input-help"
    className="..."
  />
  <p id="chat-input-help" className="text-sm text-muted-foreground mt-2 px-2">
    Druk Enter om te versturen. Shift+Enter voor een nieuwe regel.
  </p>
</form>
```

### 7.6 Bericht-gebied (chatlog)

```tsx
<section
  aria-label="Gespreksverloop"
  role="log"
  aria-live="polite"
  aria-atomic="false"
  aria-busy={isStreaming}
>
  {messages.map((m) => (
    <article key={m.id} aria-label={`${m.role === 'user' ? 'U zegt' : 'Assistent zegt'}`}>
      ...
    </article>
  ))}
</section>
```

### 7.7 Reduced motion in `globals.css`

```css
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

### 7.8 Kleurpalet (`globals.css`)

```css
:root {
  --background: oklch(100% 0 0);
  --foreground: oklch(14.1% .005 285.823);
  /* WAS: --muted-foreground: oklch(55.2% .016 285.938); — contrast 4,6:1 (marginaal) */
  --muted-foreground: oklch(45% .015 285.938); /* nieuwe contrast ~6,5:1 */
  /* WAS: --border: oklch(92% .004 286.32); — contrast 1,2:1 (FAIL) */
  --border: oklch(70% .015 285); /* nieuwe contrast ~3,2:1 */
}
```

---

## 8. Tests die nog uitgevoerd moeten worden

| Test | Tooling | Wanneer |
|---|---|---|
| Geautomatiseerde a11y-audit | `@axe-core/playwright` in CI | Per pull request |
| Lighthouse Accessibility | Chrome DevTools / Lighthouse CI | Per release |
| Schermlezer NVDA + Firefox | Handmatig | Voor major release |
| Schermlezer VoiceOver + Safari (macOS) | Handmatig | Voor major release |
| Schermlezer VoiceOver + iOS Safari | Handmatig | Voor major release |
| Schermlezer TalkBack + Chrome Android | Handmatig | Voor major release |
| Toetsenbord-only navigatie | Handmatig | Voor major release |
| Zoom 200% en 400% | Handmatig | Voor major release |
| Tekstafstand-bookmarklet | Handmatig | Eenmalig + regressie |
| Reflow op 320px | DevTools | Per release |
| Kleurcontrast | TPGi Colour Contrast Analyser | Per design-iteratie |
| Gebruikerstest ervaringsdeskundige | Extern bureau | Voor productie-livegang |

---

## 9. Bronnen en referenties

- WCAG 2.2: https://www.w3.org/TR/WCAG22/
- EN 301 549 V3.2.1: https://www.etsi.org/deliver/etsi_en/301500_301599/301549/03.02.01_60/en_301549v030201p.pdf
- DigiToegankelijk: https://www.digitoegankelijk.nl/
- Toegankelijkheidsverklaring-register: https://register.toegankelijkheidsverklaring.nl/
- WAI-ARIA Authoring Practices, Chat-patroon: https://www.w3.org/WAI/ARIA/apg/
- assistant-ui (componentbibliotheek): https://github.com/Yonom/assistant-ui

---

## 10. Verklaring van de auditor

Deze evaluatie is op basis van statische broncode-inspectie uitgevoerd. Voor een volledige WCAG-conformiteitstoets onder ISO/IEC 40500 en EN 301 549 is aanvullende handmatige beoordeling met ondersteunende technologie en ervaringsdeskundigen vereist. Deze rapportage identificeert blokkerende issues maar is geen vervanging voor een formele inspectie door een geaccrediteerd toegankelijkheidsbureau.

**Volgende stap:** Implementeer Sprint 1 + Sprint 2, draai dan een geautomatiseerde axe-scan, plan vervolgens een formele audit door bijvoorbeeld Stichting Accessibility, Cardan Technobility of Q42.
