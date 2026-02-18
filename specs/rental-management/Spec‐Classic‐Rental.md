## ADDED Requirements

### Requirement: Create a classic rental lease (bail)
The system SHALL allow the property owner to create a lease (bail de location) for a classic property. The lease MUST include: tenant identity (name, date of birth, email, phone), lease start date, rent amount, charges amount or provision, payment frequency (monthly), deposit amount, and property description. The lease content MUST comply with the minimum requirements of loi ALUR (articles obligatoires).

#### Scenario: Successful lease creation for furnished property
- **WHEN** the owner of a classic-furnished property fills in all required lease fields
- **THEN** the system creates a lease in "draft" status with the mandatory furnished-specific clauses included

#### Scenario: Successful lease creation for unfurnished property
- **WHEN** the owner of a classic-unfurnished property fills in all required lease fields
- **THEN** the system creates a lease in "draft" status with the mandatory unfurnished-specific clauses (bail de 3 ans minimum)

#### Scenario: Missing required lease fields
- **WHEN** the owner submits a lease with missing required fields
- **THEN** the system rejects the submission and highlights all missing mandatory fields

### Requirement: Send lease for electronic signature
The system SHALL allow the owner to send the lease to the tenant for signature using the built-in electronic signature system. Both the owner and tenant MUST sign. The system SHALL generate a proof file for each signature.

#### Scenario: Complete lease signing flow
- **WHEN** the owner sends the lease for signature, the tenant signs, and the owner counter-signs
- **THEN** the system generates a finalized PDF with both signatures, stores the proof files, and activates the lease

#### Scenario: Tenant does not sign within 7 days
- **WHEN** a lease has been pending tenant signature for more than 7 days
- **THEN** the system sends a reminder email to the tenant and notifies the owner

### Requirement: Manage check-in inspection (état des lieux d'entrée)
The system SHALL allow the owner to create a check-in inspection report linked to an active lease. The report MUST include: date, room-by-room condition assessment (walls, floor, ceiling, fixtures), overall condition rating per room, meter readings (water, electricity, gas), and optional photos per room.

#### Scenario: Create check-in inspection
- **WHEN** the owner creates a check-in inspection for an active lease with room assessments and meter readings
- **THEN** the system saves the report, generates a PDF, and links it to the lease

#### Scenario: Add photos to inspection
- **WHEN** the owner uploads photos for specific rooms during the inspection
- **THEN** the system stores the photos and includes them in the generated PDF report

#### Scenario: Tenant validates check-in inspection
- **WHEN** the tenant reviews and signs the check-in inspection via the signing link
- **THEN** the system records the tenant's approval and marks the inspection as "validated"

### Requirement: Manage check-out inspection (état des lieux de sortie)
The system SHALL allow the owner to create a check-out inspection report linked to an active lease. The report MUST pre-fill room data from the check-in inspection for comparison. The system SHALL highlight differences between entry and exit conditions.

#### Scenario: Create check-out inspection with comparison
- **WHEN** the owner creates a check-out inspection for a lease that has a check-in inspection
- **THEN** the system pre-fills the room list from the check-in report and displays entry conditions alongside for comparison

#### Scenario: Highlight degradations
- **WHEN** the owner marks a room condition as worse than the check-in condition
- **THEN** the system highlights the degradation and allows adding a note and photos as evidence

### Requirement: Manage charges (provisions and regularization)
The system SHALL allow the owner to define whether charges are "forfaitaires" (fixed) or "provisions sur charges" (estimated with annual regularization). For provisions, the system SHALL track actual charges and allow annual regularization.

#### Scenario: Define fixed charges
- **WHEN** the owner sets charges as "forfaitaires" with a monthly amount
- **THEN** the system includes the fixed charge amount in each rent notice with no regularization needed

#### Scenario: Annual charge regularization
- **WHEN** the owner enters the actual annual charges for a lease with provisions
- **THEN** the system calculates the difference (actual vs. provisioned), generates a regularization document, and adjusts the next rent notice accordingly

### Requirement: Generate standard letters (courriers types)
The system SHALL provide templates for common landlord letters: rent increase notice, lease termination notice (congé), late payment reminder (mise en demeure), charge regularization letter, and rent receipt request. Each letter MUST be pre-filled with lease and tenant data.

#### Scenario: Generate a late payment reminder
- **WHEN** the owner selects "mise en demeure" for a lease with unpaid rent
- **THEN** the system generates a pre-filled letter with tenant name, address, unpaid amounts, and dates, in PDF format

#### Scenario: Generate a lease termination notice
- **WHEN** the owner selects "congé" for an active lease
- **THEN** the system generates a termination notice with the required legal notice period (6 months for unfurnished, 3 months for furnished) pre-calculated from the lease end date

### Requirement: Manage classic rental lease lifecycle
The system SHALL track leases through the following statuses: draft → pending-signature → active → termination-notice → ended → archived. The system MUST enforce minimum lease durations (3 years unfurnished, 1 year furnished).

#### Scenario: Lease becomes active after signing
- **WHEN** both parties have signed the lease
- **THEN** the system updates the status to "active" and starts generating rent notices from the lease start date

#### Scenario: Owner sends termination notice
- **WHEN** the owner initiates a termination notice on an active lease
- **THEN** the system calculates the legal notice period end date and updates the status to "termination-notice"
