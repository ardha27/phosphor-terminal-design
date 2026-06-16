# Phosphor Terminal — Design Language

The shared visual system behind two live trading dashboards:

- **MERIDIAN** — `DLMM Liquidity Terminal · Solana`. An autonomous Meteora DLMM liquidity-provider agent.
- **XAU·EA** — `Martingale Learning Terminal · Gold`. A self-tuning XAU/USD MetaTrader EA.

Both render the same idea: a dense, always-live **instrument panel** an operator reads at a glance. XAU·EA is derived from MERIDIAN and retargeted to gold — same chrome, different default skin. This doc is the one source of truth for the look.

---

## The one idea

A **phosphor CRT quant terminal**. Dark, monospace, hairline-bordered cells packed edge to edge. One loud thing per screen (an italic serif hero numeral); everything else stays quiet. It should feel like a Bloomberg terminal that someone left running on a green-screen monitor at 3am — alive, dense, never decorative.

Restraint is the rule (the "Chanel" rule): the hero serif numeral is the single loud element. Everything else is hairline borders, faint labels, no gratuitous motion.

---

## Palette

Every theme is a dark base + a single accent + theme-independent PnL colors. Themes are a `[data-theme]` toggle; the only difference between the two apps is which one is default.

```
PHOSPHOR (MERIDIAN default)  bg #050705  panel #0a0f0b  accent #3dff7c  green
AMBER / GOLD (XAU·EA default) bg #0a0805  panel #110d07  accent #ffb000  gold
CYAN                          bg #04080f  panel #071120  accent #4dd8ff
VIOLET                        bg #0a0612  panel #120a1e  accent #b48cff
```

The subject picks the default: Solana → phosphor green, XAU/gold → amber. The other three stay as a toggle in the top bar.

**PnL semantics are theme-independent and never repurposed:**

```
--green #3dff7c   up / profit / win
--red   #ff5c5c   down / loss
```

`--accent` is for **chrome and emphasis only** — never to signal profit or loss. This separation is load-bearing: a green accent theme still uses the same `--green` for a winning number, so semantics never collide with branding.

Each theme also defines `--fg` / `--fg-dim` / `--fg-faint` (a three-step text ramp), `--line` / `--line-soft` (two-step border ramp), `--bg-panel` / `--bg-inset`, and a `--glow` (`0 0 22px` accent at ~25% alpha).

---

## Type

Two faces, deliberate roles. No third.

- **IBM Plex Mono** — everything structural: labels, data, tables, tickers, body. Tabular numerals on every figure (`font-variant-numeric: tabular-nums`) so columns don't jitter as they tick.
- **Instrument Serif, italic** — *only* the hero numerals: Net P/L, the live score, calendar totals. The italic serif against the mono grid is the page's voice. Using it anywhere else cheapens it.

Scale:

| Role | Size | Tracking |
|---|---|---|
| Labels (uppercase) | 8px | 0.18–0.22em |
| Body / data | 10–11px | 0.01em |
| Hero serif numeral | 42–52px | -0.01em |

Uppercase + wide tracking is the terminal register for labels. Values stay natural-case. Base body is `12px / 1.45`.

---

## Atmosphere (the signature)

The CRT chrome is what makes it unmistakable. Applied once, never fought:

- **Scanlines** — a fixed 1px `repeating-linear-gradient` over everything (`body::before`, `z-index: 999`, `pointer-events: none`).
- **Vignette** — a faint accent glow top-center, darkness creeping up from the bottom (`.vignette`).
- **Phosphor glow** — `text-shadow: var(--glow)` on the accent glyph, the live dot, and hero numerals **only**.
- **Flicker** — the hero numeral dims ~once every 9s (`@keyframes flicker`). An idle CRT, not a strobe.
- **Corner brackets** — hero panels get two accent L-brackets (`.corner::before` / `::after`).
- **Spinning glyph** — the brand glyph rotates once per 14s. The one piece of perpetual motion.
- **Rise-in** — panels fade up 7px on load (`@keyframes rise`), staggered by column.

All motion respects `@media (prefers-reduced-motion: reduce)` → animations off.

---

## Layout

A dense **3-column terminal grid** between a fixed top bar and a bottom status bar. Side columns are fixed-width; the center anchors row height. Structure encodes content — panels are bordered cells with a letter-spaced head (`label` left, `phint` right), **not** floating cards. No drop shadows, no rounded corners, no padding-as-decoration.

```
┌ topbar: ◈ BRAND ─ marquee ticker ───────────── THEME · ● LIVE · 12:00 ┐
├ secondary ticker: recent activity scrolling ─────────────────────────┤
│ LEFT  ~295px      │ CENTER 1fr            │ RIGHT ~330px              │
│  hero P/L panel ⌐ │  main chart           │  params / weights        │
│  KV stat row      │   (anchors height)    │  feed (fill + scroll)    │
│  score / gate ⌐   │  secondary chart      │                          │
├ strip (full width): radar · glance · donut ──────────────────────────┤
├ table (full width, scroll / paginate) ───────────────────────────────┤
└ statusbar: version · state · last update ────────────────────────────┘
```

Breakpoints collapse the grid predictably:

- `≤1180px` → 2 columns; right column drops full-width and flows as wrapping cards.
- `≤820px` → 1 column; the top marquee hides (panels carry the data), charts shrink, hero numeral → 42px.
- XAU·EA's analytics grid steps `8 → 5 → 3 → 2` tiles per row down to 480px.

**Panel anatomy:** `1px var(--line)` border on `var(--bg-panel)`; a `.panel-head` strip with `9px / 0.2em` dimmed uppercase label left and a fainter `.phint` right; body is hairline-divided rows. Filling panels use `flex: 1 1 0; min-height: 0` with an inner `overflow-y: auto` so the list scrolls instead of stretching the row.

---

## Recurring components

| Component | What it is |
|---|---|
| **Hero numeral** | The serif italic P/L. Glows, flickers, flips color on sign. The eye's first landing spot. |
| **KV row** | A `repeat(4, 1fr)` grid of label-over-value cells, hairline-separated. Tabular nums. |
| **Marquee ticker** | Seamless-loop scrolling stat strip (two identical halves, translateX -50%). Masked edges. |
| **Live dot** | Pulsing accent dot; turns red + stops pulsing on stale data. |
| **Mini bar rows** | `name · track · value` grids for weights, timeframe emphasis, reason scores. Fill animates width. |
| **Feed** | Tagged, timestamped, clickable event rows that open a detail modal. Line-clamped reasons. |
| **Data table** | Sticky dimmed header, hairline rows, hover-inset, right-aligned numeric columns, pill tags. |
| **Calendar modal** | Daily-realized PnL heat grid; serif total header; click a day for its log. |
| **Status bar** | Version · live state · last update, in faint 9px letter-spaced caps. |

Scrollbars are themed thin (8px, `var(--line)` thumb, transparent track).

---

## Signature element per app

The grid and chrome are shared; each app earns **one panel a generic dashboard wouldn't have**, and it gets a corner-bracket and a spot where the eye lands after the hero P/L.

- **MERIDIAN → Decision Feed + Signal Weights (Darwin).** The agent narrates *why* it deployed or closed, and shows its self-tuned signal weights recalculating. You can watch it learn.
- **XAU·EA → Learning Gate (Darwin).** Answers the operator's real question — *how many more good / bad cycles until the EA can evolve its params?* — as two phosphor progress bars (good X/5, bad Y/5) and a total (Z/50), with the apply-gate state called out when a published version isn't live on the EA yet.

---

## Copy

Operator-side language, plain and present-tense.

- State the live state plainly: "Evolving" when ready, "Holding — v12 not yet live on the EA" when the apply-gate blocks.
- Counts, not jargon: "EA needs 8 more good, 3 more bad" — not "cold-start gate unmet".
- Empty states invite, never apologize: "Curve starts after the first close."
- Labels are nouns in caps (`PERFORMANCE`, `ACTIVE PARAMS`, `DECISION FEED`); hints are terse context (`NET REALIZED`, `BY AVG PNL`, `LAST 60 · GREEN PROFIT · RED DD`).

---

## Building something new in this language — checklist

1. Dark base, one accent, untouchable green/red for PnL.
2. IBM Plex Mono for everything; Instrument Serif italic for the single hero numeral only.
3. Hairline-bordered panels with a letter-spaced uppercase head. No cards, no shadows, no rounded corners.
4. Scanlines + vignette + glow-on-accent + 9s flicker. Apply once.
5. Tabular numerals on every figure.
6. 3-column grid, center anchors height, collapse to 2 then 1.
7. Give the page exactly one signature panel that answers its core question, and bracket it.
8. Respect `prefers-reduced-motion`.
