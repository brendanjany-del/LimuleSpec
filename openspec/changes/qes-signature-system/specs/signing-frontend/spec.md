## ADDED Requirements

### Requirement: Send for signature mode
The frontend SHALL provide a page where a user can upload a PDF, visually place signature fields on the document pages using drag and drop, assign each field to a signer by entering their email and name, and send the document for signature.

#### Scenario: Complete send flow
- **WHEN** a user uploads a PDF, places signature fields, assigns signers, and clicks "Send for signature"
- **THEN** the frontend creates a signing session via the API, activates it, and displays a confirmation with the session status

#### Scenario: Drag and drop field placement
- **WHEN** a user drags a signature field onto a rendered PDF page
- **THEN** the field is positioned at the drop location with adjustable dimensions and an assignable signer

#### Scenario: Multi-page navigation
- **WHEN** a user uploads a multi-page PDF
- **THEN** the frontend renders all pages in a scrollable view allowing field placement on any page

### Requirement: Direct signing mode
The frontend SHALL provide a page where multiple co-present signers can sign a document directly without email notifications. Each signer signs in turn on the same device.

#### Scenario: Multi-signer direct signing
- **WHEN** a document is set up for direct signing with multiple signers
- **THEN** the frontend presents each signer's fields one at a time, collecting each signature before moving to the next signer

#### Scenario: Direct signing completion
- **WHEN** all co-present signers have signed their fields
- **THEN** the frontend displays a completion confirmation and provides a download link for the signed document

### Requirement: Shareable link signing mode
The frontend SHALL provide a signing page accessible via a shareable URL (`/s/:token`). This page SHALL be minimal, standalone, and embeddable in an iframe.

#### Scenario: Signer opens signing link
- **WHEN** a signer opens a valid signing URL
- **THEN** the frontend displays the document with only that signer's assigned fields highlighted, along with a "Sign" button and a "Decline" button

#### Scenario: Signer completes signing via link
- **WHEN** a signer clicks "Sign" and confirms
- **THEN** the signature is submitted to the API and a success confirmation is displayed

#### Scenario: Signer declines via link
- **WHEN** a signer clicks "Decline" and confirms
- **THEN** the decline is submitted to the API and a confirmation message is displayed

### Requirement: PDF document viewer
The frontend SHALL render PDF documents for viewing and field placement using a client-side PDF rendering library.

#### Scenario: PDF rendered in browser
- **WHEN** a PDF document is loaded in the frontend
- **THEN** all pages are rendered at readable resolution with zoom controls

#### Scenario: Responsive display
- **WHEN** the frontend is accessed on different screen sizes
- **THEN** the PDF viewer adapts to the available width while maintaining aspect ratio

### Requirement: Session status dashboard
The frontend SHALL display a simple list of signing sessions with their current status (DRAFT, PENDING, IN_PROGRESS, COMPLETED, EXPIRED, CANCELLED).

#### Scenario: View session list
- **WHEN** a user accesses the dashboard
- **THEN** the frontend displays sessions sorted by most recent, with status, document name, signer count, and creation date

#### Scenario: View session details
- **WHEN** a user clicks on a session
- **THEN** the frontend shows the full session details including per-signer status and signing progress
