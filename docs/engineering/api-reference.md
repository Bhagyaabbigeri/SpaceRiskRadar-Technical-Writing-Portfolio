---
title: SpaceRisk Radar API Reference
description: HTTP routes, parameters, authentication, and response shapes verified from Flask handlers.
---

# SpaceRisk Radar API Reference

**Summary:** The Flask application exposes public REST endpoints at `/api/*`,
versioned aliases and account routes at `/api/v1/*`, a health route, and
Socket.IO events. This guide describes the handlers currently registered in
`backend/app.py` and `backend/api_v1.py`.

**Audience:** Developers integrating a local instance or investigating the
dashboard's request and response contracts.

All requests use the same origin as the Flask-served frontend. Responses below
show the shape produced by the implementation; example names and values are
illustrative, not a live catalog snapshot. Unless noted otherwise, endpoints
are unauthenticated.

## Health and public endpoints

| Method and path | Query parameters | Purpose and response shape |
|---|---|---|
| `GET /health` | — | `{"status":"ok","tracked_objects":0,"timestamp":"<UTC ISO-8601>"}`. The HTTP process may be healthy while the asynchronous catalog load is still empty. |
| `GET /api/objects` | `at` (optional ISO-8601 time) | JSON array of propagated object records. The latest background snapshot uses `lat`, `lon`, and `alt_km`; a requested-time calculation uses `latitude`, `longitude`, and `altitude_km`. If no snapshot is ready, the route may calculate records in the latter format. |
| `GET /api/satellite` | `name` (required); `at` (optional) | One matching object record. Exact case-insensitive match is tried before substring match. |
| `GET /api/search` | `q` or `name`; `at` (optional) | `{"matches":[...],"count":N,"query":"..."}`; at most 10 propagated name matches. Empty search returns an empty `matches` array. |
| `GET /api/conjunctions` | `threshold` (default current threshold); `at` (optional) | `{"conjunctions":[...],"count":N,"threshold_km":...,"timestamp":"..."}`. |
| `GET /api/stats` | `threshold`; `at` (optional) | Counts, altitude and speed summaries, orbit distribution, and timestamp. |
| `GET /api/ground-visibility` | `at` (optional) | `{"stations":[...],"visible":[...],"visible_count":N,"timestamp":"..."}` when catalog data is available. |
| `GET /api/heatmap` | `at` (optional) | `{"cells":[...],"max_density":N,"orbit_counts":{...},"timestamp":"..."}`. At most 300 cells are returned. |
| `GET /api/debris-risk` | `threshold`; `at` (optional) | `{"clouds":[...],"event_count":N,"chain_probability_sum":...,"threshold_km":...,"timestamp":"..."}`. The values are illustrative heuristics. |
| `GET /api/maneuvers` | `at` (optional) | `{"maneuvers":[...],"count":N,"horizon_days":30,"timestamp":"..."}`. Records are local scenario inputs, not verified operator schedules. |
| `GET /api/launches` | `limit` (default 5, clamped to 1–10) | `{"launches":[...],"count":N,"source":"cache\|launch_library_2\|fallback"}`. Trajectories are generated previews. |

The optional `at` value is parsed as an ISO-8601 timestamp. If parsing fails,
the current UTC time is used. Empty catalog handling varies by endpoint: for
example, `/api/objects` returns `[]`; `/api/heatmap` returns empty `cells`,
zero `max_density`, and empty `orbit_counts`; `/api/debris-risk` returns empty
`clouds` and zero counts; and `/api/ground-visibility` returns empty station
and visibility arrays. These early-return payloads may omit fields such as a
timestamp that are present when catalog data is available.

### Object response example for a requested time

`GET /api/objects?at=2026-10-15T12:01:00Z`

```json
{
  "name": "EXAMPLE OBJECT",
  "norad_id": 12345,
  "object_id": "98067A",
  "launch_date": "~1998 (catalog seq. 67)",
  "operator": "Other",
  "decay_date": "Active (in catalog)",
  "tle_epoch": "2026-10-15T12:00:00+00:00",
  "orbit": "LEO",
  "inclination": 51.6,
  "country": "Other",
  "speed_kms": 7.6,
  "timestamp": "2026-10-15T12:01:00+00:00",
  "tle_line1": "<TLE line 1>",
  "tle_line2": "<TLE line 2>",
  "latitude": 42.31,
  "longitude": -87.45,
  "altitude_km": 421.5
}
```

`launch_date`, `operator`, and `decay_date` are derived or placeholder metadata
in this application; they should not be treated as authoritative catalog
enrichment. The orbit-class boundaries are application classifications. A
latest-snapshot object instead uses `lat`, `lon`, and `alt_km` for the final
three coordinate fields; Socket.IO's `objects` event uses that short form too.
When propagation returns at least one object, the backend also inserts a
synthetic `TEST_DEBRIS_X` record near the first object. That test record has a
smaller metadata set and should not be mistaken for a catalog object.

### Conjunction response example

```json
{
  "conjunctions": [
    {
      "obj1": "EXAMPLE OBJECT A",
      "obj2": "EXAMPLE OBJECT B",
      "sat1": "EXAMPLE OBJECT A",
      "sat2": "EXAMPLE OBJECT B",
      "distance_km": 12.5,
      "probability": 0.001059,
      "probability_pct": "0.106%",
      "risk_factors": [
        "Close proximity encounter (<50km)",
        "LEO orbital density multiplier",
        "Active coordinated payloads"
      ],
      "evasion_maneuver": "Cooperative collision avoidance maneuver: EXAMPLE OBJECT A thrust radial-in, EXAMPLE OBJECT B thrust radial-out.",
      "details": {
        "sat1_pos": {"lat": 1.0, "lon": 2.0, "alt": 400.0},
        "sat2_pos": {"lat": 1.0, "lon": 2.0, "alt": 412.5}
      },
      "risk": {"level": "CRITICAL", "color": "#ff3b30"}
    }
  ],
  "count": 1,
  "threshold_km": 20.0,
  "timestamp": "2026-10-15T12:01:00+00:00"
}
```

The numeric probability field is the application's heuristic, **not collision
probability**. See [conjunction screening](../articles/conjunction-screening.md).
The sample pair is assumed to have LEO classification and the default speed
and metadata fields used by the scoring function; the factor
`Active coordinated payloads` is a formula branch, not verified operational
capability.

## Requests and errors

Search for a name:

```http
GET /api/search?q=ISS
```

Request one object's state:

```http
GET /api/satellite?name=ISS&at=2026-10-15T12:00:00Z
```

The detail route returns `400` with `{"error":"name query parameter required"}`
when `name` is missing, and `404` with `{"error":"satellite not found","name":"..."}`
when no name matches. Propagation errors return `500` with an `error` and
`name`. The application does not define one uniform error envelope for every
endpoint.

## Versioned routes

The following GET routes under `/api/v1/` are aliases to the corresponding
public handlers: `/objects`, `/satellite`, `/search`, `/conjunctions`,
`/stats`, `/ground-visibility`, `/maneuvers`, `/debris-risk`, `/launches`, and
`/heatmap`.

Additional routes:

| Method and path | Authentication | Purpose and response |
|---|---|---|
| `GET /api/v1/openapi.json` | No | Returns the generated OpenAPI 3.0.3 document. It provides summaries but omits detailed schemas and does not enumerate every registered route. |
| `GET /api/v1/docs` | No | Serves a Swagger UI page that loads the generated specification. |
| `GET /api/v1/ephemeris/{norad_id}` | No | Returns an array of propagated states with `latitude`, `longitude`, `altitude_km`, `speed_kms`, and `timestamp`. Query: `start`, `end` (ISO-8601), `step_sec` (default 60). The loop stops after 1,000 points. A missing object returns `404`. |
| `GET /api/v1/collision-risk/{norad_id}` | No | Returns the first matching event from the most recent screen, or `{"status":"nominal","message":"No active conjunctions detected"}`. A missing object returns `404`. It is not a dedicated Pc assessment. |

The ephemeris route uses the common time parser, so absent or invalid start/end
values become the current UTC time. Use positive `step_sec` values and a
bounded interval; the route does not provide a rich query-validation error
contract.

## Socket.IO events

Socket.IO uses the application origin and the default namespace. The current
handlers do not require authentication. The backend broadcasts these events
from its background worker, approximately every three seconds when catalog
data is available:

| Event | Direction | Payload |
|---|---|---|
| `objects` | Server → clients | Array of propagated objects. Position fields use `lat`, `lon`, and `alt_km`; metadata includes name, orbit, speed, and catalog fields. |
| `conjunctions` | Server → clients | Object with `conjunctions`, `count`, `threshold_km`, and `timestamp`. Each event uses the same shape described above. |
| `stats` | Server → clients | Object with total count, conjunction count/threshold, altitude and speed summaries, orbit distribution, and timestamp. |
| `ground_visibility` | Server → clients | Object with station definitions, visible-object records, and `visible_count`. |
| `heatmap` | Server → clients | Object with binned `cells`, `max_density`, and `orbit_counts`. |
| `debris_risk` | Server → clients | Object with illustrative `clouds`, `event_count`, and `chain_probability_sum`. |
| `maneuvers` | Server → clients | Object with local scenario `maneuvers`, `count`, and `horizon_days`. |
| `set_threshold` | Client → server | JSON `{"threshold_km": <number>}`. The server clamps valid values to 10–5,000 km. |
| `threshold_updated` | Server → clients | JSON `{"threshold_km": <applied number>}` broadcast after a valid threshold update. |

For example, a client can send:

```json
{"threshold_km": 250}
```

The server logs and ignores a malformed `set_threshold` value. Since
`threshold_updated` and subsequent analysis events are broadcast, a threshold
change is not private to the client that initiated it. The browser's slider
only exposes 10–2,000 km.

## Account and preference routes

These routes require a bearer token except for signup and login. Send the token
as `Authorization: Bearer <token>`.

| Method and path | Request | Success response |
|---|---|---|
| `POST /api/v1/auth/signup` | JSON `email`, `password` (minimum 8 characters), optional `display_name`, optional `role` and `admin_invite` | `201 {"user":{...},"token":"..."}`; invalid input returns `400 {"error":"..."}`. |
| `POST /api/v1/auth/login` | JSON `email`, `password` | `{"user":{...},"token":"...","expires_in":28800}`; invalid credentials return `401`. |
| `POST /api/v1/auth/logout` | Bearer token | `{"ok":true,"message":"discard bearer token client-side"}`. |
| `GET /api/v1/auth/me` | Bearer token | `{"user":{...}}`. |
| `PATCH /api/v1/profile` | JSON `display_name` | `{"user":{...}}`. |
| `GET /api/v1/preferences` | Bearer token | Preferences, watched names, alert threshold, email-alert flag, and webhook URL. |
| `PUT /api/v1/preferences` | JSON preference fields | Updated preferences object. |
| `POST /api/v1/watched-satellites` | JSON `satellite_name` | Updated watchlist; invalid name returns `400`. |
| `DELETE /api/v1/watched-satellites/{name}` | Bearer token | Updated watchlist response. |
| `POST /api/v1/alerts/test` | Optional JSON `event` | Dispatcher result. |
| `GET /api/v1/admin/cache-status` | Admin bearer token | Cache status; protected by the admin-role check. |

Protected routes return `401 {"error":"authentication required"}` when the
bearer token is missing or invalid. Account objects contain `id`, `email`,
`display_name`, `role`, `created_at`, and `updated_at`. For example, login
returns:

```json
{
  "user": {
    "id": 1,
    "email": "reader@example.invalid",
    "display_name": "Reader",
    "role": "user",
    "created_at": "<UTC ISO-8601>",
    "updated_at": "<UTC ISO-8601>"
  },
  "token": "<bearer-token>",
  "expires_in": 28800
}
```

The default preferences response includes `preferences`,
`watched_satellites`, `alert_threshold_km`, `email_alerts_enabled`, and
`webhook_url`. Its built-in preference keys are `theme`,
`default_orbit_filter`, `show_heatmap`, `show_ground_visibility`, and
`show_debris_clouds`. `POST /api/v1/alerts/test` returns counters named
`sent_email`, `sent_webhook`, and `event_count`; delivery also depends on the
user's configured alert settings and available services.

The signup handler creates a user account and token; its source also contains a
hard-coded development invite check that can accept a requested `admin` role
when the matching invite field is supplied. Treat that behavior as unsafe for
an exposed service; the invite value is not repeated here. The API does not
expose email verification or password-reset routes. The token lifetime
defaults to eight hours and can be changed with `CRV_TOKEN_TTL_SECONDS`.

## Related guides

- [System architecture](architecture.md)
- [Data flow](data-flow.md)
- [Getting started](../guides/getting-started.md)
