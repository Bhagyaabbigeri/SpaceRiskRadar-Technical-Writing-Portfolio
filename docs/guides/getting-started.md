---
title: Run SpaceRisk Radar Locally
description: Install and start the Flask application using the commands in its repository.
---

# Run SpaceRisk Radar Locally

**Summary:** The application is served by Flask and does not require a separate
frontend build step. Python dependencies are installed from the application's
`requirements.txt`; the initial orbital catalog loads asynchronously and may
require internet access.

**Audience:** Developers setting up the original SpaceRisk Radar application
on a local Windows, macOS, or Linux machine.

## Prerequisites

- Git.
- Python 3.10 or newer.
- A modern browser.
- Internet access for current CelesTrak data and Launch Library 2 results.
- Redis is optional. The application falls back to an in-process memory cache
  when Redis cannot be reached.

No Node.js install is needed to serve the current frontend. The Flask
application serves `frontend/` from the same origin.

## Set up a virtual environment

Clone the **application** repository, not this portfolio repository:

```powershell
git clone https://github.com/Bhagyaabbigeri/SpaceRisk-Radar.git
Set-Location SpaceRisk-Radar
```

Create and activate an environment on Windows PowerShell:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

If PowerShell blocks activation, either follow your organization's script
policy or invoke the environment's interpreter directly:

```powershell
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

On macOS or Linux, activate with `source .venv/bin/activate`.

## Configuration

No third-party API key is required by the checked-in startup path. Optional
settings are:

| Variable | Purpose | Default |
|---|---|---|
| `PORT` | Flask-SocketIO listening port | `5000` |
| `REDIS_URL` | Redis cache URL | `redis://localhost:6379/0`; memory cache is used if unavailable |
| `CRV_AUTH_DB` | SQLite account database path | `backend/app_data.db` |
| `CRV_JWT_SECRET` | Bearer-token signing secret | Development-only value in source; set a private random value for any exposed instance |
| `CRV_TOKEN_TTL_SECONDS` | Account-token lifetime | `28800` seconds |
| `CRV_SMTP_HOST`, `CRV_SMTP_PORT`, `CRV_SMTP_USER`, `CRV_SMTP_PASSWORD`, `CRV_ALERT_FROM` | Optional SMTP delivery for conjunction alerts | Email delivery is not configured unless an SMTP host is set. |

In PowerShell, for a local session:

```powershell
$env:PORT = "5000"
$env:CRV_JWT_SECRET = "replace-with-a-private-random-value"
```

Do not commit secret values. This application has no need for a real production
credential in a local setup.

## Start the server

From the application repository root:

```powershell
python backend/app.py
```

The server starts background work for the TLE catalog, refresh loop, Socket.IO
updates, and alert scheduler. The initial TLE load is asynchronous, so HTTP
startup does not guarantee orbital objects have loaded yet.

Open <http://localhost:5000/>. The root endpoint serves the dashboard. Check
the health endpoint from another PowerShell window:

```powershell
Invoke-RestMethod http://localhost:5000/health
```

The response should include `"status": "ok"` and a `tracked_objects` count. A
zero count can occur before startup loading finishes or if all catalog sources
and caches are unavailable; it is not a successful catalog verification by
itself. Once objects load, `/api/objects` should return a JSON array.

Stop the foreground server with `Ctrl+C`.

## External data and local files

- The TLE fetcher requests CelesTrak sources and may use a local cache at
  `backend/tle_cache.txt` when network access fails.
- Upcoming launch data is retrieved from Launch Library 2, cached locally, and
  replaced by a bundled fallback record if retrieval fails.
- SQLite account data defaults to `backend/app_data.db`.
- If `REDIS_URL` is unavailable, route caching uses process memory.

An external service outage can change data freshness or replace remote results
with fallback data. The dashboard may still load while data sources are
unavailable.

## Verify the browser interface

After catalog data arrives, the globe should render object points and live
panels should receive REST or Socket.IO data. Browser developer tools can help
distinguish a page-loading issue from an API or Socket.IO connection issue.
See the [troubleshooting guide](troubleshooting.md) for safe checks.

## References

- [Original application README](https://github.com/Bhagyaabbigeri/SpaceRisk-Radar/blob/main/README.md)
- [Flask documentation](https://flask.palletsprojects.com/)
- [Flask-SocketIO documentation](https://flask-socketio.readthedocs.io/)

## Scope note

These commands were checked against the repository's dependency manifest,
Flask entry point, and static-folder configuration. They are not a claim that
the application was started or tested in this portfolio workspace. No
automated application test suite was found in the inspected source tree.
