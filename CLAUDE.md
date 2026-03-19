# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A static, no-build-step web calculator for an RPUK (GTA roleplay server) community. Two pages:

- **`index.html`** — Run Calculator. Single-item tool: pick item + vehicle + tier, optionally switch to Raw Gather mode, get full breakdown including vehicle capacity bars and comp value.
- **`staff.html`** — Staff Comp page. Multi-row tool: add multiple items with individual units and tiers; auto-calculates on every change; always shows all three comp values (P→P 100%, Server Proc 70%, Server Unproc 33%) side-by-side with the grand total.

No frameworks, no bundler, no dependencies beyond a Google Fonts CDN link. Open the HTML files directly in a browser to run.

## Deployment

Hosted on GitHub Pages (`main` branch), org: **GrandTheftArmaTools**:
- `index.html` → `https://grandtheftarmatools.github.io/GTA-Run-Calculator/`
- `staff.html` → `https://grandtheftarmatools.github.io/GTA-Run-Calculator/staff.html`

Push to `origin/main` to deploy. Changes are live within ~1 minute.

## Data source of truth

The CSV files are the authoritative reference for game data — **not** the JS arrays in the HTML files. When prices, weights, or capacities change, update the CSV first, then sync the JS arrays manually.

| File | Contents |
|---|---|
| `market_items.csv` | Item name, weight (kg), sell price (£), legal/fluid flags |
| `tier_table.csv` | Tier 1–6 multipliers (1×, 1.15×, 1.32×, 1.52×, 1.74×, 2×) |
| `vehicle_capacity.csv` | Vehicle type, name, virtual inventory capacity (kg) |

## Architecture

Both pages are self-contained: all CSS, HTML, and JS live in a single file each. There is no shared JS module or external stylesheet.

### Shared data (duplicated in both files)
- `items[]` — array of `{ name, sellPrice, source }` (staff.html) or `{ name, weight, rawWeight, sellPrice, source }` (index.html)
- `tiers[]` — 6 tier objects with `label` and `mult`
- `sourceGroups[]` — grouping metadata for item `<optgroup>` labels
- `compMults` — `{ p2p: 1.0, 'server-proc': 0.70, 'server-raw': 0.33 }` (index.html) / `{ p2p, proc, raw }` (staff.html)
- `fmt(n)` — formats a number as a GB locale string (comma thousands separator)

### index.html-specific
- `vehicleGroups[]` — grouped vehicle data for the vehicle dropdown
- `runMode` — `'processed'` or `'raw'`; raw mode uses `rawWeight` (= `weight + 1`) for capacity calculations
- Vehicle fill toggle only appears when item has `rawWeight !== null` AND a vehicle with finite capacity is selected

### staff.html-specific
- `rowCounter` — ever-increasing integer ID for each `.item-row`, never reset between add/remove cycles
- Each row stores its selected tier in `data-tier` attribute on the `.item-row` element
- `addRow()` / `removeRow(id)` manage dynamic DOM; remove button is disabled when only one row remains
- No Calculate button — `calculate()` fires on every input change, select change, tier click, and row add/remove
- Layout: items card and result card sit side-by-side (flex row) above 900px, stack vertically below

### staff.html mobile conventions
All interactive elements follow these rules — maintain them when adding new controls:
- Minimum `44×44px` touch target on all buttons (`min-height: 44px`, step/remove explicitly `44px`)
- `touch-action: manipulation` on all `button` elements (set globally, do not override)
- `inputmode="numeric"` on all number inputs
- `font-size: 1rem` (16px) minimum on all inputs — prevents iOS auto-zoom
- `:active` styles on every button for touch feedback
- Breakpoints: 900px (stack layout), 600px (select full-width), 400px (comp grid 2-col, tier mult text hidden)

## Key calculation formula

```
rowValue = Math.round(units × item.sellPrice × tiers[tierIdx].mult)
compValue = Math.round(total × compMults[type])
```

All monetary values are in British Pounds (£).
