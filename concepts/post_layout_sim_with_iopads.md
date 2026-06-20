# Post-Layout Simulation with IO Pads (SCL 180nm)

Post-layout simulation of a chip with IO pads is more involved than a bare inverter simulation because:
1. The PEX netlist includes IO pad subcircuits (`pvdi`, `pv0i`, `pvda`, `pv0a`, `pc3d00`)
2. These pads contain internal protection devices (resistors, diodes) that need their own model definitions
3. The testbench needs to drive and observe through pad ports, not direct core ports

---

## CDF Setup (Before Simulation)

Before the testbench can use the `final_chip` spectre view, the CDF must be configured:

**CIW → Tools → CDF → Edit**
- Scope: Cell | CDF Layer: Base
- Library: `project0`, Cell: `final_chip`

**Component Parameter tab:**
- Add entry: Name=`model`, Type=`string`, Parse as CEL=`yes`
- Click Apply

**Simulation Information tab:**
- By Simulator → choose `spectre`
- termOrder: copy port order exactly from the `.subckt` line in `final_chip.pex.netlist`
  - Check with: `grep "subckt final_chip" /path/to/final_chip.pex.netlist`
- otherParameters: `model`
- Click Apply → OK

**CDF must be re-done every time PEX is re-run** (port order can change).

---

## Spectre View Creation

The testbench instantiates `final_chip` via a `spectre` cellview. Create it by copying the symbol:

Library Manager → `final_chip` → right-click `symbol` → Copy → To View: `spectre`

This is a Cadence convention — the spectre cellview tells ADE which cell to use during simulation. The actual netlist content comes from the PEX netlist loaded in Model Libraries.

---

## Testbench Setup (`tb_final`)

Copy an existing testbench (e.g. `inverter_tb`) and replace the inverter instance:
- Instance: `project0 / final_chip / spectre`
- Set `model` CDF parameter to `final_chip` in instance properties

**Port connections for a chip-level testbench:**

| Port | Connect to |
|------|-----------|
| `vin` | Pulse voltage source (0→1.8V) |
| `EXT_vin` | Same pulse source (both input pin types) |
| `VDD` | 1.8V DC supply |
| `VDDO` | 1.8V DC supply (IO ring power) |
| `VSS` | GND |
| `VSSO` | GND (IO ring ground) |
| `EXT_vout` | Output observation node → load cap → GND |
| `PADR_vout` | Output observation node (connect to same output net) |

---

## Model Libraries for IO Pad Simulation

In ADE-L → **Setup → Model Libraries**, the following must all be included:

| File | Section | Purpose |
|------|---------|---------|
| `final_chip.pex.netlist` | (none) | PEX-extracted parasitic netlist |
| `ts18scl/default/hspice/ts18sl_scl.lib` | `tt_18` | Core MOSFET models (1.8V) |
| `ts18scl/default/hspice/ts18sl_scl.lib` | `res2t_typ` | **IO pad resistor models** ← critical |
| `ts18scl/v2.0/hspice/ts18sl_scl.lib` | `tt_hv` | High-voltage MOSFET models (IO pads) |
| `ts18scl/v2.0/hspice/ts18sl_scl.lib` | `diodes` | IO pad diode models |
| `ts18scl/v2.0/hspice/ts18sl_scl.lib` | `acc_typ` | Accumulation device models |

**Full paths on NIT Calicut server:**
```
/home/install/SCL/scl180/pdk/cdns/sclpdk_v3/HOTCODE/models/ts18scl/default/hspice/ts18sl_scl.lib
/home/install/SCL/scl180/pdk/cdns/sclpdk_v3/HOTCODE/models/ts18scl/v2.0/hspice/ts18sl_scl.lib
```

---

## The `res2t_typ` Fix — Most Common Error

**Error message:**
```
ERROR (SFE-23): The instance `XXI0/Xpc3d00/R15' is referencing an undefined model 
or subcircuit, `rnwellsti2t'.
```

**Cause:** SCL cio150 IO pad cells contain built-in protection resistors:
- `rnwellsti2t` — N-well STI resistor
- `rnmpoly2t` — N+ polysilicon resistor
- `rphpoly2t` — P+ polysilicon resistor

These are defined in the `res2t_typ` section of `ts18sl_scl.lib`. This section exists only in the **`default`** path version of the library, not in `v2.0`.

**Fix:** Add `ts18scl/default/hspice/ts18sl_scl.lib` with section `res2t_typ` to Model Libraries.

To verify the section exists:
```bash
grep -n "rnwellsti2t" /home/install/SCL/scl180/pdk/cdns/sclpdk_v3/HOTCODE/models/ts18scl/default/hspice/ts18sl_scl.lib
```

---

## What the Waveform Should Look Like

Post-layout simulation of `final_chip` (CMOS inverter with IO ring) shows:
- Input (vin): clean 0→1.8V pulse
- Output (EXT_vout / PADR_vout): inverted signal with **smooth edges** due to IO pad capacitance

The rounding is more pronounced than the bare inverter post-layout sim because the IO pad ESD protection diodes and input resistors add significant RC delay. This is expected and realistic — it's what the chip will actually behave like on silicon.

---

## ADE Environment Options

Under **Setup → Environment**:
- Switch View List: `spectre cmos_sch cmos.sch schematic verilog`
- Stop View List: `spectre`

This tells ADE to stop at the `spectre` cellview (which links to the PEX netlist) rather than descending into the schematic. If Stop View List doesn't include `spectre`, ADE may try to use the schematic view instead of the extracted netlist.
