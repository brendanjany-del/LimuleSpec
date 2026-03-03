## ADDED Requirements

### Requirement: REST API for document management
The API SHALL expose endpoints for uploading, retrieving, downloading, and deleting PDF documents under the `/api/v1/documents` namespace.

#### Scenario: Upload document via API
- **WHEN** a POST request with a PDF file is sent to `/api/v1/documents`
- **THEN** the API returns 201 with the document metadata (id, filename, pageCount, fileSize, createdAt)

#### Scenario: Get document metadata via API
- **WHEN** a GET request is sent to `/api/v1/documents/:id`
- **THEN** the API returns 200 with the document metadata

#### Scenario: Download document file via API
- **WHEN** a GET request is sent to `/api/v1/documents/:id/download`
- **THEN** the API returns 200 with the PDF file as binary stream

#### Scenario: Delete document via API
- **WHEN** a DELETE request is sent to `/api/v1/documents/:id`
- **THEN** the API returns 204 if deletion succeeds, or 409 if the document is in use

### Requirement: REST API for signing sessions
The API SHALL expose endpoints for creating, retrieving, activating, and cancelling signing sessions under the `/api/v1/sessions` namespace.

#### Scenario: Create signing session via API
- **WHEN** a POST request is sent to `/api/v1/sessions` with document ID, signers, signature fields, and options
- **THEN** the API returns 201 with the session details including signer tokens

#### Scenario: Get session status via API
- **WHEN** a GET request is sent to `/api/v1/sessions/:id`
- **THEN** the API returns 200 with session status, signer statuses, and completion percentage

#### Scenario: Activate session via API
- **WHEN** a POST request is sent to `/api/v1/sessions/:id/activate`
- **THEN** the API returns 200, transitions the session to PENDING, and triggers notifications if enabled

#### Scenario: Cancel session via API
- **WHEN** a POST request is sent to `/api/v1/sessions/:id/cancel`
- **THEN** the API returns 200 and transitions the session to CANCELLED

### Requirement: REST API for signature fields
The API SHALL expose endpoints for managing signature fields under `/api/v1/sessions/:sessionId/fields`.

#### Scenario: Add signature field via API
- **WHEN** a POST request is sent to `/api/v1/sessions/:sessionId/fields` with field details
- **THEN** the API returns 201 with the created field

#### Scenario: Update signature field via API
- **WHEN** a PUT request is sent to `/api/v1/sessions/:sessionId/fields/:fieldId`
- **THEN** the API returns 200 with the updated field, or 409 if session is not in DRAFT

#### Scenario: Delete signature field via API
- **WHEN** a DELETE request is sent to `/api/v1/sessions/:sessionId/fields/:fieldId`
- **THEN** the API returns 204, or 409 if session is not in DRAFT

### Requirement: REST API for signing operations
The API SHALL expose endpoints for signers to perform signing operations using their token.

#### Scenario: Get signing context via token
- **WHEN** a GET request is sent to `/api/v1/sign/:token`
- **THEN** the API returns 200 with the document preview, signer's fields, and session info

#### Scenario: Submit signature via token
- **WHEN** a POST request is sent to `/api/v1/sign/:token/complete`
- **THEN** the API applies the signature, updates signer status to SIGNED, and returns 200

#### Scenario: Decline signing via token
- **WHEN** a POST request is sent to `/api/v1/sign/:token/decline`
- **THEN** the API updates signer status to DECLINED and returns 200

### Requirement: REST API for verification
The API SHALL expose an endpoint for verifying signed documents.

#### Scenario: Verify signed document via API
- **WHEN** a POST request with a signed PDF is sent to `/api/v1/verify`
- **THEN** the API returns 200 with verification results (valid/invalid, signer details, timestamps)

### Requirement: API error format
All API error responses SHALL follow a consistent JSON format with `error` (error code), `message` (human-readable description), and optional `details` (field-level errors).

#### Scenario: Validation error response
- **WHEN** a request fails validation
- **THEN** the API returns 400 with `{ "error": "VALIDATION_ERROR", "message": "...", "details": [...] }`

#### Scenario: Not found error response
- **WHEN** a requested resource does not exist
- **THEN** the API returns 404 with `{ "error": "NOT_FOUND", "message": "..." }`

### Requirement: API rate limiting
The API SHALL enforce configurable rate limits per IP address to prevent abuse.

#### Scenario: Rate limit exceeded
- **WHEN** a client exceeds the configured rate limit
- **THEN** the API returns 429 with a Retry-After header

### Requirement: API versioning
The API SHALL be versioned under `/api/v1/` to allow future breaking changes without disrupting existing integrations.

#### Scenario: Versioned endpoint access
- **WHEN** a request is sent to `/api/v1/sessions`
- **THEN** the v1 API handler processes the request
