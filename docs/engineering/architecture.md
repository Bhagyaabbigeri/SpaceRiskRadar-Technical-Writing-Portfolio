---
title: SpaceRisk Radar System Architecture
description: Verified application components, boundaries, and communication paths.
---

# SpaceRisk Radar System Architecture

**Summary:** SpaceRisk Radar is a single Flask application that serves the
browser interface, exposes REST routes, and broadcasts updates with
Flask-SocketIO. Python modules handle catalog retrieval, propagation,
proximity screening, and visualization overlays.

**Audience:** Developers, engineering students, and technical writers who need
to understand the implementation before operating or documenting it.

## Component view

```mermaid
accTitle: SpaceRisk Radar component architecture
accDescr: A browser talks to Flask over HTTP and Socket.IO. Flask routes and background workers use cached TLE data, SGP4 propagation, analysis modules, SQLite account storage, Redis-or-memory caching, and launch data.
flowchart TB
    Browser[Browser: HTML, CSS, JavaScript, Three.js]
    Flask[Flask application and static-file serving]
    REST[REST routes: public and /api/v1]
    Socket[Flask-SocketIO: event stream]
    TLE[TLE fetcher and local text cache]
    Orbit[sgp4 library and coordinate conversion]
    Analysis[Conjunction, ground visibility, heatmap, debris, maneuver modules]
    Cache[Redis cache with in-memory fallback]
    Auth[SQLite user, preference, watchlist store]
    Launch[Launch Library 2 with cached/fallback data]
    Browser -->|HTTP requests| REST
    Browser <-->|Socket.IO events| Socket
    Browser -->|same-origin static assets| Flask
    REST --> TLE
    Socket --> TLE
    TLE --> Orbit
    Orbit --> Analysis
    REST --> Analysis
    REST --> Cache
    REST --> Auth
    REST --> Launch
    Socket --> Flask
```

## Responsibilities and boundaries

| Component | Verified responsibility | Important boundary |
|---|---|---|
| Browser interface | `frontend/index.html`, `ui.js`, and `globe.js` render dashboard controls and a Three.js globe. | The frontend does not implement orbital propagation. |
| Flask | `backend/app.py` serves the UI, REST endpoints, and health route. It starts background workers at module import. | A process can serve HTTP before the asynchronous catalog loader finishes. |
| REST API | Public `/api/*` handlers expose objects, search, conjunctions, statistics, overlays, and launch data. `/api/v1/*` includes aliases and additional account/ephemeris routes. | The generated OpenAPI document lists only summaries and is not a complete schema. |
| Socket.IO | A worker emits object, conjunction, stats, ground-visibility, heatmap, debris-risk, and maneuver events. A client may send `set_threshold`. | This is a live snapshot stream, not a forecast service. |
| TLE retrieval | `backend/tle_fetcher.py` fetches CelesTrak catalog text with alternate sources and a local cache. | Network failure may result in stale cached data or an empty catalog. |
| Orbit calculation | `backend/orbit_calculator.py` calls the Python `sgp4` package and converts TEME coordinates to geodetic display values. | No covariance or end-to-end accuracy estimate is exposed. |
| Analysis modules | Separate modules calculate proximity, ground-station visibility, heatmap bins, illustrative debris clouds, and maneuver previews. | Each overlay has distinct approximations; see the relevant articles. |
| Cache | `backend/cache_layer.py` attempts Redis and falls back to process-local memory. Several routes also cache raw TLE/launch data in files. | In-memory cache is not shared between processes and is lost on restart. |
| User data | `backend/auth_service.py` stores account, preferences, and watchlist records in SQLite. | Auth persistence defaults to a file under `backend`; a memory DB fallback is possible if file initialization fails. |
| Launch data | `backend/launches.py` calls Launch Library 2 and normalizes results, with a local fallback entry. | The generated trajectory is a visualization preview, not a launch vehicle trajectory prediction. |

## Storage and configuration

`REDIS_URL` selects the Redis endpoint; the default is
`redis://localhost:6379/0`. If Redis cannot connect, the cache module logs a
warning and uses memory. `CRV_AUTH_DB` can select the SQLite database path,
and `CRV_JWT_SECRET` controls the token signing secret. The source contains a
development fallback secret: replace it before exposing an instance to an
untrusted network. Do not commit credentials or production secrets.
The optional `CRV_SMTP_*` and `CRV_ALERT_FROM` settings configure email alert
delivery; without an SMTP host, email dispatch is a dry run.

The project also has deployment configuration files, but this guide describes
the application source and does not claim that a hosted service is currently
available.

## References

- [Flask documentation](https://flask.palletsprojects.com/)
- [Flask-SocketIO documentation](https://flask-socketio.readthedocs.io/)
- [Three.js documentation](https://threejs.org/docs/)
- [Python `sgp4` package](https://pypi.org/project/sgp4/)

## Implementation note

This diagram reflects imports, route registration, and event emissions found
in the repository. It does not imply that Redis, external data services, or
the catalog network connection are available in every deployment.
