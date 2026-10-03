# Tasks

## 1. Project scaffolding

- [ ] 1.1 Create the Next.js 16 app (App Router, TypeScript, Tailwind CSS 4, ESLint, `output: "standalone"`), with `npm run lint`, `typecheck`, `test` and `build` scripts; verify each script passes on the empty app
- [ ] 1.2 Add a Dockerfile (node:22-slim, multi-stage, standalone output) and a GitHub Actions workflow running lint, typecheck, unit tests, build, Playwright (Chromium) and the Docker build; verify the image serves `/` and CI passes on the PR
- [ ] 1.3 Add the fake pleb-api for tests (a Node HTTP server serving fixture JSON and recording requests) and the Playwright config that starts it and the production build with `PLEB_API_URL` pointed at it and map tiles blocked; verify a smoke test loads `/`

## 2. API client

- [ ] 2.1 Add `openapi/pleb-api.yaml` (snapshot with source-hash header) and `scripts/sync-openapi.sh`; verify the script reproduces an identical file from `../pleb-api`, and a test fails when the snapshot is stale next to a pleb-api checkout
- [ ] 2.2 Generate types with openapi-typescript and add a `server-only` openapi-fetch client reading `PLEB_API_URL`; verify typecheck catches a deliberately wrong field and importing the client from a client component fails the build
- [ ] 2.3 Validate the fixture JSON used by the fake API against the OpenAPI schemas; verify a test fails on a fixture missing a required field

## 3. Time and formatting (shared by both pages)

- [ ] 3.1 Implement `lib/hours.ts` (on-now and ends-at in the venue timezone, day hours, day grouping, 12-hour formatting, past-midnight and all-day windows); verify table-driven tests for every venue-page schedule scenario, a Friday 10pm–1am window being on now at 12:30am Saturday, and a DST transition day
- [ ] 3.2 Implement distance formatting, the source labels with "checked <date>", and the Google and Apple directions link builders; verify unit tests for each, with the Apple link format checked against Apple's Map Links documentation

## 4. Home page (venue-discovery)

- [ ] 4.1 Implement URL state (`lat`, `lng`, `r`, `when`) parsing and serialization with West Village defaults and bounds checks; verify unit tests for defaults, invalid values and round-tripping
- [ ] 4.2 Implement the `GET /api/venues` Route Handler (validates input, sets `active_at` for "now" or `day`, forwards to pleb-api, maps errors to a 502 with a stable error body); verify unit tests for the query mapping and the error case
- [ ] 4.3 Build the server-rendered home page with the time filter, the result list (name, distance, happy hour summary, first deal) and the empty and error states; verify Playwright tests for the default "now" query, a chosen day, "Any time", nothing-on-now and API-down scenarios
- [ ] 4.4 Add the MapLibre map with OpenFreeMap tiles, markers, OSM credit, marker/list selection sync and "Search this area" with the 5 km cap; verify Playwright tests for marker selection, searching a panned area, and visible attribution
- [ ] 4.5 Add geolocation (5 s timeout, West Village fallback with a notice, no prompt when the URL already has a location); verify Playwright tests for shared, denied and reload-keeps-state scenarios, and that no browser request goes to the fake API's origin

## 5. Venue page (venue-page)

- [ ] 5.1 Build `/venues/[id]` with the grouped weekly schedule, happening-now line, deals and notes, source and freshness labels, address, website, phone, attribution and directions links; verify Playwright tests for each venue-page scenario using fixture venues
- [ ] 5.2 Handle a venue with no visible happy hours and an unknown id (404 status with a link home); verify Playwright tests for both, including the response status
- [ ] 5.3 Write the README (setup, env vars, running against pleb-api locally and through its compose `web` profile, tests, the OpenAPI snapshot sync); verify each documented command runs as written

## 6. Integration check

- [ ] 6.1 Run the app against pleb-api's compose stack with its seed data (and any crawled venues): open the home page and a venue page in Playwright and check the seeded venues, their schedules and directions links render; verify `docker compose --profile web up` serves the app on :3000
