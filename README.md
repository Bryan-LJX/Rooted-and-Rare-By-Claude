# Rooted & Rare 🌿

A single-file, interactive proof-of-concept website for **Rooted & Rare**, a fictional upscale boutique nursery specializing in rare, ethically propagated tropical plants and terrariums.

Built as a frontend-only demo — there is no backend, database, or real e-commerce behind it. Every interaction (the quiz, filtering, reservations, chat) runs entirely in the browser.

## Features

- **Botanical, editorial design** — deep forest greens, warm linen, terracotta accents, Playfair Display + Inter typography.
- **Full-bleed hero video** of a greenhouse walkthrough.
- **Plant Matchmaker quiz** — a 3-step branching quiz (light, care habits, aesthetic) that scores the catalog and recommends a top match.
- **Virtual greenhouse** — a filterable grid of 10 rare plants (by difficulty and light requirement), each with a real photo and an "Inspect" modal showing water/humidity/light/pet-safety stats and a mock "Reserve Specimen" flow.
- **PoC chatbot** — a rule-based assistant (bottom-right) that answers basic questions like "what plants do you sell" or "how do I contact support," with no backend or external API.

## Tech Stack

No build step, no package manager, no server. Everything runs from static files:

- **HTML/CSS/JS** — a single `index.html`, vanilla JavaScript throughout.
- **[Tailwind CSS](https://tailwindcss.com/)** — via the CDN build (`cdn.tailwindcss.com`).
- **[Lucide Icons](https://lucide.dev/)** — via CDN.
- **[Google Fonts](https://fonts.google.com/)** — Playfair Display & Inter.
- All plant photos and the hero video are stored locally in [`images/`](images/) rather than hotlinked, so the site works offline once downloaded (fonts, Tailwind, and Lucide still load from their CDNs and require an internet connection).

## Getting Started

### 1. Get the files

Clone the repo:

```bash
git clone https://github.com/Bryan-LJX/Rooted-and-Rare-By-Claude.git
cd Rooted-and-Rare-By-Claude
```

Or download it without git: on this repo's GitHub page, click **Code → Download ZIP**, then unzip it.

### 2. Run it locally

**Option A — just open the file (simplest).** Double-click `index.html`, or open it from your browser with <kbd>Ctrl/Cmd</kbd>+<kbd>O</kbd>. Everything — quiz, greenhouse, chatbot — works straight from disk.

**Option B — serve it locally (optional, closer to production).** From the project folder:

```bash
python3 -m http.server 8000
```

Then visit `http://localhost:8000` in your browser. Any static file server (`npx serve`, VS Code's Live Server, etc.) works the same way.

No `npm install`, no build, no environment variables — there is nothing to configure.

## Project Structure

```text
.
├── index.html                # The entire site — markup, styles, and JS
├── images/                   # Locally hosted plant photos + hero video/poster
├── CLAUDE.md                 # Project context/instructions for future work
├── AI_Performance_Report.md  # Transparency report: AI successes, hallucinations,
│                              # manual fixes, and matching-logic edge cases
├── docs/
│   └── README.md             # (placeholder for future architecture notes)
└── logs/                     # (placeholder session logs)
```

## Content, Data & Attribution

This is a demo project, and its content should be read as such:

- **The business is fictional.** "Rooted & Rare" is not a real company. Any resemblance to an actual nursery of the same or a similar name is coincidental and unintended.
- **Plant care data, prices, and rarity ratings are illustrative, not verified.** Water/humidity schedules, pet-toxicity flags, and prices were authored for narrative plausibility and have **not** been checked against a horticultural reference. See [`AI_Performance_Report.md`](AI_Performance_Report.md) for a full, itemized account of what was fabricated, approximated, or should be reviewed before this content is treated as factual.
- **Trust signals are placeholders.** Claims like "Health guarantee," "White-glove shipping," and the chatbot's listed contact email are demo copy — there is no real fulfillment operation or monitored inbox behind them.

### Third-party media

- **Plant photography** — sourced from [Wikimedia Commons](https://commons.wikimedia.org/) and used under each file's respective Commons license (typically CC BY-SA or public domain). See each image's Commons file page for exact terms and photographer credit.
- **Hero video & poster image** — "Close up on Plants in Greenhouse" by **Vincuk Konan**, via [Pexels](https://www.pexels.com/), used under the [Pexels License](https://www.pexels.com/license/) (free to use).
- **Fonts** — Playfair Display and Inter, via Google Fonts, under the SIL Open Font License.
- **Icons** — [Lucide](https://lucide.dev/), under the ISC License.
- **CSS framework** — [Tailwind CSS](https://tailwindcss.com/), under the MIT License.

If you reuse this repo, please keep the above attributions intact for any third-party assets you continue to use, and swap in your own photography/video if you turn this into a real product.

## License

No license has been declared for the original code in this repository (`index.html`, its structure, and the written project docs) — by default, all rights to that original work are reserved by the repository owner. If you're the owner and want to permit reuse, consider adding an [OSI-approved license](https://choosealicense.com/) such as MIT. Third-party assets remain under their own licenses as listed above regardless of any license later added to this repo.

## Disclaimer

This project was built as an educational proof-of-concept exploring AI-assisted frontend development. It is provided as-is, with no warranty of any kind, and is not intended for production or commercial use without further review — particularly of the care/pricing data and the placeholder trust and contact claims noted above.
