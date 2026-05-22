# Wijzigingen 22 mei 2026

Branch: `feedback-round-1`. Twaalf commits, acht productie-deploys (v0.13.0 t/m v0.14.9), van `v0.12.1-multiformat` aan het begin van de dag naar `v0.14.9-feedback1` aan het eind.

## v0.13.0-feedback1 — Container-tekst in samenvatting

Lege Journalistieke samenvatting toont nu *"Wie, wat, waar, wanneer en hoe. Bij voorkeur in ± 75 woorden, maar altijd naar relevantie"*. Native HTML-placeholder-gedrag wist 'm zodra de journalist begint te typen. Zelfde string wordt gebruikt voor de inline-display, de textarea-placeholder en de reset-state na blur, zodat het veld nooit blank gaat.

## v0.14.0-feedback1 — Zittingslijst-stijl view weg

De Zittingslijst-stijl toggle in de doc-toolbar is verwijderd, samen met:

- de `localStorage` key `zl_doc_form_style`
- de JS die `body.zl-style` om en om flipte
- ongeveer 100 regels CSS-reflow-regels die de kaart-layout opbouwden

Tab-volgorde JS blijft, vereenvoudigd: stempelt nog steeds `tabindex` op de prioritaire velden en verwijdert `disabled` op locked docs zodat ze toch tabbaar zijn.

## v0.14.1-feedback1 — Sortering output

`ListZakenByDocument` ORDER BY: **zittingsdatum ASC NULLS LAST, aanvangstijd ASC NULLS LAST, created_at ASC**. Rechtbank/Gerechtshof is impliciet constant per perslijst. Ties op aanvangstijd → geen voorkeur (deterministisch op created_at). Aansluit op de Rechtspraak-spec.

## v0.14.1 build fix — templ CLI pin

Dockerfile pulde `templ@latest`, dat sinds kort `templ.ResolveAttributeValue` calls genereert die de oudere `templ` runtime in `go.mod` niet kent — build brak. CLI gepind op `v0.3.1001` zodat builder en runtime synchroon zijn.

## v0.14.2-feedback1 — Dode CSS opgeruimd

Twee leftover `.form-style-switch.doc-style-switch` regels stonden nog in `app.css` terwijl de bijbehorende markup al weg was. Verwijderd voor netheid; geen visuele impact.

## v0.14.3-feedback1 — Rechter + OvJ naar Zitting-sectie

**Rechter(s)** en **Officier van justitie** zijn verhuisd van *Betrokkenen* naar *Zitting*. Ze horen bij de hele zitting, niet per zaak, en moeten dus hoger in het formulier staan. Advocaat blijft in Betrokkenen. Tab-volgorde aangepast aan de nieuwe visuele volgorde.

## v0.14.4-feedback1 — Deadline = eerstvolgende maandag

Upload-modaal default-deadline:

- **Reguliere upload**: eerstvolgende maandag *strikt* na de zittingsdatum.
- **Spoed**: +1 werkdag (ongewijzigd).

Gebruiker kan de waarde nog overschrijven voor verzenden; JS prikt alleen een zinnige default in.

## v0.14.5-feedback1 — Handmatige perslijsten meteen *In behandeling*

`CreateManual` riep `CreateDocument` (default `status='NIEUW'`, `assigned_to=NULL`) en stuurde de gebruiker door naar de editor. De toolbar render alleen de IN_BEHANDELING-acties (*Opslaan en vrijgeven*, *Markeer als klaar*) als de doc op IN_BEHANDELING staat — dus zaken invullen kon wel maar finaliseren niet, zonder eerst via de wachtrij op Verwerken te klikken.

Fix: één extra query na creatie. `CreateManual` roept nu `AssignDocument` met de creator's user-ID, wat `status='IN_BEHANDELING'` zet en `assigned_to=<creator>` — dezelfde transitie die de Verwerken-knop op de wachtrij triggert. Geen schema-wijziging.

## v0.14.6-feedback1 — Lege woonplaats lekt geen "Gedetineerd. N" meer

`extractField` had altijd minimaal 1 karakter capture in zijn regex, dus als de bronlabel direct gevolgd werd door het volgende label (lege waarde) pakte de extractor het volgende label op als value. *Woonplaats* zonder waarde gaf `"Gedetineerd. N"`.

Fix: empty-value guard. Na de regex-match, als de captured value (case-insensitive) begint met een bekend field-label, return een lege string. Regression test gepind in `pdf_service_test.go`.

Ook in dit deploy: **voortgang.md** als Nederlandstalig overzicht voor de gebruiker.

## v0.14.7-feedback1 — Feedback batch (6 items in één deploy)

### OvJ barcode garbage trim
`cleanOvjName` knipt trailing barcode-rommel weg. Echte OvJ-namen bevatten geen cijfers, dus snijdt op het eerste cijfer en loopt terug naar de vorige spatie.

> `"mr. M.G.T. Kramer ÁG441125833166wÈ G441125833166"` → `"mr. M.G.T. Kramer"`

Regression test gepind.

### Feit + Pleeglocatie geen auto-parse meer
ANP-feedback (19/5): gebruiker vult zelf in als open veld om fouten te voorkomen bij meerdere feiten/locaties per verdenking. Side-effect: de `"gemeente Utrecht"` suffix die uit de PDF lekte is verdwenen.

### Type zitting +3 opties
Migratie `011_zaaktype_tbs_isd.sql` voegt `VERLENGING_TBS`, `WIJZIGING_VOORWAARDEN_TBS`, `TOETSING_ISD` toe aan de enum. UI dropdown uitgebreid met:

- Verlenging tbs
- Wijziging voorwaarden tbs
- Verzoek toetsing opheffing/voortzetting ISD-maatregel

### Geboortedatum dd-mm-yyyy in de tool
`displayText` voor type=date geeft nu numerieke NL-format (`02-01-2006`, matcht de PDF). De read-only weergave gebruikt een text-input zodat de browser-locale niet terugvalt op ISO.

### Markeer als klaar → redirect overzicht
`Finalize` handler leidt naar `/documenten?confirm=klaar` ipv het bewerkscherm. Download-knop direct bereikbaar.

### Uitloggen-knop
POST form naar `/logout` in de topbar, naast het user-chip.

## v0.14.8-feedback1 — Sort arrows in overzicht

**Geupload op** en **Deadline** kolommen zijn klikbaar voor sortering. Client-side sort (dataset is <100 rijen, geen query-string round-trip nodig).

- Klik op de actieve kolom toggelt asc/desc.
- Klik op een inactieve kolom gebruikt zijn `data-sort-default` (uploaddatum desc — nieuwste boven, deadline asc — vroegste boven).
- Lege waardes zinken altijd naar onder.

Deadline-kolom start pre-marked active+asc om de server-side ORDER BY visueel waarheidsgetrouw weer te geven.

## v0.14.9-feedback1 — Markeer als klaar direct klikbaar

De finalize-knop op de doc-toolbar was disabled bij eerste render en bleef dat tot een full reload, ook als de gebruiker het laatste veld via "Zaak opslaan" had ingevuld. Gebruiker moest terug naar overzicht en weer naar binnen, of refreshen.

Fix: knop in een `MarkeerKlaarSlot` templ-partial met stabiele id `markeer-klaar-slot`. `SaveZaak` rendert het partial opnieuw met verse completeness-telling en stuurt het OOB mee in de response (zelfde mechaniek als het groene bolletje per tab). Knop springt direct aan/uit op het juiste moment.

## Tooling

- **DEPLOY.md** vandaag toegevoegd voor het devops-team dat dit op Kubernetes wil draaien (poort, env-vars, migratie-model, Postgres-sizing, geen object storage nodig).
- **voortgang.md** als Nederlandstalig snapshot van wat er functioneel staat.

## Status van de openstaande items

| # | Item | Status |
|---|------|--------|
| 1 | OvJ barcode garbage trim | v0.14.7 |
| 2 | Feit + Pleeglocatie no autoparse | v0.14.7 |
| 3 | Type zitting +3 opties | v0.14.7 |
| 4 | Geboortedatum dd-mm-yyyy | v0.14.7 |
| 5 | Redirect na finalize | v0.14.7 |
| 6 | Uitloggen-knop | v0.14.7 |
| 7 | Sort arrows uploaddatum + deadline | v0.14.8 |
| 8 | Markeer als klaar direct klikbaar | v0.14.9 |
| 9 | Separate Locatie zitting dropdown | blocked op canonieke zittingslocaties-lijst van Isabelle/Jeffrey |
