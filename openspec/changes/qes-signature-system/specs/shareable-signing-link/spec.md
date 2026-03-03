## ADDED Requirements

### Requirement: Generate unique signing URL per signer
The system SHALL generate a unique, tamper-proof URL for each signer in a signing session. The URL SHALL contain a JWT token that identifies the signer and session without requiring authentication.

#### Scenario: Signing URL generated on session activation
- **WHEN** a signing session is activated
- **THEN** the system generates a unique signing URL for each signer in the format `/s/<token>`

#### Scenario: Token contains signer identity
- **WHEN** a signing URL token is decoded
- **THEN** it contains the session ID, signer ID, and expiration timestamp

### Requirement: Signing URL access control
The signing URL SHALL grant access only to the specific signer's fields and document view. The token SHALL expire according to the session's TTL.

#### Scenario: Valid token grants access
- **WHEN** a signer accesses a valid, non-expired signing URL
- **THEN** the system displays the document with only that signer's assigned signature fields

#### Scenario: Expired token denied
- **WHEN** a signer accesses a signing URL with an expired token
- **THEN** the system displays an expiration message and does not allow signing

#### Scenario: Already-signed token
- **WHEN** a signer accesses a signing URL after having already signed
- **THEN** the system displays a confirmation that signing is already complete

#### Scenario: Invalid token rejected
- **WHEN** an invalid or tampered signing URL is accessed
- **THEN** the system rejects access with an authentication error

### Requirement: Embeddable signing page
The signing page served at the signing URL SHALL be embeddable in an iframe. It SHALL not include navigation chrome (headers, menus). It SHALL communicate signing events to the parent window via postMessage API.

#### Scenario: Page loaded in iframe
- **WHEN** the signing URL is loaded inside an iframe
- **THEN** the page renders without navigation elements and fits the iframe container

#### Scenario: Signing completion communicated to parent
- **WHEN** a signer completes signing within an iframe
- **THEN** the page sends a postMessage event to the parent window with the signing result (success, signer ID, session ID)

#### Scenario: Signing declined communicated to parent
- **WHEN** a signer declines signing within an iframe
- **THEN** the page sends a postMessage event to the parent window with the decline result

### Requirement: CORS configuration for embedding
The system SHALL support configurable CORS and frame-ancestors settings to control which domains can embed the signing page.

#### Scenario: Allowed origin embeds signing page
- **WHEN** an allowed domain loads the signing page in an iframe
- **THEN** the page loads and functions correctly

#### Scenario: Disallowed origin blocked
- **WHEN** a non-allowed domain attempts to load the signing page in an iframe
- **THEN** the browser blocks the iframe based on Content-Security-Policy frame-ancestors
