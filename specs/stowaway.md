# Stowaway — spec (working title)

Five innocent sentences each smuggle a hidden word across word boundaries ("The **map led** everyone home" → MAPLE). Find all five; the five stowaways share a secret theme.

Shared infrastructure: see `daily-kit.md`.

## Core loop

1. Five sentences listed, mono type. Letters are individually addressable; spaces and punctuation are visual only (stowaways ignore them).
2. Player selects a span: tap the first letter, tap the last letter (drag also works on desktop). The candidate span previews as a spelled word above the sentence.
3. Confirm: correct → the stowaway's letters pull together and glow (warm accent), word added to the day's manifest. Wrong → strike (soft buzz). **5 strikes** for the day.
4. All five found → theme prompt: type the theme (free text, matched against an alias list, typo-tolerant). Naming it = **CAPTAIN** rank — the Queen Bee chase. Declining or missing still counts the day as solved (rank: crew).
5. Solve card: sentences with stowaways lit, theme, rank, streak, share.

Win = five found within strikes. Share = 5 span-position squares + rank flag (🚩 crew / 👑 captain equivalent in mono style), theme never leaked.

## The teeth

- Every stowaway **must span at least one word boundary** (never fully inside one word — that's a word search). Fri/Sat: spans cross **two** boundaries ("poli**CE DAR**ted" is one; "wa**S NOW** here" hides SNOW across two words — aim three-word spans late-week).
- Decoy engineering: sentences also contain *near-misses* — 3–4 letter real words across boundaries that are NOT the stowaway ("are adept" hides AREA and READ). Strikes make guess-spamming expensive; the theme is the disambiguator, which makes the first find the hardest and the last a sprint — that accelerating-clarity curve is the game.
- Theme obfuscation dial: Mon theme = concrete category (TREES); Sat theme = lateral ("things that fall" — RAIN, NIGHT, PRICES…). The theme prompt accepts reasonable aliases.
- Sentence naturalness is load-bearing: a sentence that reads like a puzzle leaks. Author for boring plausibility — errands, weather, small talk. The blandness is camouflage.

## Puzzle format

```json
{
  "n": 137,
  "theme": "TREES",
  "aliases": ["tree", "trees", "kinds of trees"],
  "items": [
    { "s": "The map led everyone home.", "w": "MAPLE", "at": [4, 9] }
  ]
}
```

- `at` = start/end letter indices into the sentence's letters-only string; validator recomputes from `w` and fails on mismatch or multiple occurrences.
- Obfuscation per `daily-kit.md`.

## Validator (authoring-side)

- For each item: `w` occurs **exactly once** in the sentence's letter stream, at `at`, and spans ≥1 word boundary (≥2 on hard days).
- Cross-contamination sweep: no *other* theme word (from the full theme list, aliases included) hides anywhere in any of the day's sentences.
- Near-miss report: list all dictionary words length ≥4 spanning boundaries in each sentence — authors verify decoy density (2+ per sentence mid-week) and that no near-miss is a fair theme member.
- Naturalness is authored judgment; validator can't score it. Curation pass required.

## UI specifics

- Sentences as rows of letter tiles with true word spacing; selected span bridges with a cold-accent rail; confirmed stowaway pulls letters together (reduced-motion: highlight only) and re-renders warm + underlined.
- Strike pips in corner HUD; manifest (found words) accumulates at top — it doubles as the theme-guessing worksheet.
- Wrong-confirm buzz is gentle; never shame a near-miss (it spelled a real word — show it faintly gray as acknowledgment: "READ — not aboard").
- Synth: tick on select, hollow knock on near-miss, warm chord on find, small flourish at Captain.

## Content pipeline

- 365 days × (theme + aliases + 5 sentences) authored by Claude, curated for naturalness and theme-alias coverage; validator batch must pass clean.
- Gift layer per `daily-kit.md`: author mode validates a hand-written sentence set (Check: occurrence uniqueness + boundary rule). Hiding her name across a sentence boundary is the intended first custom puzzle.
