# PCB Specification — Cistern Level Sensor Interface

Interface board carrying the ESP8266 and ADS1115 modules, the burden resistor
and a 19 V to 5 V converter. Replaces the breadboard prototype for permanent
installation.

**Version:** 1.1 — 18 August 2026
**Status:** built, routed, ERC and DRC clean. Not yet fabricated.
**KiCAD project:** [`../hardware/`](../hardware/)

> **v1.1 is a deliberate simplification of v1.0.** The original specification
> called for a full input-protection chain and a split analogue/digital ground.
> Both were dropped. Section 14 records what changed and why — read it before
> assuming a part exists.

---

## 1. Purpose and scope

Carrier board that interfaces a 4–20 mA hydrostatic level probe to an ESP8266
running ESPHome, replacing the breadboard prototype for permanent installation
in a garden shed.

**Out of scope:** the probe, the junction box at the cistern shaft, the
enclosure, and external wiring.

### 1.1 Design philosophy

The board is a **carrier for three off-the-shelf modules**, not an integrated
design:

- Hand-solderable, no reflow or fine-pitch work
- Modules are individually replaceable if damaged in the field
- Avoids RF layout, antenna matching and USB design for the ESP
- Total assembly time under 30 minutes

This costs board area versus an integrated design. For a one-off in a shed,
that trade is worth making.

---

## 2. Functional requirements

| ID | Requirement | v1.1 |
|---|---|---|
| F1 | Accept a 2-wire 4–20 mA loop-powered probe, 12–32 V DC compliance | ✅ met |
| F2 | Convert loop current to a voltage readable by a 16-bit ADC | ✅ met |
| F3 | Single-supply operation from one DC input | ✅ met (19 V) |
| F4 | Distinguish cable break (0 mA) from empty tank (4 mA) | ✅ met |
| F5 | Survive a probe short circuit (≥ 30 mA) without damage | ⚠️ partial — see §5.3 |
| F6 | Survive reverse polarity on the input without damage | ❌ **descoped** |
| F7 | Break out unused ADC channels for future expansion | ❌ descoped |
| F8 | Provide test points for two-point calibration in situ | ❌ descoped — probe R1 directly |

F5 is partial rather than failed: a 30 mA probe fault still produces only 3.0 V
at the burden, which the ADS1115 tolerates (§3.3). What is gone is protection
against surge and gross overvoltage.

---

## 3. Electrical specification

### 3.1 Supply

| Parameter | Value |
|---|---|
| Input voltage | 19 V DC nominal (4.5–28 V accepted by U1) |
| Input current | ~100 mA typical, 300 mA supply recommended |
| Reverse polarity | **Not protected** |
| Internal rails | 5 V (MP1584 buck), 3.3 V (D1 mini onboard LDO) |

### 3.2 Measurement chain

| Parameter | Value |
|---|---|
| Loop current range | 4–20 mA (0–30 mA without damage) |
| Burden resistor | 100 Ω, 0.1 %, ≤ 50 ppm/K, ≥ 0.25 W |
| Signal at burden | 0.400 V (4 mA) … 2.000 V (20 mA) |
| Fault signal | 0.000 V (break), 3.000 V (30 mA short) |
| ADC | ADS1115, 16 bit, gain ±4.096 V, 125 µV/LSB |
| Effective resolution | ≈ 12 800 counts over range |

> **The burden resistor sets system accuracy.** Its tolerance appears directly
> in the reading. A 1 % part would dominate the probe's own 0.2–0.5 % error.
> Metal film, not carbon — carbon drifts up to 1000 ppm/K and the shed sees
> −10 to +40 °C.

### 3.3 ADC supply decision

**ADS1115 runs from 3.3 V, not 5 V.**

Its input must stay below VDD + 0.3 V. At 3.3 V that ceiling is 3.6 V; the
worst-case fault signal is 3.0 V, leaving 0.6 V of margin. Running from 3.3 V
also puts the I²C bus natively at ESP level, so **no level shifter is
required**.

Running the ADC at 5 V would allow a 150 Ω burden and marginally better
resolution, at the cost of a level shifter or an out-of-spec I²C connection.
The extra resolution is meaningless — the probe is two orders of magnitude
coarser than either option.

---

## 4. Block diagram

```
  19 V DC in (J1)
      │
      ├──────────────────────────────────► J2.1  Probe (+)  [LOOP+]
      │
   [U1  MP1584EN buck 19→5 V]
      │
      5 V ──► D1 mini (5V pin) ──► onboard LDO ──► 3.3 V ──► ADS1115 VDD
                    │                                            │
                    └──────────── I²C (SDA/SCL, 3.3 V) ──────────┘
                                                                 │
  J2.2  Probe (−) ──┬────── [R2 1 k] ──┬──────────────────► ADS1115 A0
     [LOOP-]        │                  │
                 [R1 100 Ω]        [C3 100 nF]
                    │                  │
                   GND ──────────────  GND
```

Single ground net, poured on both layers — see §10.

---

## 5. Circuit detail

### 5.1 Input

No protection components. J1 feeds U1's input and J2.1 directly.

> **This is the deliberate compromise of v1.1.** There is no fuse, no series
> diode and no TVS. Feed the board only from a known-good 19 V supply and check
> polarity at J1 before every power-up. If the board is ever moved somewhere
> with a long buried supply run, revisit §14.

### 5.2 Buck converter

**U1 — MP1584EN adjustable module**, 4.5–28 V in, adjustable out, ~3 A capable.

> ### ⚠ Set U1 to 5.0 V before fitting U2
>
> The MP1584's output is set by a multi-turn trimpot, and the as-shipped
> setting is arbitrary — often well above 5 V. Power the board with the D1 mini
> socket **empty**, set the output to 5.0 V with a meter, then fit the ESP.
> This is silkscreened on the board.

A 7805 linear regulator will not do: 19 V in, 5 V out at 200 mA is 2.8 W of
dissipation in a sealed enclosure.

### 5.3 Measurement front end

| Ref | Part | Function |
|---|---|---|
| R1 | 100 Ω, 0.1 %, ≤50 ppm/K, 0.25 W metal film, axial | **Burden** — converts loop current to voltage |
| R2 | 1 kΩ, 1 %, 0402 | Series limit into the ADC's internal protection diodes |
| C3 | 100 nF, C0G, 0805 | Anti-alias / buck-noise filter, τ = 100 µs with R2 |

R2 and C3 form a low-pass at 1.6 kHz. The signal is quasi-static — a cistern
level changes over hours — so this costs nothing and removes switching noise
from the buck. That matters more here than it would have with the v1.0 Recom
module: the MP1584 is a noisier switcher.

**Neither R2 nor C3 affects accuracy.** R2 sits in a 1 kΩ : ~6 MΩ divider with
the ADS1115 input, contributing about 0.02 % gain error — and that is a
systematic gain term, which the two-point `calibrate_linear` removes outright.
R2 carries under a microamp and dissipates around a nanowatt, so its tolerance,
dielectric and power rating are all irrelevant. R1 is the opposite in every
respect.

---

## 6. Connectors and interfaces

| Ref | Type | Pins | Function |
|---|---|---|---|
| J1 | Screw terminal, 5.08 mm | 2 | 19 V DC input — silkscreened `+19V` / `GND` |
| J2 | Screw terminal, 5.08 mm | 2 | Probe loop — silkscreened `LOOP+` / `LOOP−` |
| U1 | Module, 8 holes in 4 corner pairs | — | MP1584EN buck |
| U2 | Socket header, 2 × 1×8 | 16 | D1 mini |
| U3 | Socket header, 1×10 | 10 | KY-053 ADS1115, `VDD` and `A3` ends marked |

> **J2 polarity is silkscreened**, and it matters: the probe is 2-wire loop
> powered, and reversing it reads 0 mA — indistinguishable from a cable break
> at the ESPHome level.

### 6.1 Module orientation

**U3's body overhangs one side of its header** by about 15 mm. It must face
*away* from U2 — there is only 7.4 mm of clearance toward the ESP but 22 mm on
the other side, where it passes over R1/R2/C3, all low enough to sit under a
socketed module. The board is silkscreened `U3 BODY THIS SIDE (NOT TOWARD U2)`.

**U2's body overhangs the top board edge** by about 5 mm, deliberately: that is
the antenna end, and copper is kept clear beneath it.

---

## 7. Bill of materials

| Ref | Part | Package | Qty | Notes |
|---|---|---|--:|---|
| U1 | MP1584EN buck module | 22.3 × 16.8 mm, 8 THT holes | 1 | **Set to 5.0 V first** |
| U2 | Wemos D1 mini (ESP8266) | Module on sockets | 1 | |
| U3 | Joy-IT KY-053 (ADS1115) | Module on 1×10 socket | 1 | Also sold as RB-ADC01 |
| R1 | 100 Ω, 0.1 %, ≤50 ppm/K | Axial DIN0207, 10.16 mm | 1 | **Accuracy-critical** |
| R2 | 1 kΩ, 1 % | 0402 | 1 | Non-critical |
| C3 | 100 nF, C0G | 0805 | 1 | |
| J1, J2 | Screw terminal 5.08 mm | 2-pin | 2 | |
| H1–H4 | M3 mounting holes | 3.2 mm | 4 | NPTH |

Eight placed parts. Estimated component cost excluding the three modules:
**under 5 €.**

---

## 8. Mechanical

| Parameter | Value |
|---|---|
| Board outline | 70 × 60 mm |
| Thickness | 1.6 mm |
| Mounting | 4 × M3, 3.2 mm dia., 5 mm in from each corner |
| Component height | ≤ 20 mm (D1 mini on sockets is tallest) |
| Connector edge | J1 and J2 along one long edge for cable entry |

Terminal blocks on a single edge simplify cable glanding — the enclosure needs
penetrations on one side only.

---

## 9. Fabrication

| Parameter | Specification |
|---|---|
| Layers | 2 |
| Material | FR-4, TG 130 or better |
| Copper weight | 1 oz (35 µm) |
| Min trace / space | 0.25 mm signal, 0.2 mm clearance |
| Min drill | 0.3 mm (vias), 1.0 mm (module holes) |
| Surface finish | HASL lead-free, or ENIG |
| Solder mask | Green, both sides |
| Silkscreen | White, top side |

Within standard capability at JLCPCB, Aisler and PCBWay. No controlled
impedance, no blind or buried vias, no special stackup.

**Quantity:** 5 pieces is the typical minimum and costs about the same as 1–2.

Gerbers and drill files are exported to
[`../hardware/output/gerbers/`](../hardware/output/gerbers/).

---

## 10. Layout, as routed

| Parameter | Value |
|---|---|
| Signal tracks | 0.25 mm |
| GND, +19 V | 0.8 mm |
| +5 V, +3V3 | 0.5 mm |
| Vias | 0.6 mm outer / 0.3 mm drill, 6 used |
| Clearance | 0.2 mm |
| Segments | 68 |

Routed with the [KiCADRoutingTools](https://github.com/drandyhaas/kicadroutingtools)
plugin's headless `route.py`.

**Ground is a single net, poured on both F.Cu and B.Cu.** v1.0 called for split
AGND/DGND joined at a star point; §14 explains why that was dropped.

Layout intent worth preserving if the board is ever re-laid:

1. Analogue parts (R1, R2, C3, U3) grouped in one corner, ≥ 15 mm from the ESP
   antenna. As built the separation is over 30 mm.
2. No copper under the D1 mini's antenna, which overhangs the top edge.
3. Keep the ground pour continuous between U3 and R1 — it is what makes the
   return path short (§11).

---

## 11. Grounding

The v1.0 concern was that the burden's return current would share a narrow
ground **trace** with the ESP: 200 mA of WiFi burst through 10 mΩ of shared
copper is 2 mV, or 0.12 % of full scale — comparable to the probe's entire
tolerance, given away by a layout shortcut. v1.0 answered it with separate AGND
and DGND pours joined by a 0 Ω link at the buck output.

v1.1 answers it with a **solid pour on both layers** instead. Return current in
a plane concentrates under its own outgoing trace, and the spreading resistance
between U2's ground and the analogue corner is well under a milliohm —
roughly 0.1 mV at 200 mA, some twenty times better than the trace case. The
star-point resistor became redundant and was removed.

The consequence: **do not fragment the pour** between U3 and R1. The plane is
doing the work the star link used to.

---

## 12. Verification

### 12.1 Board bring-up, before fitting modules

| Step | Check | Pass criteria |
|---|---|---|
| 1 | Visual, solder bridges | None |
| 2 | Resistance J1+ to J1− | > 10 kΩ (no short) |
| 3 | Apply 19 V, measure U1 output | Adjust trimpot to 5.00 V ± 0.05 V |
| 4 | Confirm 5 V at U2's 5V socket pin | 4.95–5.05 V |

**Step 3 is not optional and must precede fitting U2.**

### 12.2 With modules fitted

| Step | Check | Pass criteria |
|---|---|---|
| 5 | I²C scan in ESPHome log | Device found at 0x48 |
| 6 | Inject 4.00 mA into J2 | 0.400 V across R1 ± 0.5 mV |
| 7 | Inject 20.00 mA into J2 | 2.000 V across R1 ± 2 mV |
| 8 | Open J2 | 0 V across R1, fault sensor active |

Steps 6–7 need a current source; a bench supply with current limit and a series
ammeter is adequate. If unavailable, substitute a precision resistor across the
loop and calculate.

### 12.3 Calibration after installation

Two-point, measuring across R1 — see [`calibration.md`](calibration.md). This
corrects burden tolerance, probe offset and ADC gain error in one step. **Do not
skip it** — the theoretical curve is a starting point, not a calibration.

---

## 13. Open items

| Item | Notes |
|---|---|
| U1 pad pitch | Reconstructed from a photo of the actual module (22.3 × 16.8 mm, corner pairs on 2.54 mm, columns 19.05 mm apart). **Confirm with calipers before ordering.** |
| Enclosure selection | Determines final mounting |
| Cistern inner diameter | Needed for litre conversion, not for the PCB |
| Probe intake height | Defines zero point, not for the PCB |

U3's pinout was confirmed against the physical module and is no longer open.

---

## 14. What changed from v1.0, and why

v1.0 specified 21 placed parts on an 80 × 60 mm board. v1.1 has 8 on 70 × 60 mm.
The driver was a deliberate decision to let the modules do the work and keep
only what earns its place.

| Dropped | v1.0 rationale | Why it went |
|---|---|---|
| F1 PTC, D1 Schottky | Fault current limit, reverse polarity | Supply is a known-good brick in a dry shed; polarity is silkscreened and checked at commissioning |
| D2 TVS 33 V, D3 TVS 5 V | Surge from the buried probe cable | Accepted risk. **The one to reinstate first** if the board ever sees a long outdoor run |
| C1 100 µF, C2 100 nF | Bulk and HF decoupling | Redundant — the MP1584 module carries its own input and output capacitors |
| R3 0 Ω star link | Single AGND↔DGND connection | Superseded by the solid two-layer pour (§11) |
| TP1–TP4 | In-situ calibration points | R1 is an axial part; its leads take meter probes directly |
| J3 spare ADC breakout | F7 future expansion | Not needed; A1–A3 remain unconnected at U3 |
| J4 I²C expansion header | Future sensors | Not needed |

| Changed | v1.0 | v1.1 |
|---|---|---|
| Supply | 24 V | **19 V** — surplus laptop brick |
| Buck | Recom R-78E5.0-0.5, fixed 5 V | **MP1584EN**, adjustable — cheaper, but must be preset |
| Ground | Split AGND/DGND + star link | Single net, poured both layers |
| Board | 80 × 60 mm | 70 × 60 mm |

**The honest summary:** v1.1 trades robustness for simplicity. In a dry shed on
a known supply that is a reasonable trade. It would not be for an outdoor
enclosure, a shared supply rail, or a site with overhead power nearby.
