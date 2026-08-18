# Cistern Level Interface — hardware

Carrier board for a 4–20 mA hydrostatic level probe feeding an ESP8266 running
ESPHome. Simplified variant: modules do the work, one precision resistor sets
the accuracy.

**Board:** 70 × 60 mm, 2 layers, 4 × M3 holes 5 mm in from each corner.

## Status

| Stage | State |
|---|---|
| Schematic | Complete — ERC **0 errors, 0 warnings** |
| Netlist | Verified against intent (8 nets) |
| PCB placement | Complete — no courtyard overlaps |
| Ground pour | GND pour on F.Cu and B.Cu |
| Routing | Complete — 68 segments, 6 vias, **0 unconnected** |
| DRC | **0 errors** (5 cosmetic silkscreen warnings) |

Routed with the KiCADRoutingTools plugin (`route.py`), 2 layers, 0.2 mm
clearance ceiling, 0.25 mm signal tracks, 0.8 mm on GND/+19V and 0.5 mm on
+5V/+3V3, 0.6/0.3 mm vias. Verified three ways: the plugin's `check_connected.py`
(all nets fully connected), its `check_drc.py` at the routed 0.2 mm floor (no
violations), and KiCAD's own `kicad-cli pcb drc` (0 violations, 0 unconnected).

To re-route, work on a copy made with `copy_board.py` (a bare `cp` strands the
sibling `.kicad_pro` that carries the DRC floor) and route to a **fresh** output
path each time.

## Bill of materials

| Ref | Value | Notes |
|---|---|---|
| U1 | MP1584EN buck module | **Set output to 5.0 V before fitting U2** |
| U2 | Wemos D1 mini (ESP8266) | On socket headers |
| U3 | Joy-IT KY-053 (ADS1115) | On a 1×10 socket header |
| R1 | 100 Ω 0.1 % ≤50 ppm/K, ≥0.25 W | **Accuracy-critical — metal film, not carbon** |
| R2 | 1 kΩ 1 %, **0402** | Series limit into the ADC input. Non-critical: any tolerance/dielectric is fine (see below) |
| C3 | 100 nF C0G, 0805 | Anti-alias / buck-noise filter |
| J1 | Screw terminal 5.08 mm, 2-pin | 19 V DC in |
| J2 | Screw terminal 5.08 mm, 2-pin | Probe loop |

## Measurement chain

100 Ω burden, ADS1115 at ±4.096 V gain, running from 3V3 so the I²C bus is
natively at ESP level — no level shifter.

| Loop current | Voltage at R1 |
|---|---|
| 0 mA (cable break) | 0.000 V |
| 4 mA (empty) | 0.400 V |
| 20 mA (full) | 2.000 V |

0.000 V is what distinguishes a broken cable from an empty tank — keep that
check in the ESPHome config.

## Before you order boards

1. Both module footprints are now built from the actual hardware:
   - **U1 (MP1584):** 22.3 × 16.8 mm, four corner connections each doubled on
     2.54 mm pitch (8 holes), columns 19.05 mm apart. Viewed from the top:
     **IN left, OUT right, `+` is the lower hole of each pair.**
   - **U3 (KY-053):** plain 1×10 header on 2.54 mm pitch, pin order
     **VDD, GND, SCL, SDA, ADDR, ALRT, A0, A1, A2, A3**. Silkscreen marks the
     VDD and A3 ends.
2. **U3 mounting orientation:** the header runs along one long edge of the
   module and its ~26 × 15 mm body overhangs ~15 mm to *one* side. It must face
   **away from U2** — there is only 7.4 mm of clearance toward the ESP, but
   22 mm on the other side, where it overhangs only R1/R2/C3 (all low-profile,
   fine under a socketed module). Silkscreen says which side.
3. Route the board, then re-run DRC.
3. Confirm your probe's minimum loop compliance is under 17 V (19 V supply
   minus the 2 V burden drop at 20 mA).

## Why R2 is not critical (but R1 is)

R2 feeds the ADS1115's high-impedance input (~6 MΩ), so it carries well under a
microamp and dissipates around a **nanowatt** — a 62.5 mW 0402 is overkill by
roughly seven orders of magnitude. It forms a 1 kΩ : 6 MΩ divider contributing
~0.02 % gain error; a ±1 % tolerance moves that by 0.0002 %, and thick-film
tempco over the shed's temperature swing does nothing measurable. That 0.02 % is
a *systematic gain* error anyway, so the two-point `calibrate_linear` removes it.

**R1 is the opposite.** It is the burden that converts loop current to voltage,
so its tolerance and drift land directly in the reading. It must stay 0.1 %,
≤50 ppm/K metal film. Do not substitute a general-purpose thick-film part there.

## Grounding

The original spec worried about the burden's return current sharing a narrow
ground *trace* with the ESP: 200 mA of WiFi burst through 10 mΩ of shared copper
is 2 mV, or 0.12 % of full scale.

That concern is largely answered by the solid GND pour on both layers rather than
by a star link. Return current in a plane concentrates under its own outgoing
trace, and the plane's spreading resistance between U2's ground and the analog
corner is well under a milliohm — roughly 0.1 mV at 200 mA, some twenty times
smaller than the trace case, so the 0 Ω star-point resistor was dropped.

If you ever re-route or cut the pour, keep that in mind: the plane is doing the
work, so do not fragment it between U3 and R1.

## Deliberately omitted

No fuse, no reverse-polarity diode, no TVS. This board has **no input
protection** — feed it only from a known-good 19 V PSU and check J1 polarity
before every power-up. Also dropped versus the original spec: the spare-ADC
breakout (F7) and the dedicated test points (F8); calibrate by probing R1's
leads directly.

## Calibration

Two-point, in situ, per the original spec §11.3:

1. Probe hanging in air → note the voltage as the 0 cm point.
2. Probe at a known depth → note the voltage.
3. Enter both into `calibrate_linear` in the ESPHome config.

The theoretical curve is a starting point, not a calibration.

## Files

- `cistern-level-interface.kicad_pro` / `.kicad_sch` / `.kicad_pcb`
- `lib/cistern.kicad_sym` — `ADS1115_Breakout`, `MP1584_Module` symbols
- `lib/cistern.pretty/` — `MP1584EN_Buck_Module`, `Fuse_Bourns_MF-R025_Radial`
  (the fuse footprint is unused in this variant; kept for the protected variant)
- `output/` — schematic PDF, PCB PDF/SVG, BOM CSV, 3D top render
