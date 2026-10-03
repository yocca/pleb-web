# Spec Delta

## Purpose

Shows everything Pleb knows about one venue's happy hour, where that came from, and how to get there.

## ADDED Requirements

### Requirement: Weekly schedule
The venue page SHALL show the venue's happy hours for each weekday in the venue's local time, in 12-hour format. Consecutive days with identical hours SHALL be grouped ("Mon–Fri 4–7pm"), windows ending after midnight SHALL read as such ("10pm–1am"), and all-day happy hours SHALL read "All day".

#### Scenario: Weekday happy hour
- **WHEN** a venue has 16:00–19:00 on days 1 to 5
- **THEN** the page shows "Mon–Fri 4–7pm"

#### Scenario: Mixed schedule
- **WHEN** a venue has 16:00–19:00 Monday to Friday, 22:00–01:00 Thursday to Saturday, and all day Sunday
- **THEN** the page lists each window and day group, and Sunday shows "All day"

### Requirement: Happening now
The venue page SHALL say whether a happy hour is running now and, if so, when it ends.

#### Scenario: Running now
- **WHEN** the page is opened at 5:30pm on a weekday for a 4–7pm weekday happy hour
- **THEN** the page says the happy hour is on now until 7pm

### Requirement: Deals and conditions
The venue page SHALL list each deal with its price when known and its qualifier, plus any notes that apply to the whole happy hour.

#### Scenario: Deals listed
- **WHEN** the happy hour has "Draft beer, $6" and "Oysters, $1.50 each" with the note "bar area only"
- **THEN** both deals and the note are shown

### Requirement: Source and freshness
For each happy hour, the venue page SHALL say where it came from (the venue's website, Pleb's own visit, a user photo or the owner), link to the source page when there is one, and show when it was last checked. Unverified information SHALL be labeled so people know to double-check it. The page SHALL NOT show menu images or copied menu text.

#### Scenario: From the venue's website
- **WHEN** a happy hour came from `https://venue.example/happy-hour`, checked on October 3, and is unverified
- **THEN** the page says it is from the venue's website, links to that page, shows "checked Oct 3", and suggests confirming with the venue

### Requirement: Directions
The venue page SHALL offer directions to the venue in Google Maps and in Apple Maps as plain links that open those apps or sites, without embedding either.

#### Scenario: Directions links
- **WHEN** the user chooses "Directions in Google Maps"
- **THEN** Google Maps opens with directions to the venue's coordinates and name

### Requirement: Venue details and credit
The venue page SHALL show the venue's address, website and phone when known, and the venue's data attribution (for example "© OpenStreetMap contributors").

#### Scenario: Attribution
- **WHEN** a venue's data came from OpenStreetMap
- **THEN** its page shows "© OpenStreetMap contributors"

### Requirement: Venue without a listed happy hour
A venue reachable by link but with no visible happy hour SHALL still have a page, which says no happy hour is listed yet and still shows its details and directions.

#### Scenario: No happy hour
- **WHEN** the user opens the page of a venue with no visible happy hours
- **THEN** the page says no happy hour is listed and shows the address and directions

### Requirement: Unknown venue
A venue page for an id pleb-api does not know SHALL return a not-found page with HTTP status 404 and a link back to the home page.

#### Scenario: Bad link
- **WHEN** the user opens `/venues/does-not-exist`
- **THEN** the response status is 404 and the page links home
