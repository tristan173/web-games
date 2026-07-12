# Daily Kit — shared chassis for the word-game lane

Common infrastructure for Ration, Misquote, and Stowaway. Build once, reuse in each single-file game. Same rules as the arcade lane: one self-contained `.html`, vanilla JS, zero dependencies, zero build.

## Daily system

- Puzzle number `N` = days since the game's epoch date (its launch day), computed from the player's **local** date. Rollover at local midnight.
- A year of puzzles ships embedded in the file as a JSON array. Index by `N % puzzles.length` so the game never dies if we forget to refresh content.
- Answers ship in the file (necessarily), lightly obfuscated (base64) so a casual view-source doesn't spoil. Not security, just keeping the ritual clean.
- "Yesterday's solution" always reachable from the menu. No archive of older days in v1.
- Mid-solve state survives refresh (persist on every move).

## Streaks & stats (localStorage)

- Keys namespaced per game and versioned: `<game>.v1.state`, `<game>.v1.stats`, `<game>.v1.settings`.
- Stats: current streak, max streak, days played, win distribution (per-game meaning of "win" defined in each spec).
- Streak increments on solving day `N` having solved day `N-1`. A solved custom (gift-link) puzzle never touches streak or stats.
- DST/timezone test required: no double-count or skipped day across a clock change.

## Share format

Wordle conventions, no spoilers, no URL required (but include one):

```
<GAME> #137
<emoji result grid — per-game>
🔥 12
```

One tap to copy. Never leak letters, words, or theme.

## Gift layer (the anniversary machine)

- Custom puzzles pack into the URL hash: `<game>.html#p=<base64url(JSON)>`. No server, no expiry.
- Hidden author mode inside the same file (e.g. long-press the title): form in → validated puzzle → link out. Runs the same validation logic the daily content passed, minus the parts needing the big authoring dictionary.
- Custom puzzles show an optional dedication line ("for …, from …") on the solve card.
- Custom play is clearly badged CUSTOM; daily stats untouched.

## House style (inherited from Ping)

- Near-black blue field; one cold accent + one warm accent per game (each spec names its pair); mono uppercase type; tiny corner HUD.
- Tiny WebAudio synth: distinct tick / error / solve voices, all soft. Mute toggle persisted.
- Phone-first layout (she plays on her phone): thumb-reach interactions, no hover-dependent UI, test at 390×844 and desktop.
- The delight visible, the rigor load-bearing.

## Dev bar (house convention, from Mate in 4)

- `?tune` reveals the dev bar: **BUILD number** (const in file, bumped every tuning round — it's the truth-teller against stale Safari caches) + today's date.
- Daily-game essentials: **day time-travel** (next/prev puzzle), reset today, sim streak. Without time-travel you can't test rollover, streak logic, or the weekly difficulty ramp.
- Phone testing per the handoff pattern: the nohup http.server on 8080, reached via the `.local` mDNS hostname.

## Accessibility

- Never color-only signals: pair every color state with a shape/underline/weight change (colorblind-safe).
- Respect `prefers-reduced-motion`.
- Contrast ≥ 4.5:1 for text on field.
- Tap targets ≥ 44px.

## Acceptance checklist (every game)

- [ ] Single file, no network calls after load, works offline once cached
- [ ] ≤ ~150 KB including a year of puzzles
- [ ] iPhone Safari + desktop Chrome/Firefox verified
- [ ] 365 embedded puzzles, all passing that game's validator
- [ ] Refresh mid-solve loses nothing
- [ ] Streak survives DST change (simulate)
- [ ] Share text correct on first solve, repeat visit, and custom puzzle
- [ ] Gift link round-trip: author mode → link → fresh browser solves it
