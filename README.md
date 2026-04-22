# Hyper Level Shifter — Integrated Power Variant

A through-hole PCB that pairs an ESP32-S3 Super Mini with a SN74AHCT125N quad bus buffer to shift 3.3V GPIO signals up to 5V, **plus integrated USB-C PD power** for driving 12V SK6812 RGBW LED strips with [HyperK](https://github.com/hyperion-project/hyperion.ng) / [HyperHDR](https://github.com/awawa-dev/HyperHDR) ambilight setups.

> **Two variants exist:**
> - `main` branch — signal-only level shifter, external 5V power
> - `integrated-power` branch (this) — USB-C PD input, onboard 12V→5V buck, 12V to strip

## Power Architecture

```
USB-C PD charger (45W, 12V)
  → PD trigger board (J5, 9×14mm) → 12V rail
      ├─ 12V → buck module (U2, MP1584EN) → 5V → ESP32 + IC
      └─ 12V → screw terminals pin 1 → LED strip power (~120 LEDs @ 3A max)
```

## Screw Terminal Pinout

Each terminal is independent (one per LED strip channel):

| Pin | Signal | Description |
|-----|--------|-------------|
| 1   | +12V   | LED strip power (direct from PD trigger) |
| 2   | DATA   | Level-shifted 5V data signal (via 68Ω) |
| 3   | GND    | Ground |

Pinout matches SK6812 RGBW 12V strip connector order — plug straight in.

## Signal & Power Path

```
USB-C PD → J5 (PD trigger, 12V out)
  → vcc_12v rail → U2 buck IN+ → U2 OUT+ → 5V → ESP32 5V pin + U1 VCC
  → vcc_12v rail → term_a/term_b pin 1 (LED strip power)

ESP32 GP1 (3.3V) → U1 1A → 1Y → 68Ω (R1) → term_a pin 2 (DATA ch1)
ESP32 GP2 (3.3V) → U1 2A → 2Y → 68Ω (R2) → term_b pin 2 (DATA ch2)
GND ──────────────────────────────────────→ term_a/term_b pin 3
```

## BOM

| Ref | Part | Notes |
|-----|------|-------|
| J1, J2 | 1×9 female 2.54mm pin header | ESP32 socket |
| U1 | SN74AHCT125N | DIP-14, e.g. LCSC C5907 |
| — | DIP-14 IC socket | Order separately |
| J3, J4 | 3-pin 2.54mm screw terminal | KF128-2.54-3P or Phoenix MPT-0.5/3-2.54 |
| R1, R2 | 68Ω 1/8W axial resistor | Any 62–100Ω 1/8W axial |
| C1 | 100nF ceramic disc capacitor | 50V, 2.5mm pitch |
| U2 | MP1584EN buck module, 5V fixed | 22.3×17mm, AliExpress — search "MP1584EN mini buck 5V fixed" |
| J5 | USB-C PD trigger board | 9×14mm, AliExpress — search "PD QC decoy trigger module" — configure for 12V before mounting |

## ⚠️ Footprints Needing Physical Verification

**U2 and J5 footprints were estimated from product images.** Measure with calipers when modules arrive and update pad positions in KiCAD before ordering the PCB.

See `CLAUDE.md` for exact details on what to measure.

## Assembly Notes

- Configure J5 (PD trigger) for **12V output** by bridging the correct solder pads on its front **before** mounting it on the PCB — you won't have access after
- J5 mounts back-down with USB-C overhanging the PCB edge
- U2 (buck module) mounts flat-face-down, soldering its castellated corner pads to the PCB
- Use a **45W+ USB-C PD charger** that supports 12V/3A output
- Install DIP-14 socket first, then insert SN74AHCT125N
- ESP32-S3 Super Mini plugs into J1/J2 with USB-C toward the "USB Port ↑" silkscreen

## Project Structure

Built with [atopile](https://atopile.io) 0.15.x.

```
ato.yaml                        # Build config
elec/src/
  level_shifter.ato             # Top-level module
  esp32s3_super_mini.ato        # ESP32 socket module
  parts/
    Header1x9F/                 # 1×9 female header
    SN74AHCT125N/               # DIP-14 quad buffer
    ScrewTerminal_2.54_3P/      # 3-pin screw terminal
    Resistor_TH_68R/            # 68Ω axial resistor
    Capacitor_TH_100nF/         # 100nF ceramic disc cap
    BuckModule_5V/              # MP1584EN 5V buck module
    PowerConnector_1x2/         # PD trigger board connection
elec/layouts/default/           # KiCAD PCB layout
```
