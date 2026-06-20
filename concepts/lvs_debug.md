# LVS Debugging — Reading RVE & Fixing Mismatches

## What LVS Does

Calibre nmLVS extracts a netlist from your layout (layout netlist) and compares it against your schematic CDL (source netlist). If they match electrically, result is **CORRECT**. If not, it reports discrepancies.

## Running LVS

`Virtuoso Layout → Calibre → Run nmLVS`

Key inputs:
- **Layout:** GDS/OASIS streamed from Virtuoso (or direct)
- **Source:** your edited CDL file
- Rule file: `_LVS.header_` (auto-filled)

## Reading the RVE Window

After LVS runs, click **Show RVE**. The Comparison Results tab shows:

```
Layout Cell / Type | Source Cell | Nets      | Instances  | Ports
IOPAD              | IOPAD       | 22L, 22S  | 36L, 36S   | 8L, 8S
```

- **L** = Layout count, **S** = Source (schematic) count
- All matching = ✅ CORRECT
- Any mismatch = ❌ INCORRECT

## Discrepancy Categories

| Category | Meaning |
|---|---|
| Incorrect Ports | Pin missing or extra in layout vs schematic |
| Incorrect Nets | Net connectivity doesn't match |
| Incorrect Instances | Device/subcircuit count or type mismatch |
| Property Errors | W/L or other device parameters don't match |
| Unmatched Objects | Instance exists in one side but not the other |

## Common Issues & Fixes

### Missing Port
```
WARNING: Unattached label "VSSO" at location (343.745, 156.405) on layer 112
WARNING: Unattached port "VSSO" at location (...) in cell "IOPAD"
```
**Cause:** Pin label placed in layout but not connected to any metal shape, or placed inside a subcell instead of at the top level.

**Fix:** Go to top-level layout, place pin on the correct metal shape. Make sure you're editing the top cell (check title bar) not inside an instance.

### Wrong Pin Count (Instances)
```
Instances: 36L, 42S (-6)
```
**Cause:** Pin order mismatch in CDL causes LVS to see different subcircuits than expected. Some instances get matched incorrectly, others are unmatched.

**Fix:** Correct pin order in CDL instance calls to match PDK subcircuit definitions exactly.

### Unmatched Objects
```
LAYOUT: X21/R0 (435,130,206,770) R(RW)    ** unmatched instance **
SOURCE: ** unmatched instance **            Xpv0a/R2 R(RW)
```
**Cause:** A resistor inside `pv0a` subcircuit isn't being matched — usually because the pin order mismatch caused the whole subcircuit to be matched incorrectly.

**Fix:** Same as above — fix pin order for `pv0a` in CDL.

## Workflow for Debugging LVS

1. Run LVS → get INCORRECT
2. Open RVE → Comparison Results tab
3. Check the summary row first: Ports / Nets / Instances counts
4. Expand **Discrepancies** tree
5. Click each discrepancy → bottom panel shows details
6. Fix the issue in layout or CDL
7. Re-run LVS
8. Repeat until CORRECT

## Tips

- Fix **port errors first** — they cascade into net and instance errors
- **Pin order errors** in CDL cause massive instance mismatches — always check this if you see large instance count differences
- The **Unmatched Objects** section is very useful — it shows exactly which device on each side has no match
- Highlighting in RVE (click discrepancy → H key) zooms to the error in layout — useful but sometimes doesn't work; read the bottom panel text instead
- `Warning: Ambiguity points were found and resolved arbitrarily` in a CORRECT result is fine — just means some symmetric nets were resolved by the tool

## LVS CORRECT Output

```
Ports:     8    8
Nets:      22   22
Instances: 36   36
           ########
           # CORRECT #
           ########
```
