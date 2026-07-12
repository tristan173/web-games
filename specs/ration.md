# Ration — spec (working title)

Clued answers fight over one exact bank of letter tiles. Plausible wrong answers bankrupt the letters another clue needs.

Audience calibration: primary player is a tenacious PhD-grade solver. Default difficulty sits around a NYT Thursday crossword; the week ramps to genuinely hard. Err toothy — she should sometimes carry it around all morning.

Shared infrastructure: see `daily-kit.md`.

## Core loop

1. The day's puzzle shows K clues (4–6), each with its answer length, e.g. `CASUAL CONVERSATION (4)`.
2. Below: one shared bank of letter tiles = the exact multiset of all answers combined (~20–30 tiles).
3. Player fills any slot in any order by tapping tiles or typing (typing consumes a matching bank tile; delete returns it). Edits are free and unlimited.
4. SUBMIT unlocks only when every slot is full — which, by construction, means the bank is empty. **No feedback of any kind before a full submission.** This is the spine of the game: you cannot spray-and-pray one answer at a time; you commit a complete world.
5. On submit: correct answers lock (accent + underline + soft blip); wrong answers shake and clear, returning their tiles.
6. Budget: **4 submissions** (settings can tighten to 3). Out of submissions → puzzle over, solution shown, streak broken.

Win = all answers locked within budget. Share grid = one row of K squares per submission (locked-green vs dark), Wordle-style.

## The teeth

Every puzzle is built around **decoys**: wrong answers that fit the clue and the visible tiles at the moment you'd naturally commit them.

Two engineered types (validator classifies, see below):

- **Type E (early-block)** — taking the decoy makes some other clue's intended answer immediately unspellable. Detectable *before* submitting by a player doing global lookahead. Rewards deduction.
- **Type L (late-block)** — every other intended answer remains individually spellable; the bankruptcy only surfaces when the last slot won't fill. Punishes greed, forces whole-board thinking. The cruelest and best.

Difficulty dials: number of clues, answer lengths, decoy count and type mix, clue ambiguity.

Weekly ramp:

| Day | Clues | Decoys (min) | Notes |
|---|---|---|---|
| Mon–Tue | 4 | 2, Type E | warm-up, ~5–10 min |
| Wed–Thu | 5 | 3, mixed | baseline teeth |
| Fri | 5–6 | 4, ≥1 Type L | hard |
| Sat | 6 | 5, ≥2 Type L chained | the one she brags about |
| Sun | 5 | 3, mixed | recovery + playful clue voice |

**Iron Ration** (settings toggle, suggested on Sat): submissions report only *how many* answers are wrong, not which. Mastermind-grade. 6 submissions in this mode.

Clue voice: crossword-tight with engineered ambiguity — every decoy is a *fair* reading of the clue. `Casual talk (4)` wants CHAT and BLAB to both occur to you; the bank's single B settles it. No trivia-wall clues; the difficulty lives in the letter economy, not obscurity.

Hints: opt-in only, never pushed. Each hint reveals the first letter of one chosen answer; max 2; each costs a star on the solve card and is marked in the share (💡). Tenacious solvers must be able to ignore hints entirely without UI nagging.

## Puzzle format

```json
{
  "n": 137,
  "clues": [
    { "c": "Feline, familiarly", "len": 3, "a": "TOM" },
    { "c": "Casual conversation", "len": 4, "a": "CHAT" }
  ]
}
```

- `a` fields base64-obfuscated in the shipped file.
- The bank is **derived at load** from the answers — never stored separately, so it can't desync.
- Bank displayed sorted A–Z, duplicate letters as repeated tiles. A SHUFFLE button re-scatters the bank (pure fidget tool, Bee players love it; no state effect).

## Validator / solver (authoring-side Node script, not shipped)

Input per puzzle: clues, intended answers, and a **candidate list** per clue — every plausible alternate answer of the right length the author (or Claude) can produce, decoys included. Uniqueness is only provable relative to candidate lists, so the tooling has two layers:

**Check 1 — uniqueness (exact-cover).** Backtracking over one-candidate-per-clue selections; the only selection whose combined multiset equals the bank must be the intended answer set. Exhaustive, no early exit on first solution (we need to *count*).

**Check 2 — blind-spot sweep.** Enumerate *all* partitions of the bank multiset into words of the required lengths using a real dictionary (length-indexed, memoized DFS over sorted letter-multiset keys; ENABLE-class list with a frequency floor). Any partition other than the intended one is reported with its words mapped to slots. The author triages each: if a word is a fair answer to its clue, add it to candidates and re-run Check 1 (usually → reword the clue); if not, ignore. This catches the alternates nobody thought of.

**Check 3 — trap report.** For each candidate decoy `d` on clue `i`: classify Type E (some other intended answer unspellable from bank − d) vs Type L (all individually spellable but no full completion). Emits per-puzzle teeth metrics; batch mode emits the week table so we can see the ramp at a glance. (A decoy that permits a full alternate completion would already have failed Check 1.)

Batch mode runs the full year file and fails loudly. No puzzle ships unvalidated.

## UI specifics

- Layout: clues stacked top (numbered, mono); each clue's empty tile-outline slots directly under it; bank grid bottom, thumb zone; SUBMIT bar above bank with submission pips.
- Cold accent = bank tiles; warm accent = placed tiles; locked answers = warm fill + solid underline (color-independent signal).
- Wrong-answer clear on submit: brief shake, tiles fly back to bank (skip under reduced-motion).
- Solve card: answers, decoy post-mortem ("BLAB was lying in wait — one B"), stars (submissions unused, hints unspent), streak, share button. The post-mortem line is the personality of the game; write it dry.

## Content pipeline

- 365 puzzles authored by Claude (Opus with this spec), curated by us: clue voice pass, decoy fairness pass.
- First batch = 30 puzzles + validator output reviewed before the shell gets built (dailies review: play a week's worth ourselves).
- Author mode (gift layer, `daily-kit.md`): custom puzzles run Check 1 only, against author-entered alternates; skip Check 2 (no dictionary in-file).

## Open questions (for design review, not blockers)

1. Submission budget: 4 feels right vs Waffle-style star pressure at 3 — playtest.
2. Should Sat auto-suggest Iron mode or stay opt-in? (Lean opt-in; she'll find it.)
3. Solve-card stars formula: submissions-based only, or time-flavored? (Lean no timer anywhere — timers cheapen tenacity.)
4. K=6 on phone: does the layout still breathe at 390px? Prototype early.
