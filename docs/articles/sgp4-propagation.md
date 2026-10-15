---
title: How SGP4 Orbital Propagation Works
description: Follow TLE input through SGP4 to a position used in a tracking interface.
---

# How SGP4 Orbital Propagation Works

**Summary:** Orbital propagation estimates an object's state at a time other
than the element epoch. SGP4 is a model designed for the mean elements
distributed in TLE/GP formats. SpaceRisk Radar delegates the propagation step
to the `sgp4` Python library and then applies its own coordinate conversion.

**Audience:** Engineering students, software developers, and technical writers
who need a practical introduction to satellite position pipelines.

## Propagation in plain language

An orbital element set describes an orbit in a model-specific form at a
reference epoch. **Propagation** applies a mathematical model to estimate the
object's state at a requested time. It is a calculation, not a new observation.
Prediction quality depends on the input elements, the elapsed time, model
assumptions, and events such as maneuvers.

## What SGP4 is for

The Simplified General Perturbations 4 (SGP4) model is used with the
corresponding mean-element conventions distributed in TLEs. Implementations
include perturbation terms intended for this model/data pairing. Deep-space
objects are handled through the related SDP4 branch in standard implementations.
The model is widely used for public satellite tracking, but it is not a
high-fidelity orbit determination service and does not replace measurement
updates or object-specific uncertainty data.

Vallado and colleagues document a modernized SGP4 implementation and provide
verification cases in [Revisiting Spacetrack Report #3](https://celestrak.org/publications/AIAA/2006-6753/).
The [`sgp4` package documentation](https://pypi.org/project/sgp4/) describes the
Python interface used by this project.

## From TLE to dashboard coordinates

In `backend/orbit_calculator.py`, SpaceRisk Radar:

1. Parses the two TLE lines with `Satrec.twoline2rv`.
2. Converts the requested UTC timestamp to Julian-date components.
3. Calls `satellite.sgp4(jd, fr)`.
4. Checks the returned error code and raises a `ValueError` if propagation
   fails.
5. Receives position in kilometres and velocity in kilometres per second in
   the TEME frame.
6. Converts position to latitude, longitude, and altitude using project code:
   a GMST-based TEME-to-ECEF rotation followed by a Bowring-style WGS-84
   geodetic conversion.
7. Computes speed as the magnitude of the returned velocity vector.

The result supplies coordinates and speed to the API and the browser globe.
The distinction between the library call and the project's post-processing is
important: SpaceRisk Radar does not implement the SGP4 force model itself, but
it does implement the coordinate conversion around the library result.

```mermaid
accTitle: TLE propagation pipeline
accDescr: TLE lines and a requested UTC time are passed to the SGP4 library; its TEME position and velocity are converted into display coordinates and speed for the API and globe.
flowchart LR
    A[TLE lines and epoch] --> B[sgp4 Satrec.twoline2rv]
    C[Requested UTC time] --> D[Julian date]
    B --> E[SGP4 propagation]
    D --> E
    E --> F[TEME position and velocity]
    F --> G[Project coordinate conversion]
    G --> H[Latitude, longitude, altitude]
    F --> I[Velocity magnitude]
    H --> J[API and globe]
    I --> J
```

## Model and implementation boundaries

The library's SGP4 result is expressed in TEME, not directly in geodetic
latitude and longitude. The project's conversion uses its own GMST expression
and WGS-84 ellipsoid constants; it is not a full high-precision Earth-orientation
transformation. The dashboard shows propagated point estimates and does not
display a covariance or formal uncertainty envelope.

Results should be interpreted near the TLE epoch and with awareness that a
maneuver or stale elements can invalidate a forward estimate. This portfolio
does not claim a measured error bound for the project's end-to-end coordinates.

## References

- Vallado et al., [Revisiting Spacetrack Report #3](https://celestrak.org/publications/AIAA/2006-6753/), AIAA 2006-6753.
- [Python `sgp4` package](https://pypi.org/project/sgp4/).
- CelesTrak, [GP data formats](https://celestrak.org/NORAD/documentation/gp-data-formats.php).

## Implementation note

SpaceRisk Radar invokes the external `sgp4` package; it does not independently
re-implement SGP4. Its subsequent coordinate conversion, altitude classification,
and presentation logic are application code and have their own assumptions.
