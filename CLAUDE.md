# HL2-MRF101 Project — Claude Agent Context

## Project Overview

This is a 100W HF power amplifier companion board for the Hermes-Lite 2.x SDR, using an NXP MRF101AN MOSFET. The PCB is designed in KiCad 4. Current hardware revision is **r0.4** (files in `hardware/r0.4/`).

## Repository Layout

```
bom/                        Bill of materials (CSV + interactive HTML)
hardware/r0.4/              KiCad project files (schematic, PCB, cache)
libs/                       Custom symbol and footprint libraries
  HL2-MRF101.lib            Custom symbols (Coupler, MCP4561, WE-DCT)
  hermeslite.lib            Hermes-Lite symbol library (symbols only — NOT footprints)
  n2adr.lib                 N2ADR custom symbols (relay, MCP23008, etc.)
  n2adr.pretty/             Custom footprints (relays, connectors, transformers)
  T50.pretty/               T50 toroid footprints
measurements/               S-parameter plots from test builds
```

## BOM Status

`bom/HL2_MRF101_r04.csv` — fully updated for Mouser sourcing. Key notes:

- All parts now have Mouser part numbers
- A **Notes** column (7th column) was added; it contains winding data, substitution notes, and flags
- Rows with designators prefixed `MATERIAL-` (MATERIAL-CORE-T50-2, MATERIAL-CORE-T50-6, MATERIAL-WIRE) are build materials with no PCB reference — treat them as shopping list items, not components
- **R36, R37, R38** are PureSignal attenuator resistors with value TBD — do not assign values without consulting the builder
- The two groups of 100nF 0805 capacitors (C70 group and C78 group) now share the same Mouser part (810-C2012X7R2A104K125AA); they were previously split across Mouser and Farnell

### T50 Wound Inductor Turn Counts (L1–L12)

All wound on T50 iron-powder cores, 22 AWG enameled copper wire. T50-2 (red, AL=49) is recommended for all. Formula: N = 100 × √(L_µH / AL)

| Ref | Target | T50-2 turns | Achieved | T50-6 turns | Achieved |
|-----|--------|-------------|----------|-------------|----------|
| L1, L7 | 224 nH | 7 | 240 nH (+7%) | 7 | 196 nH (−13% — use T50-2) |
| L2, L8 | 385 nH | 9 | 397 nH (+3%) | 10 | 400 nH (+4%) |
| L3, L9 | 680 nH | 12 | 706 nH (+4%) | 13 | 676 nH (−1%) |
| L4, L10 | 1.1 µH | 15 | 1102 nH (0%) | 17 | 1156 nH (+5%) |
| L5, L11 | 1.7 µH | 19 | 1769 nH (+4%) | 21 | 1764 nH (+4%) |
| L6, L12 | 3.1 µH | 25 | 3062 nH (−1%) | 28 | 3136 nH (+1%) |

Cores: Mouser 673-T50-2 (×12) and optionally 673-T50-6 (×6 for higher-band filters L1–L3/L7–L9).
Wire: Mouser 829-MW0220.100P (~1 m total for all 12 inductors).

## Known Open Issues (not yet fixed — investigate before making changes)

### MEDIUM — R29 missing from schematic
- R29 (2kΩ, 0805, Mouser 71-CRCW08052K00FKEAC) is listed in the BOM but has **no matching symbol in the schematic** (`HL2_MRF101.sch` has zero occurrences of "R29")
- This is a real discrepancy. Either:
  - R29 was accidentally dropped from the schematic during the r0.3→r0.4 revision, or
  - R29 is no longer needed and should be removed from the BOM
- Do NOT silently remove it from the BOM or add it to the schematic without confirming with the user which is correct

### MEDIUM — Rescue library (191 of 209 components)
- Almost all components use `HL2_MRF101-rescue` as their symbol library (`L HL2_MRF101-rescue:...` in the schematic)
- This happened when KiCad's rescue process was triggered due to missing/renamed libraries
- The design opens and works correctly, but rescue libraries are brittle when sharing the project or upgrading KiCad versions
- Fix requires remapping each symbol to the canonical KiCad library (Device:C_Small, Device:R_Small, etc.) — a significant manual effort in the KiCad schematic editor
- Do not attempt to fix this by editing the .sch file by hand; it must be done via KiCad's "Change Symbol" function

### LOW — CC206 Vishay capacitor package code unverified
- C16, C17, C21, C29, C30 use Mouser part numbers with "CC206" in the part number (e.g. 603-CC206JRNPOBBN221)
- These are in 1206 (C_1206_3216Metric) footprints in the PCB
- The CC206 Vishay code *may* indicate a 2010 case rather than 1206 — needs datasheet verification before ordering
- If they turn out to be 2010 size, replacement NP0 1206 parts will need to be sourced and the BOM updated (no PCB change needed if a 2010-bodied part can fit a 1206 pad, but confirm first)

### LOW — C6, C31 89pF non-standard value
- 89pF is not an E24 standard value; Vishay CC126JRNPOBBN890 is listed
- If unavailable, substitute 91pF (603-CC126JRNPOBBN910) — note the slightly higher value will shift the filter corner frequency very slightly
- Confirm with the builder before substituting

## Library Configuration Notes

- `hardware/r0.4/sym-lib-table` — lists symbol libraries: HL2-MRF101, n2adr, HL2_MRF101-rescue, hermeslite (all correct)
- `hardware/r0.4/fp-lib-table` — lists **footprint** libraries: T50 and n2adr only (hermeslite was removed — it is a symbol library, not a footprint library, and no hermeslite footprints are used in the PCB)
- Do not re-add hermeslite to fp-lib-table

## Git Workflow

- Feature branch: `claude/bom-mauser-compatibility-8y3rf8`
- PRs #1, #2, #3 were merged into master by the owner
- Always create a new branch for new work; use the existing branch only if continuing the same task
- Commit messages should explain WHY not WHAT
