## ADDED Requirements

### Requirement: Create signing session
The system SHALL allow creating a signing session from an uploaded document with defined signature fields and a list of signers. Each signer SHALL be identified by an email address and a display name.

#### Scenario: Create session with multiple signers
- **WHEN** a signing session is created with a document, signature fields, and a list of signers
- **THEN** the system creates a session in DRAFT status, assigns each signer a unique identifier, and associates each signature field with its designated signer

#### Scenario: Create session without signature fields
- **WHEN** a signing session creation is attempted with no signature fields defined
- **THEN** the system rejects the creation with a validation error

### Requirement: Session state machine
The system SHALL manage signing session lifecycle through the following states: DRAFT, PENDING, IN_PROGRESS, COMPLETED, EXPIRED, CANCELLED. Transitions SHALL follow these rules:
- DRAFT → PENDING (when session is activated/sent)
- PENDING → IN_PROGRESS (when first signer signs or opens their signing page)
- IN_PROGRESS → COMPLETED (when all signers have signed)
- PENDING → EXPIRED (when session TTL expires)
- IN_PROGRESS → EXPIRED (when session TTL expires)
- DRAFT → CANCELLED (manual cancellation)
- PENDING → CANCELLED (manual cancellation)
- IN_PROGRESS → CANCELLED (manual cancellation)

#### Scenario: Activate session
- **WHEN** a DRAFT session is activated
- **THEN** the session transitions to PENDING and signing tokens are generated for each signer

#### Scenario: First signer begins
- **WHEN** the first signer accesses their signing page for a PENDING session
- **THEN** the session transitions to IN_PROGRESS

#### Scenario: All signers complete
- **WHEN** the last remaining signer signs their designated fields
- **THEN** the session transitions to COMPLETED and the signed PDF is generated

#### Scenario: Session expires
- **WHEN** the configured TTL elapses for a PENDING or IN_PROGRESS session
- **THEN** the session transitions to EXPIRED and all unsigned signer tokens are invalidated

#### Scenario: Cancel active session
- **WHEN** a DRAFT, PENDING, or IN_PROGRESS session is cancelled
- **THEN** the session transitions to CANCELLED and all unsigned signer tokens are invalidated

### Requirement: Signer status tracking
Each signer within a session SHALL have an individual status: PENDING, SIGNED, or DECLINED.

#### Scenario: Signer signs their fields
- **WHEN** a signer completes signing all their assigned fields
- **THEN** the signer's status transitions from PENDING to SIGNED

#### Scenario: Signer declines to sign
- **WHEN** a signer explicitly declines the signing request
- **THEN** the signer's status transitions from PENDING to DECLINED and the session creator is notified

### Requirement: Optional signing order
The system SHALL support an optional sequential signing order. When enabled, each signer SHALL only be able to sign after the previous signer in the order has completed.

#### Scenario: Sequential signing enforced
- **WHEN** sequential signing is enabled and signer #2 attempts to sign before signer #1
- **THEN** the system blocks the attempt and indicates that a previous signer must complete first

#### Scenario: Parallel signing (default)
- **WHEN** no signing order is specified
- **THEN** all signers can sign in any order concurrently

### Requirement: Session TTL configuration
The system SHALL support a configurable time-to-live (TTL) for signing sessions. The default TTL SHALL be 7 days.

#### Scenario: Custom TTL
- **WHEN** a session is created with a custom TTL of 30 days
- **THEN** the session expires 30 days after activation

#### Scenario: Default TTL applied
- **WHEN** a session is created without specifying a TTL
- **THEN** the session expires 7 days after activation

### Requirement: Retrieve session status
The system SHALL provide the current state of a signing session including overall status, per-signer status, and completion percentage.

#### Scenario: Query session status
- **WHEN** the status of an active session is requested
- **THEN** the system returns session state, list of signers with their individual statuses, and the number of completed signatures vs total
