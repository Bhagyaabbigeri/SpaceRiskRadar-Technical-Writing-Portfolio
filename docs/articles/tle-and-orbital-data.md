---
title: Understanding Satellite Orbital Data and TLEs
description: Read a Two-Line Element set, understand its epoch, and use it responsibly.
---

# Understanding Satellite Orbital Data and TLEs

**Summary:** A Two-Line Element set (TLE) is a standardized, compact description
of an object's orbit at a particular epoch. Tracking software combines those
elements with a compatible propagation model to estimate where the object will
be at another time. A TLE is not a live position, a guarantee of accuracy, or a
complete uncertainty description.

**Audience:** Readers new to orbital tracking, software developers, and
technical communicators documenting systems that consume public element sets.

## What a TLE contains

A TLE (Two-Line Element set) consists of two fixed-width element lines. Catalog
files commonly put an object-name line before those two lines; that
name-plus-elements record is often called a three-line element (3LE) format.
The two TLE data lines contain a catalog number, epoch, inclination, right
ascension of the ascending node, eccentricity, argument of perigee, mean
anomaly, mean motion, and other values used by the SGP4 model. Line 1 also
contains fields such as the element-set number and a drag-related term. The
format is compact and positional: characters occupy defined columns, so it
should be parsed as a format rather than treated as free-form text.

The orbital elements are mean elements fitted to observations using conventions
associated with the SGP4/SDP4 model. They are not interchangeable with generic
Keplerian elements or arbitrary state vectors. For field definitions and
format details, consult the [CelesTrak TLE FAQ](https://celestrak.org/columns/v04n03/).

## The epoch is the reference time

The **element epoch** is the time associated with the fitted element set. It
does not mean the observation was made at that instant, nor does it say when
the data provider published the file. Propagating far from the epoch generally
makes the estimate less dependable because the model cannot account for every
unmodeled force, tracking update, or maneuver.

Freshness therefore has at least two meanings:

1. **File age:** how recently the application fetched or cached the record.
2. **Element age:** elapsed time between the TLE epoch and the time at which a
   position is requested.

A newly downloaded catalog may still contain element sets with different
epochs. Check an object's epoch, not only the fetch timestamp.

## How a tracking application uses a TLE

An application passes the two element lines to a propagator configured for TLE
input, then requests a state at a chosen time. The state can be transformed into
coordinates suitable for mapping or rendering. A tracking view may repeat this
for a catalog and update the display as time advances.

SpaceRisk Radar's `backend/tle_fetcher.py` downloads a catalog of
name-plus-two-element-line records from CelesTrak, with alternate CelesTrak and
AMSAT sources, and stores a local text cache in `backend/tle_cache.txt`. A
healthy cache is considered fresh for two hours. If a refresh fails, existing
cached content may be used. The backend parses name/line-1/line-2 blocks, and
`backend/orbit_calculator.py` passes the element lines to `Satrec.twoline2rv`
from the `sgp4` Python package. The element epoch is decoded from line 1 and
returned in object metadata.

The interactive propagation loop uses at most 2,000 catalog entries per cycle.
The configured limit is a workload bound, not a completeness guarantee. See
[the data-flow guide](../engineering/data-flow.md) for the complete processing
path.

## What a TLE cannot establish by itself

A TLE does not provide a covariance or a probability distribution describing
position uncertainty. It does not identify every object reliably by a
human-readable name, prove that an object remains maneuver-free, or guarantee a
position accuracy for a given number of days. A TLE-only application should not
turn a propagated point estimate into an operational collision warning without
additional data and validated analysis.

## References

- CelesTrak, [Frequently Asked Questions: Two-Line Element Set Format](https://celestrak.org/columns/v04n03/).
- CelesTrak, [NORAD General Perturbations (GP) data documentation](https://celestrak.org/NORAD/documentation/gp-data-formats.php).
- Python `sgp4`, [TLE formats and element sets](https://pypi.org/project/sgp4/).

## Implementation note

This article describes the code currently present in the original repository.
The application fetches public GP/TLE data and reports an epoch parsed from the
TLE. It does not retrieve covariance from that TLE, and the age of a cache file
is not a substitute for checking the epoch of each element set.
