# Proposal

## Why

pleb-api serves venues with happy hours and the crawler fills them in, but there's no way to see the results. A first web app lets Andrew check real West Village data as it arrives, and it's the product itself: find a happy hour near you, now or later this week.

## What Changes

- New Next.js (App Router, TypeScript) app in this repo.
- A home page with a map and a list of nearby venues that have happy hours. It starts at the user's location if they allow it, otherwise at the West Village. Filters: "happening now" (the default), a chosen weekday, or any time. Moving the map searches the new area.
- A venue page showing the weekly happy hour schedule in the venue's local time, the deals, where the information came from (the source link and when it was checked), and "Directions" links to Google Maps and Apple Maps.
- OpenStreetMap credit on every page that shows venue data or map tiles.
- All pleb-api calls happen on the server. The browser only talks to this app, and the API stays internal.
- TypeScript types generated from a snapshot of pleb-api's `openapi.yaml`, with a sync script, like pleb-agent's schema snapshot.
- Tests (unit and Playwright end-to-end against a fake API), a Dockerfile for pleb-api's compose `web` profile, and CI.

Out of scope: accounts, photo uploads, business claims, a native app, and search by venue name. Uploads and claims come in later changes.

## Capabilities

### New Capabilities
- `venue-discovery`: finding venues with happy hours near a location: the starting location, the map and list, the time filters, the area search, and what each result shows.
- `venue-page`: one venue's page: its schedule, deals, source and freshness, directions links, and the not-found and no-happy-hour cases.

### Modified Capabilities
- None.

## Impact

- New code in pleb-web only; pleb-api and pleb-agent are unchanged. The app reads `PLEB_API_URL`, which pleb-api's compose `web` service already sets.
- New dependencies: next, react, maplibre-gl, openapi-typescript and openapi-fetch, plus dev dependencies vitest and @playwright/test.
- Map tiles come from a free, keyless OpenStreetMap-based tile service, so no map API key or billing is needed.
