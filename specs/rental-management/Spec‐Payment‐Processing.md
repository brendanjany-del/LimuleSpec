## ADDED Requirements

### Requirement: Owner Stripe Connect onboarding
The system SHALL allow the property owner to connect their Stripe account via Stripe Connect (Standard accounts). The onboarding MUST be completed before the owner can receive any payments. The system SHALL track the onboarding status.

#### Scenario: Successful Stripe Connect onboarding
- **WHEN** the owner initiates Stripe Connect onboarding and completes the Stripe-hosted form
- **THEN** the system stores the connected account ID, marks the owner as "payment-ready", and displays a confirmation

#### Scenario: Incomplete onboarding
- **WHEN** the owner starts but does not complete the Stripe onboarding
- **THEN** the system keeps the status as "onboarding-incomplete" and provides a link to resume

#### Scenario: Owner without Stripe tries to collect payment
- **WHEN** an owner without a connected Stripe account attempts to request a payment from a tenant
- **THEN** the system blocks the action and prompts the owner to complete Stripe Connect onboarding first

### Requirement: Collect one-time payment from tenant (seasonal stay)
The system SHALL allow the owner to request a one-time payment from a tenant for a seasonal rental. The system SHALL send a payment link to the tenant via email. Payment MUST be processed via Stripe Checkout with the funds directed to the owner's connected Stripe account.

#### Scenario: Successful seasonal payment
- **WHEN** the owner requests payment for a confirmed seasonal contract and the tenant completes payment via the Stripe Checkout link
- **THEN** the system records the payment as "completed", updates the contract payment status, and notifies the owner

#### Scenario: Tenant does not pay within 48 hours
- **WHEN** a payment link has been sent but the tenant has not paid within 48 hours
- **THEN** the system sends a reminder email to the tenant and notifies the owner

#### Scenario: Payment fails
- **WHEN** the tenant's payment attempt fails (insufficient funds, card declined)
- **THEN** the system records the failure, notifies both parties, and keeps the payment link active for retry

### Requirement: Collect recurring rent payment (classic rental)
The system SHALL allow the owner to request monthly rent payments from tenants of classic rentals. The system SHALL send a payment link via email each month based on the rent due date defined in the lease.

#### Scenario: Successful monthly rent payment
- **WHEN** the tenant pays the monthly rent via the payment link
- **THEN** the system records the payment, generates a receipt, and updates the rent tracking for that month

#### Scenario: Rent payment overdue
- **WHEN** the rent due date passes without payment
- **THEN** the system marks the month as "overdue", sends a reminder to the tenant, and notifies the owner

### Requirement: Owner fund withdrawal
The system SHALL allow the owner to view their balance and payment history. Stripe Connect handles payouts automatically to the owner's bank account based on the configured payout schedule.

#### Scenario: View payment dashboard
- **WHEN** the owner accesses the payment section
- **THEN** the system displays total collected, pending payouts, completed payouts, and a transaction history list

#### Scenario: View payout details
- **WHEN** the owner clicks on a specific payout in the history
- **THEN** the system displays the payout amount, date, associated rental payments, and Stripe payout ID

### Requirement: Payment receipts and records
The system SHALL generate a payment receipt for every successful payment. Receipts MUST include: payer name, amount, date, property reference, payment purpose (rent, seasonal stay, deposit), and a unique receipt number.

#### Scenario: Automatic receipt generation
- **WHEN** a payment is successfully completed
- **THEN** the system generates a PDF receipt and makes it available for download by both owner and tenant

#### Scenario: Tenant accesses their receipts
- **WHEN** a tenant accesses their payment link page after payment
- **THEN** the system displays a download link for the receipt PDF
