# KDP Audit Report: From Vibe to Production

## Summary
- **Status**: NEEDS WORK
- **Manuscript type**: Technical nonfiction (AI / software engineering)
- **Format**: Markdown source (19 files) + compiled PDF interior
- **Target**: Both (eBook + paperback)

## Critical Gaps (Must Fix)
1. **Print trim size is A4, not a KDP US standard.** The compiled interior is 8.27 x 11.69 in (A4). KDP US trims for this kind of book are 6x9 (trade) or 7x10 (code-heavy technical). The print interior must be regenerated at a chosen trim. *Decision needed.*
2. **Print margins have no gutter and are not mirrored.** The current interior uses symmetric ~28mm/25mm margins. For a 151-300 page B&W paperback, KDP requires an inside (gutter) margin of at least 0.5 in, mirrored across odd/even pages. Regenerating at the chosen trim fixes this.
3. **No eBook file in an ideal format.** Source is markdown plus a print PDF. An EPUB with a hyperlinked TOC gives the best Kindle result. *(Generating now in Phase 5.)*
4. **Paperback full-wrap cover missing.** Only a front cover exists. Paperback needs a single wraparound PDF: back cover + spine + front, sized to trim, page count, and paper. *(Building in Phase 6; spine depends on the trim and final page count.)*
5. **No copyright page.** Front matter has cover, title page, TOC, and introduction, but no copyright page. KDP and readers expect one (copyright line, rights reserved, edition, ISBN/ASIN slot). Easy to add.

## Warnings (Should Fix)
1. **Color interior drives up paperback cost.** The 18 coral hero images and colored diagrams make this a color print interior, roughly 3-4x the per-page printing cost of B&W. Standard move for a technical book is a grayscale paperback interior (the eBook keeps full color). *Decision needed.*
2. **eBook cover ratio and extension.** `cover.png` is 1696 x 2528 px (ratio 1.49:1); KDP's ideal eBook cover is 1.6:1, e.g. 1600 x 2560. The file is also JPEG data with a `.png` extension. A correctly sized `ebook-cover.jpg` will be produced in Phase 6.
3. **Listing metadata absent.** Blurb, keywords, categories, and author bio are not yet written. *(Generating in Phase 3.)*
4. **SVG diagrams for Kindle.** SVGs are fine in the print PDF; for the EPUB they will be rasterized to PNG, since Kindle's SVG support is inconsistent. *(Handled in Phase 5.)*

## Interior Formatting
- [ ] Trim size: 8.27 x 11.69 in (A4) — not a KDP US standard
- [ ] Margins: symmetric, no mirrored gutter (needs inside >= 0.5 in for this page count)
- [x] Fonts embedded and standard: Helvetica body, Courier code (9pt, above the 7pt minimum)
- [x] Page numbers start after front matter, centered
- [x] Table of Contents present (will be hyperlinked in the EPUB)

## Cover Specifications
- [ ] eBook cover: 1696 x 2528, ratio 1.49:1 (ideal 1.6:1), JPEG mislabeled `.png`
- [ ] Paperback full-wrap: not yet built (front only)
- [x] Front cover content: title legible, author present, high contrast, no prohibited content

## Metadata
- [x] Title and subtitle: "From Vibe to Production" / "The Definitive Guide to Shipping Real AI Agents"
- [ ] Description/blurb (Phase 3)
- [ ] Categories, up to 3 BISAC (Phase 3)
- [ ] Keywords, up to 7 (Phase 3)
- [ ] Author bio (Phase 3)
- ISBN: not required; KDP provides a free ASIN (eBook) and ISBN (paperback)

## eBook Validation
- [x] Largest image 2.6 MB (under the 5 MB per-image guideline)
- [ ] EPUB not yet generated; epubcheck is not installed (manual checks will be noted)

## Recommended Next Steps
1. Decide print trim (6x9 vs 7x10) and paperback interior color (grayscale vs color). These set spine width and print cost.
2. Generate the EPUB (Phase 5) and the listing artifacts (Phase 3) now; neither needs the trim decision.
3. Regenerate the print interior at the chosen trim with mirrored margins, add a copyright page, then build the full-wrap cover with the real page count (Phase 6).
4. Produce a correctly sized `ebook-cover.jpg` (1600 x 2560).
