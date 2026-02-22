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

- [ ] 2.1 Install and configure NextAuth.js (Auth.js) with Prisma adapter
- [ ] 2.2 Implement email/password registration with password validation rules (8+ chars, uppercase, lowercase, digit)
- [ ] 2.3 Implement email verification flow (send verification email via Resend, 24h expiry, activation on click)
- [ ] 2.4 Implement login page with email/password and error handling (generic "invalid credentials" message)
- [ ] 2.5 Implement Google OAuth provider (auto-create verified account on first login)
- [ ] 2.6 Implement password reset flow (email link, 1h expiry, invalidate existing sessions on reset)
- [ ] 2.7 Implement user profile page (edit name, email, phone — email change triggers re-verification)
- [ ] 2.8 Configure JWT session management (7-day inactivity expiry, silent token refresh)
- [ ] 2.9 Add auth middleware to protect all routes except login/register/public pages

## 3. Property Management (property-management)

- [ ] 3.1 Create property creation form (name, address, rental type selector: seasonal / classic-furnished / classic-unfurnished)
- [ ] 3.2 Implement French address fields with 5-digit postal code validation
- [ ] 3.3 Create dashboard page listing all user properties as cards (name, address, type, occupancy status)
- [ ] 3.4 Create empty state component for dashboard when user has no properties
- [ ] 3.5 Create property detail/edit page (editable name, address, characteristics — rental type locked after creation)
- [ ] 3.6 Implement furniture/accessories inventory management (name, quantity, condition, photo upload) — conditional on seasonal or furnished type
- [ ] 3.7 Implement photo upload for furniture items via R2 with thumbnail display
- [ ] 3.8 Implement property soft-delete with active contract check (block deletion if active contracts exist)

## 4. Electronic Signature System (shared)

- [ ] 4.1 Integrate SignaturePad.js component for capturing handwritten signatures in the browser
- [ ] 4.2 Implement pdf-lib integration to embed captured signature images into PDF documents
- [ ] 4.3 Build signing workflow: generate unique token link, send email to signer, signing page with document preview + SignaturePad
- [ ] 4.4 Implement proof file system: record signer IP, timestamp, SHA-256 hash of signed PDF, store in database
- [ ] 4.5 Build signature status tracking (pending → tenant-signed → counter-signed → complete)
- [ ] 4.6 Implement signing reminder email (auto-send after 7 days if unsigned)

## 5. Document Generation Engine (document-generation)

- [ ] 5.1 Set up react-pdf or Puppeteer for PDF generation from HTML/MDX templates
- [ ] 5.2 Create base PDF template system with owner logo, header/footer, and variable interpolation
- [ ] 5.3 Create seasonal rental contract PDF template (property, owner, tenant, dates, guests, pricing, conditions, annexes, signature placeholders)
- [ ] 5.4 Create classic lease (bail) PDF template — furnished variant with loi ALUR mandatory sections
- [ ] 5.5 Create classic lease (bail) PDF template — unfurnished variant with loi ALUR mandatory sections
- [ ] 5.6 Create inspection report (état des lieux) PDF template with room-by-room table, meter readings, photos, comparison mode
- [ ] 5.7 Create rent notice (avis d'échéance) PDF template
- [ ] 5.8 Create rent receipt (quittance de loyer) PDF template compliant with art. 21 loi du 6 juillet 1989
- [ ] 5.9 Create standard letter templates: mise en demeure, congé bailleur, congé locataire, régularisation de charges, révision de loyer
- [ ] 5.10 Implement per-property template customization (logo, header text) with global fallback
- [ ] 5.11 Implement PDF storage on R2 with download endpoint (pre-signed URLs)

## 6. Seasonal Rental Management (seasonal-rental)

- [ ] 6.1 Create seasonal contract creation form (check-in/out dates, guests, price, cleaning fee, tenant identity)
- [ ] 6.2 Implement date overlap validation against existing confirmed contracts and blocked dates
- [ ] 6.3 Implement custom conditions editor with reusable templates (save/load/apply)
- [ ] 6.4 Implement "Send for signature" flow: generate PDF → send email to tenant → track status
- [ ] 6.5 Implement tenant signing page and owner counter-signing flow
- [ ] 6.6 Implement seasonal contract lifecycle state machine (draft → pending-signature → signed → confirmed → in-progress → completed → archived)
- [ ] 6.7 Implement automatic status transitions via cron job (confirmed → in-progress on check-in date, in-progress → completed on check-out date)
- [ ] 6.8 Create contract cancellation flow (allowed before in-progress, frees dates)
- [ ] 6.9 Create seasonal rental history view with status and date range filters

## 7. Classic Rental Management (classic-rental)

- [ ] 7.1 Create lease (bail) creation form with all mandatory fields (tenant, dates, rent, charges, deposit, property description)
- [ ] 7.2 Implement lease type logic: furnished (1-year min) vs unfurnished (3-year min) with appropriate clauses
- [ ] 7.3 Implement lease signature flow (send to tenant → tenant signs → owner counter-signs → lease activates)
- [ ] 7.4 Create check-in inspection form (room-by-room: walls, floor, ceiling, fixtures — condition rating, meter readings, photo upload)
- [ ] 7.5 Create check-out inspection form with pre-fill from check-in data and side-by-side comparison
- [ ] 7.6 Implement degradation highlighting on check-out (worse condition than check-in → flag + notes + photos)
- [ ] 7.7 Implement charges management: forfaitaires (fixed) vs provisions sur charges (with annual regularization calculation)
- [ ] 7.8 Implement standard letter generation (select letter type → pre-fill from lease data → generate PDF)
- [ ] 7.9 Implement lease lifecycle state machine (draft → pending-signature → active → termination-notice → ended → archived)
- [ ] 7.10 Implement legal notice period calculation for termination (6 months unfurnished, 3 months furnished)

## 8. Deposit and Guarantee Management (deposit-and-guarantee)

- [ ] 8.1 Implement classic rental deposit recording (amount, date, payment method) with legal maximum validation (1 month unfurnished, 2 months furnished)
- [ ] 8.2 Implement deposit return flow: full return (no degradation) or partial return with itemized deductions and justification document
- [ ] 8.3 Implement deposit return deadline reminders (notify owner 3 weeks after check-out, enforce 1-month/2-month legal deadlines)
- [ ] 8.4 Integrate Stripe Setup Intents for seasonal credit card holds (authorize without capturing)
- [ ] 8.5 Implement credit card hold release flow (cancel Setup Intent) and partial/full capture flow (create Payment Intent for damages)
- [ ] 8.6 Implement Visale support: activation on lease, guided workflow with steps, URLs, visa number field, unpaid rent claim reminder (30 days)
- [ ] 8.7 Implement Locapass support: activation on lease, guided workflow with steps, URLs, reference number field

## 9. Payment Processing (payment-processing)

- [ ] 9.1 Integrate Stripe Connect Standard accounts: onboarding flow (redirect to Stripe-hosted form, store connected account ID)
- [ ] 9.2 Implement onboarding status tracking (incomplete → ready) and block payment actions until onboarding complete
- [ ] 9.3 Implement one-time payment collection for seasonal rentals: generate Stripe Checkout session → send payment link email → handle webhook for success/failure
- [ ] 9.4 Implement recurring rent payment links: auto-generate Stripe Checkout link per month based on lease due date → send via email
- [ ] 9.5 Implement payment reminder (48h for seasonal, overdue for classic) and failure notification emails
- [ ] 9.6 Implement payment dashboard for owner: total collected, pending payouts, completed payouts, transaction history
- [ ] 9.7 Implement payment receipt PDF auto-generation on successful payment (payer, amount, date, property, purpose, receipt number)
- [ ] 9.8 Set up Stripe webhooks handler for payment events (checkout.session.completed, payment_intent.failed, account.updated)

## 10. Rent Management (rent-management)

- [ ] 10.1 Implement automatic rent notice generation: cron job 5 days before due date → generate PDF → email to tenant
- [ ] 10.2 Implement rent receipt (quittance) auto-generation on full payment and partial receipt on partial payment
- [ ] 10.3 Create rent tracking view per lease: month-by-month table with due, paid, date, status (paid/partial/overdue/pending), document links
- [ ] 10.4 Implement manual payment recording (cash, check, bank transfer) with receipt generation
- [ ] 10.5 Implement late payment detection with configurable grace period (default 5 days) and alert emails to both parties
- [ ] 10.6 Implement annual rent summary PDF generation (12 months, totals, statuses)

## 11. Calendar and Synchronization (calendar-and-sync)

- [ ] 11.1 Integrate FullCalendar React component with month/week views for seasonal properties
- [ ] 11.2 Display reservations color-coded by status (confirmed=green, pending=orange, draft=gray, blocked=red)
- [ ] 11.3 Implement click-on-reservation popover showing contract summary (tenant, dates, price, status)
- [ ] 11.4 Implement date blocking: select range → add reason → save → display in red → prevent contract creation on blocked dates
- [ ] 11.5 Implement iCal import: add external URL, fetch and parse iCal feed, display external events with source label and distinct color
- [ ] 11.6 Implement automatic iCal sync: background job every 15 minutes, store last sync timestamp, display last sync time in UI
- [ ] 11.7 Implement manual sync trigger button per external calendar source
- [ ] 11.8 Implement iCal export: generate unique URL per property returning confirmed + blocked dates in iCal format
- [ ] 11.9 Handle iCal fetch failures gracefully (keep last data, show last sync time, notify owner)

## 12. Final Integration and Polish

- [ ] 12.1 Implement Prisma multi-tenant middleware (auto-filter all queries by userId)
- [ ] 12.2 Add RGPD compliance: data export endpoint, account deletion with cascade, privacy policy page
- [ ] 12.3 Add responsive design pass on all pages (mobile-friendly dashboard, forms, calendar)
- [ ] 12.4 Add loading states, error boundaries, and toast notifications across the application
- [ ] 12.5 Write E2E tests for critical flows: registration, property creation, seasonal contract + signature + payment, classic lease + inspection + rent cycle
- [ ] 12.6 Configure Vercel deployment with environment variables (Stripe, Resend, R2, Supabase)
- [ ] 12.7 Set up GitHub Actions CI pipeline (lint, type-check, test, build)
