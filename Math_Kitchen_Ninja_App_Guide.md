# Math Kitchen Ninja — App Guide (project context)

Part of the **Kitchen Ninja HQ** suite. Single-HTML-file, iPad-optimised math app for J.J.'s son (Year 8 now). Cooking + chemistry framing, Singapore-math (bar model) method, ADHD-friendly, 12-year-old language.

---

## 1. Files

| File | Levels | Content |
|---|---|---|
| `math-part1-foundations.html` | 1–9 | Foundations |
| `math-part2-geometry-data.html` | 10–19 | Geometry & Data |
| `math-part3-advanced-exam.html` | 20–28 | Advanced & Exam |
| `index.html` (Kitchen Ninja HQ hub) | – | Links to every app, shows progress, holds sync + backup code |

- File names are hard-coded in the hub (`index.html`), in each part's `FILE_MAP`, and in the part-tab buttons. **Do not rename them** without editing all three places.
- Each part is fully self-contained (CSS + JS + data in one file). Parts share progress through one localStorage key, so they only work "as one app" when served from the same origin (e.g. the Vercel site).
- Parts 1 and 2 were rebuilt from browser-saved copies (saved-page comment and hidden form-detection tag removed; JS syntax-checked). Part 3 was intact.

## 2. The 28 levels

1 Don't Panic! · 2 Positive & Negative Numbers · 3 Order of Operations · 4 One-Step Equations · 5 Two-Step Equations · 6 Fractions · 7 Decimals · 8 Percentages · 9 Ratio
10 Geometry · 11 Area & Perimeter · 12 Angles · 13 Graphs · 14 Probability · 15 Word Problems · 16 Algebra Patterns · 17 Sequences · 18 Powers, Roots & Indices · 19 Algebraic Expressions
20 Circles · 21 Volume & Surface Area · 22 Averages & Range · 23 Pythagoras' Theorem · 24 Transformations · 25 Speed, Distance & Time · 26 Mixed Challenge · 27 Exam Survival · 28 Master Chef Showdown

Each level = **10 missions** (last one is a "boss" / Chef Challenge). Total 280 missions: 212 `calc`, 28 `boss`, 22 `solve`, 8 `step`, 5 `identify`, and a few `check`/`circle`/`classify`/`explain` (Level 1).

## 3. How the code works (for future edits)

**Level data** — a `LEVELS` object per file, only for that file's range:

```js
5: { title: 'Two-Step Equations', color: 2, emoji: '🍳',
     missions: [
       { type: 'calc', instruction: '...', problem: '2x + 3 = 11', answer: '4', hint: '...' + someSVG() },
       ...
       { type: 'boss', ..., solution: [ {action:'÷2', detail:'...', check:'9'}, ... ] }
     ] }
```

- `answer` can list alternatives with `|` (e.g. `'subtract 8|-8'`).
- Answer checking: lowercase, spaces removed, `−` → `-`; numbers compare numerically (`5.0` = `5`, `£`, `,`, `%` tolerated). `identify`/`classify` accept comma lists in any order. `step`/`explain` are lenient (partial match).
- Hints are HTML strings and often append a visual from the SVG helper functions (`numberLineSVG`, `pizzaSVG`, `ratioBarSVG`, `percentBarSVG`, `pieChartSVG`, triangle/angle/shape/circle/cylinder/coordinate/transformation/speed helpers, `balanceScaleSVG`, `priceChainSVG`, etc.).
- Built-in helper tools: column-working grid (`openWorkspace`), Fraction Helper, BIDMAS Help with practice.

**Navigation between parts** — `FILE_MIN/FILE_MAX`, `FILE_MAP`, `fileForLevel(n)`. Jumping across files uses `?level=N`. `TOTAL_LEVELS = 28` is hard-coded, and `currentUnlockedLevel()` loops to it.

**Progress state** — localStorage key `math-complete-state`:

```js
{ xp: 0,
  levelProgress: { "5": 10 },                 // missions reached (10 = level done)
  levelMissionResults: { "5": ["correct","wrong",null,...] } }
```

- **Important:** a wrong answer still advances progress (and gives 5 XP). So a level can be "finished" with mistakes hiding inside. `levelMissionResults` is the only record of which ones were wrong.
- All levels are unlocked (`isLevelUnlocked` returns `true`; the sequential lock is commented out).

**XP & ranks** — correct = 15 XP, boss correct = 30, wrong = 5. Ranks: Apprentice 0 · Prep Cook Ninja 300 · Chopping Champion 700 · Sizzle Sensei 1200 · Sauce Samurai 1800 · Flambé Fighter 2500 · Wok Warrior 3300 · Kitchen Guardian 4200 · Head Chef 5200 · Master Chef Ninja 6300. One clean first pass of all 280 missions is about 4,620 XP, so the top two ranks need extra content — a natural hook for the next seasons.

**Cloud sync** — KVdb.io, bucket = shared Family Code stored in `kn-sync-code` (legacy `math-family-code`), data under `/math`. `mergeStates()` takes max XP, max progress, and merges results (correct beats wrong beats null). Sync UI lives only on the HQ hub. **Only `xp`, `levelProgress`, `levelMissionResults` are synced** — any new state field must be added to `mergeStates()` and `pushCloudState()` or it will not sync.

**Hub coupling** — `index.html` reads `math-complete-state` for the XP preview and hard-codes part ranges `1–9 / 10–19 / 20–28` for the per-part done-counts. The hub's backup/restore bundle lists each app's localStorage key explicitly.

## 4. Design and content rules (do not break)

- Cooking/chemistry framing everywhere: word problems, UI labels, praise, hints. Kitchen experiments = math experiments.
- **Singapore method first:** bar models, number bonds, "draw it before you solve it". The real exam has no pictures, so kid draws models himself.
- Language: short sentences, one idea per line, concrete before abstract, 12-year-old vocabulary. Explain terms with English logic.
- ADHD-friendly: small chunks, visible progress, instant feedback, XP/ranks, no long walls of text. Sessions should be finishable in about 5–10 minutes.
- Visual thinker: every idea gets a picture. Exam anxiety: he can freeze and write nothing, so always train "write *something* first" (method marks).
- Honest simplification: where something is simplified, tell him to check with his teacher.
- Dark warm palette: Nunito + Bangers, browns/golds, `#e8482c` accent. Each app has a 🏠 back-to-HQ link.
- Workflow: phased delivery, confirm each phase, syntax-check + run in jsdom/Node before handing over.

---

## 5. Roadmap: keeping the app alive through Year 9, Year 10, GCSE and competitions

### The problem
He loves *new* and hates re-doing *finished*. Classic "revise the old levels" will fail. The app should never ask him to redo; it should keep giving him something new that quietly uses the old.

### Golden rule
**Finished stays finished. New always arrives. Old skills go undercover inside new dishes.**

### Pillar A — Seasons (each year is a new "menu")
Keep Levels 1–28 frozen as **Season 1: Year 8 Foundations** (a complete trophy). Add new seasons with **reserved level-number ranges** so the existing state format and shared XP just keep working (no migration):

| Season | Level numbers | Files (suggested names) |
|---|---|---|
| 1 Year 8 (done) | 1–28 | `math-part1/2/3-*.html` |
| 2 Year 9 | 101–1xx | `math-y9-part1-*.html`, … |
| 3 Year 10 | 201–2xx | `math-y10-part1-*.html`, … |
| 4 GCSE / IGCSE | 301–3xx | `math-gcse-*.html` |
| 5 Competition | 401–4xx | `math-comp-*.html` |

Keep each file about 9–10 levels (matches today's ~130 KB files). On the hub, show the **next season as a visible locked door** ("Year 9 Menu — opens when Master Chef Showdown is cleared"): a cliffhanger he can see. Each season adds new ranks above Master Chef Ninja (e.g. Michelin-style ranks).

### Pillar B — Undercover review (interleaving)
- Every new level tags missions with `uses: [oldLevelIds]`. About 30% of each new level's missions need an old skill as one step (e.g. Year 9 "reverse percentages" needs Season 1 ratio + equations).
- Label them as **"Prep Station"** ingredient steps inside the new dish, never as "review". Kid perception: new dish; reality: old skill practised.
- Kid-facing example: *"Today's dish: Reverse Percentage Soup 🍲. Prep step: you already know how to find 20% (Level 8). Grab it!"*

### Pillar C — Fresh numbers every time (question generators)
- Turn `calc` missions (212 of 280) into templates: `gen: () => { const a = rnd(3,9); ... return {problem, answer, hint} }`.
- Same skill, new numbers each visit, so it never feels like "the same old question". Highest priority: percentages, ratio, equations, fractions, Pythagoras, area/volume.
- Word problems get a bank of cooking/chemistry story shells with swappable ingredients, quantities and units.

### Pillar D — Stars on finished dishes (additive mastery)
- Completed level = permanent ✅ **Cooked**.
- Extra layers add on top and never remove: ⭐ → ⭐⭐ → ⭐⭐⭐ (Bronze/Silver/Gold), earned by a **Mystery Box** (5 questions, fresh numbers, about 5 minutes, new challenge framing).
- Stars give bigger XP than the first pass, and ranks above Head Chef need stars. He gets *more* reward for revisiting, without ever "redoing".

### Pillar E — Fix-it Recipes (use the wrong answers already recorded)
- Read `levelMissionResults`: any `'wrong'` becomes a "Fix the Recipe 🔧" mini-quest with fresh numbers of that exact mission type.
- Because wrong answers currently count as "done", this quietly closes the hidden gaps and feels like a new targeted quest, not a repeat.

### Pillar F — The Fridge and the Daily Special (spaced return, ADHD-sized)
- Store `lastSeen` per topic. Hub shows a **Fridge** ("🥬 Fractions — 14 days old, use it before it spoils!").
- **Daily Special** = one button, 5 questions, 5–7 minutes: 3 from the current season, 2 from the Fridge, one visible progress bar, then done. A finished-for-today stamp gives the "finished" feeling.

### Pillar G — Exam technique track (GCSE and school exams)
- Method marks: "Show working" mode where partial steps earn points (trains him to write something instead of freezing). Extend the Panic Button and sentence starters idea from Math Exam Lab.
- Command words (*Show that, Hence, Estimate, Explain, Justify*), calculator vs non-calculator sections, marks per question, a timed "Dinner Service" paper mode, and a "commonly lost marks" list (units, rounding, sign errors).
- Bar model to algebra bridge: draw the bar, write the equation, solve; keeps Singapore method alive into GCSE.

### Pillar H — Competition track ("Iron Chef Arena")
- Puzzle-style problems: number theory basics (primes, HCF/LCM, divisibility), counting, logic, geometry puzzles, algebra tricks, sequences.
- Teach Singapore-style **heuristics as "Chef's Tricks"**: draw a model, guess and check, work backwards, make a list or table, look for a pattern, try a simpler case.
- Candidate targets: UKMT Junior/Intermediate Challenges, Kangaroo, AMC 8/10, Singapore/Thai olympiad-style papers (confirm which he actually wants).

### Topic map (draft; align with actual exam board)
- **Year 9:** direct/inverse proportion, reverse and compound percentages, expanding/factorising, linear graphs (y = mx + c), simultaneous equations (intro), standard form, index laws, Pythagoras applications, polygon and parallel-line reasoning, circle arc/sector, composite shapes, tree diagrams and Venn diagrams, averages from tables, scatter graphs.
- **Year 10:** quadratics (factorise, formula, complete the square), simultaneous equations, trigonometry (SOHCAHTOA), similarity/congruence, circle theorems, vectors (intro), surds, algebraic fractions, inequalities on graphs, functions, histograms, cumulative frequency, conditional probability.
- **GCSE year:** sine/cosine rule, ½ab sin C, 3D trigonometry, proof, graph transformations, gradients and rates, full mixed papers.
- **Chemistry crossover (his hook):** concentration, density, percentage yield, scaling recipes, rate-of-reaction graphs, standard form (big/small numbers), unit conversions, compound measures.
- **Exam board: Cambridge IGCSE Mathematics (0580)** (confirmed by J.J.). See `IGCSE_Syllabus_Map.md` for the topic-by-topic map, the exam-technique rules, and the season plan tagged with Cambridge syllabus codes. Still to confirm with the school: Core vs Extended tier, and the syllabus version for his exam year (the 2025–2027 syllabus will not be his).

### Build order (phased)
1. **Done:** clean Parts 1–2, this guide.
2. **Add-on pages that don't touch the finished files:** `math-daily-special.html` (Mystery Box + Fridge + Fix-it) reading `math-complete-state` read-only and saving new stars/lastSeen to a **new key** (e.g. `mathDailySpecialState_v1`). Add the new key to the hub's backup bundle and to cloud sync.
3. **Season 2 (Year 9), 1st file**, with `uses` tags and generators, and new ranks; hub tile and locked-door for the next season; update hub part-count ranges.
4. Season 2 remaining files; then Season 3, GCSE exam-technique, Competition arena.
5. Optional: a small build script that stitches shared engine + level data into single HTML files, so edits stop needing 130 KB hand edits.

### Watch-outs
- `TOTAL_LEVELS = 28` and hub ranges `1–9/10–19/20–28` are hard-coded and need updating when seasons are added.
- Any new state key must be added to cloud sync (`mergeStates` / `pushCloudState`), the hub progress preview, and the hub backup/restore bundle.
- Keep the same-origin hosting (all files on one site) or progress will not be shared between parts.
- Keep the active-exam badge convention (`active:true` flag) for exam-tagged apps; the math app could publish a season badge the same way.

---

## 6. Status log — Season 2 (Year 9) built

| File | Levels | Content |
|---|---|---|
| `math-y9-part-a-number-kitchen.html` | 101–110 | Primes, HCF/LCM, standard form, index laws, estimating, limits of accuracy, ratio, rates & density, percentages II, time/money |
| `math-y9-part-b-algebra-kitchen.html` | 111–120 | Expand/factorise, equations, inequalities, change subject, simultaneous, nth term, straight-line graphs, real-life graphs, sets/Venn, quadratic graphs |
| `math-y9-part-c-shape-data-kitchen.html` | 121–130 | Angle reasons, bearings, nets/Euler, compound shapes, arcs & sectors, volume, Pythagoras problems, similarity, charts/scatter, combined probability |

**How Season 2 files differ from Season 1**
- Every mission is a generator (`LV(title, emoji, color, code, [() => mission, ...])`), so numbers are fresh on every page load. Level progress is still keyed by mission index, so progress and sync are unchanged.
- `code` = IGCSE syllabus code per level; `prep:` on a mission shows the green "🥕 Prep Station" chip (an old Season 1 skill used inside a new dish).
- Algebra answers use `alt()` helpers to accept every ordering (`3x+12` / `12+3x`) and `²`/`^2`.
- Ranks extended in these files only: Michelin Star Ninja 7500 XP, Kitchen Legend 9000 XP.
- `TOTAL_LEVELS = 130` in all three Season 2 files; each file's `FILE_MAP` lists Season 1 parts + Y9 A/B/C. Season 1 files do not know about Season 2 (the hub links to it).
- Hub (`index.html`): Season 2 row with Part A/B/C chips and done-counts (101–110 / 111–120 / 121–130). The station glow layer (`.station::before`) must keep `pointer-events: none` or chips become untappable on iPad.

**Next:** Season 3 (Year 10, levels 201+), the Daily Special / Mystery Box page, Fix-it Recipes. When Season 3 starts, update `TOTAL_LEVELS`, each `FILE_MAP`, and the hub ranges.
