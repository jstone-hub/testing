# Signal vs. Noise — Endor Labs booth game

A 60-second vulnerability-triage game for a trade show booth. Players swipe through a
deck of simulated alerts and decide which are **really exploitable** and which are
**noise** — the point being that CVSS severity alone is a bad signal, and reachability
is what matters.

Two self-contained files, no build step, no dependencies, no network:

| File | What it is |
| --- | --- |
| `signal-vs-noise.html` | the game itself — this is the only file the booth strictly needs |
| `index.html` | a landing page that explains the game and launches it in an overlay |

Double-click either one, or serve the folder (`python3 -m http.server`) and the
landing page comes up at `/`.

---

## Running it at the booth

**Player tablet** — open `signal-vs-noise.html` in the browser and put it in
fullscreen / kiosk mode. Works in portrait or landscape, touch or mouse.
(Open `index.html` instead if you want the explainer page in front of the game —
its **Play** button runs the game full-bleed in an overlay, and **Close** resets it
for the next player.)

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

## The landing page

`index.html` is a standalone marketing/explainer page using the same design tokens as
the game. It covers the pitch (severity vs. reachability, shown with two real cards
from the deck that a CVSS sort gets backwards), the ~92% noise-reduction figure, what
a round involves, and how to run it at a booth.

It's independent of the game — you can host it on its own, drop it, or lift sections
out of it. The only coupling is the relative link to `signal-vs-noise.html`, so keep
the two files in the same directory.

The **92% figure** is presented as an Endor Labs claim and the proportion bar is drawn
to exactly that ratio (920 noise / 80 reachable per 1,000). Confirm the number against
current marketing before the page goes anywhere public.

---

## Branding

Both files use the Endor Labs design system. Every colour resolves from one labelled
block at the top of each file's `:root` — a **BRAND** group and a **SEVERITY / STATUS**
group. Nothing brand-coloured is hardcoded anywhere else in either stylesheet, so a
future palette change is an edit to those two blocks.

`rgba()` needs channels rather than a hex, so each tinted colour carries a companion
`*-rgb` token. The comments flag which pairs must stay in sync — change `--accent` and
you must change `--accent-rgb` to match.

### Mapping

| Design system | Used for |
| --- | --- |
| `--accent` `#26D07C` (el-green-moon) | primary accent: eyebrows, CVE ids, focus rings, hairlines |
| `--accent-bright` `#00F078` | CTA gradient end, glow |
| `--el-neon-lime` `#31FF94` | "Fix Now", correct answers, hover states |
| `--el-nebula-cyan` `#3FE1F3` | gradient headline end stop, Low severity, "noise filtered" stat |
| `--el-warning-pink` `#E9004A` | "Ignore", incorrect answers |
| `--el-signal-red` `#FF4444` | Critical severity |
| `--el-moon-forest` `#000000` / `--bg-subtle` / `--bg-elevated` | page, panel and card surfaces |
| `--fg-muted` `#B7C3BE` / `--fg-subtle` `#6E7C78` | secondary and tertiary text |
| `--border-subtle` `rgba(38,208,124,.2)` | the green-tinted hairlines, at .16 / .32 |

Buttons are full pills with black ink on green, matching the site's *Book a Demo*.
The wordmark is set as type — **ENDOR** at weight 800, LABS at 400, uppercase.

### Three deliberate deviations

1. **High and Medium severity stay orange and amber.** The brand ramp has no orange,
   and the severity scale has to read red → orange → amber → cyan for players to parse
   it at a glance. Substituting brand pinks would break the ordering — the trap cards
   depend on "Critical" reading as alarming instantly.
2. **Code still uses a monospace stack.** The system specifies Switzer for Primary,
   System *and* Mono. Switzer isn't monospaced, and package names, CVE ids and the
   score columns need fixed-width to line up — losing that costs the whole
   scanner-UI texture the game trades on.
3. **The typeface is the system stack** (SF Pro on iPad, Segoe on Windows) rather than
   Switzer, per the brief. Switzer is free from Fontshare — drop the `.woff2` in, add
   an `@font-face`, and change `--sans` to make it exact.

### Still a placeholder

**The logo.** The design system lists `brand-logos`, `brand-lockups` and
`brand-auri-logos`, but no files came with it. The wordmark is currently set in the
system typeface rather than the real logotype, and the favicon is a lime tile with a
drawn "E". Both are stand-ins. Drop the real SVG in and replace the five `.wordmark`
spans (two in `index.html`, three in `signal-vs-noise.html`).

### Verification

- Every text/background pair checked against WCAG: 22 pairs, all passing. The tightest
  is the Critical badge at 4.65:1 — the severity badge fills sit at 10% opacity
  specifically so the brand red clears 4.5:1 as small text.
- The re-skin's structure was proven neutral first: the tokenisation commit before it
  compared 113,490 computed style values across 11 views and changed nothing.

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
