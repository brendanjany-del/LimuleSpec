## ADDED Requirements

### Requirement: Send signing invitation email
The system SHALL send an email to each signer when a signing session is activated, containing a personalized signing link and session details (document name, requester name, expiration date).

#### Scenario: Invitation email sent on session activation
- **WHEN** a signing session transitions from DRAFT to PENDING with notifications enabled
- **THEN** the system sends an invitation email to each signer with their unique signing link

#### Scenario: Notifications disabled
- **WHEN** a signing session is activated with notifications disabled
- **THEN** no invitation emails are sent

### Requirement: Send signing reminder
The system SHALL support sending reminder emails to signers who have not yet signed, either manually triggered or automatically at a configurable interval.

#### Scenario: Manual reminder sent
- **WHEN** a reminder is triggered for a signer with PENDING status
- **THEN** the system sends a reminder email with the signer's signing link

#### Scenario: Reminder not sent to completed signer
- **WHEN** a reminder is triggered for a signer with SIGNED status
- **THEN** no reminder email is sent

### Requirement: Send completion notification
The system SHALL send a notification email to the session creator when all signers have completed signing.

#### Scenario: All signers completed
- **WHEN** the signing session transitions to COMPLETED
- **THEN** the system sends a completion notification to the session creator with a link to download the signed document

### Requirement: Send decline notification
The system SHALL notify the session creator when a signer declines to sign.

#### Scenario: Signer declines
- **WHEN** a signer declines the signing request
- **THEN** the system sends a notification to the session creator indicating which signer declined

### Requirement: Configurable email transport
The system SHALL support configurable SMTP transport settings. The notification service SHALL be entirely disableable for headless/embedded integrations.

#### Scenario: SMTP configured
- **WHEN** the system is configured with valid SMTP credentials
- **THEN** emails are sent via the configured SMTP server

#### Scenario: Notifications globally disabled
- **WHEN** the notification service is configured as disabled
- **THEN** all email-sending operations are silently skipped without errors

#### Scenario: SMTP delivery failure
- **WHEN** an email fails to send due to SMTP error
- **THEN** the system logs the error and retries up to 3 times with exponential backoff, but does not block the signing workflow
