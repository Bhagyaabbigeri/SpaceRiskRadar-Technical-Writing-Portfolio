---
title: Space Technology Glossary
description: Consistent definitions for orbital data, screening, and web terms used in this portfolio.
---

# Space Technology Glossary

| Term | Definition |
|---|---|
| **API** | Application programming interface: a defined way for software components to exchange requests and responses. |
| **Conjunction** | A close approach between two space objects. Screening identifies candidate proximity events; it does not by itself establish a collision. |
| **Element epoch** | The reference time associated with an orbital element set. It is distinct from the time a file was downloaded. |
| **GEO** | Geostationary orbit: a geosynchronous orbit with a circular, equatorial path, making the satellite appear approximately fixed over one longitude on Earth. |
| **Ground-station visibility** | A geometric line-of-sight condition from a ground site to a satellite above a specified elevation angle. Visibility is not the same as a confirmed communications link. |
| **HEO** | High Earth orbit; a broad classification label. SpaceRisk Radar assigns this label to objects at or above its 36,500 km altitude boundary; this is an application bin, not a universal orbit definition. |
| **Kessler syndrome** | A potential cascading process in which collisions generate debris that may contribute to further collisions. It is a risk scenario, not an automatic outcome of any one conjunction. |
| **LEO** | Low Earth orbit, a region of Earth orbit commonly described as below roughly 2,000 km altitude. Exact boundaries vary by context. |
| **MEO** | Medium Earth orbit, the broad region between LEO and GEO. SpaceRisk Radar uses a simplified altitude classification rather than a standards authority. |
| **Miss distance** | The separation between two objects at a defined closest-approach time. SpaceRisk Radar's `distance_km` is a same-time approximate distance and is not a predicted miss distance. |
| **Orbital debris** | Human-made objects in orbit that no longer serve a useful function, including fragments and inactive hardware. |
| **Orbital propagation** | Estimating an object's orbital state at a time from a reference state or element set and a mathematical model. |
| **REST** | A set of web API architectural constraints commonly implemented using HTTP methods and resource-oriented URLs. |
| **SGP4** | Simplified General Perturbations 4, a propagation model used with corresponding mean elements such as those distributed in TLE/GP data. |
| **SDP4** | Simplified Deep Space Perturbations, the deep-space branch associated with SGP4 implementations for objects with longer orbital periods. |
| **SSA** | Space Situational Awareness: knowledge and understanding of the space environment, including objects, their behavior, and hazards. |
| **TEME** | True Equator, Mean Equinox: a reference frame used by SGP4 outputs. It is not the same as Earth-fixed latitude and longitude. |
| **TLE** | Two-Line Element set: a fixed-width representation of mean orbital elements and related data for use with the SGP4 model family. |
| **WebSocket** | A persistent, bidirectional communication channel over a single connection. SpaceRisk Radar uses Socket.IO, which can use WebSocket transport and other supported transports. |

For implementation-specific details, see the [TLE article](../articles/tle-and-orbital-data.md),
[SGP4 article](../articles/sgp4-propagation.md), and
[conjunction article](../articles/conjunction-screening.md).
