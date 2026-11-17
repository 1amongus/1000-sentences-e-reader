# 1000 Sentences E-Reader

A single-file, Duolingo-style guided reader for the Samsaadhanii "1000
Sentences" book. It turns the e-reader's word-by-word grammatical analysis —
padaccheda, anvaya, karaka graphs — into a learning path: one sentence per
screen, tap-a-word analysis, meaning checks, and a tile sentence builder,
with XP, hearts, and a friendly Acharya mascot.

- **Live site:** https://1amongus.github.io/1000-sentences-e-reader/
- **Repository:** https://github.com/1amongus/1000-sentences-e-reader
- **Coverage:** 20 chapters (units), 200 lessons, 996 sentences (numbered
  1–999 with gaps), verified live.

## Principles

These rules govern everything in this project. Please keep them if you build
on it.

1. **Stick to the source.** Sentence texts, pada words, morph analyses,
   karaka relations, anvaya order, and Hindi glosses reproduce the
   Samsaadhanii e-reader's analysis verbatim. The app's contribution is
   interface and presentation only: no builder-composed roles, no silent
   "corrections" of the source's tagging, no recomposed anvaya. Where the
   source has no graph, the app draws one from the relation data and says
   so; where analysis is absent, nothing is invented to fill the gap.
2. **Attribute.** The home page is the lessons index plus the Samsaadhanii
   attribution, linking to https://sanskrit.uohyd.ac.in/scl/.
3. **Permission.** Prof. Amba Kulkarni granted full permission to render
   these books (14 November 2026), including rendering other Samsaadhanii
   books as well. At her suggestion, relations containing द्योतक will be
   grayed in the graphs (presentation only, source data unchanged) —
   counted (92 words across 84 sentences) but not yet applied.
4. **Non-commercial, open source.** The author intends this project to stay
   non-commercial and open source. No license file has been added yet — ask
   the author before redistributing.

## What's in this repo

| File | What it is |
|---|---|
| `index.html` | The app. One self-contained file (~4.8 MB). |
| `version.json` | `{"version": 3}`. The app fetches it to detect newer deploys; bump it together with `APP_VERSION` in the page on every release (see Publishing). |
| `README.md` | This file. |
| `samsaadhanii-source-data.md` | The complete retrieved dataset: all 5,718 analysis records with all 25 fields, grouped by sentence, plus provenance, the placeholder-row accounting, and a field glossary. |

## Architecture

### One file, no build, no server

`index.html` is a single self-contained HTML document: two `<style>` blocks,
one `<script>` block, no framework, no bundler, no build step, no backend,
no database. It runs anywhere a static file runs — open it directly in a
browser, serve the folder over HTTP, or publish it with GitHub Pages (which
is how the live site is served).

- **One runtime network call.** `checkForUpdate()` fetches `version.json`
  (cache-busted, `no-store`) on load and every 5 minutes, and shows an
  "Update now" banner when the file reports a version newer than the page's
  `APP_VERSION = 3`. Everything else is local; if the fetch fails (e.g. the
  file opened from disk), the app carries on silently.
- **Fonts are the only other external dependency:** Google Fonts (Nunito,
  Tiro Devanagari Sanskrit) with system fallbacks.
- **Mascot art is embedded.** The Friendly Acharya mascot is stored as
  base64 data URIs (`MASCOT_SRC`, four `<img>` tags) — no separate image
  files to deploy.
- **No persistence.** The page uses no `localStorage` or `sessionStorage`:
  lesson progress, XP, hearts, and streak live in memory only, so reloading
  resets the session. (Adding persistence is a natural first contribution;
  see below.)
- **Responsive, phone-first.** 12 media queries; one sentence per screen
  with a pinned Continue button. Three views: `view-home` (tile index),
  `view-lesson` (the guided lesson), `view-complete` (chapter-complete
  celebration with confetti).
- **Graphs are SVG drawn at runtime** from the relation data (see below) —
  no fetched graph images.
- **Speech** uses the browser's `speechSynthesis` (`speak()`); there are no
  audio files.

### Data embedded in the page

All sentence data is embedded as JavaScript literals in the `<script>`
block. The main structures:

- `SENTENCES` — 996 sentence records, looked up by number through `BY_N`.
  Each record follows the chapter-JSON shape: `n` (true sentence number),
  `chapter`, `position_in_chapter`, `text`, `analysis_available`, `words`,
  `verb`, `roles`, `graph` (the source's graph path, or `null` for 7
  sentences with no source graph).
- Word records: `{ word, lemma, gloss, morph, karaka, anvaya, hindi,
  implied, karaka_alt }`. `gloss` is the app-side English learner rendering
  (or `null` — 2,972 words have none; nothing was invented). `karaka_alt`
  carries the source's `possible_relations` as alternate relations.
  `implied` marks words the analysis supplies that are not in the sentence
  text, e.g. `(त्वम्)`.
- `UNITS` — 20 chapter entries with app-side titles and descriptions
  (e.g. "Chapter 2 · अहं — 'I'").
- `KARAKAS`, `KRIYA`, `REL_KARAKA` — relation-label maps used by the graph
  builder and the word panel.
- `CH1_NS` — Chapter 1's sentence numbers in order, which is how the
  missing No. 3 is handled: lessons are built from this list, not from
  arithmetic.
- `PRACTICE_POOL`, `PRACTICE_STEPS`, `PRAISE`, `ICON_PATHS`, `MASCOT_SRC`,
  `APP_VERSION` — practice content, praise strings, icon paths, mascot art,
  and the deploy version.

Lessons hold 5 sentences each: `lessonNumbers(idx)` maps a lesson index to
its sentence numbers (unit = `idx / 10`, 10 lessons per unit, 200 lessons
total), skipping numbers that do not exist. `lessonRange` gives the first
and last sentence of a lesson.

### Rendering engine

All rendering is plain functions in the one `<script>` block. The main ones:

| Area | Functions | What they do |
|---|---|---|
| Home / curriculum | `renderCurriculum`, `expandUnit`, `sentTileHTML`, `lessonNumbers`, `lessonRange`, `lessonState` | The tile index: 20 expandable units, 10 lessons each, sentence tiles. Home shows the lessons index and attribution only (a November 2026 simplification removed the hero, counts strip, search, browser block, graph explorer, and data section from home; some of those builders remain in the file). |
| Lesson flow | `startLessonFor`, `startLessonAtSentence`, `startAnyLesson`, `buildStep`, `renderStep`, `nextStep`, `finishLesson`, `exitLesson`, `canCheck` | Start a lesson (or jump to any sentence), walk its steps with the pinned Continue button, and finish with the completion view. Exercises sit one tap away from anywhere: a home-screen card, a floating button, and second place inside every lesson. |
| Step types | `buildStep`, `mcqStep`, `renderBuild`, `showWordPanel`, `padaAnvayaHTML`, `padacchedaText`, `anvayaParts`, `anvayaWords`, `glossText` | The tile sentence builder, the meaning-check multiple choice, tap-a-word analysis panels, and the padaccheda + anvaya panels. |
| Practice | `startPractice`, `PRACTICE_POOL`, `PRACTICE_STEPS` | Mixed review: alternating builder and MCQ steps over a pool of sentences. |
| Graphs | `buildGraphsHTML`, `buildRelGraphsHTML`, `karakaGraphSVG`, `relationGraphSVG`, `relGraphModel`, `relParse`, `relLabel`, `karakaOf`, `isVerbWord`, `dfs`, `dispGroups` | Draw the karaka graph as SVG from the relation data: the verb as the central hub with the karaka roles around it, nodes tappable for word detail. Sentences with no identified verb (78) use a fallback hub rule; dangling karaka head references (sentences 111, 124) are skipped, never crashed on. |
| Navigation | `setActiveNav`, `onMain`, `scrollTopAfterRender` | Sidebar navigation between Home, Practice, and the lesson views. The Quests and Profile sidebar items are placeholders that show "not built yet" toasts. |
| Update check | `checkForUpdate` | Fetches `version.json` (see above) and reveals the update banner. |
| Speech / feedback | `speak`, `blip` | Pronunciation via `speechSynthesis`; short feedback sounds. |

### Source data vs app-side text

Keep this distinction when editing. It is the heart of the project.

- **Verbatim from Samsaadhanii:** sentence texts, pada words, morph
  analyses, karaka relations and head indices, anvaya numbers, Hindi
  glosses (underscores shown as spaces), graph structure, verb/role data.
- **App-side, not from the source:** the English glosses (short learner
  renderings written for the app in Monier-Williams/Apte style — not copied
  from any dictionary; `null` where no rendering exists), lemmas (derived
  by the `lemma_of` rule), unit titles and descriptions, lesson grouping
  (5 sentences per lesson), XP/hearts/streak/praise, the mascot, quiz
  distractors, and all UI copy.
- **Quirks preserved from the source** (documented in the build notes so a
  future editor does not "fix" them): sentence 614's shifted surface column
  and anvaya jump from 4.1 to 6.1; verbs with null karaka (603, 609, 616);
  sentence 610's implied (सः) whose source surface is just `.`; sentence
  460's analysed नगरस्य against the printed नगरं; the space before the
  danda in 462 and 484; unprinted subject rows kept in the tables.

## Data: how it was retrieved and generated

### Retrieval

The data was downloaded on 11 November 2026 from the public data directory
of the Samsaadhanii e-reader (a plain Apache listing — no login, no API key,
no scraping of rendered pages):

```
https://ereaders.samsaadhanii.in/books/data/books/1000%20Sentences/
```

Three files were retrieved:

| File | Contents |
|---|---|
| `slokas.json` | 997 entries: 996 sentences plus 1 stale duplicate (see below). Entry fields: `chaptno`, `slokano`, `sparts` (one line per sentence; the legacy `spart1`/`spart2` fields are always empty). |
| `analysis.json` | 5,718 word records, 25 fields each (see the field glossary in `samsaadhanii-source-data.md`). |
| `chapters.json` | The chapter list `01`–`20` (plus a stray unpadded `"1"` entry, preserved as retrieved). |

An earlier phase (7–9 November) had built the first 100 sentences from a
snapshot of the rendered site (embedded `SENT`/`ANA`/`GLOSS` variables in
the v1 mockup) plus live-site screenshots, via `build_import.py` (see
`NOTES.md` in the import workspace). The November 11 download superseded
it: the extraction reproduces the snapshot's analysis with zero
differences, and merges with the earlier import with zero differences.

Filtering and quirks a reuser must know:

- **Placeholder rows.** 819 of the 5,718 records are placeholders, not
  words: rows whose `word` is `.` (460) or `-` (359) with all analysis
  fields `-`. The extractor excludes them; the dump file includes them
  verbatim so the dataset is complete.
- **Stale duplicate.** Chaptno `1` (unpadded) holds 10 records for slokano
  `024` — a stale duplicate of sentence 24 (slokas text with a trailing
  danda, 10 records including a `.` row, 2 differing Hindi /
  possible_relations values). The padded `01` version is authoritative;
  the app excludes the duplicate. Net accounting: 5,718 − 819 placeholders
  − 9 stale-duplicate records = 4,890 real words.
- **Gaps.** Nos. 3, 825, and 895 do not exist; there is no No. 1000
  (Chapter 20 ends at 999). Chapter counts: 50 each, except Chapters 1, 17,
  18, and 20 with 49.
- **No merged entries.** Unlike the Gita book, every entry here is exactly
  one sentence.
- **Graph paths are names, not files.** The `graph` field names
  `assets/graphs/<chapter>_<n>_1.svg`, but no static graph files exist at
  any public URL (they return 404): the e-reader draws graphs at runtime
  from the relation data. This app does the same. Seven sentences
  (24, 72, 524, 540, 654, 893, 917) have no source graph at all.
- **`-` means absent.** The source's own marker for a missing value; it is
  preserved everywhere, never replaced with a guess. Five fields
  (`english_meaning`, `samAsa`, `prayoga`, `sarvanAma`,
  `name_classification`) are `-` for every record, and `part1`/`part2` are
  `null` throughout.
- **Sentence 697** is counted as having full analysis, although its verb's
  lemma and morph are `-` in the source.

The complete dump, with every field of every record, is in
`samsaadhanii-source-data.md` in this repo.

### Generation pipeline

1. **Extract** — `extract_chapters.py` reads the three JSONs and writes
   per-chapter files `ch01.json`–`ch20.json` (re-runnable; each file carries
   its source, scope, counts, and notes). Rules: exclude the stale
   duplicate; exclude the 819 placeholder rows (validated: after filtering,
   all 68 snapshot analysis entries reproduce exactly); strip leading /
   trailing whitespace from sentence texts (52 entries carry a leading
   space); keep the source's graph path or `null`; leave `gloss` null where
   the `GLOSS_EN` map has no entry (English glosses are not in the source).
2. **Build into the page** — the chapters were loaded into the single HTML
   file in chunks (per the project's standing rule: split big builds into
   smaller pieces, publish each separately), with the tile-index redesign
   and padaccheda + anvaya panels added along the way. The build scripts
   live outside this repo (in the author's workspace); the extraction step
   above is the reproducible part, and the page's data shapes are
   documented in the Architecture section.
3. **Verify** — chapter totals reconciled (996 sentences, 4,890 words);
   each chunk checked live after publishing.

**Hard-won lesson:** after publishing to GitHub Pages, allow ~15 minutes
for CDN cache lag before concluding a deploy is stale — verify with a
cache-busting `?v=` URL. (An early "broken buttons" report traced to a
page-script crash on load, not to caching; verify in the exact file the
user opens.)

## How to build on top

### Run it locally

No install, no build. Either open `index.html` in a browser, or serve the
folder:

```sh
python3 -m http.server 8000
# then open http://localhost:8000/
```

Serving over HTTP is the better test: the `version.json` update check
fetches relatively, which `file://` pages may block (the failure is caught
and silent, so the app still works from disk).

### Make changes

- **Styles:** the two `<style>` blocks — palette, typography, lesson
  chrome, and the graph styles.
- **UI copy:** search the `<script>` block for the string you want to
  change. Unit titles and descriptions live in `UNITS`; praise strings in
  `PRAISE`; step headings in the step builders.
- **Data fixes:** if you believe a sentence's analysis is wrong, check
  `samsaadhanii-source-data.md` (or `analysis.json` at the source URL)
  first. If the app differs from the source, that is a bug — fix the app.
  If the source itself looks wrong, leave it: the project's rule is to
  reproduce the source. (Two sentences, 421 and 430, are known to pair
  display tokens to the wrong analysis rows — see Known issues.)

### Ideas with pointers

- **Persist progress.** Lesson state, XP, hearts, and streak are plain
  in-memory variables; saving/restoring them with `localStorage` is a
  small, contained change.
- **Build Quests and Profile.** The sidebar items exist and show "not built
  yet" toasts; the views behind them are unbuilt.
- **Fix the known issues** (below): the 421/430 pairing, the home-index
  FULL ANALYSIS behavior, and the lesson-step badge labels.
- **Restore search.** Sentence search existed on the home page before the
  November 12 simplification; `renderBrowser` and the sentence index it
  used remain in the file.
- **New exercise types.** Follow `buildStep` (tile builder) and `mcqStep`
  (meaning check): a step builder returns the step's HTML and wiring, and
  `PRACTICE_STEPS` shows how steps compose into a practice round.
- **Gray the द्योतक relations.** The commitment is made and the count is
  known (92 words across 84 sentences). Implementation: in the graph
  builders, render nodes whose displayed relation contains द्योतक in a
  muted style — presentation only, source data unchanged.
- **Sentence audio.** `speak()` already pronounces sentences via
  `speechSynthesis`; it could be extended with per-sentence controls or
  recorded audio (recordings would be new, separately sourced material —
  none exists in the source data).

### Add another book

Prof. Kulkarni's permission covers rendering other Samsaadhanii books. The
recipe mirrors this book's pipeline:

1. Fetch that book's `slokas.json` / `analysis.json` / `chapters.json`
   from the same `books/data/books/<Book>/` directory pattern.
2. Extract per-chapter files with the same rules (exclude placeholder
   rows, exclude stale duplicates, preserve gaps, keep `gloss` null where
   no rendering exists).
3. Shape the data like `SENTENCES` / `UNITS` and add the home-screen
   sections.
4. Keep the attribution, the stick-to-the-source rule, and the
   verse-text-only treatment of gaps.

### Use the data elsewhere

`samsaadhanii-source-data.md` is the complete, citable dump — reuse it
directly, or re-download the JSONs from the source URL above. Apply the
placeholder-row rule and exclude the stale duplicate (both documented in
the dump), respect the `-` convention, and keep the Samsaadhanii
attribution with any reuse.

### Publish

GitHub Pages serves this repo's root; `index.html` is the entry point and
`version.json` must sit beside it. On every release, bump `version.json`
and `APP_VERSION` in the page together — the update banner compares the
two. The file is ~4.8 MB, so prefer `git` CLI or the web upload for
updates, and avoid editing large files in the web file editor. After a
deploy, verify live with a `?v=<commit>` URL (see the cache-lag note
above).

## Current state and known limitations

- Complete and live: 20 units, 200 lessons, 996 sentences.
- Home page: lessons index and attribution only, by design.
- Progress resets on reload (no persistence yet).
- Quests and Profile sidebar items show "not built yet" toasts.
- Known issues, noted during the build and awaiting a fix: sentences 421
  and 430 pair two display tokens to the wrong analysis rows; home-index
  FULL ANALYSIS rows open the reader only for the "up next" lesson (all
  analyses are reachable through the lesson tiles); lesson-step badges show
  the source book's chapter labels, which differ from the unit numbering.
- द्योतक graying is committed and counted but not yet applied.
- The lesson overlay scrolls internally on phones for the largest graphs;
  the rest of the app has zero page scroll by design.

## Credits

- Sentence analysis: the Samsaadhanii e-reader, Computational Linguistics
  Group, University of Hyderabad — https://sanskrit.uohyd.ac.in/scl/ —
  developed under the guidance of Prof. Amba Kulkarni. E-reader library:
  https://ereaders.samsaadhanii.in/books/ereaders/
- Permission to render these books, including other Samsaadhanii books,
  granted by Prof. Kulkarni on 14 November 2026.
- Built by Sarath Oruganti. Non-commercial, open-source intent; see
  Principles above.
