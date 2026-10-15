---
title: Understanding Space Debris and Kessler Syndrome
description: The orbital debris cascade concept and what the project's simplified simulation represents.
---

# Understanding Space Debris and Kessler Syndrome

**Summary:** Orbital debris includes human-made objects in orbit that no longer
serve a useful function, such as spent rocket bodies and fragments. Collisions
can create more fragments, which may increase the chance of later collisions.
Kessler syndrome describes a potential cascading process; it is not a claim
that every debris encounter causes a runaway cascade.

**Audience:** Space technology readers, students, and developers explaining
debris visualizations without overstating their predictive value.

## Why debris and altitude matter

Objects in orbit travel at high relative speeds. A collision can fragment both
objects and produce debris that remains in orbit, depending on the resulting
orbits and atmospheric drag. The debris environment varies with altitude,
inclination, object population, and time: lower-altitude fragments can be
removed more quickly by atmospheric drag, while objects at higher altitudes may
remain much longer. These are broad tendencies, not a simple altitude-only
forecast.

In their 1978 paper, Donald Kessler and Burton Cour-Palais described how
collision-generated debris could become a significant source of additional
collisions in low Earth orbit. The term is now used for the cascading-collision
concept. For current context, see [NASA's Orbital Debris Program Office](https://orbitaldebris.jsc.nasa.gov/)
and its [orbital debris resources](https://orbitaldebris.jsc.nasa.gov/measurements/).

## What the project's simulation does

`backend/kessler.py` takes the closest five conjunction events, looks up their
objects by name, and constructs a hypothetical debris-cloud record for each
event. It assigns a cloud center from the average of the two objects'
latitude, longitude, and altitude values. A formula based on the event's
distance sets a fragment count and a primary probability-like value. Other
objects within a configured radius are assigned a simple distance-weighted
chain score.

The resulting API payload contains `clouds`, `event_count`, and
`chain_probability_sum`. It is a deterministic visualization aid seeded from
the current screening results. It does not propagate fragments, evolve the
environment over time, sample uncertainty, or model validated collision
fragmentation physics.

In the current code, the module first sets
`d = max(0.1, float(event.get("distance_km") or 999999.0))`. Its illustrative
fragment-count field is `int(80 + max(0, 500 - d) × 1.8)`, and the primary
probability-like field is `min(0.85, 0.015 × (50 / d)^1.35)`. Objects within a
250 km cloud radius are assigned a linearly distance-weighted chain score; only
scores above `0.0005` are retained, and at most 12 affected objects are
included per cloud. These constants describe the implementation only; the
fragment-count field is not the output of a physical fragmentation model.

## Why the output is illustrative

The simulation's formulas are hand-written heuristics, not an empirically
validated collision or breakup model. It has no stochastic uncertainty model
or future orbital evolution. A colored cloud, fragment count, or summed chain
score therefore must not be treated as an operational prediction of debris
growth or collision likelihood.

Use the simulation to explain the *idea* that collision-generated fragments
could affect other objects. Do not quote its numeric values as measured debris
counts, calibrated probabilities, or forecasts.

## References

- Kessler, D. J. and Cour-Palais, B. G. (1978), “Collision Frequency of
  Artificial Satellites: The Creation of a Debris Belt,” *Journal of Geophysical
  Research*, 83(A6), 2637–2646.
  [doi:10.1029/JA083iA06p02637](https://doi.org/10.1029/JA083iA06p02637).
- NASA, [Orbital Debris Program Office](https://orbitaldebris.jsc.nasa.gov/).
- NASA, [Orbital Debris Quarterly News](https://orbitaldebris.jsc.nasa.gov/quarterly-news/).

## Implementation note

SpaceRisk Radar visualizes a simplified, hypothetical scenario rather than
simulating an orbital debris population. Its use of current conjunction
screening results also inherits the screening method's limitations described in
[Understanding Satellite Conjunction Screening](conjunction-screening.md).
