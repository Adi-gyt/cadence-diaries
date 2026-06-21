# 🧪 cadence-diaries

> Documenting my analog VLSI internship at NIT Calicut (June–July 2026) under Prof. Dhanaraj.  
> Tools: Cadence Virtuoso · SCL 180nm PDK · Calibre nmDRC/nmLVS/xRC · MobaXterm (X11) · Remote NIT server
> Deliverable: Tapeout-ready GDS (12MB) — CMOS inverter with full IO ring, seal ring, and dummy fill

The manual covers *how* to use Virtuoso. This repo covers *why* things work the way they do.

---

## 📁 Structure

```
cadence-diaries/
├── logs/           # Daily internship logs
├── concepts/       # Theory & concept notes
└── schematics/     # 38 screenshots from Virtuoso & Calibre (embedded in logs) 
```

---

## ✅ Progress

| Task | Status |
|------|--------|
| Inverter schematic (pmos_18 + nmos_18, SCL 180nm) | ✅ |
| Symbol creation | ✅ |
| Testbench with load cap | ✅ |
| Transient simulation | ✅ |
| DC simulation (VTC curve) | ✅ |
| Propagation delay (tpLH, tpHL) | ✅ |
| PVT corner analysis (ADE XL, 12 corners) | ✅ |
| Inverter layout (body taps, M1 routing, PR boundary) | ✅ |
| DRC — inverter (0 violations) | ✅ |
| LVS — inverter (CORRECT) | ✅ |
| PEX — inverter (xRC, 0 errors) | ✅ |
| Post-layout simulation — inverter | ✅ |
| IO ring design (cio150 pad cells, full ring layout) | ✅ |
| CDL export & manual editing (stub fix, pin order) | ✅ |
| DRC — IOPAD (6 waivable AA density errors) | ✅ |
| LVS — IOPAD (CORRECT) | ✅ |
| final_chip schematic + symbol | ✅ |
| final_chip layout (IO ring + inverter core, via stacks) | ✅ |
| DRC — final_chip (6 waivable AA density errors) | ✅ |
| LVS — final_chip (CORRECT) | ✅ |
| PEX — final_chip (xRC, 0 errors) | ✅ |
| Seal ring integration | ✅ |
| Dummy fill (Calibre DUMMY_FILL mode) | ✅ |
| final_chip_withdummy assembly | ✅ |
| DRC/ARC sign-off (4 waivable M2.A.1, 0 antenna violations) | ✅ |
| Post-layout simulation — full chip with IO pads | ✅ |
| GDS stream-out (final_chip_withdummy.gds — 12MB) | ✅ |

---

## 📅 Log Index

| Day | Date | What happened |
|-----|------|--------------|
| — | Jun 11 | Setup — MobaXterm, X11, first look at Virtuoso |
| [Day 1](logs/day01.md) | Jun 12 | Inverter schematic, symbol, testbench, transient, DC, propagation delay, PVT corners |
| [Day 2](logs/day02.md) | Jun 15 | Inverter layout, DRC clean, LVS correct, PEX, post-layout sim |
| [Day 3](logs/day03.md) | Jun 16 | IO ring theory, cio150 stream-in, pad schematics/symbols, IOPAD schematic, IO ring layout, first DRC |
| [Day 4](logs/day04.md) | Jun 17 | IOPAD DRC fixes (1030 → 6 errors), CDL export & editing, LVS CORRECT, final_chip schematic + symbol |
| [Day 5](logs/day05.md) | Jun 18 | final_chip layout, signal routing, via stacks, DRC + LVS CORRECT |
| [Day 6](logs/day06.md) | Jun 19 | PEX, seal ring, dummy fill, final assembly, DRC/ARC/LVS sign-off, post-layout sim, GDS stream-out |

---

## 📖 Concept Notes

| File | Topic |
|------|-------|
| [cmos-inverter.md](concepts/cmos-inverter.md) | CMOS inverter operation, sizing, VTC, propagation delay |
| [pvt-corners.md](concepts/pvt-corners.md) | PVT corner analysis — process, voltage, temperature |
| [layout-fundamentals.md](concepts/layout-fundamentals.md) | Layer stack, pcells, fingers vs multipliers, body taps, abutment |
| [physical-verification.md](concepts/physical-verification.md) | DRC, LVS, PEX flow — what each step does and why |
| [lvs_debug.md](concepts/lvs_debug.md) | Reading Calibre RVE, common LVS failures and fixes |
| [io_rings.md](concepts/io_rings.md) | IO ring theory, pad cell types, power nets, abutting rules |
| [pad_cells_io_ring.md](concepts/pad_cells_io_ring.md) | cio150 pad cells, PAD vs PADR, pin orders, AA density rules |
| [cdl_editing.md](concepts/cdl_editing.md) | CDL export, stub problem, manual editing, pin order |
| [cdl_netlisting.md](concepts/cdl_netlisting.md) | CDL netlisting flow for IO ring LVS |
| [via-stacks.md](concepts/via-stacks.md) | Multi-layer via stacks — when and how to place them |
| [seal_ring_and_dummy_fill.md](concepts/seal_ring_and_dummy_fill.md) | Seal ring sizing, dummy fill generation, final assembly |
| [post_layout_sim_with_iopads.md](concepts/post_layout_sim_with_iopads.md) | Post-layout sim setup for full chip with IO pads |
| [gotchas.md](concepts/gotchas.md) | Common pitfalls, wrong PDK paths, Virtuoso quirks |
| [virtuoso-workflow.md](concepts/virtuoso-workflow.md) | Virtuoso window overview, simulation flow, shortcuts |

---

## 🔧 Setup

- **EDA Tool:** Cadence Virtuoso IC618 (NIT Calicut server, `vlsilab16.nitc.ac.in`)
- **PDK:** SCL 180nm (`ts18sl_scl.lib`) — Semiconductor Complex Limited
- **Verification:** Calibre nmDRC / nmLVS / xRC v2022.4
- **Remote Access:** MobaXterm with X11 forwarding
- **Library:** `project0`
- **IO Pad Library:** `cio150` (1.8V pads, 4M1L process)
