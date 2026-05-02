# Caveats — Pittsburgh 311 map

Things to flag for the client before this ships, and things to fix in follow-up passes.

## Data caveats

- **`CityGovURL` is empty on every record** in the current export (12,234/12,234 null). Roadmap idea #4 ("did this make it into the city's 311 queue?") cannot be answered from this dataset alone — it requires matching against Pittsburgh's open 311 data on WPRDC by space + time + category.
- **4-year time window without normalization.** A location with "12 reports" could be one event from 2022 or a chronic problem in 2026. The rap-sheet popup's first/last-seen dates are the disambiguator, but a derived "reports per active year" or recency-weighted score would sharpen the headline.
- **Long-tail free-text hazard labels.** Roughly 1,000 of 12,234 reports use ad-hoc names (e.g. "Cory, the snow and ice needs cleared from the bike lane"). The client-side normalizer in `index.html` collapses the obvious cases, but a server-side pass to canonicalize labels at ingestion is the right fix.
- **`ApproxAddress` has spacing/punctuation variants** ("Car parked on sidewalk " with trailing space, etc.), so spatial aggregation uses a snapped-coordinate grid (~11m at Pittsburgh's latitude) rather than address strings. Two reports on opposite sides of a wide intersection may end up in different buckets.

## UI / product caveats

- **Photo grid uses BunnyCDN at `dashcamprod.b-cdn.net`** (`?alt=media&width=N` for image resizing). This is the same CDN the production map uses; if it's ever swapped for direct Firebase storage URLs, both the rap-sheet popup and the viewport photo panel need their `CDN_BASE` constant updated.
- **Video thumbnails use `<video preload="metadata">` first frames** rather than real posters. Most dashcam clips start mid-frame so the poster is usually informative, but a few will look dim or odd. A real fix is generating posters server-side at ingestion.
- **Brush-to-filter is mouse-only.** Touch and keyboard equivalents are not implemented yet.
- **Single city, hard-coded center.** If other cities are coming (the URL `dashcam.bike/maps/pittsburgh` implies they are), the page should be parameterized by city — bbox, center, dataset URL.
- **5MB GeoJSON ships uncompressed.** Fine for a single map page; if reused at scale, switch to gzip on the host or move to a tiled format (PMTiles / vector tiles).

## Zoom behavior

The map enforces `minZoom: 11` and applies zoom-aware visibility rules so the visualization stays legible across scales:

- **z ≤ 11 (regional):** singletons (one-off reports) are auto-hidden regardless of the toggle, and the bike-infrastructure layer is detached even when toggled on. Without these, both layers collapse into an unreadable multicolor blob over Pittsburgh.
- **z = 12–14 (city / neighborhood):** the intended sweet spot. All locations and infra render at full detail.
- **z ≥ 15 (street / parcel):** marker radius scales up so individual singletons are ~6px at z=18 instead of 3px, readable without dominating clusters.

Implications for the client:

- Hiding singletons at z=11 reduces the displayed "locations" count (e.g. 4,448 → 1,676 at default Pittsburgh extents). The map shows the sidebar count *after* singleton-hiding, so users may notice it shift on zoom — that's intentional, not a bug.
- The `minZoom: 11` floor means the map cannot show "all of Allegheny County" or compare Pittsburgh against its suburbs at a glance. If that view is wanted later, the right fix is server-side aggregation to neighborhood polygons (choropleth) — not lowering the zoom floor, which would just bring back the blob.
- Zoom thresholds (11, 12, 15) are hand-tuned to Pittsburgh's data density. Other cities may need different breakpoints; once a second city is loaded, profile it before reusing these constants.

## Scope caveats

- Roadmap items #3–#5 are not implemented yet. The MVP covers #1 and #2 only.
- No automated tests. The single-file build is small enough that visual QA + a deploy preview is reasonable, but if this grows, splitting out the data-shaping logic for unit tests would pay off quickly.
