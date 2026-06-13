# CMOS Inverter

## Circuit

```
        VDD
         |
    [PMOS] ← gate = IN
         |
        OUT
         |
    [NMOS] ← gate = IN
         |
        GND
```

- PMOS: source → VDD, drain → OUT, gate → IN
- NMOS: source → GND, drain → OUT, gate → IN

## Operation

| VIN | PMOS | NMOS | VOUT |
|-----|------|------|------|
| Low (0) | ON (conducting) | OFF | High (VDD) |
| High (VDD) | OFF | ON (conducting) | Low (GND) |

## Voltage Transfer Characteristic (VTC)

Five regions based on VIN:
1. **Region 1** (VIN near 0): PMOS in linear, NMOS off → VOUT = VDD
2. **Region 2**: PMOS in saturation, NMOS in saturation
3. **Region 3** (VIN = VM): Both in saturation — high gain, sharp transition
4. **Region 4**: PMOS in saturation, NMOS in linear
5. **Region 5** (VIN near VDD): PMOS off, NMOS in linear → VOUT = 0

### Key Parameters
- **VM** (Switching Threshold): VIN where VOUT = VIN
  - Ideal: VM = VDD/2
  - VM = (VTHn + VTHp + VDD) / 2 (approximate)
- **VOH** = VDD, **VOL** = GND (rail-to-rail swing — key CMOS advantage)
- **NMH** (Noise Margin High) = VOH − VIH
- **NML** (Noise Margin Low) = VIL − VOL

## Sizing
- To center VM at VDD/2: **(W/L)p / (W/L)n = µn / µp ≈ 2**
- Typical starting point: PMOS W = 2µm, NMOS W = 1µm, both L = 180nm (for SCL 180nm)

## Propagation Delay
- **tpHL**: high-to-low output transition (NMOS discharging CL)
- **tpLH**: low-to-high output transition (PMOS charging CL)
- **tp** = (tpHL + tpLH) / 2
- Reduce by: increasing W, reducing L, reducing load capacitance

## Power Dissipation
- **Static power**: ~0 (no DC path between VDD and GND in steady state)
- **Dynamic power**: P = α · CL · VDD² · f
  - α = activity factor, f = switching frequency

## In SCL 180nm PDK
- Devices: `nmos2v` (NMOS, 1.8V), `pmos2v` (PMOS, 1.8V)
- VDD = 1.8V
- VTHn ≈ 0.5V, VTHp ≈ −0.5V (approximate)
