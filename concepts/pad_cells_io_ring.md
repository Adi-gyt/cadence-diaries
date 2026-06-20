# IO Pad Cells & IO Ring Design

## What is an IO Ring?

The IO ring is a set of pad cells placed around the periphery of a chip. It acts as the interface between the internal core logic and the external world (PCB, bond wires). Every signal going in or out of the chip passes through an IO pad.

## PAD vs PADR

In SCL 180nm `cio150` pad cells, there are two internal terminals:

| Terminal | Full Name | Role |
|---|---|---|
| `PAD` | Bond Pad | Connects to the bond wire / external pin. Output driver side. |
| `PADR` | Pad Receiver | Internal receiver side. Connects to core logic. |

**Rule:**
- Gate **inputs** (signals going INTO the core) → connect to `PADR`
- Signals going OUT from the core → connect to `PAD` (via `EXT_vout`)

So for an inverter chip:
- `vin` (inverter input) ← `PADR_vout` (receiver side of output pad)
- `vout` (inverter output) → `EXT_vout` (PAD side of output pad)

This is because SCL PDK guidelines require gate inputs to always connect to PADR for ESD protection and signal integrity reasons.

## cio150 Pad Cell Types (4M1L)

| Cell | Type | Function |
|---|---|---|
| `pvdi` | Power | VDD input pad (digital) |
| `pv0i` | Power | VSS input pad (digital) |
| `pvda` | Power | VDDO input pad (analog/output supply) |
| `pv0a` | Power | VSSO input pad (analog/output ground) |
| `pc3d00` | Signal | Bidirectional signal pad (input/output) |
| `pfrelr` | Filler | Corner/filler cell — no electrical function, fills gaps |

## GDS Location (NIT Calicut server)
```
/home/install/SCL/scl180/iopad/cio150/4M1L/gds/tsl18cio150_4lm.gds
```

## CDL Reference File
```
/home/install/SCL/scl180/iopad/cio150/4M1L/cdl/tsl18cio150.cdl
```

## Pad Cell Pin Orders (from PDK)

These must be matched exactly when writing CDL instance calls:

| Cell | Pin Order |
|---|---|
| `pvdi` | VDD VSS VDDO VSSO |
| `pv0i` | VSS VDD VDDO VSSO |
| `pvda` | VDDO VDD VSS VSSO |
| `pv0a` | VSSO VDD VSS VDDO |
| `pc3d00` | PAD PADR VDD VSS VDDO VSSO |

## Abutting Rules

IO pad cells must be placed **edge-to-edge with zero gap**. Any misalignment between adjacent pad cells causes:
- Metal spacing violations (M1/M2/M3) at the boundary
- Poly spacing violations (P.1) near routing
- Irregular active area polygons (AA.C violations) from merged diffusion shapes

**Fix:** Use Virtuoso snap-to-grid and align tool (`A`) to ensure perfect abutment. If errors persist, redo the placement cleanly rather than trying to fix individual shapes.

## AA Density Violations

`AA.C` rules check that Active Area (diffusion) covers a minimum percentage of any 100×100µm window across the chip. With only an inverter core + sparse IO ring, density is naturally low.

- These are **waivable** at the IO ring stage
- Resolved later by **dummy fill** (Calibre FILL / nmDRC DUMMY_FILL)
- A warning in the rule file states violations will cause GDS rejection — but dummy fill in the final assembly step resolves this before tapeout
