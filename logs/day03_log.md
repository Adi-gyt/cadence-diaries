# Day 03 — IO Ring Design (Manual 2, Pages 1–12)
**Date:** 16 June 2026  
**Intern:** intern02_2026 | NIT Calicut VLSI Lab (vlsilab16)  
**Tool:** Cadence Virtuoso Layout Suite XL | SCL 180nm PDK | Calibre nmDRC

---

## Summary
Worked through Manual-2 up to section 2.4 (Physical Verification). Completed the full IO ring design flow: library setup, pad schematic/symbol creation, top-level IOPAD schematic, IO ring layout, pin placement, and first DRC run.

---

## What I Did

### 1. IO Ring Theory (Section 2.1)
- Understood what IO rings are — interface circuits between core logic and bond pads
- Learned the SCL 180nm IO pad family:
  | Library | Voltage | Use Case |
  |---------|---------|----------|
  | cio150  | 1.8V    | Core voltage IO (used in this design) |
  | cio250  | 3.3V    | General purpose / mixed signal |
  | pio520  | 5–6V    | High voltage tolerant pads |

### 2. Streaming in cio150 Library (Section 2.1)
- Created new library `cio150` attached to tech `ts018_scl_prim`
- CIW → File → Import → Stream
- Filled in Stream In dialog:
  - **Stream File:** `/home/install/SCL/scl180/iopad/cio150/4M1L/gds/tsl18cio150_4lm.gds`
  - **Library:** cio150
  - **View:** layout
  - **Attach Tech Library:** ts018_scl_prim
  - **Layer Map:** `/home/scl_workshop/workshop/cds_master/SCLSL18_4M1L/ts018_scl_prim/ts018_scl_prim.layermap`
  - **Object Map:** `/home/scl_workshop/workshop/cds_master/SCLSL18_4M1L/ts018_scl_prim/ts018_scl_prim.objectmap`
- Clicked Translate — stream completed with 0 errors, 3 warnings (ignorable)

### 3. Identifying Pad Cells (Section 2.2)
- Manual listed: pvd, pvda, pvG, pvGa, pc3d00 — but actual cell names in CDL differ
- Grepped CDL file to find correct names and pin lists:

```bash
grep -i "\.subckt pvdi\b\|\.subckt pv0i\b\|\.subckt pvda\b\|\.subckt pv0a\b\|\.subckt pc3d00\b" \
/home/install/SCL/scl180/iopad/cio150/4M1L/cdl/tsl18cio150.cdl
```

**Pin list extracted:**
| Cell | Pins |
|------|------|
| pvdi | VDD VSS VDDO VSSO |
| pv0i | VSS VDD VDDO VSSO |
| pvda | VDDO VDD VSS VSSO |
| pv0a | VSSO VDD VSS VDDO |
| pc3d00 | PAD PADR VDD VSS VDDO VSSO |

### 4. Creating Pad Schematics and Symbols
- For each of the 5 cells in cio150 library:
  - File → New → Cellview (schematic)
  - Added pins using `P` — all set to **inputOutput**, **signal** type
  - Drew wires connecting pins
  - Check and Save
  - Design → Create Cellview → From Cellview (symbol)
  - Pin arrangement: VDD/VDDO left, VSS/VSSO right, PAD top, PADR bottom (for pc3d00)

### 5. IOPAD Top-Level Schematic (Section 2.2)
- Created new schematic `IOPAD` in project0 library
- Instantiated 6 pad symbols (press `I`):
  - pvda, pvdi (top power pads)
  - pv0a, pv0i (bottom power pads)
  - 2x pc3d00 (left input pad, right output pad)
- Added net labels (`L`): VDD, VDDO, VSS, VSSO on all power pins
- Added schematic pins (`P`) — inputOutput for all:
  - VDD, VDDO, VSS, VSSO (power interface)
  - EXT_vin, vin (left pc3d00 — PAD and PADR)
  - EXT_vout, PADR_vout (right pc3d00 — PAD and PADR)
- Check and Save → created IOPAD symbol

### 6. IO Ring Layout (Section 2.3)
- Created layout view via Connectivity → Generate → All From Source
  - I/O Pins tab: all pins set to M3 pin layer, 0.28 x 0.28 size
- Placed IO cells from cio150 library in ring formation:
  - **Top:** pfrelr (corner), pfeed30000, pvdi, pfeed30000, pvda, pfeed30000, pfrelr (corner)
  - **Left:** pfeed30000, pc3d00 (input), pfeed30000
  - **Right:** pfeed30000, pc3d00 (output), pfeed30000
  - **Bottom:** pfrelr (corner), pfeed30000, pv0a, pfeed30000, pv0i, pfeed30000, pfrelr (corner)
- Abutted all cells using `A` — purple PR boundaries must overlap with no gaps
- **pfrelr** = corner cells | **pfeed30000** = filler cells (30000 width)

### 7. Pin Placement in Layout
- Auto-generated pins were placed near origin — moved each to correct M3 label location
- To move: select pin → `M` → place on M3 label inside respective pad cell
- Verified each pin by pressing `Q` — checked XL Status = OK

**Pin placement results:**
| Pin | Cell | XL Status |
|-----|------|-----------|
| VDD | pvdi/pvda | OK ✅ |
| VDDO | pvda | OK ✅ |
| VSS | pv0i/pv0a | OK ✅ |
| VSSO | pv0a | OK ✅ |
| vin | pc3d00 (left) PADR | OK ✅ |
| EXT_vin | pc3d00 (left) PAD | OK ✅ |
| PADR_vout | pc3d00 (right) PADR | OK ✅ |
| EXT_vout | pc3d00 (right) PAD | OK ✅ |

### 8. DRC Run (Section 2.4)
- Calibre → Run nmDRC
- Custom tab: DRC Run Type = BLOCK
- **Total errors: 1313**
  - P.1: 1000 (grid errors — likely from PDK cells, waivable)
  - CS.W.1: 128, CS.S.1: 150 (spacing)
  - M2.S.2: 6, M3.S.2: 12, ML.S.2: 10
  - DENSITY_PRINT_FILES: 0 ✅
- Anand noted alignment/abutting issues need fixing → to be resolved Day 4

---

## Tomorrow (Day 4)
- [ ] Fix cell abutting errors (main DRC source)
- [ ] Re-run DRC and get clean/waived result
- [ ] Export CDL netlist (CIW → File → Export → CDL)
- [ ] Edit CDL: comment individual pad subckts, copy from tsl18cio150.cdl, fix pin order
- [ ] Run Calibre nmLVS (Analog mode)

---

## Key Server Paths (NIT Calicut specific)
```
GDS:        /home/install/SCL/scl180/iopad/cio150/4M1L/gds/tsl18cio150_4lm.gds
CDL:        /home/install/SCL/scl180/iopad/cio150/4M1L/cdl/tsl18cio150.cdl
Layer Map:  /home/scl_workshop/workshop/cds_master/SCLSL18_4M1L/ts018_scl_prim/ts018_scl_prim.layermap
Object Map: /home/scl_workshop/workshop/cds_master/SCLSL18_4M1L/ts018_scl_prim/ts018_scl_prim.objectmap
FOUNDRY:    /home/install/FOUNDRY/analog/180nm/
```
