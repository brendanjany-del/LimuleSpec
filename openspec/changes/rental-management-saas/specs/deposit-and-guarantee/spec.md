## ADDED Requirements

### Requirement: Collect security deposit via bank transfer for classic rental
The system SHALL allow the owner to record a security deposit payment for a classic rental lease. The deposit amount MUST match the lease terms (max 1 month rent excluding charges for unfurnished, max 2 months for furnished). The system SHALL track deposit status: pending, received, held, partially returned, fully returned.

#### Scenario: Record deposit received
- **WHEN** the owner records that the tenant has paid the security deposit via bank transfer
- **THEN** the system marks the deposit as "received" with the date and amount, and links it to the lease

#### Scenario: Deposit amount exceeds legal maximum
- **WHEN** the owner tries to record a deposit amount exceeding the legal maximum for the property type
- **THEN** the system displays a warning indicating the legal maximum and prevents saving

### Requirement: Collect deposit or credit card hold for seasonal rental
The system SHALL allow the owner to choose between collecting a cash/transfer deposit or placing a credit card hold (empreinte CB) for seasonal rentals. For credit card holds, the system SHALL use Stripe Setup Intents to authorize without capturing.

#### Scenario: Create credit card hold
- **WHEN** the owner requests a credit card hold and the tenant enters their card details via the Stripe-hosted form
- **THEN** the system creates a Setup Intent, confirms the card is valid, and records the hold linked to the seasonal contract

#### Scenario: Collect cash/transfer deposit for seasonal rental
- **WHEN** the owner records a cash or transfer deposit for a seasonal contract
- **THEN** the system records the deposit amount, date, and payment method, linked to the contract

### Requirement: Return security deposit for classic rental
The system SHALL enforce the legal deposit return rules: the owner MUST return the deposit within 1 month after check-out if no degradation, or within 2 months if degradations are noted. The system SHALL allow the owner to deduct amounts for documented damages (referencing the check-out inspection).

#### Scenario: Full deposit return without degradation
- **WHEN** the check-out inspection shows no degradation and the owner initiates deposit return
- **THEN** the system generates a deposit return document with the full amount and records the return date

#### Scenario: Partial deposit return with deductions
- **WHEN** the owner initiates deposit return with deductions for documented damages
- **THEN** the system requires itemized deductions with justification, generates a deduction summary document, and records the partial return

#### Scenario: Deposit return deadline reminder
- **WHEN** the deposit has not been returned within 3 weeks of check-out
- **THEN** the system sends a reminder to the owner with the legal deadline

### Requirement: Release credit card hold for seasonal rental
The system SHALL allow the owner to release a credit card hold at the end of a seasonal stay, or capture a partial/full amount if damages occurred.

#### Scenario: Release hold without capture
- **WHEN** the owner releases the credit card hold after an uneventful stay
- **THEN** the system cancels the Setup Intent and records the hold as "released"

#### Scenario: Capture partial amount from hold
- **WHEN** the owner captures a partial amount from the credit card hold for documented damages
- **THEN** the system creates a Payment Intent for the specified amount, charges the card, and records the capture with justification

### Requirement: Support Visale and Locapass guarantee schemes
The system SHALL provide information and guided workflows for Visale (Action Logement guarantee) and Locapass (deposit advance) schemes. The system SHALL store the guarantee reference number and status for each lease.

#### Scenario: Enable Visale for a lease
- **WHEN** the owner activates Visale support on a classic lease
- **THEN** the system displays the Visale application steps, relevant URLs, required documents, and a field to store the Visale visa number once obtained

#### Scenario: Enable Locapass for a lease
- **WHEN** the owner activates Locapass support on a classic lease
- **THEN** the system displays the Locapass application steps, relevant URLs, and a field to store the Locapass reference number

#### Scenario: Visale guarantee reminder
- **WHEN** a lease with Visale support has an unpaid rent for more than 30 days
- **THEN** the system reminds the owner to file a claim with Visale and provides the claim procedure steps and URLs
