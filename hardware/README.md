# KiCAD project

The interface board itself. **The design is documented in
[`../docs/pcb-specification.md`](../docs/pcb-specification.md)** — circuit,
BOM, rationale and the v1.0 → v1.1 change log. This file covers only the
mechanics of working with the KiCAD project.

## Status

| Stage | State |
|---|---|
| Schematic | ERC **0 errors, 0 warnings** |
| Netlist | Verified against intent (8 nets) |
| Placement | No courtyard overlaps |
| Routing | 68 segments, 6 vias, **0 unconnected** |
| DRC | **0 errors** (5 cosmetic silkscreen warnings) |
| Fabrication outputs | Gerbers, drill, BOM, PDFs in `output/` |

The silkscreen warnings are U2's outline crossing the board edge, which is the
deliberate antenna overhang, plus reference designators overlapping the
connector labels.

## Layout

```
hardware/
  cistern-level-interface.kicad_pro   project (also carries the DRC floor)
  cistern-level-interface.kicad_sch   schematic
  cistern-level-interface.kicad_pcb   board
  fp-lib-table, sym-lib-table         point at lib/ — needed for the custom parts
  lib/cistern.kicad_sym               ADS1115_Breakout, MP1584_Module symbols
  lib/cistern.pretty/                 MP1584EN_Buck_Module footprint
  output/                             generated: Gerbers, drill, BOM, PDFs, render
```

`lib/cistern.pretty/Fuse_Bourns_MF-R025_Radial.kicad_mod` is unused in v1.1. It
is kept because the protected variant in the change log would need it.

## Custom parts

Both module footprints were built from the physical hardware, not datasheets:

- **MP1584EN** — 22.3 × 16.8 mm, four corner connections each doubled on
  2.54 mm pitch (8 holes), columns 19.05 mm apart. Top view: IN left, OUT
  right, `+` is the lower hole of each pair. Pad pitch still wants a caliper
  check before ordering.
- **ADS1115 (KY-053)** — plain 1×10 header on 2.54 mm, order VDD, GND, SCL,
  SDA, ADDR, ALRT, A0, A1, A2, A3. Confirmed against the module.

## Regenerating outputs

```bash
kicad-cli sch export pdf -o output/cistern-level-interface-schematic.pdf cistern-level-interface.kicad_sch
kicad-cli pcb export gerbers -o output/gerbers cistern-level-interface.kicad_pcb
kicad-cli pcb export drill -o output/gerbers/ cistern-level-interface.kicad_pcb
kicad-cli pcb drc --severity-error cistern-level-interface.kicad_pcb
```

## Re-routing

Routed with the [KiCADRoutingTools](https://github.com/drandyhaas/kicadroutingtools)
plugin, headless:

```bash
python route.py in.kicad_pcb out.kicad_pcb --layers F.Cu B.Cu --clearance 0.2 --track-width 0.25 --via-size 0.6 --via-drill 0.3 --power-nets "*GND*" "*19V*" "*5V*" "*3V3*" --power-nets-widths 0.8 0.8 0.5 0.5
```

Two rules from the plugin's own docs that are easy to get wrong:

- Copy boards with its `copy_board.py`, not `cp`. A bare copy strands the
  sibling `.kicad_pro` that carries the DRC floor, and KiCAD then grades
  correct copper as phantom violations.
- Route to a **fresh** output path each run. Re-using one makes `route.py` read
  back its own `.kicad_pro` and silently change the result.

Verify with all three — they check different things:

```bash
python check_connected.py out.kicad_pcb
python check_drc.py out.kicad_pcb --clearance 0.2 --clearance-margin 0.1
kicad-cli pcb drc --severity-error out.kicad_pcb
```

A DRC-clean board can be entirely disconnected; isolated copper has no
clearance conflicts.
