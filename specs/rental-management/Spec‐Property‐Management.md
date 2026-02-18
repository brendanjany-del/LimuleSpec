## ADDED Requirements

### Requirement: Create a new property
The system SHALL allow an authenticated user to create a new property by providing a name, an address, and a rental type. The rental type MUST be one of: "seasonal", "classic-furnished", or "classic-unfurnished".

#### Scenario: Successful property creation
- **WHEN** an authenticated user submits a valid name, address, and rental type
- **THEN** the system creates the property associated with the user and redirects to the property detail page

#### Scenario: Property creation with missing required fields
- **WHEN** an authenticated user submits the form with missing name or address
- **THEN** the system rejects the submission and highlights the missing fields

### Requirement: List user properties
The system SHALL display all properties belonging to the authenticated user on the dashboard. Each property card MUST show the name, address, rental type, and current occupancy status.

#### Scenario: User with multiple properties
- **WHEN** an authenticated user with 3 properties accesses the dashboard
- **THEN** the system displays 3 property cards with name, address, rental type, and occupancy status for each

#### Scenario: User with no properties
- **WHEN** an authenticated user with no properties accesses the dashboard
- **THEN** the system displays an empty state with a call-to-action to create the first property

### Requirement: Edit property details
The system SHALL allow the property owner to edit the property name, address, and characteristics. The rental type MUST NOT be changed after creation (a new property must be created instead).

#### Scenario: Successful property edit
- **WHEN** the property owner updates the property name or address
- **THEN** the system saves the changes and displays a confirmation

#### Scenario: Attempt to change rental type
- **WHEN** the property owner attempts to change the rental type
- **THEN** the system prevents the change and displays a message explaining that a new property must be created

### Requirement: Manage property address
The system SHALL store a complete French address for each property: street number, street name, complement, postal code, city. The postal code MUST be a valid 5-digit French postal code.

#### Scenario: Valid address entry
- **WHEN** the owner enters a complete address with a valid postal code
- **THEN** the system saves the address and displays it formatted on the property page

#### Scenario: Invalid postal code
- **WHEN** the owner enters a postal code that is not 5 digits
- **THEN** the system rejects the input and requests a valid French postal code

### Requirement: Manage furniture and accessories inventory
The system SHALL allow the owner of a seasonal or classic-furnished property to manage an inventory of furniture and accessories. Each item MUST have a name, a quantity, a condition (new, good, fair, poor), and an optional photo. This inventory SHALL NOT be available for classic-unfurnished properties.

#### Scenario: Add furniture item to seasonal property
- **WHEN** the owner of a seasonal property adds a furniture item with name, quantity, and condition
- **THEN** the system saves the item and displays it in the property inventory list

#### Scenario: Attempt to manage inventory on unfurnished property
- **WHEN** the owner of a classic-unfurnished property tries to access the inventory section
- **THEN** the system hides or disables the inventory section with a message indicating it is not applicable

#### Scenario: Upload photo for furniture item
- **WHEN** the owner uploads a photo for a furniture item
- **THEN** the system stores the photo and displays a thumbnail in the inventory list

### Requirement: Delete a property
The system SHALL allow the owner to delete a property only if it has no active contracts or ongoing rentals. Deletion MUST require explicit confirmation.

#### Scenario: Delete property with no active contracts
- **WHEN** the owner confirms deletion of a property with no active contracts
- **THEN** the system soft-deletes the property (archived) and removes it from the dashboard

#### Scenario: Attempt to delete property with active contract
- **WHEN** the owner attempts to delete a property that has an active contract
- **THEN** the system blocks deletion and lists the active contracts that must be terminated first
