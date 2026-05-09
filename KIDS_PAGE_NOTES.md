# Deer Park Kids Page — Build Notes

`kids.html` is a single self-contained file. No external CSS, JS, fonts, or images. Drop it on the server at `/kids` (or `/kids.html`) and it works.

## What's in it

1. **Hero** — civic-tone headline matching the parent site's serif/italic treatment ("Deer Park, *explained*"). Eyebrow tag flags it as a free learning resource.
2. **AQI Stoplight** — 4-light dashboard (Green/Yellow/Orange/Red) with a kid-readable explanation of AQI. Default value is 42 (Good) so the page never looks broken pre-wire-up.
3. **"What's a refinery?"** — 5-step scroll explainer with a sticky inline SVG that morphs through stages: ship arrives → tower heats oil → products separate → distribution → people who run it. Uses IntersectionObserver, falls back to step 1 visible if unsupported.
4. **Shelter-in-Place "Safety Superhero" checklist** — 6 click-to-check items (keyboard accessible: Tab + Space/Enter), live progress counter, reset button. Framed positively, never alarming.
5. **5-question quiz** — Multiple choice, immediate feedback with a "Why:" explanation per question, animated progress bar, and a Deer Park Safety Star SVG badge at the end.
6. **Printable activity sheet** — Hidden behind `@media print`. Includes:
   - SVG line-art Deer Park skyline (smokestacks, distillation tower, storage tanks, an air monitor) for coloring
   - 10×10 word search with 8 hidden words (AIR, SAFE, INSIDE, RADIO, CLEAN, GREEN, OXYGEN, HELP) — generated client-side
   - 5 fill-in-the-blank sentences
   - Name/Grade/Date header that only appears on print
7. **Schema.org** — Full `LearningResource` JSON-LD in the `<head>` with audience, age range, learning objectives, and `hasPart` for each section. Includes a `BreadcrumbList`. Plus a nested `Quiz` entity. This is the SEO indexability play.

## Wiring the AQI value to the existing site's data

The page exposes a global function:

```js
window.setDeerParkAQI(value)   // e.g. window.setDeerParkAQI(73)
```

Wherever the homepage already calls AirNow / TCEQ and updates the AQI ticker, add one line at the end of that callback:

```js
if (typeof window.setDeerParkAQI === 'function') window.setDeerParkAQI(currentAQI);
```

Alternative: set `<body data-aqi="73">` server-side and call `window.setDeerParkAQI()` with no args — it'll read the attribute. The function handles all 6 EPA bands (Good → Hazardous), updates the stoplight, the badge color, the band label, and the kid-friendly message.

If you want it to share a fetch with the homepage, lift the AQI fetch into a small shared script (e.g. `/assets/aqi.js`) and have both pages subscribe. For now the standalone default keeps the page looking complete out of the box.

## Where to link this from the existing site

**P0 — Add to top nav.** Insert "For Kids" between "Air Quality" and "Jobs" in the main `nav-jump` row. It belongs next to Air Quality because that's the same audience (parents checking on kids).

**P1 — Hero promo strip.** Just below the hero photo on the homepage, add a slim 1-line strip:

> *New: A free [kids' page about Deer Park air, safety, and how a refinery works](/kids) — for classrooms and families.*

This is the "this is a real publication, not a domain flip" signal — a serious civic site doesn't build a kids' education page unless it cares about the community.

**P2 — Footer "Explore" column.** Add `For Kids & Families →` alongside the existing `About Pemex Deer Park` and `Port of Houston Guide` links.

**P3 — AQI module deep-link.** Wherever the homepage's AQI box renders, add a small "Explain to a kid →" link that anchors to `/kids#aqi-module`.

## Partnership / outreach angles

These are real institutions in/near Deer Park that would actually use this. A short, friendly outreach email to each is high-leverage for both community trust AND backlinks (which compounds the SEO value of the LearningResource markup).

- **Deer Park ISD curriculum coordinator** — pitch as a free supplement to their elementary science / community-awareness units. ISD already does shelter-in-place drills (industry-adjacent district), so the safety section is directly classroom-relevant. Ask: would they link from their parent resources page?
- **San Jacinto College — Process Technology program & Early Childhood program** — Process Tech is the primary career pipeline into PMX/LyondellBasell/etc., so the refinery explainer is on-brand for their community outreach. Their Early Childhood program runs a lab preschool that could use the activity sheet.
- **Harris County Public Library — Deer Park branch** — they do family STEM nights. The printable activity sheet is the hook. Offer to provide a print-ready PDF (one-pager).
- **Deer Park LEPC (Local Emergency Planning Committee)** — federally mandated body that handles community shelter-in-place education. Our checklist mirrors their guidance; ask them to vet it and link to it from their community resources.
- **Deer Park ISD PTAs** — the activity sheet + quiz is exactly what PTAs share in newsletters before standardized testing weeks or summer.
- **Texas A&M AgriLife / 4-H Harris County** — they run environmental literacy programs; the AQI explainer is curriculum-aligned.

For each, the ask is small: "We made this free thing for the community — would you take a look and consider linking it / printing it?" Even a 20% response rate gives you several local-domain inbound links to a `LearningResource`-marked page, which Google treats very kindly.

## Accessibility notes

- All SVGs have `<title>` elements and `aria-label`; decorative SVGs are `aria-hidden`
- Skip-to-main link
- Keyboard: checklist items are `role="checkbox"` with Space/Enter handling; quiz options are real `<button>`s; Tab order is sensible top-to-bottom
- `prefers-reduced-motion` disables all animations and smooth scroll
- Color contrast: navy text on cream paper exceeds WCAG AA. Stoplight uses both color AND text labels (GO/OK/EASY/IN) so it doesn't rely on color alone
- Reading level is grade 4–6 (intentionally readable by ~age 9)
- Font sizes: 18px body, 1.15rem lede, large tap targets on mobile

## Mobile

Mobile-first throughout. The stoplight stacks vertically below 380px, the scrolly grid collapses to single column below 880px, and the checklist + quiz options are full-width tap targets. Tested mentally at 375px (iPhone SE) and 768px (iPad).

## Print

Hitting Cmd/Ctrl-P (or the in-page print button) hides the header nav, the hero meta row, the AQI dashboard, the refinery scrolly, the checklist, the quiz, and most of the footer. Only the activity sheet block prints — clean letter-size, .6in margins, with a Name/Grade/Date row at the top and a small attribution at the bottom. Coloring SVG strokes are forced to black with no fills.

## What I'd do next (not in scope, but cheap wins)

- Spanish version (`/kids/es`) — large Spanish-speaking population in Deer Park; literal direct translation of this page would take ~1 hour
- A 1-page print-optimized PDF version of the activity sheet (use the existing print stylesheet, save to PDF, host as `/kids/activity.pdf` for libraries to download)
- Add OG image — a simple SVG-to-PNG export of the safety star badge would make this page share well on Facebook (where Deer Park parents actually are)
- Track the print button click as an analytics event — this is the single best engagement signal for whether teachers are actually using it
