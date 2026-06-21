# CDL Netlist — Export & Manual Editing

## What is CDL?

CDL (Circuit Description Language) is a SPICE-like netlist format used by Calibre LVS as the **source netlist** (schematic side). It describes the circuit as subcircuit definitions with terminal connections.

```
.SUBCKT cellname pin1 pin2 pin3
* internal components
.ENDS cellname
```

---

## Exporting CDL from Virtuoso

`CIW → File → Export → CDL`

| Field | Value |
|---|---|
| Library | your library (e.g. `project0`) |
| Cell | top cell name (e.g. `IOPAD` or `final_chip`) |
| View | `schematic` |
| Output Netlist File | `<cellname>.cdl` |
| Run Directory | full path to output folder |
| Netlisting Mode | **Analog** |
| Scale | **meter** |
| Include File | leave empty |

> **Why Analog mode?** IO pad cells contain analog components (ESD diodes, protection resistors) that standard digital LVS mode doesn't handle correctly.

---

## The Stub Problem

When Cadence exports CDL for a schematic that uses **PDK library symbols**, it generates empty stub subcircuits for those cells. A stub looks like:

```spice
.SUBCKT pvda VDD VDDO VSS VSSO
*.PININFO VDD:B VDDO:B VSS:B VSSO:B
.ENDS
```

This has no transistor content — just pin info. Calibre LVS cannot match this against the layout, which has the full transistor-level geometry.

---

## Fix: Replace Stubs with Full PDK Definitions

1. Back up the exported file first:
   ```bash
   cp IOPAD.cdl IOPAD_backup.cdl
   ```

2. Open in gedit (or any text editor)

3. Open the PDK reference CDL:
   ```
   /home/install/SCL/scl180/iopad/cio150/4M1L/cdl/tsl18cio150.cdl
   ```

4. For each pad cell used (pvdi, pv0i, pvda, pv0a, pc3d00):
   - **Comment out** the stub in your CDL by adding `*` to every line
   - **Copy** the full transistor-level subcircuit from the PDK CDL
   - **Paste** at the top of your file, after the `.PARAM` line

5. Fix pin order in instance calls (see below)

6. Save

---

## CDL File Structure (after editing)

```
* auCdl Netlist header
.PARAM

.SUBCKT pvda VDDO VDD VSS VSSO       ← full PDK definition pasted here
[transistor content]
.ENDS pvda

.SUBCKT pvdi VDD VSS VDDO VSSO
[transistor content]
.ENDS pvdi

... (all 5 pad cells) ...

****************                      ← original stubs commented out
*.SUBCKT pvda VDD VDDO VSS VSSO
*.PININFO ...
*.ENDS
...

****************                      ← your top cell — leave untouched
.SUBCKT IOPAD EXT_vin EXT_vout PADR_vout VDD VDDO VSS VSSO vin
*.PININFO ...
Xpvdi VDD VSS VDDO VSSO / pvdi
...
.ENDS
```

---

## Pin Order — Critical for LVS

CDL is **positional** — pin connections are assigned by position, not by name. A single swap causes LVS to see completely wrong net connections, cascading into large instance count mismatches.

**Correct pin orders from PDK:**

| Cell | Pin Order |
|---|---|
| `pvdi` | VDD VSS VDDO VSSO |
| `pv0i` | VSS VDD VDDO VSSO |
| `pvda` | VDDO VDD VSS VSSO |
| `pv0a` | VSSO VDD VSS VDDO |
| `pc3d00` | PAD PADR VDD VSS VDDO VSSO |

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

---

## W/L Units — Uppercase U

Calibre LVS treats `W=1.0u` and `W=1.0U` differently. The SCL 180nm LVS runset expects **uppercase U**. Lowercase causes property errors even if the value is numerically correct.

```spice
MM1 VOUT VIN VSS VSS N W=1.0U L=0.18U m=1.0   ← correct
MM2 VOUT VIN vdd vdd P W=2.0U L=0.18U m=1.0
```

---

## Watch Out: Orphaned `.SUBCKT` Lines

The PDK CDL contains a rogue line between `pvda` and `pv0a`:
```
.SUBCKT pvdc VDDC VDD VSS VDDO VSSO
```
There is no matching `.ENDS` for this. Calibre silently fails to parse the entire file and reports a misleading error:
```
Source primary cell not found in source database
```

**Fix:** Delete this line entirely from your CDL.

---

## LVS Flow for IO Ring

`Virtuoso Layout → Calibre → Run nmLVS`

- Custom tab: Analog Mode = **Yes**
- Inputs tab: uncheck "Export from source viewer" → manually point to your edited CDL
- Rest of settings same as standard LVS (rule file auto-filled)

---

## Comments in CDL

`*` at the start of a line = comment. Used to:
- Comment out stub definitions
- Add notes
- Note: `.PININFO` lines are always prefixed with `*` (`*.PININFO`) — this is intentional metadata for Cadence tools only, not an error
