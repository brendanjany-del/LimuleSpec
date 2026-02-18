## ADDED Requirements

### Requirement: Generate seasonal rental contract PDF
The system SHALL generate a PDF contract for seasonal rentals using a predefined template. The PDF MUST include: property details, owner details, tenant details, dates, number of guests, pricing breakdown (rent, cleaning fee, total), conditions, annexes, and signature placeholders. The template MUST be customizable per property.

#### Scenario: Generate contract PDF from draft
- **WHEN** the owner clicks "Generate PDF" on a draft seasonal contract
- **THEN** the system populates the template with contract data and produces a downloadable PDF

#### Scenario: Regenerate PDF after contract edit
- **WHEN** the owner edits a draft contract and regenerates the PDF
- **THEN** the system produces an updated PDF reflecting the changes

### Requirement: Generate classic rental lease PDF (bail)
The system SHALL generate a lease PDF for classic rentals compliant with loi ALUR requirements. The PDF MUST include all mandatory sections: identification of parties, property description, rent and charges, deposit amount, lease duration, special conditions, and mandatory legal notices (diagnostics, informations obligatoires).

#### Scenario: Generate furnished lease PDF
- **WHEN** the owner generates a PDF for a classic-furnished lease
- **THEN** the system produces a PDF with all mandatory furnished-specific clauses and a 1-year minimum duration mention

#### Scenario: Generate unfurnished lease PDF
- **WHEN** the owner generates a PDF for a classic-unfurnished lease
- **THEN** the system produces a PDF with all mandatory unfurnished-specific clauses and a 3-year minimum duration mention

### Requirement: Generate inspection report PDF (état des lieux)
The system SHALL generate a PDF inspection report from the check-in or check-out inspection data. The PDF MUST include: date, parties present, room-by-room condition table, meter readings, photos, and signature placeholders.

#### Scenario: Generate check-in inspection PDF
- **WHEN** the owner finalizes a check-in inspection
- **THEN** the system generates a PDF with all room conditions, meter readings, and embedded photos

#### Scenario: Generate comparison check-out PDF
- **WHEN** the owner finalizes a check-out inspection for a lease with a check-in inspection
- **THEN** the system generates a PDF showing side-by-side entry and exit conditions per room with degradations highlighted

### Requirement: Generate rent notice and receipt PDFs
The system SHALL generate standardized PDF documents for rent notices (avis d'échéance) and rent receipts (quittances de loyer). These documents MUST comply with French legal requirements.

#### Scenario: Generate rent notice PDF
- **WHEN** the system generates a monthly rent notice
- **THEN** the PDF includes property address, tenant name, period, rent breakdown (rent + charges), total due, and due date

#### Scenario: Generate rent receipt PDF
- **WHEN** a rent payment is recorded as complete
- **THEN** the PDF includes property address, tenant name, period covered, amounts paid (rent + charges separately), and payment date

### Requirement: Generate standard letters PDF (courriers types)
The system SHALL generate pre-filled PDF letters from templates. Available letter types MUST include: late payment reminder (mise en demeure), lease termination notice (congé bailleur), lease termination by tenant (congé locataire), charge regularization notice, and rent increase notice. Each letter MUST be pre-filled with data from the associated lease.

#### Scenario: Generate mise en demeure
- **WHEN** the owner selects "mise en demeure" for a lease with overdue rent
- **THEN** the system generates a PDF letter pre-filled with tenant details, overdue amounts, dates, and legal references

#### Scenario: Generate congé bailleur
- **WHEN** the owner selects "congé bailleur" for an active lease
- **THEN** the system generates a PDF with the required legal notice period pre-calculated, legal motives section, and mandatory legal mentions

### Requirement: Customize document templates
The system SHALL allow the owner to customize document templates by adding their logo, header/footer text, and custom sections. Customizations MUST apply per property or globally for all properties.

#### Scenario: Add logo to all documents
- **WHEN** the owner uploads a logo in their settings
- **THEN** the system includes the logo in the header of all generated PDFs

#### Scenario: Customize template per property
- **WHEN** the owner sets a custom header text for a specific property
- **THEN** the system uses that custom header only for documents generated for that property

#### Scenario: Reset to default template
- **WHEN** the owner resets a property's template customization
- **THEN** the system reverts to the global default template for that property's documents
