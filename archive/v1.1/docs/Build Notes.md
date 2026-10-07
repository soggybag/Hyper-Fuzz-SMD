# Hyper Fuzz SMD: build notes

## v1.1 review and changes (2026-10-02)

v1.0 (16 mm pots, fab files generated 2026-09-27, never ordered) is tagged `v1.0-16mm` in git.

### Schematic review

**Checked OK:**
- **VREF:** double-filtered (R23 1k + C7 22µ, then R24/R25 + C8 10µ) and buffered by U2B. Robust against the UF3 failure (supply ripple into a high-gain stage), even with ceramic DC-bias derating.
- **Gain stage:** 1.5–48. C taper on the rheostat is the right choice for gain.
- **Limiter:** ±3.3 V. **SYMMETRY:** ±0.77 V.
- **Wavefolder:** ladder and difference amp as the original.
- **TONE:** 16 kHz–310 Hz. U3B spare half parked.
- No CMOS; wire pads are the larger UF-style; header centred on the footswitch.

**Fixed in v1.1:**
- **R27 1M, IN → GND:** input pull-down. C1's input side floated when bypassed, which causes a pop on switching.
- **C12 1n:** at the R1/C1 junction; with R1 1k it's a ~160 kHz RF filter.
- **C13 100p C0G across R3:** tames hiss and RF at gain 48.
- **U3 TL072 → LM358 (C7950):** TL072 inputs must stay ~3–4 V above ground or the output can phase-reverse. U3A (output buffer) follows the folded signal, which can swing down to ~1.5 V. LM358 inputs work down to 0 V (the original builds used LM358).

**Noted, not changed:**
- U1B + input reaches 3.6 V at the SYMMETRY extreme (borderline for TL072 worst case; fine typically).
- C2 10µ (ceramic, ~4.4 V DC) derates, so the bass corner at full GAIN is ~30 Hz instead of 16 Hz. Harmless.
- A scratchy SYMMETRY pot will thump (it's DC-coupled into the clipping chain).

### PCB v1.1: 9 mm pots and new knob layout

- **Pots:** Alpha **RD901F-40** 9 mm (KiCad `Potentiometer_THT:Potentiometer_Alpha_RD901F-40-00D_Single_Vertical`).
  - Mounted on the back (pot side), so the footprints are on B.Cu, rotated 90°. The pins are 7.5 mm below the shaft with the same lug order as Ugly Face 1 (knobs turn the right way).
  - **The two mounting tabs must be soldered.**
  - Excluded from the BOM and position files (hand-soldered).
- **Knob grid (board coordinates; outside view is x-mirrored):**
  - Top row (outside view): GAIN · FOLD · SYMMETRY, shafts at y = 108, x = 169 / 150 / 131 (19 mm pitch).
  - Bottom row: **TONE under GAIN** (169, 133) and **VOLUME under SYMMETRY** (131, 133).
  - **LED D12** centred between them at (150, 133).
- **Drilling changed** vs v1.0. The top row moved ~9.4 mm down because the 9 mm pots must sit fully on the board. The Rat Energizer no longer shares this drilling; it can follow later. **Update the enclosure drilling template.**
- **Wire pads moved** out from under the pot bodies: 9V (140.5, 102.9), G (159.5, 102.9), OG (123.6, 101.6), IG (176.4, 101.6). D11 moved to (150, 103.6).
- Fully re-routed (Freerouting, GND routed too, then the pours re-filled). DRC 0, unconnected 0, ERC 0. Connectivity verified identical to the schematic netlist (the CLI parity "net_conflict" messages are the known KiCad 10 false positives).
- Silkscreen: knob labels moved; title now **HYPER FUZZ v1.1**, bottom-left.
- **JLC files for v1.1 generated 2026-10-02** after layout review: `production/HyperFuzz.zip` (Gerbers + drill), `bom.csv` (54 SMD placements), `positions.csv` (all top), `HyperFuzz_hand_solder.csv`.
- The RD901F 3D model isn't in KiCad's library (only affects renders).

## Future idea: a better tone control (2026-10-02)

- **Already there:** TONE (RV4 B50k) is a **one-pole treble cut**: R22 1k + up to 50k into C4 10n, about 16 kHz down to 310 Hz, buffered by U3A. It can only darken the sound.
- **Why consider more:** the wavefolder adds a lot of upper harmonics. A treble cut alone tends to swing from fizzy to muffled, and there's no way to add body or shape the mids.
- **Options (cheapest first):**
  1. **Tilt (Big Muff style) tone:** passive LP/HP blend with a mid scoop at noon. One knob, a few passives; needs make-up gain (U3A could become a gain stage).
  2. **Active Baxandall (bass + treble):** reuse The Sultan v4 block. Two knobs, or one dual-gang. The best range, but it needs more panel space (a 6th knob or a concentric pot).
  3. **Mid / "contour" switch:** a toggle that adds a mid scoop or boost to the current TONE. No extra knob.
- If this goes in, revisit U3's role: it's an LM358 output buffer in v1.1, which is fine for a passive network. For an active Baxandall, use a TL072 with the input range kept away from ground.

## BOM check (2026-10-02)

- Board BOM vs schematic: **every value and LCSC number matches**, and nothing is missing.
- Every LCSC number checked against its value/package: either proven on a received board (UF1, UF3, Trem v1) or looked up on JLC (C1790 100p C0G, C7950 LM358DR2G, C17593 27k, C17380 1M2, C26113 2M2, C17797 680k, C26110 1M5, C17501 180k, C4310 1k5, C1779 4u7 25V X5R, C1729 22n X7R, C28636 560, C27834 5k1). **All correct.**
- Basic vs Extended status **could not be read reliably** from JLC's pages for the high-value resistors (1M2, 1M5, 2M2, 680k, 180k, 1k5, 5k1, 560). Check the fee summary at JLC's BOM step.

