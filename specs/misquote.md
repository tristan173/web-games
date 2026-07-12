# Misquote — spec (working title)

Two famous quotes with their words tangled together. Swap words until both read true. Waffle's soul, one level up.

Shared infrastructure: see `daily-kit.md`.

## Core loop

1. Two lines of word-tiles, mono uppercase:
   `ACTIONS SPEAK MIGHTIER THAN WORDS`
   `THE PEN IS LOUDER THAN THE SWORD`
2. Tap two words (any line, any position) to swap them. Words displaced anywhere — within a line or across lines.
3. Words in their correct home position render in that line's accent with a solid underline; all others neutral. Live feedback, Waffle-style.
4. **Swap budget = par + 5**, where par is the true minimum number of swaps (precomputed, see validator). Stars = unused swaps, exactly Waffle's economy. Budget exhausted → fail state, solution plays out, streak broken.
5. Solve card: both quotes clean, attributions, a one-line why-this-pairing note, stars, share.

Win = both quotes restored within budget. Share = ★ count + swaps-used-over-par bar, no words leaked.

## The teeth

- Difficulty dials: quote length (total 10–24 words), number of displaced words (60–90% of mobile words), and **cross-plausibility** — pairs are chosen so displaced words are grammatical in their wrong homes ("The pen is louder than the sword" reads fine; that's the trap).
- Weekly ramp: Mon short proverbs, few displacements → Sat two long quotes, near-full derangement, high cross-plausibility; occasional Sat variant tangles **three** quotes.
- Duplicate words (THE, THAN…) are the honest difficulty: multiple correct targets exist, and a careless solver wastes swaps shuffling interchangeable words. Engine treats any placement of interchangeable duplicates as correct (see validator) — the *player's* challenge is realizing which words are interchangeable.
- She may not know a quote — deliberately include one less-famous line most weeks; the grammar-and-fit deduction must carry a solver who's never heard it. (This is what separates it from trivia.)

## Puzzle format

```json
{
  "n": 137,
  "lines": [
    { "words": ["ACTIONS","SPEAK","LOUDER","THAN","WORDS"], "by": "proverb" },
    { "words": ["THE","PEN","IS","MIGHTIER","THAN","THE","SWORD"], "by": "Edward Bulwer-Lytton" }
  ],
  "scramble": [ 2, 8 ],
  "note": "One says shout, the other says write."
}
```

- `lines[].words` = the true quotes; `scramble` = the permutation applied at load (stored as swap pairs or a permutation array — generator's choice, validator-approved). Deriving the start state from truth + permutation means the solved state can't desync.
- Obfuscation per `daily-kit.md`.

## Generator + validator (authoring-side)

- Author supplies the quote pair (+ attribution + note). Generator selects the mobile word set and a permutation with: no displaced word landing on an identical word (auto-merge duplicates first), grammatical cross-plausibility preserved (authored judgment, flagged not computed), and target par hit.
- **Par computation is exact and duplicate-aware:** with duplicate words, minimum swaps = minimize (n − cycles) over all assignments of tiles to identical-word slots — brute-force the assignment for small duplicate groups (they're tiny: THE ×3 at worst). Par ships as data; the in-game budget derives from it.
- Validator confirms: permutation restores exactly; par correct by independent recompute; no unintended *alternate* fully-underlined state (any arrangement the engine would score as solved must be word-for-word equal to truth up to interchangeable duplicates).
- Copyright posture: lean proverbs, idioms, public-domain literature, historical speech; short attributed lines otherwise. No song lyrics, ever.

## UI specifics

- Each line owns one accent (cold line / warm line); correct-position = that line's accent + underline; selected word = raised tile; swap = crossing slide animation (reduced-motion: instant).
- Swap pips deplete along the bottom; par shown as a notch on the pip bar.
- Long quotes wrap; tiles stay ≥44px tall; test 24 words at 390px early.
- Synth: soft tick per swap, low buzz on budget last-3, two-chord resolve per completed line, small fanfare on both.

## Content pipeline

- 365 pairs authored by Claude, curated by us for: pairing wit (the mashups should be *funny* mid-solve), cross-plausibility, par range.
- The `note` line on the solve card is the personality; write it dry, never explain the joke.
- Gift layer per `daily-kit.md`: author mode takes two quotes → generates scramble at chosen difficulty → link. This is the anniversary machine — first-dance lyric is off-limits (lyrics rule), but vows, inside jokes, and toasts are exactly the material.
