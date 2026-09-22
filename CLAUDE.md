# CLAUDE.md — Master Instruction & Context Document

## Project Goal

Build a proof-of-concept (PoC) webpage. The business was decided during this
project: **Rooted & Rare**, an upscale boutique nursery specializing in rare,
ethically propagated tropical plants and terrariums. The site is a single,
self-contained frontend deliverable — no backend, no database, no real order
fulfillment. It exists to demonstrate an interactive shopping/discovery
experience, not to run a real store.

## Status

- [x] Business/niche decided — Rooted & Rare (boutique rare-plant nursery)
- [x] Site structure defined
- [x] Visual design/branding decided (see Design System below)
- [x] Content drafted (hero copy, quiz copy, 10-plant catalog)
- [x] PoC page implemented (`index.html`)
- [x] Real photography and hero video sourced and integrated
- [ ] Care data / pricing verified against real horticultural sources (see `AI_Performance_Report.md` §2)
- [ ] Match-scoring tie-break logic reviewed (see `AI_Performance_Report.md` §4)

## What's Been Built

`index.html` is a single-file, self-contained web app (Tailwind CSS via CDN,
Lucide icons via CDN, vanilla JS — no build step, no framework, no backend).
It contains:

1. **Sticky nav** — logo, anchor links (Home / Plant Matchmaker / Greenhouse), CTA button.
2. **Hero** — full-bleed looping background video of a greenhouse (muted,
   autoplay, `poster` fallback) with a gradient overlay for text legibility,
   headline, value proposition, and two CTAs. Video sourced from Pexels
   (credited on-page and in the footer).
3. **Plant Matchmaker quiz** (`#quiz`) — a 3-step card quiz (lighting → care
   lifestyle → aesthetic preference) with a progress bar, back navigation,
   and inline-style/CSS-`transition`-based slide animations between steps
   (see the "Manual Interventions" note below for why it's not `@keyframes`).
4. **Results dashboard** (`#results`) — a "Your Top Match" hero card computed
   from the quiz answers via a deterministic point-scorer (`scorePlant()`),
   plus an always-browsable "Explore the Greenhouse" grid of all 10 plants,
   filterable by difficulty and light requirement.
5. **Inspect modal** — click any plant card to see care stats (water,
   humidity, light, pet safety) and a "Reserve Specimen" button that shows a
   toast confirmation and flips to a persistent "Reserved" state.
6. **Plant catalog** — 10 plants, each with a real photo sourced from
   Wikimedia Commons (see Content Sourcing below).
7. **Footer** — brand mark, photo/video attribution.

## Design System

- **Palette:** deep forest greens (`forest-*`), warm cream/linen background
  (`linen`), terracotta accent (`terracotta-*`), a muted gold for rarity/
  difficulty accents (`gold-*`). Defined in the inline `tailwind.config` in
  `index.html`.
- **Type:** Playfair Display (serif, headings) + Inter (sans, body), loaded
  via Google Fonts.
- **Tone:** premium, botanical, editorial — not a generic SaaS look.

## Content Sourcing

- **Plant photography:** real photos, not illustrations, hotlinked from
  Wikimedia Commons via the `Special:FilePath/<file>.jpg?width=700` pattern.
  Every URL was verified with a live HTTP request before being committed to
  the source. Two of the ten (Hoya Kerrii and Ficus Tineke) are
  approximated with the closest available *cultivar* photo rather than the
  exact named cultivar — flagged in `AI_Performance_Report.md`.
- **Hero video:** "Close up on Plants in Greenhouse" by Vincuk Konan, sourced
  from Pexels (free license), hotlinked directly from `videos.pexels.com`.
- **Care stats, prices, rarity ratings:** authored/estimated for narrative
  plausibility, **not** verified against a horticultural reference. Treat as
  placeholder content — see `AI_Performance_Report.md` §2 before using this
  data in front of real customers.

## Known Limitations / Follow-Ups

Full detail lives in `AI_Performance_Report.md`. Summary:

- Care/toxicity/price data needs a real horticultural review pass.
- The match-scoring tie-break (equal scores fall back to catalog array
  order) can produce a top match a human would disagree with — see §4 of
  the report for a concrete reproduced example and suggested fixes
  (weighting dimensions, surfacing ties instead of silently picking one).
- No automated test suite exists; verification so far has been manual
  click-through testing in the Claude Code browser pane after each change.

## Structure

- `CLAUDE.md` — this file; master context and instructions for future sessions.
- `logs/` — granular state and context retention logs, one file per session (currently placeholders).
- `docs/README.md` — technical/architecture notes as the PoC grows (currently a placeholder).
- `index.html` — the PoC webpage itself. This is the entire deliverable.
- `AI_Performance_Report.md` — objective assessment of where the LLM succeeded, hallucinated, needed manual debugging, or produced matching-logic edge cases.

## Notes for Future Sessions

Read this file first, then `AI_Performance_Report.md` for known risks, then
the most recent file in `logs/` for session-level state before continuing
work.
