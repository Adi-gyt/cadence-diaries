# Layout Fundamentals
**Context:** SCL 180nm PDK, Cadence Virtuoso Layout Suite XL

---

## Layer Stack (SCL 180nm)

| Layer | Purpose |
|-------|---------|
| WN | N-well (defines PMOS region) |
| ACTIVE | Diffusion area (source/drain/channel) |
| GC | Gate Connection — polysilicon gate |
| CS | Contacts (connects active/poly to Metal 1) |
| M1 | Metal 1 — first routing layer |
| XP | P-implant (defines PMOS active) |
| XN | N-implant (defines NMOS active) |

In Virtuoso, each layer has a purpose tag — `drw` (drawing), `pin`, `label` etc. Always draw on the `drw` purpose unless placing pins.

---

## Pcells

The SCL 180nm PDK provides parameterized cells (pcells) for transistors — `pmos_18` and `nmos_18`. These auto-generate the correct geometry (active, gate, contacts, implants) based on the parameters you set.

Key parameters:
- **W (width):** gate width in µm
- **L (length):** gate length in µm (minimum 0.18µm for SCL 180nm)
- **nf (fingers):** number of gate fingers
- **m (multiplier):** number of parallel instances

---

## Fingers vs Multipliers

**Fingers (nf):**
- Splits the gate width into multiple parallel fingers physically
- Does NOT change circuit behaviour — same total W/L
- Safe to modify at layout stage
- Reduces gate resistance by shortening each finger

**Multipliers (m):**
- Creates multiple parallel copies of the entire transistor including diffusion
- Changes the effective diffusion area → changes W/L ratio → changes circuit behaviour
- Must be set during schematic design only, never changed at layout stage

---

## Body Taps

The bulk terminal of every transistor must be connected to its supply to prevent latch-up.

In SCL 180nm pcells, body taps are built in — enable them via:
`Q → Parameter tab → Show Tap Props = Yes`

| Transistor | Tap | Connect to |
|------------|-----|------------|
| pmos_18 | topTap + topTapM1 | VDD |
| nmos_18 | bottomTap + bottomTapM1 | VSS |

`topTapM1` / `bottomTapM1` adds a Metal 1 connection on top of the tap, making it easy to route.

**Why it matters:** Without bulk connections, floating wells can cause latch-up — a parasitic SCR (thyristor) in the substrate can trigger and short VDD to VSS permanently, destroying the circuit.

---

## Abutment

When two adjacent transistors share a terminal (e.g. both drains connect to VOUT):
- Place them with the shared terminal facing each other
- Drag until the terminals overlap
- Virtuoso auto-detects the overlap and merges the node

This eliminates the need for an explicit metal connection between the two drains and saves area.

---

## Minimum Spacing Rules (SCL 180nm)

| Rule | Value |
|------|-------|
| NMOS active to PMOS N-well edge | 0.43µm minimum |
| Gate length (minimum) | 0.18µm |
| M1 width (minimum) | 0.23µm |

Use the ruler (`K`) to measure spacing and align (`A`) to snap to grid. Always verify with DRC after placement.

---

## PR Boundary

The PR (Place and Route) Boundary defines the physical extent of a cell. It is required for:
- Calibre DRC/LVS to know the cell boundary
- Placement in larger designs (the router uses it for abutment between cells)
- Correct GDS export

Draw it as a rectangle on the `prBoundary` layer enclosing all geometry with some margin.

---

## Useful Shortcuts (Virtuoso Layout Editor)

| Key | Action |
|-----|--------|
| K | Ruler — measure distances |
| A | Align selected objects |
| Q | Edit properties of selected object |
| P | Draw path (for routing) |
| R | Draw rectangle |
| Shift+S | Check and Save |
| F | Fit view |
| Z | Zoom in |
