# Images

Empty for now — the installation is not built yet.

## Shot list

Photographs worth taking as the build progresses. Each answers a question that
text handles badly.

### Probe and shaft

- [ ] **Probe with the protective cap removed**, showing the diaphragm.
      Answers "which end actually measures" — the reference point for depth
      measurement during calibration.
- [ ] **Cable cross-section**, showing the capillary between the conductors.
      This is the detail people get wrong when buying; a photo settles it.
- [ ] **Probe suspended in the stilling tube**, before lowering.
      Shows the 20–30 cm standoff from the floor.
- [ ] **Shaft from above** with the probe installed and the junction box
      mounted near the top.

### Junction box

- [ ] **Interior**, showing where the capillary terminates and the
      pressure-equalising filter sits.
- [ ] **Mounting position** relative to the maximum water level.

### Electronics

- [ ] **Breadboard prototype** — useful precisely because it is untidy; it
      shows the circuit is only a handful of components.
- [ ] **Assembled interface board**, with both modules seated, before it goes
      in the enclosure.
- [ ] **MP1584 trimpot being set to 5.0 V** with the meter reading visible and
      the D1 mini socket still empty. This is the step that kills the ESP if
      done in the wrong order, so it is worth documenting.
- [ ] **Enclosure interior**, with terminal blocks, modules and the burden
      resistor visible.
- [ ] **Close-up of the burden resistor** with its markings legible.

### Home Assistant

- [ ] **Dashboard card** with a real level reading.
- [ ] **72-hour history graph** covering an irrigation cycle and the refill
      afterwards. This is the single most informative screenshot for anyone
      with an open-bottomed cistern — it shows the recovery curve directly.
- [ ] **Calibration**: multimeter across R1, ideally with the reading visible.

## Conventions

- Landscape, 1600 px wide is plenty
- Compress before committing; a repo does not need 4 MB photos
- Descriptive filenames: `probe-diaphragm.jpg`, not `IMG_2847.jpg`
- Reference from the docs as `![description](../images/filename.jpg)`