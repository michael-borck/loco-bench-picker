# LocoBench Picker: v0.1 prototype

Interactive GPU selection tool for people building local AI inference nodes. Reads the LocoBench benchmark dataset, plots it across user-chosen axes, and highlights the Pareto frontier so you can see the candidates that aren't dominated on both cost and performance.

Not "top 3 cards." Not editorial. A plot that honestly shows the tradeoffs, and lets the shortlist fall out of the data.

## What this is

A proof-of-concept, standalone web page. The UX is live; the data is hand-authored placeholder numbers that resemble plausible benchmark results. Before v0.2, the placeholder file gets replaced by output from the real LocoBench harness, and nothing else in the frontend needs to change.

## How to run

No build step. No server required.

```bash
cd loco-bench-picker-prototype
open index.html
```

Or, if your browser blocks local `fetch()` (Safari, some Chromium policies):

```bash
cd loco-bench-picker-prototype
python3 -m http.server 8000
# visit http://localhost:8000
```

Dependencies are CDN-hosted (D3 v7, Google Fonts). First paint needs internet; subsequent loads are cached by the browser.

## What works in v0.1

- Scatter plot with 24 GPUs, from the real LocoBench floor (GTX 950 2 GB, GTX 960, GTX 1050 Ti, GTX 1060 3 GB, GTX 980 Ti, GTX Titan X Maxwell) through mainstream consumer through current-gen flagships (RTX 5090), plus server cards (M40, P40, P100, V100 16 GB, V100 32 GB). The CC 5.0 Maxwell floor is explicitly inclusive: 2 GB cards are here, they run what they can, the picker shouldn't make someone with a GTX 950 feel excluded from learning local AI.
- Three axis controls: X axis (cost / VRAM / bandwidth / power / year / CC), Y axis (benchmark metric / spec), benchmark selector (four benchmarks in v0.1).
- Six filters in the sidebar: min VRAM, min compute capability, Tensor Cores requirement, max cost, class (consumer / server / both), show-estimated toggle.
- Pareto frontier overlay, drawn in amber, connects the non-dominated points. Toggle-able.
- Colour codes dots by architecture family. Filled = measured, dashed outline = estimated.
- Model-size shortcut: "I want to run 13B at Q4_K_M" auto-sets the min VRAM slider and selects the matching benchmark.
- Currency toggle (AUD / USD) using a canonical USD value × exchange-rate multiplier. Persists via `localStorage`.
- Hover tooltips with card specs, benchmark value, and confidence tag.
- Click a dot → detail panel with full specs, all benchmark results, and notes.
- Explicit framing at the top: *"Each measurement is a tier floor, not a ranking."*

## Aesthetic direction

Lab-bench instrument. Dark warm-navy background (not pure black, but a slight paper-under-lamp warmth). IBM Plex Sans + Mono throughout. Amber phosphor for important data (Pareto frontier, measured values, data callouts). Cyan for interactive controls. Corner brackets on the plot frame and section headers (borrowing from oscilloscope / DSO graphics). No marketing polish.

## Data contract

The entire frontend reads one file: `picker-data.json`.

Entities:

- `currencies`: `{ code, symbol, from_usd }` exchange multipliers. Canonical value is USD.
- `cost_brackets`: `$` through `$$$$$`, each with a USD low/high and a disclaimer (postage excluded, patience required at low brackets, etc.).
- `benchmarks`: definitions of what is measured. Includes `higher_is_better` and `model_vram_gb`.
- `model_shortcuts`: keyed by model size × quantisation, maps to VRAM requirement + matching benchmark ID.
- `architecture_colors`: per-architecture-family colour (consumer) plus a single `server` colour for datacenter cards.
- `gpus`: card entries with specs and a `results` object keyed by `benchmark_id`. Each result has `value`, `confidence` (`measured` / `community` / `estimated`), optional `tested_on`, optional `notes`.

Schema version is declared in `schema_version`. When it changes, bump and migrate.

## The confidence field

- `measured`: LocoBench harness actually ran this on this card.
- `community`: external contributor supplied the number (source should be in `notes`).
- `estimated`: inferred from specs, no real run. Rendered as an open dot with a dashed outline and an `ESTIMATED` tag in the tooltip.

This is how "no data yet" is handled. You emit an `estimated` entry derived from bandwidth / architecture, the tool displays it honestly, and the user can toggle estimates off if they want.

## LocoBench → schema transformer

The real work for v0.2 is writing a small script that reads whatever LocoBench produces (CSV, JSON log, result files) and emits:

```js
gpu.results[benchmark_id] = {
  value: <number>,
  confidence: 'measured',
  tested_on: '<ISO date>',
  source_run_id: '<optional>',
}
```

Everything else (filters, axes, Pareto computation, UI) consumes only the merged `picker-data.json`.

## What v0.2+ would add

- **Real data.** Replace placeholder values with actual LocoBench harness output.
- **Shareable URLs.** Serialize filter state + selected card into the URL so you can send someone a specific view.
- **Detail panel as a route.** Move from inline to `/card/rtx-3090-24gb`-style so you can deep-link.
- **"Similar or better" list** on the detail panel: uses the "floor" framing to surface cards that strictly dominate the selected one (same VRAM + higher bandwidth, or newer arch at same specs, or strictly more VRAM).
- **More benchmarks.** Agent benchmarks (SWE-Bench), additional quantisations, additional models, latency metrics (time-to-first-token).
- **Community contribution flow.** Pull-request-based submission of benchmark results, validated against the schema.
- **Multi-GPU scenarios.** Pairs and trios (NVLink or PCIe tensor parallelism) as virtual entries.
- **Power-efficiency axis.** `tokens_per_sec / watt` as a derived metric.

## Files

```
loco-bench-picker-prototype/
├── index.html          standalone page (inline CSS + JS; loads D3 from CDN)
├── picker-data.json    hand-authored placeholder dataset (24 GPUs, 4 benchmarks)
└── README.md           this file
```

## License

Part of LocoLabo, an independent self-funded research lab. Data is placeholder and for demonstration only. Do not use these numbers to make real purchase decisions. Wait for v0.2 with real LocoBench output.
