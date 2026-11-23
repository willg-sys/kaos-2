# KAOS-2 RF Test Board (rev A)

Small 2-layer prototype (72 × 46 mm, 1.6 mm) to prove the 144.39 MHz APRS transmit chain and the 915 MHz LoRa link before the flight RF board. The RF MCU (Raspberry Pi Pico) stays off-board and connects through the 2×9 header J1.

## What's on it
- **144.39 MHz APRS TX:** Semtech SX1278 (direct-FSK AFSK from Pico PIO), Jauch 32 MHz TCXO, L-match to Semtech's measured PA_BOOST load (~13 + j7 Ω, AN1200.19), 7-pole 0.1 dB Chebyshev low-pass (fc ≈ 170 MHz), output pi-pad footprint, edge SMA (J3).
- **915 MHz LoRa:** Ai-Thinker Ra-01H (SX1276) with its own supply filter, pi-match footprint and a 50 Ω coplanar trace to edge SMA (J5).
- Bench 3.3 V input (J2) with current-measure jumper (JP1), test points, u.FL probe footprint (J4, DNP) between match and filter.

## Files
| File | What |
|---|---|
| `kaos2-aprs-proto.kicad_pro/.kicad_sch/.kicad_pcb` | KiCad project (KiCad 10 format) |
| `kaos2-aprs-proto.kicad_dru` | Custom DRC rule (KiCad 8+) waiving edge clearance for the edge-mount SMA pads |
| `fab/kaos2-rf-test_gerbers.zip` | Gerbers + drill exported from KiCad 10 (Protel extensions, single Excellon file) for OSH Park — or upload the `.kicad_pcb` directly |
| `fab/assembly_top.pdf` | Assembly drawing with all reference designators (generated before the KiCad 10 conversion; placement unchanged) |
| `fab/kaos2-aprs-proto_schematic.pdf` | Schematic PDF |

## Key values (144 MHz chain)
PA_BOOST → L4 33 nH series, C1 62 pF shunt; L5 220 nH choke to VR_PA (C8 100 nF at pin 25, C6 100 pF + C7 10 nF at the choke end) → DC block → 7-pole low-pass → pad (R2 0 Ω) → J3. Exact filter values are on the schematic. All RF caps C0G ±2 %; high-Q wire-wound inductors ±2 %.

## Firmware notes
- RegTcxo (0x4B) bit 4 TcxoInputOn = 1; XTB left open.
- Raise RegOcp from the 100 mA default to ~120–140 mA (PA draws ~87 mA at +17 dBm).
- NRESET / Ra-01H RESET must float at power-on (no pull-up): drive open-drain from the Pico.
- Never transmit on both radios at once. Pin assignments are in the J1 table on the schematic.

## Status / before ordering
- DRC (KiCad 10): 0 errors, 0 unconnected. Remaining warnings are expected: KiCad 7 footprint copies vs KiCad 10 library (do not update U1 from library — its thermal vias were enlarged to 0.3 mm drill for OSH Park), and local-label net naming (`RF_NODE` vs `/RF_NODE`).
- Do **not** run Update PCB from Schematic on this board: the footprint↔symbol links don't match and it scrambles placement.
- DNP: C0, C23, C24, J4, R1, R3, R4, R5, R11 (flags set on the board).
- Ra-01H footprint checked against Ai-Thinker's package drawing: 2.0 mm pitch, 8 pads/side, pin 1 (ANT) toward J5.
- Confirm OSH Park 2-layer minimums and dielectric constant (the 915 MHz trace is designed for εr ≈ 4.5).
- Bring-up: transmit only into a dummy load / 30 dB attenuator; measure power, current and harmonics (2nd, 3rd, 11th). Transmitting on 144.39 MHz requires a licensed operator and callsign.
