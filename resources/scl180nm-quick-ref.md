# SCL 180nm PDK — Quick Reference

## Device Names
| Device | Model Name | Max VDS/VGS |
|--------|------------|-------------|
| NMOS (1.8V) | `nmos2v` | 1.8V |
| PMOS (1.8V) | `pmos2v` | 1.8V |
| NMOS (3.3V) | `nmos33` | 3.3V |
| PMOS (3.3V) | `pmos33` | 3.3V |

## Typical Parameters (approximate, 1.8V devices)
| Parameter | NMOS | PMOS |
|-----------|------|------|
| VTH | ~0.5V | ~−0.5V |
| µCox (µA/V²) | ~270 | ~90 |
| Min L | 180nm | 180nm |
| Typical W (inverter) | 1µm | 2µm |

## Supply Voltages
- **1.8V domain:** VDD = 1.8V, VSS = 0V
- **3.3V domain:** VDD = 3.3V, VSS = 0V

## Common Simulation Setups

### Transient (Digital signals)
```
Analysis: tran
Stop time: 100n (adjust based on circuit speed)
Step: 10p
```

### DC Sweep (VTC)
```
Analysis: dc
Sweep variable: VIN source
From: 0, To: 1.8, Step: 1m
```

### AC (Frequency Response)
```
Analysis: ac
Sweep: logarithmic
From: 1, To: 1G (Hz)
Points per decade: 20
```

## Testbench Sources
| Source | Use |
|--------|-----|
| `vpulse` | Digital input: V1=0, V2=VDD, per=T, rise/fall=0.1n |
| `vdc` | Supply rails |
| `vsin` | AC analysis input |
| `idc` | Bias current source |
