# 🧪 cadence-diaries

> Documenting my analog VLSI internship at NIT Calicut (June–July 2026) under Prof. Dhanaraj.  
> Tools: Cadence Virtuoso · SCL 180nm PDK · MobaXterm (X11) · Remote NIT server

The manual covers *how* to use Virtuoso. This repo covers *why* things work the way they do.

---

## 📁 Structure

```
cadence-diaries/
├── logs/           # Daily internship logs
├── concepts/       # Theory & concept notes
├── schematics/     # Screenshots from Virtuoso
└── resources/      # Cheatsheets, PDK notes
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
| Layout | ✅ |
| DRC | ✅ |
| LVS | ✅ |
| Post-layout simulation | ✅ |

---

## 📅 Log Index

| Date | What happened |
|------|--------------|
| [Jun 11](logs/2026-06-09.md) | Setup — MobaXterm, X11, first look at Virtuoso |
| [Jun 12](logs/2026-06-12.md) | Schematic, symbol, testbench, transient, DC, delay, PVT corners |
| [Jun 15](logs/day02.md) | Layout, DRC clean, LVS correct, PEX, post-layout sim |

---

## 🔧 Setup

- **EDA Tool:** Cadence Virtuoso (NIT Calicut server, `vlsilab16.nitc.ac.in`)
- **PDK:** SCL 180nm (`ts18sl_scl.lib`)
- **Remote Access:** MobaXterm with X11 forwarding
- **Library:** `project0`
