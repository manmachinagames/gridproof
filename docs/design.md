# Gridproof: Daily Logic Puzzles — design

Date: 2026-11-12. Status: agreed design, ready to build. Part of the Proof series; the series plan lives in `manmachinagames/proof_core`, `docs/proof_series_structure.md`.

Built in Godot 4.7 (GDScript) on `proof_core` and the series conventions (the other games' `CLAUDE.md`). Fully playable offline. 12 languages, controller support, Steam Deck friendly.

---

## 1. Pitch

Six classic logic puzzles where every puzzle is fair: exactly one solution, reachable by logic alone, never by guessing.

- **Proven fair.** Every puzzle is checked twice before it ships: an exact solver proves the solution is unique, and a human-style solver proves it can be solved step by step with named techniques, no trial and error.
- **Hints that explain.** A hint shows the next deduction and why it holds, in words, with the cells that prove it highlighted.
- **Daily three.** Three puzzles a day, three different types at rising difficulty, the same for everyone, one shareable result line.
- **Learn mode.** Short interactive lessons for every puzzle type and technique, built from the same solver.
- **Respects the player**: no ads, no hint purchases, no energy. Coins are earned by playing and spent on looks.

Store subtitle: "Daily Logic Puzzles".

---

## 2. Puzzle types at launch

| Type | Rules (short) | Sizes at launch |
|---|---|---|
| Sudoku | Fill 1–9 so every row, column and 3×3 box holds each digit once | 9×9; 6×6 for easy and Learn |
| Nonograms | Shade cells so each row's and column's runs match its clues | 5×5 to 25×25 |
| Kakuro | Fill 1–9 so each run sums to its clue with no digit repeated in a run | 6×6 to 14×14 |
| Slitherlink | Draw one closed loop along grid edges; each number counts the loop edges around its cell | 5×5 to 15×15 |
| Hashi | Connect islands with straight bridges (at most two between a pair, no crossings); each island's number is its bridge count; all islands connected | 7×7 to 15×15 |
| Numberlink | Join each pair of matching numbers with a path; paths never cross and together fill every cell | 5×5 to 12×12 |

Sizes are tuned by playtesting, not fixed up front. Not trademarked names: "Sudoku" and "Kakuro" are generic outside Japan; avoid "Picross", "Flow" and Nikoli's protected brand names.

Gated content drops (section 10) add more types later; Nurikabe is drop 1.

---

## 3. The fairness guarantee

Every puzzle a player sees passes both checks:

1. **Unique.** An exact solver (backtracking with constraint propagation, independent of the technique solver) counts solutions up to two and finds exactly one.
2. **Logic-only.** The technique solver solves it from the start using only named techniques from the type's technique list, never trial and error. Every step it takes is a deduction it can explain.

**Techniques** are defined per type as an ordered list, each with an id, a tier (1–4) and an explanation template (a translation key with placeholders). Examples: Sudoku naked single (tier 1) to X-wing (tier 4); Nonograms overlap and edge logic; Slitherlink corner and loop-closing rules; Hashi bridge counting and isolation rules.

**Difficulty** = the highest technique tier needed, adjusted by how many hard steps the solution needs. Four bands per type: **Easy, Medium, Hard, Expert**, thresholds calibrated per type from generated puzzles.

---

## 4. Play

### 4.1 Common to every type

- Undo and redo are unlimited; restart any time.
- **Hint**: the technique solver finds the next deduction from the player's current state, highlights the cells involved and shows its explanation. If the player has made a mistake, the hint says so first and points at it.
- **Check**: optional; marks wrong entries.
- A **clean solve** = no hint, no check, no undo. Hints and checks make a game assisted; assisted games count for nothing (stats, records, quests, achievements, coins). Undo alone only removes the clean flag. Same fair-play rules as the other Proof games.
- Notes: types with numbers (Sudoku, Kakuro) have pencil marks, with optional auto-candidates (assisted only if the player turns on "auto-remove candidates"; showing candidates the player entered is never assisted).

### 4.2 Input per type

- **Sudoku, Kakuro**: select a cell, type or pick a number; a notes toggle; keyboard digits; controller with a radial number picker.
- **Nonograms**: click to fill, right click to mark empty; drag fills or marks a straight run; satisfied clues dim.
- **Slitherlink**: click or drag along edges to draw line; right click marks an edge as no line.
- **Hashi**: drag from island to island to add a bridge (a second drag makes it double, a third removes); islands show how many bridges they still need.
- **Numberlink**: drag from a number to draw a path; dragging over a path cuts it back.

Every type is fully playable by mouse, keyboard and controller. Drawing input must feel precise: no lost drags at small cell sizes.

---

## 5. Learn mode

Per type: a rules lesson, then one lesson per technique in tier order. Each lesson is a small generated position where that technique is the next deduction, with the explanation shown step by step, then two or three practice positions where the player finds it. Lessons track mastery; the type's free play suggests the band that fits the techniques the player has learned. Learn positions come from the same technique solver, so they are always correct.

---

## 6. Content

Unique, logic-only puzzles can be slow to generate, especially large ones. Content is therefore **generated offline, verified, and shipped as data**:

- **Daily pool**: three puzzles per day for at least ten years ahead.
- **Free-play pools**: per type, size and band, large enough that a player rarely repeats (target: thousands per type).
- A dev tool generates the pools with seeds, runs both checks on every puzzle, records band and technique profile, and writes compact data files. CI verifies a random sample of each pool on every push; the tool can verify every puzzle (slow).
- **On-the-fly generation** only for type and size combinations where generation plus both checks reliably finishes in under 300 ms on a mid-range PC; otherwise play draws unplayed puzzles from the pools.
- Pool data is append-only once released: never reorder or remove, so dailies and puzzle codes stay stable.

---

## 7. Modes

- **Daily three**: three puzzles of three different types, Easy → Medium → Hard. The type rotation follows a fixed schedule from the daily seed salt (never change it after release) so every type comes around at least every two days. One shareable result line covering all three; each puzzle counts separately for stats.
- **Free play**: pick a type, size and band.
- **Packs**: per type, curated sequences with stars and steady progression.
- **Learn**: section 5.
- **Gauntlet**: a mixed-type run of rising difficulty against one timer.
- **Puzzle codes**: share any pool or generated puzzle as a short code.

---

## 8. Stats and records

Buckets: type × size × band. Per bucket: solves, clean solves, best time, average time. Per type: techniques mastered in Learn. Daily history, current and best daily streak (shown, never used to pressure: no streak-loss warnings or penalties). Lifetime totals, daily quests and achievements as in the other Proof games (100 Steam achievements maximum, gated content included).

---

## 9. Looks, shop and coins

Same system as the other Proof games, through `proof_core`: palettes, backdrops and effects, presets, the Appearance dialog as the shop, Reduce motion.

New parts: **grid styles** (lines, paper, chalk, ink…), **number fonts** and **fill styles** for nonograms and paths. A good free set, including a high-contrast style and a colour-blind-safe path palette for Numberlink.

Coins: each clean or counted solve pays by type size and band; counted dailies pay double; quests and achievements pay as in the other games. Replaying an already-solved pool or pack puzzle pays nothing. Assisted games pay nothing.

---

## 10. Content updates (gated features)

Same content-drop system as Tileproof (`docs/design.md`, section 12) and Slideproof: built to release quality, shipped locked behind `CONTENT_DROP`, unlocked by updates, always tested in CI.

Planned drops, each a new puzzle type (it joins the daily rotation when unlocked):

1. Nurikabe
2. Net (rotate tiles to connect a network)
3. Code-breaking (Mastermind-style deduction; no text, ideal for the daily)
4. Battleship solitaire
5. Lights Out variants
6. Killer Sudoku, colour nonograms

---

## 11. Architecture

- `proof_core` as a submodule at `addons/proof_core`, pinned to a release tag, from day one. CI checks it out separately with the `PROOF_CORE_TOKEN` secret (Mineproof's workflow is the reference). Game shell (app, ui, meta) starts from the newest Proof game.
- **Every puzzle type is a module** in `core/puzzles/<type>/` behind one interface (`PuzzleType`): grid model and rules check, exact solver, technique solver with its technique list, generator, rating, compact codec. Its UI lives in `ui/puzzles/<type>/` (board view and input). Adding a type = its two folders plus one registry line; removing one = deleting them. Shared code across types (grid, technique step, hint presentation, generic exact search helpers) lives in `core/puzzles/common/`.
- Content pools in `content/<type>/`; the pool tool in `tools/`.
- Determinism as in the other games: all randomness through `SeededRng`; each type's generator has a `VERSION` that goes into codes.

---

## 12. Milestones

Each milestone ends with: all tests passing, CI green on push, screenshots of every changed screen checked, and a review by a separate reviewer agent against this document. A failed review is fixed and re-reviewed before the next milestone starts. Checkpoints (★) pause for Iiro to play and give feedback.

The acceptance bar for **every puzzle type** (M1–M6, M10): the exact solver finds exactly one solution for every puzzle in a sample of 1,000 per band; the technique solver solves all of them without trial and error; each band holds a reasonable share of generated puzzles; a seeded generator gives identical output across runs; the type is playable by mouse, keyboard and controller with hints that explain; screenshots at 1280×720 and the 760×540 minimum.

**M0 — Scaffold.** proof_core submodule, shell from the newest Proof game with its game rules stripped, the `PuzzleType` interface and registry, CI. `CLAUDE.md` written.
Accept: boots with no errors; CI green; no other game's code or text left.

**M1 — Sudoku, end to end ★.** Rules, both solvers, techniques with explanations, generator, rating, pool tool, board view with notes, hints and check, free play.
Accept: the per-type bar.
Checkpoint A: Iiro plays Sudoku, judging hint explanations and input feel.

**M2 — Nonograms.** **M3 — Kakuro.** **M4 — Slitherlink.** **M5 — Hashi.** **M6 — Numberlink.**
Accept: the per-type bar for each.

**M7 — Content and Daily three ★.** Full pools for every type and band, the ten-year daily pool, the daily schedule, the Daily three screen and share line.
Accept: pool sample verification in CI; full verification run recorded; daily schedule deterministic and covering every type at least every two days; pools append-only (a test pins existing entries).
Checkpoint B: Iiro plays dailies across a week of dates.

**M8 — Learn mode.** Lessons for every type and technique.
Accept: every lesson's positions verified (the taught technique is the next deduction); lessons complete by mouse, keyboard and controller.

**M9 — Meta.** Stats and records, clean solves, quests, achievements, replays, coins and the shop with the new parts, packs, Gauntlet, puzzle codes, all on `proof_core` (extend core where Gridproof is the second user of something).
Accept: tests for stats buckets, coin rules (no replay farming) and purchases; codes round-trip.

**M10 — Gated drop 1: Nurikabe.** Built to the per-type bar, locked behind `CONTENT_DROP`.
Accept: the per-type bar; invisible in a launch build; visible with the gate raised; CI tests it either way.

**M11 — Art and polish ★.** Final grid, font and fill styles, backdrops and effects tuned for this game, feel pass on every input, all text in 12 languages (technique explanations included), Steam Deck sizing, achievements list, store screenshots.
Accept: every screen screenshot-checked in English and at least Finnish and Japanese; minimum window size works; Reduce motion covers every animation.
Checkpoint C: Iiro plays the release candidate.

---

## 13. Out of scope for the first release

Steam Workshop, online leaderboards, multiplayer, music, mobile builds, user-made puzzle submission.
