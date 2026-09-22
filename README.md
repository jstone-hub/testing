# Signal vs. Noise — Endor Labs booth game

A 60-second vulnerability-triage game for a trade show booth. Players swipe through a
deck of simulated alerts and decide which are **really exploitable** and which are
**noise** — the point being that CVSS severity alone is a bad signal, and reachability
is what matters.

Everything lives in **one self-contained file**: `signal-vs-noise.html`.
No build step, no dependencies, no network. Double-click it, or serve the folder.

---

## Running it at the booth

**Player tablet** — open `signal-vs-noise.html` in the browser and put it in
fullscreen / kiosk mode. Works in portrait or landscape, touch or mouse.

**Second screen (leaderboard)** — open `signal-vs-noise.html#board` in a second
window or on a second display. It opens straight into large-type "booth display"
mode and refreshes automatically as scores come in.

> Both screens must be the **same browser profile on the same machine** — scores are
> kept in `localStorage`, which is per-device. Two separate laptops won't share a
> leaderboard. (That's the v1 tradeoff of having no backend.)

### Controls

| Input | Action |
| --- | --- |
| Swipe right / tap **Fix Now** / `→` | classify as a real, exploitable risk |
| Swipe left / tap **Ignore** / `←` | classify as noise |
| Tap the reveal banner / `Space` | skip the explanation and move on |
| `Esc` or the `✕` | end the round early |
| `L` | jump to the leaderboard |

Arrow keys are there so you can test on a laptop without a touchscreen.

---

## Editing the content

### Cards

The card pool is the `CARDS` array at the top of the `<script>` block. Add, remove or
reword entries freely — the game logic reads the array and never hardcodes anything
about it. Each card looks like this:

```js
{ pkg:"left-pad-utils", version:"2.3.1", ecosystem:"npm",
  cve:"CVE-2025-31447", severity:"Critical", cvss:9.8,
  desc:"Remote code execution via unsanitised template expansion in `padTemplate()`.",
  context:"Imported by `src/api/invoice-renderer.ts`; `padTemplate()` runs on customer-supplied invoice titles.",
  fix_now:true,
  why:"Untrusted input reaches the vulnerable function on a live request path. Severity and reachability agree — fix it." }
```

| Field | Notes |
| --- | --- |
| `severity` | `Critical` / `High` / `Medium` / `Low` — drives the badge colour |
| `cvss` | `0.0`–`10.0`, drives the score meter |
| `context` | the reachability clue; this is what makes the card answerable |
| `fix_now` | `true` = correct answer is **Fix Now**, `false` = **Ignore** |
| `why` | the one-liner shown on reveal |

Text in \`backticks\` renders as inline code. All text is HTML-escaped, so an
apostrophe or an angle bracket in a description is safe.

The 30 shipped cards are deliberately balanced so severity is a misleading signal:
roughly a third are **Critical/High but correctly Ignore** (unreachable, compiled out,
behind an allowlist, test-only) and roughly a third are **Medium/Low but correctly Fix
Now** (reachable on an auth path, on user uploads, in the audit trail).

### Timing and pacing

`CONFIG`, just below the card array:

```js
roundSeconds:           60,    // round length
cardsPerRound:          18,    // 15–20 works well; capped at the pool size
pauseTimerDuringReveal: false, // true = reading the explanation costs no clock
revealMsCorrect:        900,
revealMsIncorrect:      2100,  // longer — the player has to read the "why"
swipeThreshold:         110,   // px of drag that commits an answer
soundOn:                true,
leaderboardSize:        10
```

`pauseTimerDuringReveal` is the one worth knowing about. Left `false`, a round is
exactly 60 seconds of wall clock, which keeps booth throughput predictable — but time
spent reading explanations is time not spent answering. Set it `true` and players get
a full 60 seconds of *decisions*, at the cost of rounds running ~80 seconds.

---

## Scoring

Score is **accuracy** — correct calls ÷ cards answered — not speed. Answering fewer
cards carefully beats blasting through the deck. The end screen also breaks the score
into "real risks caught" and "noise filtered" so players can see which way they were
fooled.

The leaderboard ranks by accuracy, then by cards answered (so a 94% over 17 cards
beats a 94% over 15), then by who got there first.

Name and email on the end screen are optional and stay on the device — nothing is sent
anywhere. Booth staff can wipe the board with **Clear** on the leaderboard screen.

---

## Note on the content

Every package name, CVE identifier, version and finding in this game is **invented**.
None of it describes a real vulnerability in real software, and a disclaimer to that
effect is on the start screen.

The CVE IDs use the realistic `CVE-2025-XXXXX` shape that was asked for. Be aware that
this is the same numbering space real identifiers are assigned from, so a given ID
could coincidentally collide with a real, unrelated CVE. If that's a concern for
legal, the quickest fix is a search-and-replace of `CVE-2025-` to something obviously
fictional (`DEMO-2025-`, `ENDOR-2025-`) in the `CARDS` array — nothing else needs to
change.
