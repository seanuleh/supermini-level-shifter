# CLAUDE.md

## Project

atopile 0.15.x PCB project. All components are through-hole. Build entry: `elec/src/level_shifter.ato:LevelShifter`.

## Key Design Decisions

- ESP32-S3 Super Mini is socketed via 2× `Header1x9F` (1×9 female 2.54mm headers), NOT using the castellated ESP32S3SuperMini_package footprint directly
- SN74AHCT125N: all 4 ~OE pins tied to GND (always enabled); VCC from ESP32 5V pin (ic.14)
- Only channels 1 & 2 of the IC are used; channels 3 & 4 inputs (3A, 4A) are tied to GND
- Both screw terminals (term_a, term_b) carry identical signals for top/bottom orientation choice
- Screw terminal: pin 1 = OUT1 (GP1), pin 2 = OUT2 (GP2), pin 3 = GND
- 68Ω series resistors sit between IC outputs and screw terminals
- 100nF ceramic disc decoupling cap placed close to IC VCC (pin 14)

## Parts Pattern

All parts use CNONE (hand-solder). Unique CNONE IDs per part to avoid atopile footprint mismatch warnings:
- `CNONE_HDR1X9F` — 1×9 female header
- `CNONE_SN74AHCT125N` — SN74AHCT125N DIP-14
- `CNONE_SCREW3P254` — 3-pin 2.54mm screw terminal
- `CNONE_R68R_TH` — 68Ω 1/8W axial resistor
- `CNONE_C100NF_TH` — 100nF ceramic disc cap

## Footprints

All footprints are copied locally from the KiCAD standard library. Source paths:
- `Connector_PinSocket_2.54mm.pretty/PinSocket_1x09_P2.54mm_Vertical.kicad_mod`
- `Package_DIP.pretty/DIP-14_W7.62mm.kicad_mod`
- `TerminalBlock_Phoenix.pretty/TerminalBlock_Phoenix_MPT-0,5-3-2.54_1x03_P2.54mm_Horizontal.kicad_mod`
- `Resistor_THT.pretty/R_Axial_DIN0204_L3.6mm_D1.6mm_P5.08mm_Horizontal.kicad_mod`
- `Capacitor_THT.pretty/C_Disc_D3.0mm_W1.6mm_P2.50mm.kicad_mod`

## ESP32-S3 Super Mini Pinout (socket mapping)

Left header (J1, header_left) — pin 1 at top toward USB-C:

| Header pin | ESP32 signal |
|-----------|-------------|
| 1 | TX |
| 2 | RX |
| 3 | GP1 |
| 4 | GP2 |
| 5 | GP3 |
| 6 | GP4 |
| 7 | GP5 |
| 8 | GP6 |
| 9 | GP7 |

Right header (J2, header_right) — pin 1 at top:

| Header pin | ESP32 signal |
|-----------|-------------|
| 1 | 5V |
| 2 | GND |
| 3 | 3V3 |
| 4 | GP13 |
| 5 | GP12 |
| 6 | GP11 |
| 7 | GP10 |
| 8 | GP9 |
| 9 | GP8 |

Headers must be spaced **15.5mm center-to-center** (pin row to pin row, verified by physical measurement). The module body is 18mm wide outer edge to outer edge, but the through-hole pin rows are inset ~1.25mm each side.

## atopile Notes

- `#pragma experiment("TRAITS")` required in all component files
- Do NOT use `has_part_removed` — it drops net assignments. Use `CNONE_*` pattern instead
- GP3 is a strapping pin on ESP32-S3 — keep idle-high, do not use for level shifter inputs
- atopile build updates nets in the .kicad_pcb without touching placement or routing
