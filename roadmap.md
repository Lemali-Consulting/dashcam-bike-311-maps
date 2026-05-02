# dashcam.bike — Pittsburgh 311 map roadmap

Goal: turn the existing points-on-a-map into a tool that's useful for cyclists, advocates, and journalists — not just a dashboard. Source data is `AggregatedHazards_pittsburgh.geojson` (12,234 reports, Apr 2022 – Apr 2026, fields: `HazardName`, `Timestamp`, `Hour`, `DayOfWeek`, `ApproxAddress`, `TweetURL`, `ImageOrVideoFilepath`, lat/lon).

## 1. Repeat-offender locations *(MVP)*

Cluster reports by snapped coordinate or `ApproxAddress` and surface anywhere with ≥3 hits as graduated symbols. A pothole reported once is noise; the same intersection reported 12 times is an advocacy artifact.

- Popup = "rap sheet": count, first/last seen, hazard-type breakdown, photo strip linking to source tweets.
- Toggle to dim singletons so chronic locations dominate the map.
- Eventually: persistent shareable URL per location (`/maps/pittsburgh#loc=...`) so advocates can link to a specific block.

## 2. Hour × DayOfWeek heatmap *(MVP)*

7×24 small-multiple beside the map. Filters the map (and vice versa) so the user can pull on the "weekday 8–10am bike-lane parking" thread. `Hour` and `DayOfWeek` are already in the data.

- Hazard-type filter chips above the heatmap.
- "Brush" a region of the heatmap to filter map markers to that time window.

## 3. Bike infrastructure + crash overlay

Pull Pittsburgh bike-lane geometry (WPRDC) and PennDOT crash data. Color hazards by the infrastructure class they fall on — protected lane vs. painted lane vs. sharrow vs. nothing — to answer "does the infra we paid for actually work?"

- Stretch: per-segment hazard density normalized by lane-mile.

## 4. City-response gap

`CityGovURL` is empty in the current export, so the comparison has to come from elsewhere:

- Match dashcam reports against Pittsburgh's open 311 dataset (WPRDC) by space + time + category.
- Show "reported to city — still open N days" overlays. % of dashcam reports that ever made it into the city queue is itself a story.

## 5. Photo-forward cluster view

Side panel showing a photo grid for the current viewport, sorted by recency or by repeat-offender rank. Turns the map into evidence rather than abstraction. Each report already has `ImageOrVideoFilepath`.

## Later / stretch

- Routing: "safest route from A to B" weighted by hazard density.
- Per-council-district aggregation with per-capita and per-lane-mile rates for advocacy targeting.
- Time-series sparkline per repeat-offender location to show whether a spot is getting worse or better.
- Hazard-name normalization — current data has long-tail free-text entries (e.g. "Cory, the snow and ice needs cleared from the bike lane") that should collapse into the canonical categories before charting.

## Open questions

- Is the geojson regenerated from a live source, and on what cadence? (Affects whether the map should fetch a static file or hit an API.)
- Where should this be hosted — replacing `dashcam.bike/maps/pittsburgh` directly, or in parallel?
- Any other cities in the pipeline? (Affects whether to parameterize by city up-front.)
