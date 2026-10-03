# Spec Delta

## Purpose

Lets people find bars and restaurants with a happy hour near them, either happening now or on a chosen day, on a map and in a list.

## ADDED Requirements

### Requirement: Starting location
The home page SHALL search around the user's current location when the browser shares it. When location is denied, unavailable or slow, it SHALL search around the West Village (40.7336, -74.0027) and say that it did.

#### Scenario: Location shared
- **WHEN** the user allows location access at 40.7350, -74.0010
- **THEN** the map centers there and the results are venues near that point

#### Scenario: Location denied
- **WHEN** the user denies location access
- **THEN** the map centers on the West Village, the results are West Village venues, and the page says it is showing the West Village

### Requirement: Time filter
The home page SHALL filter results by time: "Now" (a happy hour running at this moment, the default), a chosen weekday (a happy hour on that day), or "Any time". Times SHALL be evaluated in each venue's own timezone.

#### Scenario: Default is now
- **WHEN** the user opens the home page on a Tuesday at 5:30pm New York time
- **THEN** only venues with a happy hour running at 5:30pm on Tuesday are shown

#### Scenario: Chosen day
- **WHEN** the user picks Friday
- **THEN** venues with any happy hour on Friday are shown, whether or not one is running now

#### Scenario: Any time
- **WHEN** the user picks "Any time"
- **THEN** every nearby venue with a listed happy hour is shown

### Requirement: Map and list show the same results
The home page SHALL show results both as map markers and as a list ordered nearest first. Selecting a list entry SHALL highlight its marker, and selecting a marker SHALL highlight its list entry.

#### Scenario: Selecting a marker
- **WHEN** the user taps a venue's marker
- **THEN** that venue's list entry is highlighted and scrolled into view, with a link to its venue page

### Requirement: Result summary
Each result SHALL show the venue name, its straight-line distance, and its happy hour for the filtered time: the end time if one is running now ("Until 7pm"), otherwise that day's hours ("4–7pm"). It SHALL show the first listed deal when there is one.

#### Scenario: Running now
- **WHEN** a venue's happy hour runs 4pm–7pm and it is 5:30pm
- **THEN** its result says it is on now until 7pm

#### Scenario: Past midnight
- **WHEN** a venue's late happy hour runs 10pm–1am and the user picks that day
- **THEN** its result shows "10pm–1am"

### Requirement: Search this area
When the user moves or zooms the map, the page SHALL offer to search the visible area, and searching SHALL replace the results with venues around the new map center within the visible area, capped at 5 km.

#### Scenario: Panning to another neighborhood
- **WHEN** the user pans the map and chooses "Search this area"
- **THEN** the results are venues around the new center, nearest first

### Requirement: Shareable state
The location and time filter SHALL be reflected in the page URL, so reloading, going back, or sharing the link shows the same search.

#### Scenario: Reload
- **WHEN** the user picks Friday, searches another area, and reloads the page
- **THEN** the same area and the Friday filter are shown, without asking for location again

### Requirement: Empty and error states
The page SHALL explain why there are no results: nothing running now (offering "Any time"), or no covered venues in the area (offering to jump to the West Village). When pleb-api cannot be reached, it SHALL say so and offer to retry, without showing stale or partial results as current.

#### Scenario: Nothing on now
- **WHEN** no nearby happy hour is running at the current time
- **THEN** the page says none is on right now and offers to show any time

#### Scenario: API down
- **WHEN** pleb-api returns an error or does not respond
- **THEN** the page shows an error message with a retry action

### Requirement: Server-side API access
The browser SHALL NOT call pleb-api directly. All pleb-api requests SHALL be made by the web app's server, so pleb-api needs no public address and no CORS.

#### Scenario: Browser traffic
- **WHEN** a user loads the home page and searches another area
- **THEN** every request the browser makes goes to the web app's own origin or the map tile service

### Requirement: Map credit
The map SHALL credit OpenStreetMap contributors and the tile provider wherever map tiles or venue data are shown.

#### Scenario: Attribution visible
- **WHEN** the home page shows the map
- **THEN** "© OpenStreetMap contributors" is visible on it
