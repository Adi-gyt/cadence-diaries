# Concept: CDL Netlisting and LVS for IO Ring

## What is a CDL Netlist?
CDL (Circuit Description Language) is a SPICE-like netlist format used for LVS (Layout vs Schematic) verification. It describes the circuit as subcircuit definitions with terminal connections.

## CDL File Structure
```
.SUBCKT cellname pin1 pin2 pin3
* internal components
.ENDS cellname
```

## Why Manual CDL Editing is Needed for IO Ring
When Virtuoso exports CDL from the IOPAD schematic, it generates individual subcircuit definitions for each pad cell (pvdi, pv0i etc.) based on your schematic. However, these auto-generated definitions are incomplete — they don't have the actual transistor-level content of the IO pad cells.

The fix:
1. Export CDL from Virtuoso schematic
2. Comment out auto-generated pad subckts
3. Copy the real subckt definitions from the PDK CDL file
4. Paste at the top of the exported CDL
5. Fix pin order in the main IOPAD subckt to match PDK order

## Step-by-Step CDL Export
1. CIW → File → Export → CDL
2. Library Browser → select project0, IOPAD, schematic
3. Output Netlist File: `IOPAD.cdl`
4. Run Directory: `/home/intern02_2026/cds/project0/IOPAD/layout/lvs/`
5. Netlisting Mode: **Analog**
6. Scale: **meter**
7. Include File: leave empty
8. Click OK

## CDL Editing Steps
```bash
# Open generated CDL
gedit IOPAD.cdl

# 1. Find and comment all individual pad subckts:
# .SUBCKT pvdi ...   →   *.SUBCKT pvdi ...
# .SUBCKT pv0i ...   →   *.SUBCKT pv0i ...
# etc.

# 2. Copy subckts from PDK CDL:
grep -A 50 "\.SUBCKT pvdi\b" /home/install/SCL/scl180/iopad/cio150/4M1L/cdl/tsl18cio150.cdl
# Paste at top of IOPAD.cdl

# 3. Fix pin order in main IOPAD subckt to match PDK order
```

## LVS Flow for IO Ring
- Tool: **Calibre nmLVS**
- Calibre → Run nmLVS
- Custom tab: Select Analog Mode = **Yes**
- Inputs tab: uncheck "Export from source viewer" → add edited IOPAD.cdl
- Rest same as inverter LVS (Manual-1 section 7.3)

## Key Difference from Standard LVS
Analog mode is needed because IO pad cells have analog components (diodes, resistors for ESD) that standard digital LVS mode doesn't handle correctly.
