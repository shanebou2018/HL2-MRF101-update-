100W Amplifier Companion for the Hermes-Lite 2.x
================================================

See the [Wiki](https://github.com/mathisschmieder/HL2-MRF101/wiki) for details.

This is a work in progress to create a 100W power amplifier companion board for the [Hermes-Lite 2.x](http://www.hermeslite.com) using the NXP MRF101AN MOSFET. It is based on WA2EUJ's [Single NXP MRF-101 Eval Board](https://sites.google.com/site/rfpowertools/) that won 1st place in the NXP Design Challenge.

First builds of revision 0.2 are successful. Several changes and additions were done by M5EVT to revision 0.3. Revision 0.4 is the current design with further fixes including corrected library paths and schematic/PCB updates.

## Revision history

- **r0.2** – First successful builds
- **r0.3** – Changes and additions by M5EVT
- **r0.4** – Issue fixes (#2 #3 #4 #5 #8 #9), library path corrections, schematic and PCB updated (current)

## BOM

The BOM (`bom/HL2_MRF101_r04.csv`) is fully sourced for Mouser Electronics. All filter inductors (L1–L12) are hand-wound on T50 iron-powder cores; winding turn counts are included in the BOM notes column.

**Note:** R36, R37, R38 (PureSignal attenuator) are marked TBD — values depend on your specific installation.
