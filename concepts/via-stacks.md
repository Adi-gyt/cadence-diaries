# Via Stacks in Multi-Layer Metal Routing

## What Is a Via?

A via is a vertical metal connector between two adjacent metal layers. In the SCL 180nm PDK, metal layers are stacked M1 → M2 → M3 → (higher), and vias sit between each adjacent pair.

| Via Type | Connects |
|----------|----------|
| V1 (M1_M2) | M1 ↔ M2 |
| V2 (M2_M3) | M2 ↔ M3 |
| V3 (M3_M4) | M3 ↔ M4 |

---

## The Via Stack Problem

When routing from M1 to M3, you cannot jump directly — you must pass through M2. This requires **two separate vias** and a short M2 segment in between:

```
M3 (power ring)
 |
[V2 — M2_M3 via]
 |
M2 (short segment)
 |
[V1 — M1_M2 via]
 |
M1 (cell pin)
```

**Common mistake:** Placing only the "default" via in Virtuoso inserts just one via type (usually V1). The connection appears visually complete but Calibre LVS extracts an open circuit because M1 and M3 are never actually bridged.

---

## When Does This Come Up?

Power ring connections are the most common scenario. In SCL 180nm IO ring designs:
- Power rings (VDD, VSS, VDDO, VSSO) are routed on **M3**
- Standard cell and inverter supply pins are on **M1**
- Any route from a cell's power pin to the ring needs a full M1→M2→M3 via stack

---

## How to Place a Via Stack in Virtuoso

### Method 1 — Route with layer change (auto-via)
1. Press `W` to start routing
2. Click on the M3 ring to begin
3. Press `-` key to step down a layer (M3 → M2 → M1); Virtuoso inserts vias automatically at each transition
4. Click on the M1 cell pin to finish

### Method 2 — Manual via placement
1. Route M1 segment from cell pin upward
2. `Create → Via` → select `V1_M1M2` → place at end of M1 segment
3. Route short M2 segment upward from via
4. `Create → Via` → select `V2_M2M3` → place at end of M2 segment
5. Route M3 segment to power ring

---

## How to Diagnose a Missing Via

If LVS shows an open circuit on a power net (e.g. "LAYOUT nets VDD and 11 must be connected"):

1. In Calibre RVE → Finder tab → search for the floating net number
2. Check what layer the net lives on (shown under Net Layers)
3. Compare to the layer of the destination net
4. If layers differ and no via exists between them → missing via stack

---

## Key Rule

> Always check what metal layer your destination net is on before routing. If source and destination are not on the same layer, plan the via stack explicitly before drawing wires.
