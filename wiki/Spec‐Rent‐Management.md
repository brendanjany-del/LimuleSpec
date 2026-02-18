## ADDED Requirements

### Requirement: Automatic rent notice generation (avis d'échéance)
The system SHALL automatically generate a rent notice (avis d'échéance) for each active classic lease before the rent due date. The notice MUST be generated 5 days before the due date. It MUST include: tenant name, property address, rent amount, charges amount, total due, payment due date, and payment method.

#### Scenario: Automatic notice generation
- **WHEN** an active lease exists and the rent due date is 5 days away
- **THEN** the system generates a rent notice PDF and sends it to the tenant via email

#### Scenario: Notice for lease with charge provisions
- **WHEN** a rent notice is generated for a lease with "provisions sur charges"
- **THEN** the notice displays rent amount, provision amount, and total separately

#### Scenario: Notice for lease with fixed charges
- **WHEN** a rent notice is generated for a lease with "forfaitaires" charges
- **THEN** the notice displays rent amount, fixed charges, and total

### Requirement: Rent receipt generation (quittance de loyer)
The system SHALL generate a rent receipt (quittance de loyer) when a monthly rent payment is recorded as fully paid. The receipt MUST include: tenant name, property address, period covered, rent amount, charges amount, total paid, and payment date. The receipt MUST comply with French legal requirements (loi du 6 juillet 1989, art. 21).

#### Scenario: Automatic receipt after full payment
- **WHEN** a rent payment for a given month is recorded as fully paid
- **THEN** the system generates a quittance PDF and makes it available to both owner and tenant

#### Scenario: Partial payment recorded
- **WHEN** a partial rent payment is recorded for a given month
- **THEN** the system generates a partial receipt (reçu de paiement partiel) indicating the amount paid and the remaining balance

#### Scenario: Tenant requests receipt
- **WHEN** a tenant requests a receipt for a past month
- **THEN** the system provides the previously generated receipt for download

### Requirement: Rent payment tracking
The system SHALL maintain a month-by-month rent tracking view for each active lease. Each month MUST show: amount due, amount paid, payment date, status (paid, partial, overdue, pending), and links to the notice and receipt.

#### Scenario: View rent tracking for a lease
- **WHEN** the owner opens the rent tracking view for a lease
- **THEN** the system displays a table with one row per month showing due amount, paid amount, status, and document links

#### Scenario: Mark rent as manually paid
- **WHEN** the owner records a manual rent payment (cash, check, bank transfer outside Stripe)
- **THEN** the system updates the month's status, records the payment method and date, and generates the receipt

### Requirement: Late payment detection and alerts
The system SHALL detect unpaid rent after the due date and send alerts to both the owner and tenant. The system MUST allow configurable grace periods (default: 5 days after due date).

#### Scenario: Rent overdue after grace period
- **WHEN** a rent payment has not been recorded 5 days after the due date
- **THEN** the system marks the month as "overdue", sends a reminder email to the tenant, and notifies the owner

#### Scenario: Owner adjusts grace period
- **WHEN** the owner changes the grace period for a lease from 5 to 10 days
- **THEN** the system applies the new grace period for future overdue detections on that lease

### Requirement: Annual rent summary
The system SHALL generate an annual rent summary for each lease, listing all months with payment status, totals collected, and any outstanding amounts. This summary SHALL be available for download as PDF.

#### Scenario: Generate annual summary
- **WHEN** the owner requests an annual summary for a lease and a given year
- **THEN** the system generates a PDF showing 12 months of rent data with totals and payment statuses
