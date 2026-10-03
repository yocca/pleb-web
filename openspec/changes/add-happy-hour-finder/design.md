# Design

## Context

pleb-api (merged) serves `GET /v1/venues` (venues near a point with at least one visible happy hour, filterable by `active_at` or `day`, cursor-paged, nearest first) and `GET /v1/venues/{id}`. Happy hours come with day, local `HH:MM` start and end (end before start means past midnight), `all_day`, deals, notes, confidence, `verified`, and provenance (`source_kind`, `source_url`, `observed_at`). pleb-api's compose file already has a `web` service that builds `../pleb-web` with `PLEB_API_URL=http://api:8080` on port 3000. This repo is empty apart from OpenSpec.

## Goals / Non-Goals

**Goals:**
- A fast, mobile-first page: most people will open it on a phone on the street.
- Correct times: a venue's happy hour is evaluated in its own timezone, not the visitor's.
- No API keys, accounts or billing for maps or directions.

**Non-Goals:**
- Writing anything to pleb-api (uploads and claims are later changes).
- Server-side caching beyond Next.js defaults, analytics, SEO tuning, PWA/offline.

## Decisions

**Next.js 16 App Router, TypeScript, React Server Components.** The home page and the venue page are server components that call pleb-api directly. The interactive parts (map, filters, location) are client components. Searches after the first load go through one Route Handler, `GET /api/venues`, which validates its query and forwards it to pleb-api, so the browser never sees `PLEB_API_URL`. Alternative: Server Actions for search. A GET route is cacheable, simpler to test, and maps 1:1 to the API.

**Typed client from a contract snapshot.** `openapi/pleb-api.yaml` is a copy of pleb-api's `openapi.yaml` with a source-hash header. `scripts/sync-openapi.sh ../pleb-api` refreshes it, and a test fails when it's stale next to a pleb-api checkout (the same pattern as pleb-agent's schema snapshot). `openapi-typescript` generates types at build time, and `openapi-fetch` is the client. The client lives in a `server-only` module, so importing it from client code fails the build. Alternative: hand-written types. They drift silently.

**Map: MapLibre GL JS with OpenFreeMap vector tiles.** v1 uses no Google data (see the sourcing decision), so any map works. MapLibre is open source and needs no key. OpenFreeMap serves OpenStreetMap-based tiles free with no key or registration, under its attribution terms. The style URL is configurable (`NEXT_PUBLIC_MAP_STYLE_URL`), so swapping to MapTiler, Stadia or self-hosted tiles is a config change. Alternatives: Leaflet with raster tiles from tile.openstreetmap.org, but OSM's tile policy discourages app traffic. Google Maps JS needs a key and billing.

**Location and URL state.** The URL holds `lat`, `lng`, `r` (radius in meters) and `when` (`now`, `0`–`6`, or `any`). With no `lat`/`lng`, the server renders West Village results immediately, and the client then asks for geolocation and, if granted, replaces the URL and results. People see results without waiting on a permission prompt, and a 5-second geolocation timeout falls back silently. "Search this area" sets `lat`/`lng` to the map center and `r` to the distance from center to the farthest visible corner, capped at 5,000 m.

**"Now" is evaluated by the API.** The Route Handler passes `active_at=<server now>`. For display, a small pure module (`lib/hours.ts`) works out "on now until X", that day's hours, day grouping ("Mon–Fri") and 12-hour formatting from the happy hour rows and the venue's IANA timezone with `Intl.DateTimeFormat`. It is the one place this logic lives, and it gets table-driven tests including past-midnight windows (a Friday 10pm–1am window is "on now" at 12:30am Saturday) and DST days.

**Directions links.** Google: `https://www.google.com/maps/dir/?api=1&destination=<lat>,<lng>`. Coordinates rather than a name search, because v1 has no Google place ids and a name search can land on the wrong place. Apple: a unified Maps URL, `https://maps.apple.com/directions?destination=<lat>,<lng>`, with the venue name passed where Apple's link format supports it (checked against Apple's Map Links docs during implementation). Both are plain `<a>` links that open the app on phones, built by one tested helper.

**Freshness and trust labels.** Each happy hour shows its source ("From the venue's website", "Checked by Pleb in person", "From a customer photo", "From the owner"), a link to `source_url`, and "checked <date>" from `observed_at`. Unverified rows get "Confirm with the venue before you go". The API already hides rows below 0.6 confidence, so the page shows confidence only as that label, not a number.

**Styling: Tailwind CSS 4.** Small, no runtime, and good for a mobile-first layout: map on top and list below on phones, list beside the map on wide screens.

**Testing.**
- Vitest unit tests for `lib/hours.ts`, URL-state parsing, the directions link builders and the Route Handler's validation.
- Playwright end-to-end tests run the production build against a fake pleb-api: a tiny Node HTTP server serving fixture JSON that matches the OpenAPI schema and recording queries. They cover the spec scenarios: denied location falls back to the West Village, the filters change the API query, a marker click highlights the list, "Search this area", reload keeps state, the empty and error states, venue page contents, directions hrefs, 404, and that the browser never calls the fake API's origin. Geolocation and the clock are controlled through Playwright's `geolocation`, `permissions` and `clock`.
- Map tiles are blocked in tests, and the map renders without them.

**Packaging.** `output: "standalone"` and a multi-stage Dockerfile (node:22-slim), so pleb-api's compose `web` profile works unchanged. CI runs lint, typecheck, unit tests, build, the Playwright tests (Chromium) and the Docker build.

## Risks / Trade-offs

- [OpenFreeMap is a free community service with no SLA] → Style URL is configuration. Moving to a paid provider or self-hosted PMTiles is a config change.
- [Few venues have data yet, so the map looks empty] → The empty states point to "Any time" and the West Village. Seed data in pleb-api covers development.
- [Geolocation outside NYC gives empty results] → The empty state offers to jump to the West Village.
- [Contract drift between the snapshot and pleb-api] → Hash check in tests plus the sync script, and types are generated at build so a breaking change fails typecheck.
- [Visitor clock or timezone differs from the venue's] → All "now" logic uses the venue's timezone, and the server's clock for API queries.

## Open Questions

- Whether to show venues without any listed happy hour (useful for "help us fill this in" once uploads exist). The API doesn't return them from search today, so it would need a pleb-api change. It can wait for the uploads change.
