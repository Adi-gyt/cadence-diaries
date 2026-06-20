# Seal Ring & Dummy Fill

## Seal Ring

### What It Is
A continuous metal guard structure placed at the outermost boundary of the die, just inside the scribe line. It encloses the entire chip.

### Why It Exists
- **Mechanical protection:** Die sawing creates crack propagation. The seal ring acts as a barrier, stopping cracks from reaching the active circuit area.
- **Contamination barrier:** Prevents moisture, ionic contamination, and oxidation from creeping in from the die edge.
- **Required for tapeout:** Foundries mandate it. Without a seal ring, GDS submission is rejected.

### SCL 180nm Seal Ring
- GDS source: `/home/install/SCL/scl180/ext_str/SEALRING_PAD_PDK2020.gds`
- Cell name: `SCL18_Seal_ring_4M1L_a0`
- Stream into a new library (`SEALRING`) attached to `ts018_scl_prim`

### Sizing Rules (from NIT Calicut manual)
- Inner edge to active area: **≥10µm clearance**
- Seal ring width: **11µm**
- For C2S/MPW tapeout: outer dimensions must be **2mm×2mm** or **5mm×5mm**
- For lab/demo: 1mm×1mm (1000×1000µm) is used as example

### How to Resize
1. Open seal ring layout cell
2. Select each edge with drag-select — must capture **all layers** at once (not just one)
3. Press `S` to stretch
4. Set dimension to target size

If you select only one layer before stretching, that layer moves while others stay — causes distortion and DRC errors.

### VSS Connection
After placing seal ring in `final_chip` layout:
- Draw a **TOP_M drawing** rectangle (30µm wide) from the VSS terminal of the `pv0i` pad cell down to the inner edge of the seal ring
- Place an `ML_M3` via at the junction between TOP_M wire and the M3 VSS net
- The seal ring must be grounded — it functions as a guard ring

### Placement
- Instantiate in `final_chip` layout (not a separate assembly cell)
- Place centered around the IO ring
- Save layout

---

## Dummy Fill

### What It Is
Small metal/poly/active rectangles inserted into otherwise empty regions of the layout to bring layer densities within foundry-specified bounds.

### Why It Exists
- **CMP uniformity:** Chemical Mechanical Planarization (the process that flattens each metal layer) polishes faster in sparse areas and slower in dense areas. If density varies too much, you get dishing (over-polish in sparse areas) and erosion (excess material in dense areas).
- **DRC density rules:** PDKs specify minimum and maximum density per layer within any given window (e.g. M2 density must be between 20% and 80% in any 100×100µm window).
- An inverter with IO pads leaves large empty M2/M3 areas → density violations → dummy fill required.

### How to Generate (Calibre nmDRC Custom Mode)

1. Calibre → Run nmDRC from layout
2. **Custom tab** → DRC Run Options: `DUMMY_FILL` → check all dummy layers to generate
3. **Rules tab** → set run directory
4. **Inputs tab** → Run Options: `FLAT` (flatten hierarchy for fill calculation)
5. **Outputs tab** → Format: GDSII, File: `<cell_name>__dummy.gds` (double underscore convention)
6. **Settings → Show Pages → Options** → Options tab → Upper Limit: `All`
7. Run DRC

This does not produce DRC violations — it produces a GDS file containing the fill geometry.

### Integrating Dummy Fill

1. Stream in the `__dummy.gds` into a new library (e.g. `DUMMY_FILL`) attached to `ts018_scl_prim`
2. Create a new assembly cell `final_chip_withdummy` in `project0`
3. Instantiate both `project0/final_chip/layout` and `DUMMY_FILL/final_chip/layout` at origin (0,0)
4. Select both → Q → set Origin X=0, Y=0 to ensure perfect alignment

### Naming Conflict Warning
If both `project0` and `DUMMY_FILL` libraries contain a cell named `final_chip`, GDS stream-out will rename one to `final_chip_0`. This is harmless — the hierarchy is preserved. Just be aware the top-level GDS may show `final_chip_0` for the dummy fill instance.

### LVS on Assembly Cell
**Do not run LVS on `final_chip_withdummy`.** Seal ring and dummy fill are physical-only structures with no schematic representation. LVS will fail because it cannot match the extra nets/ports introduced by these physical cells.

Always run LVS on `final_chip` (the electrical cell) directly.

---

## Workflow Summary

```
final_chip layout (Day 5)
    ↓
Add seal ring instance → connect to VSS via TOP_M
    ↓
Run Calibre DUMMY_FILL → get __dummy.gds
    ↓
Stream __dummy.gds into DUMMY_FILL library
    ↓
Create final_chip_withdummy → instantiate both at (0,0)
    ↓
DRC / ARC on final_chip_withdummy
LVS on final_chip (not withdummy)
    ↓
GDS stream-out of final_chip_withdummy
```
