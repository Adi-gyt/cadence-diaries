# Cadence Virtuoso Workflow

## Access Setup
- Remote server at NIT Calicut
- Connect via **MobaXterm** with X11 forwarding enabled
- Launch with: `virtuoso &` (in the right directory with `.cdsinit` / `cds.lib`)

## Key Windows
| Window | Purpose |
|--------|---------|
| CIW (Command Interpreter Window) | Main console; errors/warnings show here. Keep open always. |
| Library Manager | Browse libraries, cells, views |
| Schematic Editor | Draw schematics |
| Symbol Editor | Create/edit cell symbols |
| Layout Editor | Draw physical layout |
| ADE L | Run simulations |
| Waveform Viewer | View simulation results |

## Cell View Types
- **schematic** — circuit diagram
- **symbol** — abstracted block (for use in testbenches)
- **layout** — physical mask drawing
- **av_extracted** — post-layout extracted view (with parasitics)

## Schematic Flow
1. Library Manager → New Cell → schematic view
2. Place instances (`I` key or Add → Instance)
3. Wire connections (`W` key)
4. Add pins (`P` key)
5. Save (`Ctrl+S`)
6. Check & Save → verify no errors

## Symbol Creation
- From schematic: `Create → Cellview → From Cellview`
- Or manually: open symbol editor, place pins, draw body

## ADE L Simulation Flow
1. From schematic: `Launch → ADE L`
2. Setup → Model Libraries → add PDK model file (`.spi` or `.lib`)
3. Setup → Analyses → choose tran / dc / ac
4. Outputs → To Be Plotted → select nets
5. `Shift+R` or Run button → simulate
6. Results → Direct Plot or open in Waveform Viewer

## Common Analyses
| Analysis | Use |
|----------|-----|
| `tran` | Transient — time-domain signals |
| `dc` | DC sweep — VTC, operating point |
| `ac` | AC sweep — frequency response, gain, BW |

## SCL 180nm PDK Notes
- Standard cell devices: `nmos2v`, `pmos2v` (1.8V nominal)
- Also has `nmos33`, `pmos33` (3.3V devices)
- Model file typically at: `/path/to/scl180nm/models/...` (ask Anand for exact path on NIT server)
- Supply voltage: **VDD = 1.8V**

## Useful Shortcuts
| Key | Action |
|-----|--------|
| `I` | Add Instance |
| `W` | Wire |
| `P` | Pin |
| `Q` | Properties |
| `U` | Undo |
| `Ctrl+S` | Save |
| `F` | Fit view |
| `Shift+R` | Run simulation (ADE) |
