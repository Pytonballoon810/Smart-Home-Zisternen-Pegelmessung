# Hardware

Bill of materials, wiring, and selection criteria.

---

## 1. Bill of materials

### Measurement chain

| Item | Specification | Qty | ~Cost |
|---|---|--:|--:|
| **Level probe** | 4–20 mA, 2-wire, 12–32 V DC, range to suit, stainless, IP68, **vented cable** | 1 | 40 € |
| **ADC module** | ADS1115, 16-bit, I²C (Joy-IT KY-053, sold as RB-ADC01) | 1 | 12 € |
| **Microcontroller** | ESP8266 (Wemos D1 mini) | 1 | 5–8 € |
| **Buck converter** | MP1584EN adjustable module, set to 5.0 V | 1 | 2 € |
| **Burden resistor** (R1) | 100 Ω, 0.1 %, metal film, ≤ 50 ppm/K, ≥ 0.25 W | 1 | 2 € |
| **Series limiter** (R2) | 1 kΩ, 1 %, 0402 — non-critical | 1 | <1 € |
| **Filter cap** (C3) | 100 nF C0G, 0805 | 1 | <1 € |
| **Interface PCB** | 70 × 60 mm, 2-layer — see [`pcb-specification.md`](pcb-specification.md) | 1 | ~5 € (qty 5) |
| **Power supply** | 19 V DC, ≥ 300 mA | 1 | 12–20 € |

The three discrete parts, two screw terminals and the two module sockets sit on
the interface PCB. On a breadboard, only R1 is strictly needed — see §3.

### Cable run (if the probe cable is too short)

| Item | Specification | Qty | ~Cost |
|---|---|--:|--:|
| **Junction box with pressure equalisation** | e.g. TECSON T12080, IP 64, Goretex filter | 1 | 27 € |
| Extension cable | 2-core, ≥ 0.4 mm² | as needed | 5 € |

### Installation

| Item | Specification | Qty | ~Cost |
|---|---|--:|--:|
| Stilling tube | HT pipe DN 50/75, drilled | ~2 m | 10 € |
| Pipe clamps | for shaft wall | 2–3 | 5 € |
| Suspension | stainless wire or UV-resistant ties | 1 | 3 € |
| Enclosure | IP65 | 1 | 15 € |
| Terminal blocks, glands, wire | — | 1 set | 10 € |

**Total: approximately 140 €.**

---

## 2. Choosing the probe

### Measurement range

Measure the **water column**, not the cistern's build height:

```
range = (overflow height) − (pump intake height)
```

A 3 m deep cistern typically has 2.2–2.5 m of usable column — the overflow sits
20–40 cm below the ceiling, the pump 20–30 cm above the floor.

Pick the next size up, not two. Accuracy is specified as a percentage of **full
scale**: 0.5 % on a 2 m probe is 10 mm, on a 5 m probe it is 25 mm.

Exceeding the range does not damage the probe — most tolerate several times
their rated pressure. The reading simply stops rising.

### Vented cable — mandatory

The specification must state one of:

- Luftführungskabel
- vented cable / air tube
- capillary / Kapillare
- gauge pressure (as opposed to absolute)

**Without it the probe measures absolute pressure**, i.e. water column *plus*
atmospheric pressure. A normal weather system moves barometric pressure by
20–30 hPa, equivalent to 20–30 cm of water column. In a 2 m diameter cistern
that is 600–900 litres of error.

The failure mode is insidious: nothing breaks, the reading just wanders slowly.

### Cable length

Measure from the intended probe position to the dry termination point, then add
two metres. Vented cable cannot be extended with ordinary connectors — the air
path must stay open. Either buy it long enough, or budget for a junction box
with a pressure-equalising filter.

### Material

Stainless 304 is adequate for rainwater. 316L only matters if chlorides are
present.

Prefer a **removable protective cap** — an open-bottomed cistern carries
sediment, and being able to clean the diaphragm without pulling the probe is
worth having.

---

## 3. Wiring

Minimum wiring, as on the breadboard prototype:

```
   19 V (+) ─────────────────────────────►  Probe (+)     [usually red]

   Probe (−) [usually black] ──┬─────────►  ADS1115  A0
                               │
                            R1 100 Ω
                               │
   19 V (−) ───────────────────┴─────────►  ADS1115  GND ──── ESP GND

   ESP 3V3 ──────────────────────────────►  ADS1115  VDD
   ESP D2 (GPIO4) ───────────────────────►  ADS1115  SDA
   ESP D1 (GPIO5) ───────────────────────►  ADS1115  SCL
```

The interface PCB adds two parts to this chain — **R2**, a 1 kΩ series resistor
between the burden node and A0, and **C3**, 100 nF from A0 to ground. Neither
affects the measurement: R2 limits current into the ADS1115's internal
protection diodes, and C3 filters the buck converter's switching noise. R2 sits
in a 1 kΩ : ~6 MΩ divider with the ADC input, contributing about 0.02 % gain
error, which the two-point calibration removes anyway. On a breadboard both can
be omitted.

The PCB also carries the MP1584 buck that derives 5 V for the ESP, so only one
supply is needed.

**Common ground is mandatory.** The ADS1115 measures against its own GND pin;
if that is not tied to the supply negative, the reading is undefined.

### Signal levels at 100 Ω

| State | Current | Voltage |
|---|--:|--:|
| Cable break | 0 mA | 0.00 V |
| Empty (probe top) | 4 mA | 0.40 V |
| Half (mid-range) | 12 mA | 1.20 V |
| Full (range top) | 20 mA | 2.00 V |
| Short / overrange | 30 mA | 3.00 V |

### Why 100 Ω and not 150 Ω

The ADS1115 input must stay below VDD + 0.3 V. Powered from 3.3 V that ceiling
is 3.6 V.

| Burden | Signal at 20 mA | Fault at 30 mA |
|---|--:|--:|
| 100 Ω | 2.00 V | 3.00 V ✅ |
| 150 Ω | 3.00 V | **4.50 V** ❌ |

150 Ω works if the ADS1115 runs at 5 V — but then the I²C bus sits at 5 V
against a 3.3 V microcontroller, which needs a level shifter. The extra
resolution is meaningless: both options resolve far finer than the probe.

### Supply voltage

The probe needs its minimum terminal voltage *plus* the burden drop:

```
supply ≥ probe_minimum + (0.020 A × burden)
       ≥ 12 V + 2 V  =  14 V
```

**This installation uses 19 V**, which leaves 17 V at the probe at full scale —
comfortable margin over the 12 V minimum, and within the MP1584's 4.5–28 V
input range. A surplus laptop brick is ideal; they are almost all 19 or 19.5 V
and rated far beyond the ~300 mA needed.

> ### ⚠ Check polarity before connecting
>
> The interface board has **no reverse-polarity protection** — the series
> Schottky was dropped in v1.1 to cut part count. Reversing J1 destroys the
> MP1584 and probably everything downstream of it.
>
> Laptop supplies often have **three** output conductors: V+, V− and a thin
> ID/sense wire the laptop uses to identify the adapter. Identify them with a
> meter before wiring — the pair reading ≈19 V DC are your rails, and the wire
> that floats or reads an odd, drifting voltage is the ID line, which is left
> unconnected. Check on **AC volts** too: any significant AC means it is a
> transformer, not a DC supply, and must not be connected.

---

## 4. Installation in the shaft

**Suspend the probe 20–30 cm above the floor**, do not rest it on the bottom.
Sediment accumulates, and a probe lying in silt eventually measures silt.

**Use a stilling tube** — a drilled HT pipe. It holds the probe in position and
shields it from flow when the pump runs.

**Terminate the cable dry.** The capillary's open end must sit in dry air, with
desiccant if the enclosure is not otherwise dry. If the run needs a junction in
the shaft, use a box with a pressure-equalising filter; an ordinary sealed box
would block the air path and defeat the vented cable entirely.

**Define zero at the pump intake**, not at the floor. Water below the intake
cannot be pumped and should not appear as available stock.

---

## 5. Litres per centimetre

```
litres_per_cm = π × (diameter / 2)² × 10
```

| Diameter | Area | Litres per cm |
|---|--:|--:|
| 1.5 m | 1.77 m² | 17.7 |
| 1.8 m | 2.54 m² | 25.4 |
| 2.0 m | 3.14 m² | 31.4 |
| 2.2 m | 3.80 m² | 38.0 |
| 2.5 m | 4.91 m² | 49.1 |

For non-cylindrical tanks, substitute the cross-sectional area. The ESPHome
configuration does this calculation from a `substitutions` entry — no code
changes needed.