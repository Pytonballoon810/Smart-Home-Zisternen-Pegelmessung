# Zisternen-Pegelmessung

Continuous water level measurement for a garden cistern, using a hydrostatic
4–20 mA probe read by an ESP8266 and reported to Home Assistant via ESPHome.

Replaces a ten-contact float ladder that resolved 300 litres per step — too
coarse to answer the question that actually matters: *is there enough water for
tomorrow's irrigation, and how fast does the tank recover?*

---

## How it works

A 2-wire loop-powered probe sits near the bottom of the shaft and pushes 4 mA
when the water column is zero, 20 mA at full range. That current runs through a
100 Ω burden resistor, turning it into 0.400 V … 2.000 V, which a 16-bit
ADS1115 reads over I²C. ESPHome converts volts to centimetres, then to litres.

The reason for 4–20 mA rather than a voltage output: **a broken cable reads
0 mA, which is 0.000 V — clearly distinguishable from an empty tank at 4 mA.**
A voltage sensor cannot tell those apart, and a cistern that reports "empty"
because a mouse chewed the cable is worse than no sensor at all.

| State | Current | Voltage |
|---|--:|--:|
| Cable break | 0 mA | 0.00 V |
| Empty | 4 mA | 0.40 V |
| Full | 20 mA | 2.00 V |
| Short / overrange | 30 mA | 3.00 V |

---

## Repository layout

| Path | Contents |
|---|---|
| [`docs/hardware.md`](docs/hardware.md) | Bill of materials, wiring, how to choose a probe, installation in the shaft |
| [`docs/pcb-specification.md`](docs/pcb-specification.md) | Interface board: circuit, rationale, v1.0 → v1.1 change log |
| [`docs/calibration.md`](docs/calibration.md) | Two-point calibration procedure — **do not skip this** |
| [`docs/home-assistant.md`](docs/home-assistant.md) | Entities, dashboard cards, derived sensors, automations |
| [`docs/findings.md`](docs/findings.md) | Measured behaviour of an open-bottomed cistern |
| [`esphome/`](esphome/) | Device configuration and secrets template |
| [`hardware/`](hardware/) | KiCAD project, custom libraries, fabrication outputs |
| [`images/`](images/) | Photographs (shot list only — nothing built yet) |

---

## Status

| Part | State |
|---|---|
| ESPHome configuration | Written, not yet flashed |
| Interface PCB | Designed, routed, ERC/DRC clean — **not yet fabricated** |
| Probe and installation | Not yet purchased or built |
| Calibration | Pending installation |

Nothing here has been through a full build yet. The findings in
[`docs/findings.md`](docs/findings.md) come from the float ladder this project
replaces, not from the probe.

---

## Before you build one of these

**Get a vented probe.** The cable must contain a capillary venting the sensor
to atmosphere. Without it the probe measures water column *plus* barometric
pressure, and a normal weather system moves that by 20–30 cm of apparent water
— hundreds of litres of error that drifts rather than breaks. This is the single
most expensive mistake available here.

**The burden resistor sets your accuracy.** 100 Ω at 0.1 % and ≤50 ppm/K. A 1 %
part dominates every other error in the chain, including the probe's own.

**Calibrate in two points after installation.** The shipped transfer curve is
theoretical. Ten minutes with a multimeter and a tape measure is the difference
between an indication and a measurement.

**The v1.1 board has no input protection.** No fuse, no reverse-polarity diode,
no TVS — deliberately, to keep the part count at eight. Feed it from a
known-good supply and check polarity at J1. See §14 of the PCB specification
for what to reinstate if your installation is less forgiving than a dry shed.

---

## An honest caveat about open-bottomed cisterns

This cistern has an open bottom, in hydraulic contact with groundwater. That
makes it a **buffer, not a reservoir**: the level settles at the groundwater
table, rainfall above that drains away within hours, and the tank refills itself
after heavy draw-off. Measured recovery was roughly 600 litres in five hours.

The practical consequence is that usable storage is not the tank volume but the
gap between groundwater level and full — and that gap moves with the season, so
a fixed "ration below X litres" rule does not work.

[`docs/findings.md`](docs/findings.md) has the measurements. If your cistern is
sealed, none of it applies.

Groundwater abstraction is also regulated in most jurisdictions — in Germany
under the Wasserhaushaltsgesetz. Worth asking your local authority before
building something similar.

---

## Licence

Not yet chosen. Until one is added, treat this as "look, learn, but ask before
redistributing".
