# Enclosure: Hammond/Tayda 125B

Drill data: `*_125B_drill.csv` (for the Tayda Box Tool) and `*_125B_drill_template.svg` (1:1 printable template). Generated 2026-10-02 from the KiCad board (v1.1, 9 mm pots).

## Tayda Box Tool conventions (as documented by Tayda users)

- **Side A** = face (knobs, LED, footswitch, toggles); **side B** = top end (jacks).
- Each side's origin (0, 0) is its **centre**. Units are mm; **+X right, +Y up**, viewed from outside.
- Diameters in the CSV are for **raw aluminium**. For powder-coated boxes add the allowance Tayda states in the Box Tool (users report +0.2 to +0.4 mm).
- Up to 40 holes. Order the enclosure + the "125B Custom Drill Service" with the **same email** as your Box Tool account, then assign the template to the order in the Box Tool.

## Coordinate mapping (how the numbers were made)

- 125B outline on the board's User.Drawings layer: 66.5 × 121.9 mm, centre at board (150, 127.95).
- The pots/LED/toggles are on the back (panel side), so the outside view is mirrored: **X = 150 − x_board, Y = 127.95 − y_board**.

## Hole sizes used (check against your parts)

| Part | Ø mm |
|---|---|
| 9 mm Alpha pot (RD901F, M7 bushing) | 7.5 |
| 3 mm LED (direct, no bezel; use ~5–6 mm for a bezel) | 3.2 |
| 3PDT footswitch | 12.2 |
| Taiway 100 mini toggle (1/4" bushing) | 6.5 |
| 1/4" jacks (top-mounted) | 9.7 |
| DC jack (slim, top-mounted) | 8.1 |

- **Side B jack coordinates** are the common 125B top-jack layout shared on the PedalPCB forum (jacks at X ±15.2, Y 5.75; DC at 0, −4.4). They depend on your jack hardware, so verify with a printed template first.
- Hyper Fuzz and Rat Energizer share every side-A hole except the Rat's two toggles: same pot grid, LED and footswitch.
- Before ordering, print the SVG at 100% (check the 50 × 10 mm box) and test-fit the pots and PCB.
