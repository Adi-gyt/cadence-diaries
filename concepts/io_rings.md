# Concept: IO Rings

## What is an IO Ring?
An IO ring is a ring of interface cells placed around the **periphery** of a chip die. The core logic sits in the center, and IO cells surround it on all four sides — acting as translators and protectors between the delicate core and the outside world.

```
External World → Bond Pad → IO Ring Cell → Core Logic
```

## Why IO Rings are Needed

### 1. Drive Strength
Core logic operates at µA-level currents. External loads (PCB traces, connectors) need mA-level drive. IO drivers provide this boost.

### 2. Voltage Translation
Core may run at 1.8V internally, but external interfaces may expect 3.3V or 5V. IO cells handle level shifting.

### 3. ESD Protection
The first thing an external pin sees. IO ring cells have clamping diodes and protection structures to absorb electrostatic discharge.

### 4. Signal Conditioning
Schmitt triggers for noisy inputs, slew rate control for outputs (reduces EMI).

### 5. Bidirectional Capability
IO cells can be configured as input, output, or tristate (high-Z).

## Why "Ring"
Cells are arranged in a **continuous ring** so:
- Power and ground rails (VDD, VSS, VDDO, VSSO) run continuously with no breaks
- Corner cells complete the ring at four corners
- Any gap in the ring breaks the power connection → DRC/LVS failure

## SCL 180nm IO Pad Families
| Library | Voltage | Notes |
|---------|---------|-------|
| cio150  | 1.8V    | Core voltage, matches SCL 180nm core |
| cio250  | 3.3V    | For external 3.3V interfaces |
| pio520  | 5–6V    | High voltage tolerant |

## Power Nets in IO Ring
| Net | Meaning |
|-----|---------|
| VDD | Core supply voltage |
| VSS | Core ground |
| VDDO | IO ring supply voltage |
| VSSO | IO ring ground |

pvdi/pvda supply VDD/VDDO. pv0i/pv0a supply VSS/VSSO.

## Cell Types in cio150
| Cell Type | Example | Purpose |
|-----------|---------|---------|
| Power pad (VDD) | pvdi, pvda | Supply VDD/VDDO to ring |
| Power pad (GND) | pv0i, pv0a | Supply VSS/VSSO to ring |
| Analog IO pad | pc3d00 | Signal pad without buffer |
| Corner cell | pfrelr | Completes ring at corners |
| Filler cell | pfeed30000 | Fills gaps, maintains power continuity |

## pc3d00 Pin Description
| Pin | Side | Purpose |
|-----|------|---------|
| PAD | Outer (bond pad side) | Physical bond pad — connects to package pin |
| PADR | Inner (core side) | Signal connection to core logic |
| VDD/VSS | — | Core power |
| VDDO/VSSO | — | IO ring power |

## Important Rules
- Must use all 4 power pads (pvdi, pvda, pv0i, pv0a) whenever any p* IO pad is used
- At least one set of pvdi, pvda, pv0i, pv0a per edge of the ring
- All consecutive cells must be **abutted** — no gaps in the purple PR boundary
- pvdc and pv0c can optionally supply the core if core supply voltage differs from IO ring

## Abutting Cells
Press `A` in Virtuoso layout → select corner of one cell's purple border → select corner of adjacent cell → verify borders overlap exactly with no gap.

Any gap = break in power ring = DRC errors (CS spacing, P.1 grid errors).
