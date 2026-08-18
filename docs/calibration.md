# Calibration

The values shipped in the ESPHome configuration are the **theoretical**
transfer curve. They assume an exact 100 Ω burden, an exact 4.000 mA zero, and
no gain error in the ADC. None of those hold precisely.

Two-point calibration corrects all three in one step. It takes ten minutes and
is the difference between an indication and a measurement.

---

## What the theoretical curve assumes

At 100 Ω burden with a 0–3 m probe:

| Level | Current | Voltage |
|---|--:|--:|
| 0 cm | 4 mA | 0.400 V |
| 300 cm | 20 mA | 2.000 V |

Real-world deviations:

| Source | Typical error |
|---|---|
| Burden resistor tolerance (0.1 %) | ±0.1 % |
| Burden resistor tolerance (1 %) | ±1 % |
| Probe zero offset | ±0.2 % FS |
| Probe span error | ±0.3 % FS |
| ADS1115 gain error | ±0.15 % |

With a 0.1 % burden these stack to well under 1 %. With a 1 % carbon resistor,
the resistor alone dominates everything else — which is why the BOM specifies
metal film at 0.1 %.

---

## Procedure

### What you need

- Multimeter
- Tape measure
- Access to the probe before it is finally installed

### Step 1 — zero point, probe in air

Power the system with the probe hanging in air, not submerged.

Measure **across R1**, the burden resistor. On the v1.1 interface board R1 is
the axial through-hole part next to the probe terminal, so its leads take meter
probes directly — the dedicated test points from the v1.0 specification were
dropped along with the rest of the protection circuitry.

Alternatively read `sensor.zisterne_pegel_sonde_rohspannung` in Home Assistant,
which is the same node measured by the ADC itself. That avoids a trip to the
shed, though a meter is the better reference when you are establishing the
calibration in the first place.

> Expected: close to 0.400 V. If you read 0.000 V, the loop is open — check
> polarity and supply. If you read more than 0.6 V, something is wrong before
> you go any further.

### Step 2 — span point, probe at known depth

Lower the probe into water to a **measured** depth. Deeper is better; use at
least half the range if you can.

Measure the depth from the water surface to the **probe's diaphragm**, not to
the cable entry or the top of the housing. The diaphragm is at the tip, behind
the protective cap.

Note voltage and depth.

### Step 3 — enter both points

In `esphome/zisterne-pegel.yaml`, replace:

```yaml
      - calibrate_linear:
          - 0.400 -> 0.0
          - 2.000 -> 300.0
```

with your measured pairs, for example:

```yaml
      - calibrate_linear:
          - 0.412 -> 0.0        # measured in air
          - 1.243 -> 155.0      # measured at 155 cm
```

ESPHome extrapolates linearly beyond the two points, so you do not need a
measurement at full scale.

### Step 4 — verify

Flash, then compare the reported centimetre value against the tape measure at a
third depth. Deviation should be within a centimetre or two.

---

## Alternative: calibrate in place

If the probe is already installed and cannot easily be removed:

**Zero point** — wait until the cistern is at a level you can measure directly,
for instance with a weighted tape from the shaft opening. Note the voltage and
the actual water column above the probe.

**Span point** — repeat at a substantially different level. Rain or a heavy
irrigation cycle will provide one within a few days.

Two points at least a metre apart give a usable calibration. Closer together,
and any measurement error in the depth reading is amplified in the slope.

---

## Sanity checks

After calibration, confirm:

| Check | Expected |
|---|---|
| Probe in air | 0 cm ± 2 cm |
| Reading vs tape measure | within 2 cm |
| Loop disconnected | fault sensor active, 0 V |
| Reading over 24 h without inflow or draw | stable within a few cm |

The last one is worth doing. A reading that drifts overnight with no water
movement points at a temperature effect or a partially blocked capillary.

---

## Cross-checking against a float ladder

If you are replacing a float switch ladder, keep it connected for a few weeks.
Its contacts sit at known heights, so each transition is a free calibration
check:

| Float contact | Known level | Probe should read |
|---|---|---|
| tank10 | 300 L | … |
| tank20 | 600 L | … |

If probe and ladder disagree by more than one step, one of them is wrong — and
you have a way to find out which.