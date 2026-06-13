# CMOS Inverter

## Circuit

```
        vdd
         |
    [PMOS] ← gate = VIN  (pmos_18, W=2µm, L=0.18µm)
         |
        VOUT
         |
    [NMOS] ← gate = VIN  (nmos_18, W=1µm, L=0.18µm)
         |
        VSS
```

## Operation

| VIN | PMOS | NMOS | VOUT |
|-----|------|------|------|
| Low (0V) | ON | OFF | High (VDD = 1.8V) |
| High (1.8V) | OFF | ON | Low (0V) |

Rail-to-rail output swing — key CMOS advantage over older logic families.

## Voltage Transfer Characteristic (VTC)

- Sweep VIN from 0 → 1.8V (DC analysis)
- VOUT starts high, transitions sharply, ends low
- **VM** (switching threshold): VIN where VOUT = VIN ≈ 0.9V (ideally VDD/2)
- Sharp transition = high gain in that region

## Sizing
- To center VM at VDD/2: **(W/L)p ≈ 2 × (W/L)n**
- Reason: hole mobility (PMOS) ≈ half electron mobility (NMOS)
- In SCL 180nm: PMOS W=2µm, NMOS W=1µm, both L=0.18µm

## Propagation Delay

Measured at 50% point (VDD/2 = 900mV):

- **tpLH**: output transitions low→high (PMOS charging load cap)
- **tpHL**: output transitions high→low (NMOS discharging load cap)
- **tp** = (tpLH + tpHL) / 2

From simulation (TT corner, 27°C, with load cap):
- tpLH ≈ 4.286ns
- tpHL ≈ 3.276ns

## Testbench Setup
- Input: `vsource` (pulse), 0→1.8V, period ~40ns
- Supply: `vdc` = 1.8V
- Load: `cap` at output (models fanout/wire capacitance)
- Simulations: transient (time domain) + DC sweep (VTC)

## In SCL 180nm PDK
- NMOS device: `nmos_18` (W=1µm, L=0.18µm)
- PMOS device: `pmos_18` (W=2µm, L=0.18µm)
- VDD = 1.8V
- Library: `project0` on NIT Calicut server
