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

### NP0/C0G RF Filter Capacitor Sourcing (C6–C36, C86)

All bandpass filter capacitors (1206 NP0/C0G) have been standardised on the **KEMET C1206C series** (Mouser prefix `80-C1206Cxxx`), replacing the discontinued Vishay CC126/CC206 Vitramon line and Walsin 791-series parts. TDK C3216C0G parts are used for 1200pF and 2200pF where KEMET doesn't have stock. All are 50V rated (adequate for this 12V-supplied circuit).

Key substitutions made:
- C6, C31: 89pF → **91pF** (E24 standard; negligible filter shift). Part: `80-C1206C910J5GACTU`
- C7, C19, C32 (180pF): `80-C1206C181J5GACTU`
- C8, C33 (270pF): `80-C1206C271J5GACTU`
- C9, C10, C34, C35 (560pF): `80-C1206C561J5GACTU`
- C11, C23, C36 (1200pF): `810-C3216C0G2J122J085AA` (TDK, full part number updated)
- C12, C25 (39pF): `80-C1206C390G5GACTU`
- C13, C26 (56pF): `80-C1206C560J5GACTU`
- C14, C27 (100pF): `80-C1206C101J5GACTU`
- C15, C28 (150pF): `80-C1206C151J5GACTU`
- C16, C29 (220pF): `80-C1206C221J5GACTU`
- C17, C30 (470pF): `80-C1206C471J5GACTU`
- C20 (330pF): `80-C1206C331J5GACTU`
- C21 (470pF): `80-C1206C471J5GACTU`
- C22 (1000pF): `80-C1206C102J5GACTU`
- C24 (2200pF): `810-C3216C0G2J222J085AA` (TDK, full part number updated)
- C66 (220pF 0805): `80-C0805C221J5GACTU`
- C86 (33pF): `80-C1206C330J5GACTU`

**Critical:** All filter caps must be NP0/C0G dielectric — do NOT substitute X7R or other temperature-varying dielectrics.

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

### MEDIUM — L15, L16 inductor part number is invalid
- The BOM lists `994-1812LS-182XJBC` (Coilcraft 1812LS 1.8µH) for L15 and L16
- Research confirmed: the Coilcraft 1812LS series covers **12µH to 1000µH only** — a 1.8µH (code 182) value does not exist in this series
- The part number is either obsolete/discontinued or was a transcription error from the original Farnell 2345144
- **Action needed**: identify a suitable 1.8µH shielded SMD inductor that fits the L_1812_4532Metric footprint (4.5mm × 3.2mm pad area) and replace the BOM entry
- Do not order 994-1812LS-182XJBC — it will not be fulfilled

### RESOLVED — CC206 Vishay capacitor package code
- Previously flagged C16, C17, C21, C29, C30 with uncertain CC206 Vishay package code
- All Vishay CC126/CC206 parts have been replaced with KEMET C1206C series equivalents (see BOM Status above); this issue is no longer relevant

### RESOLVED — C6, C31 89pF non-standard value
- Previously substituted with KEMET 91pF (C1206C910J5GACTU); note updated in BOM

## Library Configuration Notes

- `hardware/r0.4/sym-lib-table` — lists symbol libraries: HL2-MRF101, n2adr, HL2_MRF101-rescue, hermeslite (all correct)
- `hardware/r0.4/fp-lib-table` — lists **footprint** libraries: T50 and n2adr only (hermeslite was removed — it is a symbol library, not a footprint library, and no hermeslite footprints are used in the PCB)
- Do not re-add hermeslite to fp-lib-table

## Git Workflow

- Feature branch: `claude/bom-mauser-compatibility-8y3rf8`
- PRs #1, #2, #3 were merged into master by the owner
- Always create a new branch for new work; use the existing branch only if continuing the same task
- Commit messages should explain WHY not WHAT
