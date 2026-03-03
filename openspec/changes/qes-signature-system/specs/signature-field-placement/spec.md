## ADDED Requirements

### Requirement: Define signature fields on a document
The system SHALL allow defining signature fields on a PDF document by specifying page number, position (x, y coordinates), dimensions (width, height), and the assigned signer.

#### Scenario: Add signature field to a page
- **WHEN** a signature field is defined with valid page number, coordinates, dimensions, and signer identifier
- **THEN** the system stores the field definition and associates it with the document and the designated signer

#### Scenario: Invalid page number
- **WHEN** a signature field is defined with a page number exceeding the document's page count
- **THEN** the system rejects the field with a validation error

#### Scenario: Field outside page bounds
- **WHEN** a signature field is defined with coordinates or dimensions that extend outside the page boundaries
- **THEN** the system rejects the field with a validation error

### Requirement: Multiple signature fields per document
The system SHALL support multiple signature fields on a single document, potentially on different pages and assigned to different signers.

#### Scenario: Multiple fields for different signers
- **WHEN** multiple signature fields are defined on a document, each assigned to a different signer
- **THEN** the system stores all fields and each signer sees only their assigned fields during signing

#### Scenario: Multiple fields on same page
- **WHEN** multiple signature fields are placed on the same page
- **THEN** the system stores all fields with their distinct positions

### Requirement: Update signature field position
The system SHALL allow updating the position and dimensions of a signature field before the signing session starts.

#### Scenario: Reposition field before signing starts
- **WHEN** a field's position is updated while the signing session is in DRAFT status
- **THEN** the system updates the stored coordinates and dimensions

#### Scenario: Reposition field after signing started
- **WHEN** a field's position update is attempted after the signing session has left DRAFT status
- **THEN** the system rejects the update with an error indicating the session is already active

### Requirement: Remove signature field
The system SHALL allow removing a signature field from a document before the signing session starts.

#### Scenario: Remove field before signing starts
- **WHEN** a field is removed while the signing session is in DRAFT status
- **THEN** the system deletes the field definition

#### Scenario: Remove field after signing started
- **WHEN** a field removal is attempted after the signing session has left DRAFT status
- **THEN** the system rejects the removal with an error

### Requirement: Retrieve signature fields for a document
The system SHALL return all signature fields defined for a document, including their positions, dimensions, assigned signer, and signing status.

#### Scenario: List all fields
- **WHEN** fields are requested for a document with defined signature fields
- **THEN** the system returns all fields with their page, coordinates, dimensions, assigned signer identifier, and whether they have been signed
