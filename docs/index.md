---
title: SpaceRisk Radar — Technical Writing Portfolio
description: A technical publication about the engineering and orbital concepts behind SpaceRisk Radar.
---

# SpaceRisk Radar — Technical Writing Portfolio

**Engineering Documentation, Technical Articles, and Space Technology Explainers**

This portfolio documents the engineering concepts, system architecture, data
workflows, and technical principles behind SpaceRisk Radar, a satellite
monitoring and orbital-risk visualization project. It shows how complex
technical systems can be explained through structured documentation, diagrams,
practical guides, and accessible engineering articles.

## Start with these articles

| Article | What it explains |
|---|---|
| [Satellite orbital data and TLEs](articles/tle-and-orbital-data.md) | What a two-line element set represents, how its epoch relates to freshness, and what it cannot tell you |
| [How SGP4 propagation works](articles/sgp4-propagation.md) | How an element set becomes an estimated position at a chosen time |
| [Understanding conjunction screening](articles/conjunction-screening.md) | How the project flags nearby objects and why its score is not a collision probability |
| [Space debris and Kessler syndrome](articles/space-debris-and-kessler-syndrome.md) | The real-world cascade concept and the limits of the project's illustrative simulation |

## Engineering documentation

- [System architecture](engineering/architecture.md) — verified components, responsibilities, and boundaries.
- [Data flow](engineering/data-flow.md) — how a catalog record becomes a browser visualization.
- [API reference](engineering/api-reference.md) — routes and response shapes inspected in the Flask handlers.

## Practical guides

- [Getting started](guides/getting-started.md) — install dependencies and run the application locally.
- [Dashboard user guide](guides/dashboard-user-guide.md) — use the search, filters, globe, timeline, and analysis panels.
- [Troubleshooting](guides/troubleshooting.md) — diagnose common setup and connectivity symptoms, with untested possibilities identified as such.

For definitions, see the [space technology glossary](reference/glossary.md).
For the scope and editorial approach, read [About this portfolio](about.md).

## About the project

SpaceRisk Radar is a Flask application with a browser-based visualization. Its
backend retrieves public TLE data, calls the Python `sgp4` library to estimate
object positions, screens the propagated snapshot for proximity, and serves
results over HTTP and Socket.IO. The application also includes ground-station
visibility, a binned orbital heatmap, and simplified debris-risk and maneuver
visualizations.

These are descriptions of implemented code paths, not claims of operational
space situational awareness. In particular, the conjunction routine compares
one propagated snapshot using a surface-distance approximation combined with
altitude difference. The accompanying probability field is a hand-tuned
heuristic. Neither should be used for collision-avoidance decisions.

![SpaceRisk Radar dashboard showing a 3D Earth visualization and telemetry panels](assets/dashboard.png)

*The screenshot is from the application repository. Dashboard controls and
displayed metrics are illustrative; see the [user guide](guides/dashboard-user-guide.md)
and [limitations in the conjunction article](articles/conjunction-screening.md).*

## Project source and original README

- [SpaceRisk Radar application repository](https://github.com/Bhagyaabbigeri/SpaceRisk-Radar)
- [Original application README](https://github.com/Bhagyaabbigeri/SpaceRisk-Radar/blob/main/README.md)

The application repository is the source reference. This portfolio is an
independent documentation collection; it does not replace or modify the
application's README.

## Who this documentation is for

The articles and guides are written for engineering students, software
developers, technical writers, space technology enthusiasts, and recruiters
reviewing engineering documentation work.

## How the collection is organized

The **technical articles** explain the orbital concepts behind the project.
The **engineering documentation** describes verified components, data flow,
and API behavior. The **user guides** focus on running and exploring the
application. The [glossary](reference/glossary.md) provides shared definitions,
and [About this portfolio](about.md) explains the editorial scope.

## About the author

I created this collection to demonstrate how I research, structure, and
communicate technical information using a real software system as the subject.
