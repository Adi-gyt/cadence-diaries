# Physical Verification
**Context:** Calibre nmDRC / nmLVS / xRC v2022.4, SCL 180nm PDK

---

## Overview

Physical verification is the process of confirming that a layout is both geometrically correct and electrically equivalent to the schematic, and then extracting a realistic netlist for simulation. The standard flow is:

```
Layout → DRC → LVS → PEX → Post-Layout Simulation
```

All three steps use **Calibre** (Siemens EDA) as the verification engine, invoked from Virtuoso via `Calibre → Run DRC / Run LVS / Run PEX`.

---

## DRC — Design Rule Check

**What it does:** Checks every shape in the layout against the PDK's physical design rules — minimum width, minimum spacing, enclosure rules, well rules, density rules etc.

**What it does NOT do:** Has no knowledge of the circuit. It only checks geometry.

**Tool:** Calibre nmDRC
**Rule file:** `_DRC_rules_` (SCL DRC Check ver.00_00_01 for SCL 180nm)

**Key outputs:**
- `inverter.drc.results` — database of violations
- `inverter.drc.summary` — human-readable summary

**Result to aim for:** 0 violations across all rulechecks.

**Common violations on a first run:**
- Spacing between active regions too small
- N-well not enclosing PMOS active by required amount
- Metal width below minimum
- Missing or incorrectly sized contacts

---

## LVS — Layout vs Schematic

**What it does:** Extracts a netlist from the layout by recognizing devices (transistors, resistors etc.) from their geometry, then compares this extracted netlist against the schematic netlist.

**What it checks:**
- Same number of ports, nets, and instances
- Same connectivity between devices
- Same device types and parameters

**Tool:** Calibre nmLVS

**Key outputs:**
- `inverter.lvs.report` — detailed comparison report
- Calibre RVE window — visual discrepancy navigator

**Result to aim for:** CORRECT — all ports, nets, and instances match.

**Common failures:**
- Wrong pin names in layout (e.g. `/NSS/` instead of `/VSS/`)
- Floating nets — a connection drawn in schematic not routed in layout
- Short circuits — two nets accidentally connected in layout
- Missing body tap connections

---

## PEX — Parasitic Extraction

**What it does:** Goes beyond LVS. After confirming the layout is correct, PEX extracts the parasitic resistances (R) and capacitances (C) introduced by the physical geometry — metal routing, via resistance, coupling capacitance between wires etc.

**Tool:** Calibre xRC

**Key outputs:**
- `inverter.pex.netlist` — SPICE netlist with parasitics included
- `inverter.pex.netlist.inverter.pxi` — include file with instance parameters

**Result to aim for:** xRC Errors = 0.

**Warnings to ignore:**
- Undefined ground layer names (isp, SNU, DNWELL) — these are missing PDK definitions on the server, not actual errors.

---

## Post-Layout Simulation

After PEX, the extracted netlist is loaded into ADE L alongside the original model libraries. Running simulation with this netlist gives results that account for parasitic R and C.

**Setup in ADE L:**
1. Session → Load State (load previous schematic sim state)
2. Setup → Model Libraries → add `inverter.pex.netlist`
3. Run simulation

**CDF termOrder:** Must match the pin order in the PEX netlist exactly.
Check pin order from the netlist: look for the `subckt` line, e.g.:
```
subckt inverter ( VIN VSS vdd VOUT )
```
Set CDF termOrder to: `"VIN" "VSS" "vdd" "VOUT"`

---

## Pre vs Post Layout — What Changes

| Parameter | Pre-Layout | Post-Layout |
|-----------|------------|-------------|
| Parasitics | Ideal (none) | Realistic RC included |
| Edge shape | Sharp switching | Slight rounding |
| Propagation delay | Faster (optimistic) | Slightly slower (realistic) |
| DC VTC | Same | Same (DC not affected much) |

For a minimum-size inverter at 180nm, the difference is small. For larger circuits with long routing, parasitics can significantly degrade performance.

---

## Output Directory Structure

```
inverter/layout/
├── drc/
│   ├── inverter.drc.results
│   └── inverter.drc.summary
├── lvs/
│   ├── inverter.lvs.report
│   └── inverter.lvs.report.ext
├── pex/
│   ├── inverter.pex.netlist
│   └── inverter.pex.netlist.inverter.pxi
└── arc/
```

---

## Config View

The config view (`inverter_tb_1 config`) allows switching between schematic and extracted netlist without changing the testbench. In the Hierarchy Editor:
- Set inverter's **View To Use** to `pex netlist` → post-layout sim
- Set it back to `schematic` → pre-layout sim

Useful for direct comparison within the same ADE session.
