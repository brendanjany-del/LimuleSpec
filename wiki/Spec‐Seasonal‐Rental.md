## ADDED Requirements

### Requirement: Create a seasonal rental contract
The system SHALL allow the property owner to create a seasonal rental contract for a seasonal property. The contract MUST include: check-in date, check-out date, number of guests, rental price, cleaning fee, and the tenant's identity (name, email, phone). The check-out date MUST be after the check-in date. The dates MUST NOT overlap with existing confirmed contracts for the same property.

#### Scenario: Successful contract creation
- **WHEN** the owner fills in all required fields with valid data and non-overlapping dates
- **THEN** the system creates the contract in "draft" status and displays it on the property page

#### Scenario: Overlapping dates with existing contract
- **WHEN** the owner creates a contract with dates that overlap an existing confirmed contract
- **THEN** the system rejects the creation and displays the conflicting reservation dates

#### Scenario: Check-out before check-in
- **WHEN** the owner enters a check-out date before the check-in date
- **THEN** the system rejects the input and requests valid dates

### Requirement: Customize contract conditions and annexes
The system SHALL allow the owner to add custom conditions (house rules, cancellation policy, etc.) and annexes (inventory list, access instructions) to each seasonal contract. The owner MUST be able to save reusable condition templates.

#### Scenario: Add custom conditions to contract
- **WHEN** the owner adds custom text conditions to a draft contract
- **THEN** the system saves the conditions and includes them in the generated contract PDF

#### Scenario: Save a reusable condition template
- **WHEN** the owner saves a set of conditions as a template with a name
- **THEN** the system stores the template and makes it available for future contracts

#### Scenario: Apply saved template to new contract
- **WHEN** the owner selects a saved condition template while creating a contract
- **THEN** the system pre-fills the conditions section with the template content, editable before saving

### Requirement: Send contract for signature
The system SHALL allow the owner to send a draft contract to the tenant for electronic signature. The system SHALL generate a PDF of the contract, send an email to the tenant with a unique signing link, and track the signing status.

#### Scenario: Successful contract sending
- **WHEN** the owner clicks "Send for signature" on a complete draft contract
- **THEN** the system generates the PDF, sends an email to the tenant, and updates the contract status to "pending-signature"

#### Scenario: Tenant signs the contract
- **WHEN** the tenant clicks the signing link, reviews the contract, and draws their signature
- **THEN** the system captures the signature (SignaturePad), embeds it in the PDF (pdf-lib), records the proof file (timestamp, IP, hash SHA-256), and updates the contract status to "signed"

#### Scenario: Owner counter-signs
- **WHEN** the tenant has signed and the owner clicks "Counter-sign"
- **THEN** the system captures the owner's signature, finalizes the PDF with both signatures, and updates the contract status to "confirmed"

### Requirement: Manage seasonal contract lifecycle
The system SHALL track seasonal contracts through the following statuses: draft → pending-signature → signed → confirmed → in-progress → completed → archived. The owner MUST be able to cancel a contract at any stage before "in-progress".

#### Scenario: Cancel a draft contract
- **WHEN** the owner cancels a contract in "draft" status
- **THEN** the system marks the contract as "cancelled" and frees the reserved dates

#### Scenario: Contract automatically moves to in-progress
- **WHEN** the current date reaches the check-in date of a confirmed contract
- **THEN** the system automatically updates the status to "in-progress"

#### Scenario: Contract automatically completes
- **WHEN** the current date passes the check-out date of an in-progress contract
- **THEN** the system automatically updates the status to "completed"

### Requirement: View seasonal rental history
The system SHALL display a list of all seasonal contracts for a property, filterable by status (active, completed, cancelled) and date range.

#### Scenario: Filter contracts by status
- **WHEN** the owner selects the "completed" filter on a seasonal property
- **THEN** the system displays only contracts with "completed" or "archived" status

#### Scenario: View contract details
- **WHEN** the owner clicks on a contract in the list
- **THEN** the system displays the full contract details including dates, price, tenant info, conditions, signature status, and associated documents
