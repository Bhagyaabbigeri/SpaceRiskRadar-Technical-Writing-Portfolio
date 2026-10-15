---
title: SpaceRisk Radar Data Flow
description: How public orbital elements become propagated browser data and analysis overlays.
---

# SpaceRisk Radar Data Flow

**Summary:** The backend loads public TLE records, propagates a bounded catalog
to a common time, builds display objects, runs snapshot analysis, and sends
JSON data to the browser over REST or Socket.IO.

**Audience:** Developers and reviewers tracing data provenance and
implementation boundaries.

## Live catalog-to-dashboard path

```mermaid
accTitle: Orbital catalog data flow
accDescr: CelesTrak or a fallback source supplies TLE records; the backend parses and propagates up to 2,000 objects at one UTC time, computes screening and overlays, and delivers payloads to the browser over REST and Socket.IO.
sequenceDiagram
    participant Source as CelesTrak / fallback source
    participant Fetcher as tle_fetcher.py
    participant Worker as app.py background workers
    participant Orbit as orbit_calculator.py + sgp4
    participant Risk as risk_engine.py and overlays
    participant API as Flask REST / Socket.IO
    participant UI as Browser UI and globe
    Source-->>Fetcher: Three-line TLE text
    Fetcher->>Fetcher: Read cache; fetch and refresh if needed
    Fetcher-->>Worker: Parsed TLEData records
    Worker->>Orbit: Up to 2,000 records and one UTC timestamp
    Orbit-->>Worker: Position-derived objects and velocity
    Worker->>Risk: Same-time position records
    Risk-->>Worker: Conjunctions and overlay payloads
    Worker->>API: Update in-process state and emit events
    API-->>UI: objects, conjunctions, stats, overlays
    UI->>API: REST requests for search/details/history or overlay data
    API-->>UI: JSON responses
```

## Processing stages

1. **Retrieve and cache.** At startup, `get_tles()` reads
   `backend/tle_cache.txt`. If a healthy cache is younger than two hours, it
   can be used without a network request. Otherwise, the fetcher tries the
   configured CelesTrak catalog URL, a CelesTrak active-group URL, then an
   AMSAT mirror. If a refresh fails, existing cached content may be retained.
2. **Parse element records.** `parse_tle_data()` consumes blocks with a name
   line, line 1, and line 2, then returns `TLEData` objects. The parser skips
   blocks whose two data lines do not begin with `1 ` and `2 ` respectively.
3. **Choose a time and bound the work.** The background emitter snapshots the
   catalog, takes at most 2,000 entries, and uses one current UTC timestamp for
   the cycle. Individual propagation failures are logged at debug level and
   skipped.
4. **Propagate and convert.** The orbit calculator calls `sgp4` at that time
   and converts the TEME position to latitude, longitude, and altitude. The
   speed is derived from the returned velocity vector.
5. **Prepare and screen.** The backend builds display fields and calls
   `find_conjunctions()` on the propagated positions. That module uses a
   simplified surface-arc-plus-altitude distance, not a future closest
   approach. The list also includes an injected test object when the list is
   non-empty.
6. **Calculate overlays.** Separate functions produce station visibility,
   density bins, hypothetical debris clouds, and maneuver previews. The
   maneuver and launch previews use local simplified data/geometry rather than
   operating ephemerides.
7. **Deliver to the UI.** The worker emits named Socket.IO events about every
   three seconds. REST endpoints can return the latest in-process snapshot or
   calculate a requested time/overlay. The browser handles these messages in
   `frontend/ui.js` and renders the globe and panels.

## Inputs, outputs, and dependencies

| Stage | Input | Output | Dependency or limitation |
|---|---|---|---|
| Catalog load | Network text or `backend/tle_cache.txt` | Parsed TLE records | External sources may be unavailable; cached elements may be stale. |
| Propagation | TLE lines and UTC time | TEME state, then geodetic coordinates | Python `sgp4`; custom project coordinate conversion. |
| Proximity screen | Same-time latitude, longitude, altitude | Candidate pairs under threshold | Approximate distance; no time-of-closest-approach search. |
| Ground visibility | Position and five built-in station locations | Best visible station per object | Spherical-Earth line-of-sight approximation and estimated duration. |
| Heatmap | Position, altitude, orbit label | Binned density cells | Bins are visual summaries; normalized density is not collision risk. |
| Debris scenario | Screened pairs and positions | Hypothetical cloud records | Deterministic heuristic, not a physical fragmentation model. |
| API / Socket.IO | Python payload dictionaries | JSON and events | Flask and Flask-SocketIO; event stream is broadcast. |

The backend also serves launch information from Launch Library 2 when
available, keeps a local cache, and substitutes a bundled placeholder when
remote fetch fails. The generated launch trajectory is a display path only.

## References

- [CelesTrak GP data formats](https://celestrak.org/NORAD/documentation/gp-data-formats.php)
- [Flask-SocketIO documentation](https://flask-socketio.readthedocs.io/)
- [Python `sgp4` package](https://pypi.org/project/sgp4/)

## Implementation note

The worker cadence and limits are configuration in the checked-in application
code, not service-level guarantees. No timing, throughput, availability, or
accuracy benchmark is claimed here.
