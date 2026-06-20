# CDL Netlist — Export & Manual Editing

## What is CDL?

CDL (Circuit Description Language) is a SPICE-like netlist format used by Calibre LVS as the **source netlist** (schematic side). It describes the circuit in terms of subcircuits, instances, and connections.

## Exporting CDL from Virtuoso

`CIW → File → Export → CDL`

Settings:
| Field | Value |
|---|---|
| Library | your library (e.g. `project0`) |
| Cell | top cell name (e.g. `IOPAD`) |
| View | `schematic` |
| Output Netlist File | `<cellname>.cdl` |
| Run Directory | full path to output folder |
| Netlisting Mode | **Analog** |
| Scale | **meter** |

## The Stub Problem

When Cadence exports CDL for a schematic that uses **PDK library symbols**, it generates empty stub subcircuits for those cells. A stub looks like:

```spice
.SUBCKT pvda VDD VDDO VSS VSSO
*.PININFO VDD:B VDDO:B VSS:B VSSO:B
.ENDS
```

This has no transistor content — just pin info. Calibre LVS cannot match this against the layout, which has the full transistor-level geometry.

## Fix: Replace Stubs with Full PDK Definitions

1. Open the PDK's reference CDL file:
   ```
   /home/install/SCL/scl180/iopad/cio150/4M1L/cdl/tsl18cio150.cdl
   ```

2. Find the full subcircuit definition for each pad cell used (pvdi, pv0i, pvda, pv0a, pc3d00)

3. In your generated CDL file:
   - **Comment out** all stub definitions by adding `*` at the start of every line
   - **Paste** the full PDK definitions at the top of the file (after `.PARAM`)

4. Save the file

## CDL File Structure (after editing)

```
* auCdl Netlist header
.PARAM

.SUBCKT pvda VDDO VDD VSS VSSO       ← full PDK definition
[transistor content]
.ENDS pvda

.SUBCKT pvdi VDD VSS VDDO VSSO
[transistor content]
.ENDS pvdi

... (all 5 pad cells) ...

****************                      ← commented out stubs
*.SUBCKT pvda VDD VDDO VSS VSSO
*.PININFO ...
*.ENDS
...

****************                      ← your top cell (untouched)
.SUBCKT IOPAD EXT_vin EXT_vout PADR_vout VDD VDDO VSS VSSO vin
*.PININFO ...
Xpvdi VDD VSS VDDO VSSO / pvdi
...
.ENDS
```

## Pin Order — Critical for LVS

CDL is **positional** — pin connections are assigned by position, not by name. If the order in an instance call doesn't match the order in the subcircuit definition, LVS sees wrong connections.

**Example of a mismatch:**
```spice
* PDK definition:
.SUBCKT pc3d00 PAD PADR VDD VSS VDDO VSSO

* Wrong instance call:
XI0 EXT_vout PADR_vout VDD VDDO VSS VSSO / pc3d00
*                              ^^^^ VDDO and VSS swapped!

* Correct instance call:
XI0 EXT_vout PADR_vout VDD VSS VDDO VSSO / pc3d00
```

Always verify instance pin order against the PDK subcircuit header before running LVS.

## Comments in CDL

In CDL/SPICE format, `*` at the start of a line = comment. Used to:
- Comment out stub definitions
- Add notes
- Note: `.PININFO` lines are always commented (`*.PININFO`) — this is intentional, they are metadata for Cadence tools only
