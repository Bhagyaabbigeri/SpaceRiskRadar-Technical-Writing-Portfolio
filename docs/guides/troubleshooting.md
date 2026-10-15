---
title: SpaceRisk Radar Troubleshooting
description: Source-grounded diagnostics for setup, data, API, Socket.IO, and rendering symptoms.
---

# SpaceRisk Radar Troubleshooting

This guide distinguishes behaviors visible in the source from symptoms that
have not been reproduced in this portfolio work. “Possible cause” entries are
diagnostic hypotheses, not claims that a failure occurred.

## Python dependency import fails

**Symptom:** `ModuleNotFoundError` appears when starting `backend/app.py`.

**Likely cause:** Dependencies from the application repository's
`requirements.txt` are not installed in the active Python environment, or the
virtual environment is not active.

**Check and resolve:**

```powershell
python --version
python -m pip show Flask Flask-SocketIO sgp4
python -m pip install -r requirements.txt
```

Run these commands from the application repository root with its virtual
environment active. The application manifest includes Flask, Flask-SocketIO,
Eventlet, `sgp4`, Redis, and authentication dependencies.

## Server starts but no orbital objects appear

**Symptom:** `/health` responds, but `tracked_objects` is zero or `/api/objects`
returns an empty array.

**Verified behavior:** Catalog retrieval happens asynchronously after startup.
The fetcher requests CelesTrak sources, then an AMSAT mirror, and may use the
local `backend/tle_cache.txt` file as a fallback. If no data is available,
`get_tles()` logs the error and returns an empty list.

**Checks:** Wait for the startup catalog load, inspect the server log for
`TLE pipeline failed` or fetch errors, check whether the cache file exists, and
confirm outbound network access to the data sources. Do not delete the cache
as a first troubleshooting step; it may be the only available offline data.

## Health succeeds but browser live panels do not update

**Possible cause, not reproduced here:** The browser's Socket.IO connection
may be blocked, disconnected, or unable to reach the same origin. The frontend
initializes Socket.IO and retries when unavailable; the backend emits live
events from a background worker.

**Check:** Open browser developer tools, review the Console and Network
WebSocket/Socket.IO requests, and confirm the page is opened from the Flask
origin rather than a separate file URL. Verify that `/health` and
`/api/objects` respond locally.

## Port 5000 is already in use

**Possible cause, not reproduced here:** Another process is listening on the
default port. The application reads its port from `PORT`.

**Safe workaround:** Choose a different port for the current PowerShell session
and start the application again:

```powershell
$env:PORT = "5001"
python backend/app.py
```

Then open <http://localhost:5001/>. Avoid terminating an unidentified process.

## Redis warnings appear

**Verified behavior:** `backend/cache_layer.py` attempts Redis at `REDIS_URL`.
If connection or ping fails, it logs a warning and uses in-process memory
caching. A Redis server is optional for local operation; the fallback is not
shared across processes and does not persist across restarts.

**Check:** If Redis is intended, confirm the URL and service availability. If
not, the warning is expected and can be treated as informational for local
development.

## Launch list is empty or shows a fallback record

**Verified behavior:** Launch Library 2 is requested with a 12-second timeout.
The route caches launch data for 30 minutes, and the module supplies a fallback
record if the network request fails.

**Check:** Inspect the response's `source` field (`cache`,
`launch_library_2`, or `fallback`) and confirm outbound access if live data is
expected. Do not assume that a fallback launch is scheduled.

## Globe is blank or rendering is degraded

**Possible cause, not reproduced here:** Browser WebGL support, hardware
acceleration, context limits, or a JavaScript exception may prevent the Three.js
scene from rendering.

**Check:** Try a current browser with WebGL enabled, inspect the Console for
JavaScript errors, and check whether REST responses contain objects. If REST
data exists but the canvas is blank, investigate rendering separately from
the backend data path.

## Cached data seems old

**Verified behavior:** The TLE file-cache freshness check is two hours, but
each record also has an element epoch. A successful recent fetch therefore
does not prove that every element is recent. Cached data may be served after
refresh failure.

**Check:** Compare each object's `tle_epoch` with the requested propagation
time. Do not infer freshness from file modification time alone.

## Report a reproducible issue

Capture the command used, Python version, endpoint path (without credentials),
relevant error text, and whether a local cache or Redis was enabled. Redact
tokens, passwords, webhook URLs, and other private configuration before
sharing logs.
