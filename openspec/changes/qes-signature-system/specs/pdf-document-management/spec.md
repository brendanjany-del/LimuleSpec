## ADDED Requirements

### Requirement: Upload PDF document
The system SHALL accept PDF file uploads and store them for signature processing. The system SHALL validate that uploaded files are valid PDF documents. The system SHALL reject files exceeding the configured maximum size (default 20MB).

#### Scenario: Successful PDF upload
- **WHEN** a valid PDF file under the size limit is uploaded
- **THEN** the system stores the file, generates a unique document ID, and returns the document metadata (ID, filename, page count, file size)

#### Scenario: Invalid file type rejected
- **WHEN** a non-PDF file is uploaded
- **THEN** the system rejects the upload with a validation error indicating the file type is not supported

#### Scenario: File exceeds size limit
- **WHEN** a PDF file exceeding the configured maximum size is uploaded
- **THEN** the system rejects the upload with an error indicating the file is too large

### Requirement: Retrieve document metadata
The system SHALL provide document metadata including filename, page count, file size, upload date, and document status.

#### Scenario: Retrieve existing document metadata
- **WHEN** a valid document ID is provided
- **THEN** the system returns the document metadata

#### Scenario: Document not found
- **WHEN** an invalid or non-existent document ID is provided
- **THEN** the system returns a not-found error

### Requirement: Generate document preview
The system SHALL generate page-level preview images (thumbnails) of uploaded PDF documents for use in the signature field placement UI.

#### Scenario: Preview generation for multi-page PDF
- **WHEN** a preview is requested for a multi-page PDF document
- **THEN** the system returns preview images for each page with page numbers

### Requirement: Download original and signed documents
The system SHALL allow downloading the original uploaded PDF and, once signing is complete, the signed PDF.

#### Scenario: Download original PDF
- **WHEN** a download is requested for an uploaded document
- **THEN** the system returns the original PDF file

#### Scenario: Download signed PDF
- **WHEN** a download is requested for a completed signing session
- **THEN** the system returns the signed PDF with all signatures applied

#### Scenario: Signed PDF not yet available
- **WHEN** a download of the signed PDF is requested but signing is not complete
- **THEN** the system returns an error indicating the document is not yet fully signed

### Requirement: Delete document
The system SHALL allow deletion of documents that are not part of an active signing session.

#### Scenario: Delete unused document
- **WHEN** deletion is requested for a document not linked to any active signing session
- **THEN** the system removes the document file and its metadata

#### Scenario: Delete document in active session
- **WHEN** deletion is requested for a document linked to an active signing session
- **THEN** the system rejects the deletion with an error indicating the document is in use
