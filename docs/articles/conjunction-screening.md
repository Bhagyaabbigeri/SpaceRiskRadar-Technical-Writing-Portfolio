---
title: Understanding Satellite Conjunction Screening
description: What proximity screening finds, how SpaceRisk Radar computes distance, and why its risk score is not collision probability.
---

# Understanding Satellite Conjunction Screening

**Summary:** A conjunction is a close approach between objects in space.
Screening searches for pairs that meet a chosen proximity rule. A screening
result is not, by itself, a prediction that two objects will collide. SpaceRisk
Radar compares propagated positions at one selected time using a simplified
distance calculation; its displayed probability is a heuristic score, not a
scientifically validated collision probability.

**Audience:** Readers learning about space situational awareness, developers
building tracking interfaces, and anyone interpreting the dashboard's risk
labels.

## What is a conjunction?

In spaceflight contexts, a conjunction is a close approach between two
space objects. Screening systems identify candidate events for closer
assessment. Operational collision-risk analysis typically considers the time
of closest approach, relative motion, object sizes, and uncertainty in both
objects' states. A momentary distance between two current position estimates is
not the same thing as a predicted miss distance at a future encounter.

## What SpaceRisk Radar screens

`backend/risk_engine.py` evaluates every pair in the list it receives. For each
pair it calculates a distance, then returns a result when that value is below
the configured `threshold_km`. In the live pipeline, each candidate object is
propagated to the same UTC timestamp before the pairwise comparison. The
interactive catalog is capped at 2,000 objects, and the implementation also
injects a synthetic `TEST_DEBRIS_X` near the first object when a propagated list
is non-empty. This test object is not a catalog observation.

The function named `haversine_3d` first computes great-circle surface distance
from latitude and longitude using a fixed 6,371 km Earth radius, then combines
that value and the altitude difference with a Pythagorean square root. This is
an application-level approximation. It is not a Cartesian distance between
orbital position vectors, and it does not determine the future time of closest
approach.

The default threshold is 500 km. A smaller threshold narrows the set of pairs
reported by this rule; a larger threshold includes more pairs. The threshold is
a screening parameter, not a scientifically calibrated risk boundary. The
backend clamps live Socket.IO threshold updates to 10–5,000 km, although the
dashboard slider itself ranges from 10 to 2,000 km. The REST query parameter is
read as a floating-point value but is not clamped to those limits.

## From proximity to the displayed risk label

For each pair within the threshold, the code computes a value called
`probability`. The implementation starts with
`base_score = -2.0 - 0.08 × distance_km`, adds hand-set adjustments for
categorical properties (such as orbit class, debris labels, speed, and
inclination), clamps the exponent, and evaluates a power of ten. The resulting
number is mapped to CRITICAL, HIGH, MEDIUM, or LOW labels and displayed with
risk-factor text.

Despite the field name, this is **not** a validated probability of collision
(Pc). The calculation does not use collision geometry, hard-body radius,
relative velocity at a predicted closest approach, or position covariance.
Its labels and values should not be interpreted as frequencies, calibrated
probabilities, or maneuver recommendations.

## What rigorous collision probability work needs

A collision probability assessment requires a defined encounter at a time of
closest approach, relative position and velocity, object hard-body dimensions,
and defensible uncertainty information (commonly represented through
covariance). The encounter geometry and uncertainty model are then used in a
specified probability method, with careful attention to assumptions and
screening uncertainty. SpaceRisk Radar does not implement this workflow.

## References

- NASA, [Spacecraft Conjunction Assessment and Collision Avoidance Best Practices Handbook](https://ntrs.nasa.gov/citations/20205011318).
- CelesTrak, [Frequently Asked Questions: Two-Line Element Set Format](https://celestrak.org/columns/v04n03/).

## Implementation limitations

The feature is best described as a snapshot proximity visualization:

- A single propagated instant is compared; no time-of-closest-approach search
  occurs in `find_conjunctions`.
- The coordinate-space distance is an approximation based on spherical surface
  arc plus altitude difference, not orbital Cartesian state vectors.
- TLE-derived positions lack a covariance in this application.
- The heuristic score is not a validated Pc, despite its JSON field name.
- The test debris object can create an intentionally synthetic close pair.
- The pairwise loop is quadratic in the number of objects; the application uses
  a 2,000-object propagation limit.

Do not use the display as an operational warning, collision probability,
avoidance command, or measure of real-world risk.
