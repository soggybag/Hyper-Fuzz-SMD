# Hyper Fuzz SMD v1.1: test procedure

Board: `HyperFuzz` **v1.1** (git tag `v1.1`; `archive/v1.1/` has the schematic PDF and the interactive BOM). TL072 ×2 + LM358, 125B, 9 mm pots.
General method: `../../_tools/BENCH_TESTING.md`. First build of this board.

## How it works (signal order)

| Block | Parts | What it does |
|---|---|---|
| Power / bias | D11 SS14; **VCLEAN** = +9V via R23 1k + C7 22µ; R24/R25 100k + C8 → **U2B → VREF ≈ 4.35 V** | Filtered, buffered mid-rail |
| Input | R1 1k + C12 1n (RF), R27 1M pull-down, C1 100n, R2 1M to VREF | |
| **GAIN** stage U1A | non-inverting, R3 47k ∥ C13 100p; leg R4 1k + **RV1 GAIN C100k** + C2 10µ | ×1.5 → ×48 |
| **Limiter** U1B | inverting, C3 100n + R5 180k in, R6 680k feedback; D1/D2 to the R7 6k8 / R8 1k5 tap | Output (DRIVE) limited to about ±3.3 V. **SYMMETRY** (RV3 across VCLEAN → R10 47k into U1B +, R9 10k to VREF) shifts it ±0.76 V |
| **Wavefolder** | DRIVE → **RV2 FOLD B1M** rheostat → R11–R14 ladder (27k / 27k / 27k / 100k), D3–D10 pairs to VREF; R15 27k always feeds the last node | The higher FOLD (less resistance), the more pairs conduct: more folds |
| Difference amp U2A | R16 1M2 / R17 680k / R18 680k / R19 1M2 in, R20 1M5 fb, R21 2M2 to VREF | Recombines the ladder taps (SHAPED) |
| **TONE** | R22 1k + **RV4 B50k** + C4 10n → U3A (LM358) buffer | Treble cut, ~16 kHz → ~310 Hz |
| Output | C5 1µ → **RV5 VOLUME A10k** → OUT | |
| LED | D12 from +9V, R26 4k7 → LED_K | ~1.4 mA |

## 0. Before power

| # | Check | Expected | Measured |
|---|---|---|---|
| 0.1 | Visual: U1/U2/U3 pin 1, D11 band, **D1–D10 bands** (pairs alternate), D12; pot tabs soldered | as silkscreen | ✅ |
| 0.2 | Ohms J1 → GND | tens of kΩ | From J1 to GND I get 94.5K |

## 1. First power (100 Ω; knobs at noon; no input)

| # | Check | Expected | Measured |
|---|---|---|---|
| 1.1 | LED off | **~6 mA** (two TL072 ~5 + LM358 ~0.7) | 0.75V -> 7.5mA|
| 1.2 | LED on | +~1.4 mA | 0.88V 8.8mA |

## 2. DC voltages (no input)

| # | Node | Where | Expected | Measured |
|---|---|---|---|---|
| 2.1 | +9V | U1 pin 8 | supply − ~0.25 V | 7.99V |
| 2.2 | VCLEAN | C7 + | ≈ +9V − 0.15 V | 7.88V |
| 2.3 | **VREF** | U2 pin 7 | **≈ 4.35 V** | 3.94V |
| 2.4 | U1 pin 1 (PRE) | | ≈ VREF | 3.88V |
| 2.5 | **U1 pin 7 (DRIVE)**, SYMMETRY at noon / each end | | ≈ VREF at noon; ~~VREF ± 0.76 V~~ **≈ VREF ± 2 V** at the ends (see notes) | 3.9V / 5.95V / 1.86V |
| 2.6 | U2 pin 1 (SHAPED) | | ≈ VREF (moves a little with SYMMETRY) | Min 3.9V, Max 3.89V |
| 2.7 | U3 pin 1 (TONE buffer out) | | ≈ SHAPED | 3.89V |

**Sections 0–2 notes (2026-10-13):**
- **0.2** 94.5 kΩ: no short ✓.
- **Supply current** (100 Ω measured 100 Ω): LED off 7.5 mA, LED on 8.8 mA → the LED adds **1.3 mA** ✓.
  - 7.5 mA vs the ~6 mA estimate is within the TL072s' spread (Rat Energizer: 10.7 mA with two TL072s).
- **All DC voltages are ~0.4 V low because of the 100 Ω test resistor:**
  - 9.0 V − 0.75 V (7.5 mA × 100 Ω) − 0.25 V (SS14) ≈ 8.0 V at U1 pin 8 (measured 7.99 V).
  - VCLEAN 7.88 V; **VREF 3.94 V = exactly half of VCLEAN** (4.35 V expected from a full 9 V).
  - PRE / SHAPED / TONE buffer 3.88–3.90 V ≈ VREF ✓.
  - Without the resistor everything rises proportionally.
- **2.5 SYMMETRY ±2.05 V (5.95 / 1.86 V) instead of the ±0.76 V I predicted: the board is right, my expectation was wrong.**
  - R10/R9 shift U1B's + input by ±0.69 V, and C3 blocks DC through R5, so I assumed the output simply follows (×1).
  - But D1/D2 tie the − input to the 6k8/1k5 tap. With R6 680k feedback, ~2 µA of diode current (at ~0.3 V across them) adds ~1.4 V. Solving for that balance gives ±2 V, as measured.
  - So SYMMETRY is a strong control: at its ends, DRIVE's centre sits ~2 V off VREF inside the ±3.3 V limiter swing. Expect very lopsided clipping at the extremes.
- **For section 3:** remove the 100 Ω (or expect levels ~10% low), and keep SYMMETRY at noon unless a step says otherwise.

## 3. Signal (generator: 200 Hz sine, 100 mVpp; VOLUME low; scope)

| # | Point / setting | Expected | Seen |
|---|---|---|---|
| 3.1 | U1 pin 1, GAIN min / max | ×1.5 (150 mVpp) → ×48 (~4.8 Vpp, may just clip). **GAIN up = clockwise?** | min 1.5V sine, max 7V square |
| 3.2 | U1 pin 7 (DRIVE), GAIN max | inverted, **flat-topped at about ±3.3 V** (the limiter) | 4.5V square |
| 3.3 | Same, SYMMETRY each end | the clipping shifts: one side longer than the other | min 2V square, Max 1V square |
| 3.4 | **U2 pin 1 (SHAPED), FOLD min → max** | a plain clipped wave → **folded wave** with more and more ripples on each half-cycle | Min flat with tiny spikes, through square with “ears”, ranges from 0V to 0.8V. As gain is turned down fold and symmetry bring out “bat” shapes through “ripples” |
| 3.5 | Same, slowly raise the input level | new folds appear as each ladder pair joins in (≈ 0.5 / 1 / 1.5 / 2 V steps at the ladder) | Seems like I need to turn gain all the way down to see all of the “folds” as gain goes up the signal gets progressively squared off. |
| 3.6 | U3 pin 1, TONE ends | bright (16 kHz) ↔ dark (310 Hz) | Sounds like it. |
| 3.7 | OUT, VOLUME max | same as 3.6, centered on 0 V | With Gain at min and Volume max, the output is just under unity. This setting brings out most of the wave folding sound. With Gain at max and volume max the output is just over unity. With tone at min the volume is significantly reduced. Symmetry is sounds the same across the first half and the second half of the sweep, imagine the same sounds are mirrored across each half of the range.  |

## 4. Guitar

| # | Check | Seen |
|---|---|---|
| 4.1 | FOLD from min to max at full GAIN: from fuzz to the "hyper" folded sound | Definite wave folded sound at low gain. More traditional distorted sound with octave overtone at high gain. |
| 4.2 | SYMMETRY: gated / sputtery at the ends? | Quiet almost silent at both extremes, with volume and gain at max the sound is just under unity. There is a spot when fold is about 90% where the volume jumps then it drops again. Sputtery/sizzely as we approach the center. The travel seems mirrored with either half of rotation sounding almost identical. |
| 4.3 | Knob directions all increase clockwise | Yes, with the note about Symmetry. |
| 4.4 | Any squeal or oscillation at full GAIN + FOLD (C13 100p should stop it) | No more noise than would be expected at high gain. Generally the output is a little low. I suspect some settings you might not get enough volume, many settings, low gain, low to medium fold, with volume at full are less than unity. |
| 4.5 | Does TONE have enough range, or is the "better tone control" note in `Build Notes.md` needed? | Tone is okay, could use work but is serviceable as is. |

**Section 4 notes (2026-10-14):**
- **4.1 ✓** Folding at low GAIN, a fuzzier sound with an octave-up overtone at high GAIN. Matches section 3: the limiter squares the signal at high gain, so the ladder has less to fold.
- **4.2 SYMMETRY ends almost silent:** the ~±2 V DC shift at the ends parks DRIVE near a rail, and offsets the whole ladder, so one side's diode pairs stay biased on and swallow the signal. Together with the mirrored halves, only ~¼ of the knob is useful. → v1.2: one-sided and gentler (see section 3 notes).
  - The jump in volume at ~90% FOLD is a ladder-pair alignment peak (one pair starts conducting at the signal's peak); normal for a diode ladder.
- **4.3 ✓** (SYMMETRY as above).
- **4.4 no oscillation ✓. Output a little low:** many low-GAIN, low/mid-FOLD settings are below unity at full VOLUME. → v1.2: make-up gain.
- **4.5 TONE serviceable.** The tilt-tone idea in Build Notes stays optional.

**Bench mod: R10 47k → 150k (2026-10-14):**
- SYMMETRY ends (GAIN and FOLD at noon) are now **about half of unity, no longer near-silent** ✓. The shift at the + input drops from ±0.69 V to ±0.25 V, so DRIVE stays off the rails. **The two halves still sound mirrored** (confirmed): the fix is the one-sided wiring in v1.2, not the resistor value.
- **The overall level is low:**
  - GAIN, FOLD and SYMMETRY all at min, VOLUME max: very quiet. It rises once FOLD passes ~80%, and reaches unity at FOLD max.
  - FOLD min, SYMMETRY max: quiet until GAIN ~80%; unity at GAIN max.
- **Why the action bunches at the top of FOLD:** FOLD is a **B1M (linear) rheostat** in series with a 27k–100k ladder. The ladder only gets real drive once the rheostat is down to ~100–200k, which is the last 10–20% of the rotation. That's the same problem as the B100k GAIN pot (×1.5 → ×2.8 over 75% of its travel).
- **Conclusions for v1.2:**
  1. **R10 = 150k** (keep), plus the one-sided SYMMETRY wiring.
  2. **Make-up gain is needed** (U3A ×3 or more). Most settings are below unity even at full VOLUME.
  3. **FOLD taper:** a **C1M** (reverse log: ~100k at noon) spreads the fold across the knob. Alternatively a lower value (B250k): test the feel and how clean FOLD min stays.
  4. **GAIN:** keep the C100k (the B100k test pot confirms the bunching).

## 5. After boxing

- Supply current (1.1). Check the jacks' ground lugs aren't touching the pot bodies.

## Record

| Item | Result |
|---|---|
| Unit number | |
| Date | |
| Notes | |
