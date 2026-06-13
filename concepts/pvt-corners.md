# PVT Corner Analysis

## What is PVT?
PVT = **Process, Voltage, Temperature**

Real chips don't behave exactly as simulated. Manufacturing variations and operating conditions affect circuit performance. PVT analysis checks if your circuit works across all realistic conditions.

## Process Corners

During fabrication, transistor parameters (VTH, mobility, oxide thickness) vary slightly wafer to wafer. We model this with corners:

| Corner | NMOS | PMOS | Meaning |
|--------|------|------|---------|
| TT (tt_18) | Typical | Typical | Nominal, ideal |
| FF (ff_18) | Fast | Fast | Thin oxide, low VTH — transistors switch faster |
| SS (ss_18) | Slow | Slow | Thick oxide, high VTH — transistors switch slower |
| FS (fs_18) | Fast | Slow | NMOS fast, PMOS slow |
| SF (sf_18) | Slow | Fast | NMOS slow, PMOS fast |

In SCL 180nm, corners are defined in `ts18sl_scl.lib`.

## Temperature Effect
- Higher temperature → lower carrier mobility → **slower switching**
- Lower temperature → higher mobility → **faster switching**
- Typical range tested: **-40°C to 120°C**

## Worst Cases for Inverter Delay
- **Worst case slow** (max delay): SS corner, high temperature (120°C)
- **Best case fast** (min delay): FF corner, low temperature (-40°C)

## Results from Inverter Simulation
| Metric | Nominal (TT, 27°C) | Best case | Worst case |
|--------|-------------------|-----------|------------|
| tpLH | 4.286ns | 3.162ns (FF,-40°C) | 6.171ns (SS,120°C) |
| tpHL | 3.276ns | 2.586ns (FF,-40°C) | 4.523ns (SS,120°C) |

## How to Run in ADE XL
1. Launch → ADE XL (from schematic)
2. Setup corners → load `.lib` file, select TT/FF/SS/FS/SF
3. Add temperature sweep: -40, 27, 120
4. Add outputs: tpLH, tpHL (using calculator expressions)
5. Run → Results tab shows table across all corners

## Why It Matters
- Chip must **meet timing specs at worst-case (SS, hot)**
- Power must be within limits at **best-case (FF, cold)** — fastest switching = most dynamic power
- FS/SF corners stress asymmetric circuits (like inverter with mismatched PMOS/NMOS)
