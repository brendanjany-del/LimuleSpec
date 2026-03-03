## ADDED Requirements

### Requirement: Hash document for integrity
The system SHALL compute a SHA-256 hash of the PDF document content before any signature is applied. This hash SHALL be stored and used to verify document integrity.

#### Scenario: Document hash computed on session activation
- **WHEN** a signing session is activated
- **THEN** the system computes and stores the SHA-256 hash of the original PDF document

#### Scenario: Document integrity verification
- **WHEN** a signer accesses the document for signing
- **THEN** the system verifies the current document hash matches the stored hash before allowing signing

### Requirement: Apply digital signature to PDF
The system SHALL apply a digital signature to the PDF document at each signature field location when a signer completes their signature. The signature SHALL be embedded in the PDF using a standard PDF signature format (PAdES baseline compatible).

#### Scenario: Single signer signs
- **WHEN** a signer completes their signature on all assigned fields
- **THEN** the system applies a digital signature to the PDF at each field location using the server's signing key

#### Scenario: Multiple signers sign sequentially
- **WHEN** multiple signers sign the document in sequence
- **THEN** each signature is applied incrementally to the PDF, preserving previous signatures

### Requirement: Server signing key management
The system SHALL use a configurable server-side signing key pair (RSA 2048+ or ECDSA P-256+) for applying digital signatures. The key pair SHALL be loadable from file system path or environment variable.

#### Scenario: Signing key loaded from file
- **WHEN** the system starts with a configured key file path
- **THEN** the signing key is loaded and available for signature operations

#### Scenario: Missing signing key
- **WHEN** the system starts without a configured signing key
- **THEN** the system generates a self-signed key pair and logs a warning that production use requires a proper key

### Requirement: Timestamp signatures
The system SHALL include a server-side timestamp with each signature, recording the exact date and time the signature was applied.

#### Scenario: Timestamp included in signature
- **WHEN** a signature is applied to the PDF
- **THEN** the signature metadata includes the UTC timestamp of when the signature was applied

### Requirement: Verify signed document
The system SHALL provide a verification function that checks the integrity and validity of all signatures in a signed PDF document.

#### Scenario: Valid signed document
- **WHEN** verification is requested for a properly signed document
- **THEN** the system confirms all signatures are valid and the document has not been modified

#### Scenario: Tampered document detected
- **WHEN** verification is requested for a document whose content has been modified after signing
- **THEN** the system reports that the document integrity check has failed

### Requirement: Generate signature visual representation
The system SHALL render a visual representation of the signature in the PDF at the designated field position, including the signer's name, date, and a visual indicator that the field is digitally signed.

#### Scenario: Visual signature rendered
- **WHEN** a signer completes their signature
- **THEN** the signed PDF displays the signer's name and signing date at the signature field location
