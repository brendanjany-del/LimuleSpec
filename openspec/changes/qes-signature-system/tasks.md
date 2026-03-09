## 1. Project Setup & Monorepo Structure

- [ ] 1.1 Initialize monorepo with pnpm workspaces and TypeScript strict config
- [ ] 1.2 Create package structure: packages/core, packages/api, packages/frontend, packages/shared
- [ ] 1.3 Configure shared tsconfig, ESLint, and Prettier across packages
- [ ] 1.4 Set up Prisma ORM with PostgreSQL in packages/core, define initial schema (documents, sessions, signers, signature_fields)
- [ ] 1.5 Implement StorageProvider interface with LocalStorageProvider and S3StorageProvider

## 2. Shared Types & DTOs

- [ ] 2.1 Define shared TypeScript types in packages/shared: Document, SigningSession, Signer, SignatureField, SessionStatus enum, SignerStatus enum
- [ ] 2.2 Define API request/response DTOs and JSON Schema validation schemas
- [ ] 2.3 Define standard API error format types (error code, message, details)

## 3. Core — PDF Document Management

- [ ] 3.1 Implement PDF upload with validation (file type check, size limit, pdf-lib parsing)
- [ ] 3.2 Implement document metadata extraction (page count, file size) using pdf-lib
- [ ] 3.3 Implement document preview generation (page-level thumbnails)
- [ ] 3.4 Implement document download (original and signed) via StorageProvider
- [ ] 3.5 Implement document deletion with active session guard
- [ ] 3.6 Write unit tests for pdf module

## 4. Core — Signature Field Placement

- [ ] 4.1 Implement signature field CRUD (create, read, update, delete) with page/bounds validation
- [ ] 4.2 Implement multi-field support per document with signer assignment
- [ ] 4.3 Enforce DRAFT-only mutation guard (block updates/deletes after session activation)
- [ ] 4.4 Write unit tests for placement module

## 5. Core — Signing Workflow

- [ ] 5.1 Implement session creation with signers and field associations
- [ ] 5.2 Implement session state machine (DRAFT → PENDING → IN_PROGRESS → COMPLETED/EXPIRED/CANCELLED)
- [ ] 5.3 Implement signer status tracking (PENDING → SIGNED | DECLINED)
- [ ] 5.4 Implement optional sequential signing order enforcement
- [ ] 5.5 Implement session TTL with configurable expiration (default 7 days)
- [ ] 5.6 Implement session activation (generate JWT tokens per signer, compute document hash)
- [ ] 5.7 Write unit tests for workflow module

## 6. Core — Cryptographic Signing

- [ ] 6.1 Implement server signing key management (load from file/env, auto-generate fallback with warning)
- [ ] 6.2 Implement document hashing (SHA-256) and integrity verification
- [ ] 6.3 Implement digital signature application to PDF using pdf-lib + node:crypto (PAdES baseline)
- [ ] 6.4 Implement incremental signature support (preserve previous signatures when adding new ones)
- [ ] 6.5 Implement signature visual rendering (signer name, date) at field position in PDF
- [ ] 6.6 Implement timestamp embedding in signature metadata (UTC)
- [ ] 6.7 Implement signed document verification function
- [ ] 6.8 Write unit tests for crypto module

## 7. Core — Notification Service

- [ ] 7.1 Implement configurable Nodemailer transport (SMTP settings, disable toggle)
- [ ] 7.2 Implement invitation email sending on session activation (with signing link)
- [ ] 7.3 Implement reminder email (manual trigger, skip already-signed signers)
- [ ] 7.4 Implement completion notification to session creator
- [ ] 7.5 Implement decline notification to session creator
- [ ] 7.6 Implement retry logic (3 retries, exponential backoff, non-blocking)
- [ ] 7.7 Write unit tests for notification module

## 8. Core — Shareable Signing Link

- [ ] 8.1 Implement JWT signing token generation per signer (session ID, signer ID, expiration)
- [ ] 8.2 Implement token validation (decode, expiry check, tamper detection)
- [ ] 8.3 Implement token access control (scope to signer's fields only, handle already-signed state)
- [ ] 8.4 Write unit tests for link module

## 9. API — Fastify REST Server

- [ ] 9.1 Set up Fastify server with JSON Schema validation, CORS, and rate limiting plugins
- [ ] 9.2 Implement document endpoints: POST /api/v1/documents, GET /:id, GET /:id/download, DELETE /:id
- [ ] 9.3 Implement session endpoints: POST /api/v1/sessions, GET /:id, POST /:id/activate, POST /:id/cancel
- [ ] 9.4 Implement field endpoints: POST /api/v1/sessions/:sessionId/fields, PUT /:fieldId, DELETE /:fieldId
- [ ] 9.5 Implement signing endpoints: GET /api/v1/sign/:token, POST /:token/complete, POST /:token/decline
- [ ] 9.6 Implement verification endpoint: POST /api/v1/verify
- [ ] 9.7 Implement standard error response format and global error handler
- [ ] 9.8 Configure Content-Security-Policy frame-ancestors for embeddable signing page
- [ ] 9.9 Write API integration tests

## 10. Frontend — React SPA Setup

- [ ] 10.1 Initialize React + Vite project in packages/frontend with routing
- [ ] 10.2 Set up API client layer to communicate with the Fastify backend
- [ ] 10.3 Implement PDF viewer component using react-pdf with zoom and responsive layout

## 11. Frontend — Send for Signature Mode

- [ ] 11.1 Implement PDF upload page with file picker and validation
- [ ] 11.2 Implement drag-and-drop signature field placement on rendered PDF pages (dnd-kit)
- [ ] 11.3 Implement signer assignment panel (email + name per field)
- [ ] 11.4 Implement "Send for signature" flow: create session → add fields → activate

## 12. Frontend — Direct Signing Mode

- [ ] 12.1 Implement direct signing page at /sign/:sessionId
- [ ] 12.2 Implement sequential signer turn-taking UI (one signer at a time on same device)
- [ ] 12.3 Implement completion confirmation with signed PDF download link

## 13. Frontend — Shareable Link Signing Mode

- [ ] 13.1 Implement signing page at /s/:token (minimal, no navigation chrome)
- [ ] 13.2 Implement token-based document + field loading from API
- [ ] 13.3 Implement Sign and Decline buttons with confirmation dialogs
- [ ] 13.4 Implement postMessage API for iframe embedding (sign complete, decline events)
- [ ] 13.5 Handle expired/invalid/already-signed token states with appropriate messages

## 14. Frontend — Session Dashboard

- [ ] 14.1 Implement session list page with status, document name, signer count, creation date
- [ ] 14.2 Implement session detail view with per-signer status and progress

## 15. Integration & Polish

- [ ] 15.1 End-to-end test: full send-for-signature flow (upload → place fields → send → sign via link → download signed PDF)
- [ ] 15.2 End-to-end test: direct signing flow (upload → place fields → co-present signing → download)
- [ ] 15.3 End-to-end test: session expiration and cancellation
- [ ] 15.4 Verify iframe embedding works with postMessage events and CSP frame-ancestors
- [ ] 15.5 Add environment configuration documentation (DB, SMTP, storage, signing key, CORS)
