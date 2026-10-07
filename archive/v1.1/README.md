# Hyper Fuzz SMD v1.1: archived revision

- **Git tag:** `v1.1` (commit `e570386`, 2026-10-06)
- **Tag note:** v1.1: ordered from JLCPCB (October 2026 batch)
- **Archived:** 2026-10-06 with `_tools/archive_version.py`

## Finding a part

1. Open `HyperFuzz_ibom.html` in a browser. Type a reference (R14) or a value (100k) in the filter box; the part lights up on the board.
   - Hover over a pad to see its net.
   - F / B switch sides.
2. Find the same reference in `HyperFuzz_schematic.pdf` with Cmd-F.
3. `HyperFuzz_board_front.pdf` / `_back.pdf` are printable: page 1 has the outlines and references, page 2 the copper. The back is mirrored, as you see it holding the board.

## Files

- `HyperFuzz_board_back.pdf`
- `HyperFuzz_board_front.pdf`
- `HyperFuzz_ibom.html`
- `HyperFuzz_render_bottom.png`
- `HyperFuzz_render_top.png`
- `HyperFuzz_schematic.pdf`
- `docs`/
- `parts.csv`
- `production`/

`production/` holds the files sent to JLCPCB for this revision: HyperFuzz.zip, HyperFuzz_hand_solder.csv, bom.csv, designators.csv, netlist.ipc, positions.csv.
