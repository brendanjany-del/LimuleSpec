## ADDED Requirements

### Requirement: Display reservation calendar for seasonal property
The system SHALL display an interactive calendar view for each seasonal property showing all reservations. Each reservation MUST be color-coded by status: confirmed (green), pending-signature (orange), draft (gray), blocked (red). The calendar MUST support month and week views.

#### Scenario: View monthly calendar
- **WHEN** the owner opens the calendar for a seasonal property
- **THEN** the system displays a monthly view with all reservations color-coded by status

#### Scenario: Click on a reservation
- **WHEN** the owner clicks on a reservation block in the calendar
- **THEN** the system opens a popover or side panel showing the contract summary (tenant name, dates, price, status)

### Requirement: Block dates on calendar
The system SHALL allow the owner to manually block date ranges on a seasonal property calendar. Blocked dates MUST prevent contract creation for overlapping periods. Each blocked period MUST have an optional reason (personal use, maintenance, etc.).

#### Scenario: Block a date range
- **WHEN** the owner selects a date range and clicks "Block dates"
- **THEN** the system creates a blocked period displayed in red on the calendar and prevents new contracts for those dates

#### Scenario: Unblock dates
- **WHEN** the owner removes a blocked period
- **THEN** the system frees the dates and they become available for new contracts

#### Scenario: Attempt to create contract on blocked dates
- **WHEN** the owner tries to create a seasonal contract that overlaps with blocked dates
- **THEN** the system rejects the creation and indicates which dates are blocked

### Requirement: Import external calendar via iCal URL
The system SHALL allow the owner to add one or more external iCal URLs (Airbnb, Booking, Abritel, etc.) for a seasonal property. The system SHALL periodically fetch and parse the iCal feed to display external reservations on the calendar.

#### Scenario: Add Airbnb iCal URL
- **WHEN** the owner adds an Airbnb iCal URL for a property
- **THEN** the system fetches the feed, parses the events, and displays them on the calendar with an "Airbnb" label and distinct color

#### Scenario: External reservation conflicts with local dates
- **WHEN** an external iCal feed contains a reservation that overlaps with locally available dates
- **THEN** the system displays the external reservation on the calendar and warns the owner if they try to create a local contract for those dates

#### Scenario: iCal feed fetch failure
- **WHEN** the system fails to fetch an iCal URL (network error, invalid URL)
- **THEN** the system keeps the last known data, displays the last sync time, and notifies the owner of the fetch failure

### Requirement: Automatic iCal synchronization
The system SHALL synchronize external iCal feeds every 15 minutes via a background job. The system SHALL store the last successful sync timestamp for each feed. The owner MUST be able to trigger a manual sync at any time.

#### Scenario: Automatic periodic sync
- **WHEN** 15 minutes have elapsed since the last sync for an iCal feed
- **THEN** the system fetches the feed, updates the external reservations, and records the sync timestamp

#### Scenario: Manual sync trigger
- **WHEN** the owner clicks "Sync now" for an external calendar
- **THEN** the system immediately fetches the feed, updates the calendar, and displays the new sync timestamp

### Requirement: Export local calendar as iCal feed
The system SHALL provide a unique iCal export URL for each seasonal property. This URL MUST return all confirmed and blocked dates in iCal format so the owner can import it into Airbnb, Booking, or other platforms to avoid double bookings.

#### Scenario: Generate iCal export URL
- **WHEN** the owner requests the iCal export URL for a property
- **THEN** the system generates a unique, non-guessable URL and displays it with copy and instructions for importing into external platforms

#### Scenario: External platform fetches the iCal feed
- **WHEN** an external platform (Airbnb) fetches the iCal export URL
- **THEN** the system returns a valid iCal file containing all confirmed reservations and blocked dates for the property
