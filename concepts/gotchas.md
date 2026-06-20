# Day 03 Gotchas and Tips

## 1. PDK Path — Manual Placeholder is Wrong
Manual says: `/home/<pdk_directory>/scl180/iopad/...`  
Actual path on NIT server: `/home/install/SCL/scl180/iopad/...`

Always check `/home/install/SCL/` for SCL PDK files on vlsilab16.

## 2. find command — 2>/dev/null Syntax Issue
On the lab server, the following fails:
```bash
find / -name "file*" 2>/dev/null   # FAILS — space before 2 causes parse error
```
Fix — no space:
```bash
find / -name "file*" 2>/dev/null   # make sure no space between 2 and >
```
Or just let errors show:
```bash
find / -name "file*"
```

## 3. Pad Cell Names Differ from Manual
Manual says: pvd, pvda, pvG, pvGa, pc3d00  
Actual names in CDL: **pvdi, pv0i, pvda, pv0a, pc3d00**

Always grep the CDL file to confirm cell names:
```bash
grep -i "\.subckt" /home/install/SCL/scl180/iopad/cio150/4M1L/cdl/tsl18cio150.cdl | awk '{print $2}' | sort
```

## 4. Pin Signal Type — Keep as "signal" not "ground"
When creating pad schematic pins in Virtuoso, set all pins to:
- Direction: inputOutput
- Signal Type: **signal** (NOT ground, even for VSS)

Setting VSS as "ground" type causes issues when instantiated in top level.

## 5. Generate All From Source — Pins Placed at Origin
After generating layout from schematic, all IO pins are auto-placed near the origin (0,0) as a cluster. They need to be manually moved (`M`) to the correct M3 label locations inside each pad cell.

## 6. Finding M3 Labels Inside Pad Cells
The streamed-in GDS cells contain M3(label) markers showing where each power net (VDD, VDDO, VSS, VSSO) is accessible. Click on elements inside the pad cell and press `Q` to check layer — look for `Layer: M3(label)` with the correct net name. Place your M3(pin) there.

## 7. Pin vs Text Label in Layout
- **Create → Label** or pressing `L` on a wire = just a text label, NOT an electrical pin
- **Create → Pin** = actual electrical pin recognized by LVS/Calibre
Always use Create → Pin for layout pins, not labels.

## 8. Undo Sometimes Doesn't Work in Layout
Ctrl+Z may not work after certain operations in Virtuoso Layout XL. Save frequently with Ctrl+S. If something goes wrong, close without saving and reopen.

## 9. Layout Suite L vs XL
- Layout Suite **L** = limited, no "Generate All From Source" in Connectivity menu
- Layout Suite **XL** = full featured, has Connectivity → Generate → All From Source
The lab opens XL by default but check the title bar to confirm.

## 10. DRC P.1 Errors in IO Pad Cells
P.1 (Pattern digitized grid) errors are common in IO pad GDS cells and are usually **waivable** — they come from the foundry GDS, not from your design. Show to Anand for confirmation before trying to fix them.
