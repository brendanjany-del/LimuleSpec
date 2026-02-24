## 1. Project Setup

- [x] 1.1 Initialize Next.js project with App Router and TypeScript
- [x] 1.2 Configure Prisma ORM with PostgreSQL (Supabase) connection
- [x] 1.3 Define initial Prisma schema: User, Property, Contract, Lease, Payment, Document, Inspection, FurnitureItem, CalendarEvent models
- [x] 1.4 Run initial Prisma migration and seed script
- [x] 1.5 Configure Cloudflare R2 (S3-compatible) client for file storage with pre-signed URL helpers
- [x] 1.6 Configure Resend for transactional emails with base templates (verification, notification, payment link)
- [x] 1.7 Set up Tailwind CSS and base UI component library (layout, buttons, forms, modals, cards)
- [x] 1.8 Set up environment variables structure (.env.example) for Stripe, Resend, R2, database URL

## 2. User Authentication (user-auth)

- [x] 2.1 Install and configure NextAuth.js (Auth.js) with Prisma adapter
- [x] 2.2 Implement email/password registration with password validation rules (8+ chars, uppercase, lowercase, digit)
- [x] 2.3 Implement email verification flow (send verification email via Resend, 24h expiry, activation on click)
- [x] 2.4 Implement login page with email/password and error handling (generic "invalid credentials" message)
- [x] 2.5 Implement Google OAuth provider (auto-create verified account on first login)
- [x] 2.6 Implement password reset flow (email link, 1h expiry, invalidate existing sessions on reset)
- [x] 2.7 Implement user profile page (edit name, email, phone — email change triggers re-verification)
- [x] 2.8 Configure JWT session management (7-day inactivity expiry, silent token refresh)
- [x] 2.9 Add auth middleware to protect all routes except login/register/public pages

## 3. Property Management (property-management)

- [x] 3.1 Create property creation form (name, address, rental type selector: seasonal / classic-furnished / classic-unfurnished)
- [x] 3.2 Implement French address fields with 5-digit postal code validation
- [x] 3.3 Create dashboard page listing all user properties as cards (name, address, type, occupancy status)
- [x] 3.4 Create empty state component for dashboard when user has no properties
- [x] 3.5 Create property detail/edit page (editable name, address, characteristics — rental type locked after creation)
- [x] 3.6 Implement furniture/accessories inventory management (name, quantity, condition, photo upload) — conditional on seasonal or furnished type
- [x] 3.7 Implement photo upload for furniture items via R2 with thumbnail display
- [x] 3.8 Implement property soft-delete with active contract check (block deletion if active contracts exist)

## 4. Electronic Signature System (shared)

- [x] 4.1 Integrate SignaturePad.js component for capturing handwritten signatures in the browser
- [x] 4.2 Implement pdf-lib integration to embed captured signature images into PDF documents
- [x] 4.3 Build signing workflow: generate unique token link, send email to signer, signing page with document preview + SignaturePad
- [x] 4.4 Implement proof file system: record signer IP, timestamp, SHA-256 hash of signed PDF, store in database
- [x] 4.5 Build signature status tracking (pending → tenant-signed → counter-signed → complete)
- [x] 4.6 Implement signing reminder email (auto-send after 7 days if unsigned)

## 5. Document Generation Engine (document-generation)

- [x] 5.1 Set up react-pdf or Puppeteer for PDF generation from HTML/MDX templates
- [x] 5.2 Create base PDF template system with owner logo, header/footer, and variable interpolation
- [x] 5.3 Create seasonal rental contract PDF template (property, owner, tenant, dates, guests, pricing, conditions, annexes, signature placeholders)
- [x] 5.4 Create classic lease (bail) PDF template — furnished variant with loi ALUR mandatory sections
- [x] 5.5 Create classic lease (bail) PDF template — unfurnished variant with loi ALUR mandatory sections
- [x] 5.6 Create inspection report (état des lieux) PDF template with room-by-room table, meter readings, photos, comparison mode
- [x] 5.7 Create rent notice (avis d'échéance) PDF template
- [x] 5.8 Create rent receipt (quittance de loyer) PDF template compliant with art. 21 loi du 6 juillet 1989
- [x] 5.9 Create standard letter templates: mise en demeure, congé bailleur, congé locataire, régularisation de charges, révision de loyer
- [x] 5.10 Implement per-property template customization (logo, header text) with global fallback
- [x] 5.11 Implement PDF storage on R2 with download endpoint (pre-signed URLs)

## 6. Seasonal Rental Management (seasonal-rental)

- [x] 6.1 Create seasonal contract creation form (check-in/out dates, guests, price, cleaning fee, tenant identity)
- [x] 6.2 Implement date overlap validation against existing confirmed contracts and blocked dates
- [x] 6.3 Implement custom conditions editor with reusable templates (save/load/apply)
- [x] 6.4 Implement "Send for signature" flow: generate PDF → send email to tenant → track status
- [x] 6.5 Implement tenant signing page and owner counter-signing flow
- [x] 6.6 Implement seasonal contract lifecycle state machine (draft → pending-signature → signed → confirmed → in-progress → completed → archived)
- [x] 6.7 Implement automatic status transitions via cron job (confirmed → in-progress on check-in date, in-progress → completed on check-out date)
- [x] 6.8 Create contract cancellation flow (allowed before in-progress, frees dates)
- [x] 6.9 Create seasonal rental history view with status and date range filters

## 7. Classic Rental Management (classic-rental)

- [x] 7.1 Create lease (bail) creation form with all mandatory fields (tenant, dates, rent, charges, deposit, property description)
- [x] 7.2 Implement lease type logic: furnished (1-year min) vs unfurnished (3-year min) with appropriate clauses
- [x] 7.3 Implement lease signature flow (send to tenant → tenant signs → owner counter-signs → lease activates)
- [x] 7.4 Create check-in inspection form (room-by-room: walls, floor, ceiling, fixtures — condition rating, meter readings, photo upload)
- [x] 7.5 Create check-out inspection form with pre-fill from check-in data and side-by-side comparison
- [x] 7.6 Implement degradation highlighting on check-out (worse condition than check-in → flag + notes + photos)
- [x] 7.7 Implement charges management: forfaitaires (fixed) vs provisions sur charges (with annual regularization calculation)
- [x] 7.8 Implement standard letter generation (select letter type → pre-fill from lease data → generate PDF)
- [x] 7.9 Implement lease lifecycle state machine (draft → pending-signature → active → termination-notice → ended → archived)
- [x] 7.10 Implement legal notice period calculation for termination (6 months unfurnished, 3 months furnished)

## 8. Deposit and Guarantee Management (deposit-and-guarantee)

- [x] 8.1 Implement classic rental deposit recording (amount, date, payment method) with legal maximum validation (1 month unfurnished, 2 months furnished)
- [x] 8.2 Implement deposit return flow: full return (no degradation) or partial return with itemized deductions and justification document
- [x] 8.3 Implement deposit return deadline reminders (notify owner 3 weeks after check-out, enforce 1-month/2-month legal deadlines)
- [x] 8.4 Integrate Stripe Setup Intents for seasonal credit card holds (authorize without capturing)
- [x] 8.5 Implement credit card hold release flow (cancel Setup Intent) and partial/full capture flow (create Payment Intent for damages)
- [x] 8.6 Implement Visale support: activation on lease, guided workflow with steps, URLs, visa number field, unpaid rent claim reminder (30 days)
- [x] 8.7 Implement Locapass support: activation on lease, guided workflow with steps, URLs, reference number field

## 9. Payment Processing (payment-processing)

- [x] 9.1 Integrate Stripe Connect Standard accounts: onboarding flow (redirect to Stripe-hosted form, store connected account ID)
- [x] 9.2 Implement onboarding status tracking (incomplete → ready) and block payment actions until onboarding complete
- [x] 9.3 Implement one-time payment collection for seasonal rentals: generate Stripe Checkout session → send payment link email → handle webhook for success/failure
- [x] 9.4 Implement recurring rent payment links: auto-generate Stripe Checkout link per month based on lease due date → send via email
- [x] 9.5 Implement payment reminder (48h for seasonal, overdue for classic) and failure notification emails
- [x] 9.6 Implement payment dashboard for owner: total collected, pending payouts, completed payouts, transaction history
- [x] 9.7 Implement payment receipt PDF auto-generation on successful payment (payer, amount, date, property, purpose, receipt number)
- [x] 9.8 Set up Stripe webhooks handler for payment events (checkout.session.completed, payment_intent.failed, account.updated)

## 10. Rent Management (rent-management)

- [x] 10.1 Implement automatic rent notice generation: cron job 5 days before due date → generate PDF → email to tenant
- [x] 10.2 Implement rent receipt (quittance) auto-generation on full payment and partial receipt on partial payment
- [x] 10.3 Create rent tracking view per lease: month-by-month table with due, paid, date, status (paid/partial/overdue/pending), document links
- [x] 10.4 Implement manual payment recording (cash, check, bank transfer) with receipt generation
- [x] 10.5 Implement late payment detection with configurable grace period (default 5 days) and alert emails to both parties
- [x] 10.6 Implement annual rent summary PDF generation (12 months, totals, statuses)

## 11. Calendar and Synchronization (calendar-and-sync)

- [x] 11.1 Integrate FullCalendar React component with month/week views for seasonal properties
- [x] 11.2 Display reservations color-coded by status (confirmed=green, pending=orange, draft=gray, blocked=red)
- [x] 11.3 Implement click-on-reservation popover showing contract summary (tenant, dates, price, status)
- [x] 11.4 Implement date blocking: select range → add reason → save → display in red → prevent contract creation on blocked dates
- [x] 11.5 Implement iCal import: add external URL, fetch and parse iCal feed, display external events with source label and distinct color
- [x] 11.6 Implement automatic iCal sync: background job every 15 minutes, store last sync timestamp, display last sync time in UI
- [x] 11.7 Implement manual sync trigger button per external calendar source
- [x] 11.8 Implement iCal export: generate unique URL per property returning confirmed + blocked dates in iCal format
- [x] 11.9 Handle iCal fetch failures gracefully (keep last data, show last sync time, notify owner)

## 12. Final Integration and Polish

- [x] 12.1 Implement Prisma multi-tenant middleware (auto-filter all queries by userId)
- [x] 12.2 Add RGPD compliance: data export endpoint, account deletion with cascade, privacy policy page
- [x] 12.3 Add responsive design pass on all pages (mobile-friendly dashboard, forms, calendar)
- [x] 12.4 Add loading states, error boundaries, and toast notifications across the application
- [x] 12.5 Write E2E tests for critical flows: registration, property creation, seasonal contract + signature + payment, classic lease + inspection + rent cycle
- [x] 12.6 Configure Vercel deployment with environment variables (Stripe, Resend, R2, Supabase)
- [x] 12.7 Set up GitHub Actions CI pipeline (lint, type-check, test, build)
