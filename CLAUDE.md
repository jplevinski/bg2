# Born Good — Project Notes for Claude

Single-file site: everything lives in `index.html` (HTML/CSS/JS inline, no
build step, no other source files). Every change in this repo is made
directly in that file.

## Standing instructions

- **No stat cap.** Stats (`STATS.core` / `STATS.soul`) are uncapped. The
  `max: 40` field on each stat is only a reference for scaling the visual
  bar width (`Math.min((val/s.max)*100,100)`) — never clamp or cap the
  displayed numeric value itself.
- **HP/MP must match the formulas** on the Battle Mechanics page:
  - `HP = (STR + VIT×2) × 3`
  - `MP = (WIS + INT×2) × 3`
  Whenever core stats change for a chapter, recompute HP_MP for that
  chapter and verify it against the formula before committing.
- **Elias's age is sixteen**, not fourteen — check any new/edited passages
  that mention his age.
- **No videos in chapters.** All in-chapter media are static `<img>` tags
  wrapped in `.chapter-media` (with `beastLightboxOpen('image', ...)` for
  the lightbox) — do not add autoplay/click-to-play video elements.
- **Spoiler gates are generic** — the modal/confirmation text must never
  reveal specific plot details, just a general "you may be reading ahead"
  warning. Reuse the existing `#spoiler-confirm` modal and
  `confirmSpoiler()` / `confirmSpoilerPage()` / `proceedSpoiler()` pattern
  rather than inventing new gating mechanisms.
- **Author byline stays "J.S. Plevinski, Ph.D."** — the Ph.D. is
  intentional, part of the "career educator" framing already established
  on the About page.
- **Stylistic voice constraints for Elias's prose:** avoid overusing "the
  specific…", "I ran the math", "the arithmetic", "the ledger", "the
  stubborn idiot who lives in my mouth", retrospective foreshadowing
  lines, and keep — do not reduce — "the way…" comparison constructions.
- **Time Invested figures must be calculated from real project data**,
  not guessed round numbers:
  - Writing hours = live total word count across all chapter pages ÷
    1,000 words/hour.
  - Web build hours = extrapolated from `index.html` line-count growth
    since the last time the figure was set, using the previously
    established lines-per-hour rate (check `git log -S "<old hrs text>"
    -- index.html` to find the baseline commit).
  - Music/visuals hours = weeks elapsed × the user's stated weekly hours
    (ask if unknown — do not assume).
  - Ask the user for current weeks-elapsed / weekly-hours if not already
    stated in this session; do not carry forward stale assumptions.

## Adding a new story chapter (main story or Glintjaw side-story)

Follow this exact checklist, in order, for every new chapter:

1. Add an entry to the chapters overlay (`#chapters-overlay`) / relevant
   index page (e.g. `#page-glintjaw` chapter-card list).
2. Add a chapter card to the home page (main story) if applicable.
3. Add a "→ Chapter N" nav button to the *previous* chapter's
   `.chapter-nav`.
4. Build the full chapter page: `.chapter-text`, end-of-chapter dashboard
   (`.chapter-end-dashboard.show-desktop` with collapsible
   Stats/Inventory/Abilities/Summary), and `.chapter-sidebar` with
   start-of-chapter collapsibles.
5. Extend `STATS.core` and `STATS.soul` with the new chapter's row,
   uncapped, with soul-stat deltas justified by the chapter's events.
6. Add an `HP_MP` entry, verified against the HP/MP formulas above.
7. Extend `ABILITIES` for the chapter.
8. Extend `INVENTORY` with narratively-flavored item descriptions.
9. Extend `BEGIN_INV_MAP` / `END_INV_MAP`.
10. Add all render-call invocations for the new chapter/suffix pairs:
    `renderStats` (×2), `renderInventory` (×2), `renderAbilities` (×2),
    `renderEndSummary` (×1).
11. Add `SOUL_REASONS` tooltip entries for all six soul stats
    (Mercy, Conviction, Resentment, Connection, Hope, Identity).
12. Update character profiles (`MAIN_CHARS` on the Characters page) — see
    "Character ability tracking" below.
13. Verify before committing (see below).
14. Commit and push to both the working branch and `main` (see below).

## Character ability tracking

Whenever new chapter material is added or edited — main story, Glintjaw,
or any other POV — automatically do this without being asked:

- Read the new material for any spell, ability, skill, or notable power
  used or newly acquired by a named character, not just Elias.
- For any character with an existing profile in `MAIN_CHARS` (currently
  Elias, Kip, Cassian — check for new ones as the roster grows), add
  newly revealed abilities to that character's `abilities` array. Match
  the existing entry format: `{ icon, name, type, desc }`, description
  written for the character sheet (third person / descriptive, not
  Elias's first-person chapter voice).
- If an ability already exists on the sheet but has visibly leveled up,
  changed scope, or gained a new use, update its `desc` rather than
  duplicating the entry.
- Keep Elias's `MAIN_CHARS.elias.abilities` in sync with the latest
  chapter's `ABILITIES` entry (same skills, sheet-appropriate phrasing)
  every time `ABILITIES` is extended per the chapter checklist above.
- **Create a new `MAIN_CHARS` profile automatically whenever a new named
  character is introduced** — do not wait to be asked. Build the full
  profile in the same shape as the existing entries (`name`, `role`,
  `level`, `img`, `desc`, `stats`, `soul`, `abilities`, `achievements`),
  populated from what the chapter actually reveals (leave fields like
  `soul` as `null` if the story hasn't shown that character's Soul
  Balance). Add the matching `char-card` entry to the Characters page
  (Main/Major/Minor tier, whichever fits) and wire it to
  `openCharOverlay('id')` so the new profile is actually reachable.
  Use a placeholder `img` path following the existing `images/name.png`
  convention and flag to the user that the portrait still needs to be
  supplied — don't skip creating the profile just because artwork isn't
  available yet.
- This applies retroactively too: if a character was introduced in
  earlier material and never got a profile (e.g. Silas), create one the
  next time you touch character data, rather than only handling it going
  forward.

## Verification (required before every commit)

Run a Node.js check that extracts all `<script>` blocks and confirms they
parse and, for data changes, that the render functions actually populate
their target containers:

```
node -e "
const fs = require('fs');
const html = fs.readFileSync('index.html','utf8');
const scripts = [...html.matchAll(/<script>([\s\S]*?)<\/script>/g)].map(m=>m[1]);
scripts.forEach((s,i)=>{ try { new Function(s); } catch(e){ console.log('Error',i,e.message); process.exit(1);} });
console.log('OK', scripts.length);
"
```

## Git workflow

- Working branch: `claude/mirror-borngood-repo-im10mk`, mirrored to
  `main`.
- Every change gets committed with a descriptive message and pushed to
  **both**:
  ```
  git push origin claude/mirror-borngood-repo-im10mk
  git push origin claude/mirror-borngood-repo-im10mk:main
  ```
