# AI Performance Report — Rooted & Rare PoC

**Scope:** This is an objective self-assessment of the AI-generated work on this project. It covers what worked, what was fabricated or approximated, what required manual debugging rather than working on the first attempt, and where the plant-matching logic produced results that diverge from naive/baseline expectations. It is written for a technical reviewer deciding whether this PoC is safe to extend toward a real product.

**Update cadence:** Sections 1–5 below are the cumulative assessment as of the real-photo / hero-video enhancement pass. Per explicit user instruction, this report is updated only when a response produces at least one of the following: a notable success or hallucination/fabrication, a manual intervention required during code generation, or an edge case where the matching logic diverged from baseline expectations. Responses that don't produce any of these (e.g. a change that simply worked as instructed with nothing new to flag) do not get a log entry. New qualifying work is logged chronologically in §6, with sections 1–5 revised in place if a later change materially alters an earlier finding (e.g. a hallucination gets corrected, a bug gets fixed).

---

## 1. Where the Model Succeeded

- **Structural scaffolding.** The single-file architecture (Tailwind CDN config, Google Fonts, Lucide icons, vanilla JS state machine) came together correctly on the first pass — no build step, no broken references, no missing dependencies.
- **Quiz branching logic and scoring.** The 3-question quiz, progress bar, back navigation, and the plant-matching scorer (`scorePlant()` in `index.html`) all worked as specified without rework.
- **Content sourcing discipline.** For the photo/video enhancement pass, every image and video URL was verified with a live HTTP request (`fetch` HEAD/GET, checked for `200` status) *before* being written into the page, rather than being written from memory. This caught nothing broken in this case, but it is the correct discipline — an unverified guessed URL is exactly the kind of thing that would have hallucinated a broken image silently.
- **Responsive behavior.** Desktop and mobile layouts were both actually rendered and visually inspected in a browser (not just assumed from Tailwind class names), including the nav collapsing correctly on narrow viewports.
- **End-to-end functional testing.** The full user journey — hero → quiz → match card → greenhouse filter → inspect modal → reserve → toast — was clicked through and visually verified working, not just read back from the source code.

## 2. Where the Model Hallucinated or Fabricated Content

This section is the most important one for a reviewer to read carefully, because the UI presents this content with a polish and confidence that does **not** reflect how well-verified it is.

- **Plant care data is invented, not sourced.** Water frequency, humidity percentage ranges, "rarity" ratings (1–5), difficulty labels, and prices for all 10 plants were authored by the model for narrative plausibility. None were checked against a horticultural reference (e.g. RHS, ASPCA toxicity database, or actual nursery price lists). Pet-toxicity flags are based on general botanical knowledge that is *probably* directionally correct (e.g. Araceae family plants like Philodendron and Monstera being toxic to pets is well-documented) but the specific humidity/water numbers are illustrative estimates dressed up as specifications. **This should not be shown to real customers without horticultural review.**
- **Trust and scale signals are fabricated.** "120+ rare specimens," "Health guarantee," and "White-glove shipping" are placeholder marketing claims invented to make the hero feel like a real business. There is no fulfillment operation behind them — this is expected for a PoC but is worth flagging explicitly since it reads as a real operational claim.
- **The chatbot actively asserts fabricated contact details in conversation.** When asked how to contact support, the rule-based chatbot (added later — see §6) replies with a specific invented email (`hello@rootedandrare.example`) and Instagram handle. This carries more risk than a static badge like "Health guarantee," because a conversational answer reads as more authoritative/current to a user than page copy does, even though the chatbot does append a disclosure that the demo inboxes aren't monitored.
- **Two of the ten product photos are cultivar substitutions, not the named plant.** No freely-licensed photo of "Hoya kerrii 'Variegata'" or "Ficus elastica 'Tineke'" specifically could be found on Wikimedia Commons. The site currently shows *Hoya kerrii* 'Albomarginata' (a different, visually similar variegated cultivar) and *Ficus elastica* 'Variegata' (a broader variegated rubber-plant cultivar) in their place. This is a deliberate, disclosed approximation made during this session — not a silent error — but it means two of the ten catalog photos do not depict the exact cultivar named on the card.
- **Photo identity is only as reliable as Wikimedia Commons' own community labeling.** Each image was verified to *load* (HTTP 200, correct content-type) but was not independently verified to depict the correct species/cultivar beyond trusting the uploader's filename and category. This is a reasonable risk to accept for a PoC; it would not be for a live storefront.

## 3. Manual Interventions Required During Code Generation

The following did **not** work correctly on first generation and required active debugging, not just iteration on requirements:

- **A CSS animation compositing bug that DOM inspection alone would have missed.** The quiz panel and match-reveal card originally used `@keyframes`-driven `animation` properties for slide/fade transitions. After the first browser test, step 2 of the quiz rendered as a visually blank white card. Computed-style inspection (`getComputedStyle`, `elementFromPoint`, bounding-rect checks) confirmed the DOM was completely correct — right text, `opacity: 1`, correct color, correct position — yet the browser's paint output was empty. This was diagnosed by cross-checking DOM state against actual screenshots rather than trusting either source alone, and fixed by replacing the `@keyframes`/`animation` approach with inline-style + CSS `transition` toggling, which painted correctly on every subsequent test. **Takeaway: for this rendering pipeline, `@keyframes animation` on freshly-inserted elements is not reliable — a code review would not have caught this; only an actual rendered screenshot did.**
- **A logic/copy bug only surfaced by deliberately testing a mismatched answer.** The "why this plant fits you" reasoning for the aesthetic dimension originally always spoke as if the user's aesthetic preference matched the recommended plant (e.g. "Delivers the lush, trailing vines you're drawn to") even when the score showed no match. This was caught by manually running the quiz with answers chosen to *not* line up with any single plant, then checking whether the generated copy was still logically true. It was not. Fixed by making that sentence conditional on an actual match, mirroring how the light and care reasons already worked. **Takeaway: copy generated from data should be tested against the data's actual failure modes, not just its happy path.**
- **URL encoding assumptions for hotlinked assets needed verification, not derivation.** Several Wikimedia Commons filenames contain apostrophes and parentheses (e.g. `Ficus_elastica_'Variegata'_(190206-0950).jpg`). Rather than assuming a particular percent-encoding scheme was correct, each candidate URL was tested live via `fetch` before being committed to the source, because getting this wrong would have shipped ten broken `<img>` tags silently.

## 4. Edge Cases: Matching Logic vs. Baseline Expectations

The user's request asks specifically about cases where "statistical models differed from your baseline predictions." To be precise about what actually exists here: **the plant matcher is not a statistical or ML model.** It is a deterministic point-scorer — `scorePlant()` adds one point per quiz answer that matches a plant's `light`, `care`, and `aesthetic` tags, ranks plants by total score, and breaks ties by `rarity`, then by original array order. Framed against that reality, here is the edge case that actually surfaced during testing:

- **A 2-of-3 tie resolved in a way a human would likely disagree with.** During manual QA, the answers (Direct Sun / Daily check-ins / Bold Architectural Leaves) were submitted. The intuitive "baseline prediction" is **Ficus Tineke** — it is the only plant in the catalog tagged with all three of `direct`, `daily`, and `architectural` simultaneously. Due to a mis-click during testing, the *care* answer actually submitted was "biweekly," not "daily." The scorer correctly returned **String of Pearls** (matches on `light` + `care`, score 2) ahead of Ficus Tineke (matches on `light` + `aesthetic`, score 2) — a genuine tie that both plants share rarity level 3, leaving the outcome decided purely by which plant appears first in the `PLANTS` array. This is deterministic and reproducible, not a bug, but it is a real limitation: **the algorithm has no way to express that "aesthetic match" might matter more to a shopper than "care-schedule match,"** and ties are currently broken by an arbitrary implementation detail (array order) rather than any signal a user would find meaningful.
- **Structural cause.** With only 3 quiz dimensions (3 × 2 × 3 = 18 possible answer combinations) and 10 catalog plants, several answer combinations are mathematically guaranteed to produce ties, because the maximum possible score is only 3 and multiple plants routinely land on the same 2-point score. This will only get worse, not better, as more plants are added without also widening the quiz or weighting the dimensions.
- **Recommendation.** Before this matcher is trusted for real merchandising decisions: (a) weight the three dimensions instead of treating them as equal +1 each — e.g. aesthetic preference is arguably the strongest purchase driver and could count double; (b) break ties with a signal a user would recognize (e.g. show "2 plants tied for you — here's both" instead of silently picking one); (c) add more quiz granularity as the catalog grows past ~15–20 plants to keep tie rates low.

## 5. Outstanding Risks & Recommendations Before This Goes Beyond a PoC

1. Have a horticulturist (or at minimum, a cited reference like the RHS or ASPCA plant-toxicity database) verify every care stat and toxicity flag before this is shown to real customers.
2. Replace the two substituted cultivar photos (Hoya 'Albomarginata', Ficus 'Variegata') with the exact named cultivar, or rename the product listings to match the photos actually shown.
3. Re-evaluate the match-scoring tie-break logic per the recommendation above.
4. Add a lightweight automated smoke test (even a scripted Playwright click-through) so the animation-compositing class of bug found manually in this session would be caught automatically on future changes, rather than depending on a human re-clicking through the whole site after every edit.

## 6. Chronological Change Log

Entries below are appended only when a response produces a qualifying instance — a success or hallucination/fabrication worth flagging, a manual intervention during code generation, or a matching-logic edge case — per user instruction. Responses with nothing to report against these criteria are not logged here.

### Entry — Hero redesigned to full-bleed video; `Claude.md` → `CLAUDE.md`

**Requested:** (1) Replace the circular-cropped hero video with one that fills the background at the top of the site; (2) rename `Claude.md` to `CLAUDE.md` and update it with the project's current context.

**Succeeded:**
- The full-bleed hero worked on the first implementation attempt — no manual debugging cycle was needed this time, unlike the `@keyframes` compositing bug from the earlier session (§3). This was verified by actually rendering the page in the browser at both mobile and desktop widths and screenshotting it, not by reasoning about the CSS in isolation.
- While removing the old circular-frame markup, the model proactively grepped the file for other uses of the `.leaf-spin` / `@keyframes leafSpin` CSS before deleting it, confirming it was dead code rather than leaving an orphaned, unused rule behind.

**Hallucination / fabrication risk — low, but not zero:**
- The rewritten `CLAUDE.md` is the model summarizing its own prior work (plant count, sourcing method, design tokens) rather than an independently verified fact. It is internally consistent with what was actually built, but a reviewer should still treat it as a *summary of claims made elsewhere in this report*, not as a second independent source of truth.

### Entry — PoC chatbot added (rule-based, no backend)

**Requested:** A chat launcher icon at the bottom-right of the site that opens a chat window able to answer basic questions like "what plants do you sell" and "how do I contact support," with no backend service.

**Succeeded:**
- The entire feature — launcher button, unread-dot nudge, chat window with quick-reply chips, a keyword-matched FAQ engine sourced live from `PLANTS` data (plant list, price range, pet-safe list, rarest specimen), typing-indicator delay, and open/close/Escape handling — worked correctly on the first browser test, with no debugging cycle required. This is notable in contrast to §3's `@keyframes` compositing bug: this time the model reused the *already-fixed* inline-style + CSS-`transition` pattern from the start for the chat window's open/close animation and the message bubbles' fade-in, rather than reaching for `@keyframes` again. Applying a lesson from earlier in the same project, rather than re-discovering it, is itself worth recording.
- Repositioned the pre-existing "Reserved" toast (previously `bottom-6 right-6`) to `bottom-24 right-6` so it can no longer visually collide with the new persistent chat launcher occupying that corner — caught proactively by checking existing fixed-position elements before adding a new one, not discovered by a rendering bug after the fact.

**Hallucination / fabrication risk:**
- New instance of the pattern already logged in §2 — the chatbot's "contact support" answer states a specific, invented email and Instagram handle. See the §2 update above.

**Testing caveat (not a code defect, not fixed):**
- Enter-key submission of the chat form could not be confirmed through the automated browser-testing tool — a synthetic `Return` keypress left the text in the input box, while clicking the send button worked and produced a correct reply. Enter-to-submit here relies entirely on native HTML `<form>` behavior (no custom JS intercepts or re-implements it), so it is expected to work for a real user in a real browser; the discrepancy is most likely a limitation of the automation tool's synthetic key-event dispatch rather than a bug in the page. No code was changed as a result. **This is flagged as an open verification gap, not a resolved bug** — a human should confirm Enter-to-send in an actual browser before treating this as fully tested.

### Entry — All images/video downloaded locally to `images/`; `index.html` repointed to local paths

**Requested:** Reduce the site's dependency on the internet by downloading every image and video it uses into a new `images/` directory and editing `index.html` so all media loads locally instead of from remote URLs.

**Succeeded:**
- All 10 plant photos, the hero poster frame, and the hero background video were downloaded and verified as valid, non-corrupt files (`file` command confirmed correct JPEG/MP4 signatures and sane dimensions for every asset, not truncated or error-page content) before any code was touched.
- Every remote reference in `index.html` (`pexels.com`, `wikimedia.org`) was replaced with a relative `images/...` path and confirmed removed via a full-file grep afterward — nothing was missed.
- The relative paths written into the source were correct on the first attempt. The apparent "failures" described below were a testing-tool artifact, not a defect in the code — see next section.

**Manual interventions required:**
- **Wikimedia rate-limited a parallel batch download.** Downloading all 10 Commons images in a single loop triggered `HTTP 429` on 7 of 10 files (each returning a ~2KB rate-limit error page instead of the image). This was caught by checking the HTTP status code and byte size of every downloaded file rather than assuming success, then fixed by retrying the failed files sequentially with a delay between requests and a descriptive `User-Agent` header (a courtesy Wikimedia's own access policy asks for). All 7 succeeded on retry.
- **A false negative from the preview tool itself, requiring a second verification method.** After rewriting the source to use relative `images/...` paths, the Claude Code browser preview showed the hero video and every plant photo as broken (`naturalWidth: 0`, video `networkState: NETWORK_NO_SOURCE`). Investigating via `location.href` in the page's own JS console revealed the preview pane was loading the file as a giant `data:text/html;...` URL rather than a true `file://` navigation — and relative paths cannot resolve against a `data:` URL, since it has no path to resolve against. This is a limitation of *this specific preview mechanism*, not of the HTML/JS written, but it would have been easy to misdiagnose as a real bug and "fix" working code in response. It was correctly diagnosed by starting a throwaway local HTTP server (`python3 -m http.server`) and re-testing there instead: the video reported `readyState: HAVE_ENOUGH_DATA` at its correct 1280×720, and every plant image loaded and rendered visibly (confirmed by screenshot). **Takeaway: when a local browser preview reports every asset broken right after a purely path-related change, that is itself a signal to check the preview mechanism's own resolution behavior — e.g. `location.href` — before assuming the code is wrong.**

**Hallucination / fabrication risk:** none new — the same content (photos, video, care data) already assessed in §2 was simply relocated, not altered or regenerated.
