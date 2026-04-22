# CLAUDE.md

## Project

atopile 0.15.x PCB project. All components are through-hole. Build entry: `elec/src/level_shifter.ato:LevelShifter`.

## Power Architecture

```
USB-C PD charger (45W, 12V/3A)
  → PD trigger board (9×14mm, configured for 12V) → pwr_in 2-pin header
      ├─ 12V → buck module (MP1584EN, 12V→5V) → ESP32 5V pin + IC VCC
      └─ 12V → screw terminal pin 1 → SK6812 RGBW LED strip power (up to ~120 LEDs @ 3A)
```

## Key Design Decisions

- ESP32-S3 Super Mini is socketed via 2× `Header1x9F` (1×9 female 2.54mm headers), NOT using the castellated ESP32S3SuperMini_package footprint directly
- SN74AHCT125N: all 4 ~OE pins tied to GND (always enabled); VCC from 5V buck output
- Only channels 1 & 2 of the IC are used; channels 3 & 4 inputs (3A, 4A) are tied to GND
- term_a (channel 1, GP1) and term_b (channel 2, GP2) are separate terminals for independent strip segments
- Screw terminal pinout: pin 1 = +12V, pin 2 = DATA (level-shifted via 68Ω), pin 3 = GND
- Pinout matches SK6812 RGBW strip connector order: 12V | DATA | GND
- 68Ω series resistors sit between IC outputs and screw terminals
- 100nF ceramic disc decoupling cap on IC VCC (5V rail from buck)
- Buck module and PD trigger board connect via short wire leads to through-hole headers on PCB

## Parts Pattern

All parts use CNONE (hand-solder). Unique CNONE IDs per part to avoid atopile footprint mismatch warnings:
- `CNONE_HDR1X9F` — 1×9 female header
- `CNONE_SN74AHCT125N` — SN74AHCT125N DIP-14
- `CNONE_SCREW3P254` — 3-pin 2.54mm screw terminal
- `CNONE_R68R_TH` — 68Ω 1/8W axial resistor
- `CNONE_C100NF_TH` — 100nF ceramic disc cap
- `CNONE_BUCKMOD5V` — MP1584EN 5V fixed buck module (22.3×17mm, AliExpress)
- `CNONE_HDR1X2` — 1×2 2.54mm pin header (PD trigger board power input)

## Footprints

All footprints are copied locally from the KiCAD standard library. Source paths:
- `Connector_PinSocket_2.54mm.pretty/PinSocket_1x09_P2.54mm_Vertical.kicad_mod`
- `Package_DIP.pretty/DIP-14_W7.62mm.kicad_mod`
- `TerminalBlock_Phoenix.pretty/TerminalBlock_Phoenix_MPT-0,5-3-2.54_1x03_P2.54mm_Horizontal.kicad_mod`
- `Resistor_THT.pretty/R_Axial_DIN0204_L3.6mm_D1.6mm_P5.08mm_Horizontal.kicad_mod`
- `Capacitor_THT.pretty/C_Disc_D3.0mm_W1.6mm_P2.50mm.kicad_mod`
- `Connector_PinHeader_2.54mm.pretty/PinHeader_2x02_P2.54mm_Vertical.kicad_mod` (buck module)
- `Connector_PinHeader_2.54mm.pretty/PinHeader_1x02_P2.54mm_Vertical.kicad_mod` (PD trigger input)

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

Headers must be spaced **18mm center-to-center** to match the ESP32-S3 Super Mini module width.

## atopile Notes

- `#pragma experiment("TRAITS")` required in all component files
- Do NOT use `has_part_removed` — it drops net assignments. Use `CNONE_*` pattern instead
- GP3 is a strapping pin on ESP32-S3 — keep idle-high, do not use for level shifter inputs
- atopile build updates nets in the .kicad_pcb without touching placement or routing
- 12V rail is present on the PCB — ensure wide traces (≥1mm) on 12V power paths in KiCAD layout
