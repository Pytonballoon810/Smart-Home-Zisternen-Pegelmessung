# Findings — behaviour of an open-bottomed cistern

Measured observations from a concrete cistern with an **open bottom**, i.e. in
hydraulic contact with groundwater. If your cistern is sealed, none of this
applies to you — skip this document.

Data collected August 2026 with a ten-contact float ladder (300 L resolution).
Numbers are therefore coarse; a continuous probe will refine them.

---

## The short version

An open bottom is a **buffer, not a reservoir.**

- Water level settles at groundwater height and will not rise above it for long
- Rainfall above that level drains away within hours
- After heavy draw-off, the level recovers on its own
- The equilibrium level itself tracks groundwater, which rises after rain

The practical consequence: usable storage is not the tank volume, but the
difference between groundwater level and full — and that difference changes
with the season.

---

## Observation 1 — recovery after running dry

On 11 August the irrigation system drained the cistern until the pump ran dry.
No further draw-off occurred afterwards (supply was switched to mains).

| Time | Level | Elapsed |
|---|--:|--:|
| 11.08. 11:27 | empty | — |
| 11.08. ~16:51 | ≥ 600 L | ~5 h |
| 13.08. 08:57 | 900 L | ~45 h |
| 13.–16.08. | 900 L | no change |

**Inflow rate falls as the level approaches equilibrium** — roughly 100 L/h
initially, about 8 L/h over the final 300 litres, then zero.

This is the expected Darcy behaviour: flow is proportional to the pressure
difference, which vanishes as the levels equalise.

**The practical implication is favourable.** A sealed cistern that runs dry
stays dry until it rains. This one had 600 litres back within five hours.

---

## Observation 2 — rainfall is only partly retained

21.3 mm of rain on 16 August:

| Time | Level | |
|---|--:|---|
| 16.08. 12:00 | 900 L | before rain |
| 16.08. 19:15 | 1200 L | filling |
| 16.08. **19:50** | **1500 L** | peak |
| 16.08. **22:25** | **1200 L** | ← fell back |
| 17.08. through 18.08. | 1200 L | stable |

**300 litres drained away in 2.5 hours with no water drawn.** The meter on the
outlet confirmed zero consumption during that window.

But the level settled at 1200 L, not the previous 900 L. The rain had also
**raised the groundwater table**, moving the equilibrium point up with it.

Net effect of this event: roughly 300 L retained, 300 L lost. Heavier rain
would lose a larger share, because the peak sits further above equilibrium.

---

## Observation 3 — draw-off is masked at high inflow

On 18 August the irrigation cycle drew 131 litres while 5.9 mm of rain fell.
**The level did not drop measurably.** Inflow matched the draw during the cycle
itself.

Under these conditions the tank behaves as though it had unlimited supply. That
holds only while groundwater is high.

---

## What this means for system design

### Storage is capped, not by volume, but by groundwater level

A 3000 litre tank sitting in groundwater at the 900 litre mark has 900 litres
of reliable stock and 2100 litres of unusable headroom. Rainfall filling that
headroom drains within hours.

### Do not seal the bottom without weighing the trade

Sealing it would capture the full 3000 litres — but remove the recovery. Three
weeks without rain in July is normal, and the observed recovery of 600 litres
in five hours is worth more than storage that only fills when it rains anyway.

If storage is the constraint, **additional tanks** are cheaper and lower-risk
than retrofitting a floor. Two 1000 litre IBC totes cost around 200 € and
capture exactly the water that currently drains away. Sealing an existing
cistern is expensive, and in high groundwater it can create buoyancy problems.

### A fixed low-level threshold does not work

Because the equilibrium point moves — 900 L in dry weather, 1500 L after rain —
any fixed "ration below X litres" rule is sometimes too aggressive and
sometimes ineffective.

Better approaches, in increasing order of complexity:

1. Threshold set from the observed dry-weather equilibrium, updated seasonally
2. Rate-of-change: ration when the level falls faster than it recovers
3. Predictive: compare tomorrow's requirement against level plus expected inflow

None of these are implemented here yet — a continuous probe is a prerequisite,
since 300 litre steps cannot resolve a recovery rate.

### Water quality

Groundwater is harder than rainwater and may carry iron and manganese, both of
which precipitate on contact with air and clog drip emitters. Two observations
in this installation are consistent with that: a greenhouse drip line fell from
1.02 to 0.56 L/min over a few days, and the outlet flow meter's impeller has
jammed on debris before.

Sediment ingress through an open bottom is the likely mechanism for both.

---

## Method notes

**Resolution was the limiting factor.** A 300 litre step cannot resolve a
recovery rate; the numbers above are bounds, not measurements. This is precisely
why the installation is moving to a continuous probe.

**Draw-off was independently verified** by a flow meter on the outlet, so
"no consumption" periods are confirmed rather than assumed.

**One confounder remains:** it is not possible to separate roof runoff from
groundwater inflow with a single level sensor. Both raise the level. Separating
them would need a flow meter on the inlet — worth having if the question
matters to you.

---

## Legal note

Groundwater abstraction is regulated in most jurisdictions. In Germany it falls
under the Wasserhaushaltsgesetz and is generally subject to permit, with
exemptions for small domestic quantities that vary by state and district.

An open-bottomed cistern in contact with groundwater is arguably abstraction.
This is not legal advice — if you are planning something similar, ask your
local water authority first.