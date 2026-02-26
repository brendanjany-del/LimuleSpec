# LS-Immo — Complete Project Reconstruction Prompt

## Overview

**LS-Immo** is a French property management SaaS application (gestion locative) built with Next.js. It supports two main rental types:
- **Seasonal** (location saisonnière) — short-term vacation rental contracts
- **Classic** (location classique) — long-term leases (furnished or unfurnished), compliant with French law (Loi ALUR, Code civil articles 1713+)

### Core Features
1. **User Authentication** — Credentials (email/password with bcrypt) + Google OAuth via NextAuth.js v5 beta
2. **Property Management** — CRUD for properties with address, rental type, soft delete, logo/header customization
3. **Furniture Inventory** — Per-property furniture items with condition tracking (NEW/GOOD/FAIR/POOR) and photo uploads
4. **Seasonal Contracts** — Full lifecycle (draft → pending signature → signed → confirmed → in progress → completed), digital signatures, Stripe payments
5. **Classic Leases** — Long-term lease management with rent tracking, charge provisions, Visale/Locapass guarantees
6. **Inspection Reports** (État des lieux) — Room-by-room condition assessment with meter readings (water, electricity, gas), photo uploads, check-in/check-out comparison
7. **Digital Signatures** — Token-based signing workflow with signature_pad canvas, IP logging, document hash verification
8. **PDF Generation** — Server-side PDF creation via pdf-lib for contracts, leases, inspections, rent notices/receipts, letters (mise en demeure, congé, etc.)
9. **Payment Processing** — Stripe Connect for online payments (checkout sessions), manual payment tracking (cash, check, bank transfer), deposit management (including credit card holds)
10. **Calendar** — FullCalendar-based views, iCal external calendar sync (Airbnb, Booking.com), manual blocked dates, iCal feed export
11. **Document Management** — File upload/download via Cloudflare R2 (S3-compatible), organized by type (CONTRACT_PDF, LEASE_PDF, etc.)
12. **Transactional Emails** — Via Resend for verification, password reset, signature requests, payment confirmations
13. **Rent Management** — Monthly rent entries generation, due date tracking, overdue detection, partial payments, annual summaries

### Tech Stack
- **Framework**: Next.js 16.1.6 with App Router (React 19, TypeScript 5)
- **Database**: PostgreSQL via Prisma ORM v7.4.1 with `@prisma/adapter-pg` (PrismaPg adapter)
- **Auth**: NextAuth.js v5 beta (`next-auth@5.0.0-beta.30`) with `@auth/prisma-adapter`
- **Payments**: Stripe Connect (`stripe@20.3.1`)
- **Storage**: Cloudflare R2 via `@aws-sdk/client-s3` and `@aws-sdk/s3-request-presigner`
- **PDF**: `pdf-lib@1.17.1` for server-side generation
- **Calendar**: `@fullcalendar/react@6.1.20` + `node-ical@0.25.4` + `ical-generator@10.0.0`
- **Email**: `resend@6.9.2`
- **Signatures**: `signature_pad@5.1.3`
- **Styling**: Tailwind CSS v4 with `@tailwindcss/postcss`
- **Testing**: Vitest v4 with jsdom, `@testing-library/react`, `@testing-library/jest-dom`
- **Password Hashing**: `bcryptjs@3.0.3`

### Project Structure
```
ls-immo/
├── prisma/
│   ├── schema.prisma           # Database schema
│   └── seed.ts                 # Database seeder
├── src/
│   ├── __tests__/              # Test files
│   │   ├── auth-helpers.test.ts
│   │   ├── ical-parser.test.ts
│   │   └── pdf-signature.test.ts
│   ├── app/
│   │   ├── (auth)/             # Auth pages (login, register, etc.)
│   │   ├── (dashboard)/        # Protected dashboard pages
│   │   ├── api/                # API routes
│   │   ├── sign/[token]/       # Public signature page
│   │   ├── globals.css
│   │   ├── layout.tsx
│   │   └── page.tsx
│   ├── components/
│   │   ├── layout/sidebar.tsx
│   │   ├── providers.tsx
│   │   ├── signature/signature-pad.tsx
│   │   └── ui/                 # Reusable UI components
│   ├── lib/
│   │   ├── auth.ts             # NextAuth configuration
│   │   ├── auth-helpers.ts     # Auth utility functions
│   │   ├── email.ts            # Resend email helpers
│   │   ├── ical-sync.ts        # iCal parsing/sync
│   │   ├── pdf-signature.ts    # PDF signature embedding
│   │   ├── pdf-templates/      # PDF generation templates
│   │   ├── prisma.ts           # Prisma client singleton
│   │   ├── r2.ts               # Cloudflare R2 helpers
│   │   └── stripe.ts           # Stripe client
│   ├── generated/prisma/       # Generated Prisma client (gitignored)
│   └── middleware.ts           # Auth middleware
├── .env.example
├── .gitignore
├── eslint.config.mjs
├── next.config.ts
├── package.json
├── postcss.config.mjs
├── prisma.config.ts
├── tsconfig.json
└── vitest.config.ts
```

---

## Complete Source Code

Below is every source file in the project. Recreate each file at the specified path.


### `.env.example`

```
# Database (Supabase PostgreSQL)
DATABASE_URL="postgresql://user:password@host:5432/dbname"

# NextAuth.js / Auth.js
NEXTAUTH_URL="http://localhost:3000"
AUTH_SECRET="your-secret-here"

# Google OAuth
GOOGLE_CLIENT_ID=""
GOOGLE_CLIENT_SECRET=""

# Stripe Connect
STRIPE_SECRET_KEY=""
STRIPE_PUBLISHABLE_KEY=""
STRIPE_WEBHOOK_SECRET=""

# Cloudflare R2 (S3-compatible)
R2_ENDPOINT="https://your-account-id.r2.cloudflarestorage.com"
R2_ACCESS_KEY_ID=""
R2_SECRET_ACCESS_KEY=""
R2_BUCKET_NAME="ls-immo"

# Resend (transactional emails)
RESEND_API_KEY=""
EMAIL_FROM="LS Immo <noreply@ls-immo.fr>"

# App
NEXT_PUBLIC_APP_URL="http://localhost:3000"

```

### `.gitignore`

```
# See https://help.github.com/articles/ignoring-files/ for more about ignoring files.

# dependencies
/node_modules
/.pnp
.pnp.*
.yarn/*
!.yarn/patches
!.yarn/plugins
!.yarn/releases
!.yarn/versions

# testing
/coverage

# next.js
/.next/
/out/

# production
/build

# misc
.DS_Store
*.pem

# debug
npm-debug.log*
yarn-debug.log*
yarn-error.log*
.pnpm-debug.log*

# env files
.env
.env.local
.env.*.local

# vercel
.vercel

# typescript
*.tsbuildinfo
next-env.d.ts

/src/generated/prisma

```

### `package.json`

```json
{
  "name": "ls-immo",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "eslint",
    "db:generate": "prisma generate",
    "db:migrate": "prisma migrate dev",
    "db:push": "prisma db push",
    "db:seed": "npx tsx prisma/seed.ts",
    "db:studio": "prisma studio",
    "test": "vitest run",
    "test:watch": "vitest"
  },
  "prisma": {
    "seed": "npx tsx prisma/seed.ts"
  },
  "dependencies": {
    "@auth/prisma-adapter": "^2.11.1",
    "@aws-sdk/client-s3": "^3.995.0",
    "@aws-sdk/s3-request-presigner": "^3.995.0",
    "@fullcalendar/core": "^6.1.20",
    "@fullcalendar/daygrid": "^6.1.20",
    "@fullcalendar/interaction": "^6.1.20",
    "@fullcalendar/react": "^6.1.20",
    "@prisma/adapter-pg": "^7.4.1",
    "@prisma/client": "^7.4.1",
    "@react-pdf/renderer": "^4.3.2",
    "bcryptjs": "^3.0.3",
    "ical-generator": "^10.0.0",
    "next": "16.1.6",
    "next-auth": "^5.0.0-beta.30",
    "node-ical": "^0.25.4",
    "pdf-lib": "^1.17.1",
    "pg": "^8.18.0",
    "prisma": "^7.4.1",
    "react": "19.2.3",
    "react-dom": "19.2.3",
    "resend": "^6.9.2",
    "signature_pad": "^5.1.3",
    "stripe": "^20.3.1",
    "tsx": "^4.21.0"
  },
  "devDependencies": {
    "@tailwindcss/postcss": "^4",
    "@testing-library/jest-dom": "^6.9.1",
    "@testing-library/react": "^16.3.2",
    "@types/bcryptjs": "^2.4.6",
    "@types/node": "^20",
    "@types/react": "^19",
    "@types/react-dom": "^19",
    "eslint": "^9",
    "eslint-config-next": "16.1.6",
    "jsdom": "^28.1.0",
    "tailwindcss": "^4",
    "typescript": "^5",
    "vitest": "^4.0.18"
  }
}

```

### `tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "ES2017",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "react-jsx",
    "incremental": true,
    "plugins": [
      {
        "name": "next"
      }
    ],
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": [
    "next-env.d.ts",
    "**/*.ts",
    "**/*.tsx",
    ".next/types/**/*.ts",
    ".next/dev/types/**/*.ts",
    "**/*.mts"
  ],
  "exclude": ["node_modules"]
}

```

### `next.config.ts`

```typescript
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  /* config options here */
};

export default nextConfig;

```

### `postcss.config.mjs`

```javascript
const config = {
  plugins: {
    "@tailwindcss/postcss": {},
  },
};

export default config;

```

### `eslint.config.mjs`

```javascript
import { defineConfig, globalIgnores } from "eslint/config";
import nextVitals from "eslint-config-next/core-web-vitals";
import nextTs from "eslint-config-next/typescript";

const eslintConfig = defineConfig([
  ...nextVitals,
  ...nextTs,
  // Override default ignores of eslint-config-next.
  globalIgnores([
    // Default ignores of eslint-config-next:
    ".next/**",
    "out/**",
    "build/**",
    "next-env.d.ts",
  ]),
]);

export default eslintConfig;

```

### `vitest.config.ts`

```typescript
import { defineConfig } from "vitest/config";
import path from "path";

export default defineConfig({
  test: {
    environment: "jsdom",
    globals: true,
    include: ["src/**/*.test.{ts,tsx}", "__tests__/**/*.test.{ts,tsx}"],
  },
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
    },
  },
});

```

### `prisma.config.ts`

```typescript
// This file was generated by Prisma, and assumes you have installed the following:
// npm install --save-dev prisma dotenv
import "dotenv/config";
import { defineConfig } from "prisma/config";

export default defineConfig({
  schema: "prisma/schema.prisma",
  migrations: {
    path: "prisma/migrations",
  },
  datasource: {
    url: process.env["DATABASE_URL"],
  },
});

```

### `prisma/schema.prisma`

```prisma
generator client {
  provider = "prisma-client"
  output   = "../src/generated/prisma"
}

datasource db {
  provider = "postgresql"
}

// ─── User & Auth ────────────────────────────────────────

model User {
  id            String    @id @default(cuid())
  email         String    @unique
  name          String?
  phone         String?
  passwordHash  String?
  emailVerified DateTime?
  image         String?
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt

  accounts         Account[]
  sessions         Session[]
  properties       Property[]
  conditionTemplates ConditionTemplate[]
  stripeAccountId  String?
  stripeOnboarded  Boolean  @default(false)
  logoUrl          String?
  headerText       String?
}

model Account {
  id                String  @id @default(cuid())
  userId            String
  type              String
  provider          String
  providerAccountId String
  refresh_token     String?
  access_token      String?
  expires_at        Int?
  token_type        String?
  scope             String?
  id_token          String?
  session_state     String?

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@unique([provider, providerAccountId])
}

model Session {
  id           String   @id @default(cuid())
  sessionToken String   @unique
  userId       String
  expires      DateTime

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)
}

model VerificationToken {
  identifier String
  token      String   @unique
  expires    DateTime

  @@unique([identifier, token])
}

// ─── Property ───────────────────────────────────────────

enum RentalType {
  SEASONAL
  CLASSIC_FURNISHED
  CLASSIC_UNFURNISHED
}

model Property {
  id          String     @id @default(cuid())
  userId      String
  name        String
  rentalType  RentalType
  streetNumber String?
  streetName   String
  complement   String?
  postalCode   String
  city         String
  createdAt   DateTime   @default(now())
  updatedAt   DateTime   @updatedAt
  deletedAt   DateTime?

  user             User              @relation(fields: [userId], references: [id], onDelete: Cascade)
  furnitureItems   FurnitureItem[]
  contracts        Contract[]
  leases           Lease[]
  calendarEvents   CalendarEvent[]
  externalCalendars ExternalCalendar[]
  documents        Document[]
  logoUrl          String?
  headerText       String?
}

// ─── Furniture / Inventory ──────────────────────────────

enum ItemCondition {
  NEW
  GOOD
  FAIR
  POOR
}

model FurnitureItem {
  id         String        @id @default(cuid())
  propertyId String
  name       String
  quantity   Int           @default(1)
  condition  ItemCondition @default(GOOD)
  photoUrl   String?
  createdAt  DateTime      @default(now())
  updatedAt  DateTime      @updatedAt

  property Property @relation(fields: [propertyId], references: [id], onDelete: Cascade)
}

// ─── Seasonal Contract ──────────────────────────────────

enum ContractStatus {
  DRAFT
  PENDING_SIGNATURE
  SIGNED
  CONFIRMED
  IN_PROGRESS
  COMPLETED
  CANCELLED
  ARCHIVED
}

model Contract {
  id             String         @id @default(cuid())
  propertyId     String
  status         ContractStatus @default(DRAFT)
  checkIn        DateTime
  checkOut       DateTime
  guests         Int
  rentalPrice    Float
  cleaningFee    Float          @default(0)
  tenantName     String
  tenantEmail    String
  tenantPhone    String?
  conditions     String?
  createdAt      DateTime       @default(now())
  updatedAt      DateTime       @updatedAt

  property   Property    @relation(fields: [propertyId], references: [id], onDelete: Cascade)
  signatures Signature[]
  payments   Payment[]
  deposit    Deposit?
  documents  Document[]
}

model ConditionTemplate {
  id        String   @id @default(cuid())
  userId    String
  name      String
  content   String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)
}

// ─── Classic Lease ──────────────────────────────────────

enum LeaseStatus {
  DRAFT
  PENDING_SIGNATURE
  ACTIVE
  TERMINATION_NOTICE
  ENDED
  ARCHIVED
}

enum ChargeType {
  FIXED
  PROVISION
}

model Lease {
  id              String      @id @default(cuid())
  propertyId      String
  status          LeaseStatus @default(DRAFT)
  tenantName      String
  tenantEmail     String
  tenantPhone     String?
  tenantBirthDate DateTime?
  startDate       DateTime
  endDate         DateTime?
  rentAmount      Float
  chargesAmount   Float
  chargeType      ChargeType  @default(FIXED)
  depositAmount   Float
  paymentDueDay   Int         @default(1)
  gracePeriodDays Int         @default(5)
  terminationDate DateTime?
  createdAt       DateTime    @default(now())
  updatedAt       DateTime    @updatedAt

  // Guarantee
  visaleEnabled    Boolean @default(false)
  visaleNumber     String?
  locapassEnabled  Boolean @default(false)
  locapassNumber   String?

  property    Property     @relation(fields: [propertyId], references: [id], onDelete: Cascade)
  inspections Inspection[]
  rentEntries RentEntry[]
  signatures  Signature[]
  deposit     Deposit?
  documents   Document[]
}

// ─── Inspection (état des lieux) ────────────────────────

enum InspectionType {
  CHECK_IN
  CHECK_OUT
}

model Inspection {
  id        String         @id @default(cuid())
  leaseId   String
  type      InspectionType
  date      DateTime       @default(now())
  validated Boolean        @default(false)
  meterWater      String?
  meterElectricity String?
  meterGas         String?
  createdAt DateTime       @default(now())
  updatedAt DateTime       @updatedAt

  lease  Lease           @relation(fields: [leaseId], references: [id], onDelete: Cascade)
  rooms  InspectionRoom[]
  signatures Signature[]
  documents  Document[]
}

model InspectionRoom {
  id           String        @id @default(cuid())
  inspectionId String
  name         String
  walls        ItemCondition @default(GOOD)
  floor        ItemCondition @default(GOOD)
  ceiling      ItemCondition @default(GOOD)
  fixtures     ItemCondition @default(GOOD)
  notes        String?
  photos       String[]

  inspection Inspection @relation(fields: [inspectionId], references: [id], onDelete: Cascade)
}

// ─── Signature ──────────────────────────────────────────

enum SignatureRole {
  TENANT
  OWNER
}

model Signature {
  id           String        @id @default(cuid())
  token        String        @unique @default(cuid())
  role         SignatureRole
  signedAt     DateTime?
  signerIp     String?
  documentHash String?
  signatureUrl String?
  createdAt    DateTime      @default(now())

  contractId   String?
  leaseId      String?
  inspectionId String?

  contract   Contract?   @relation(fields: [contractId], references: [id], onDelete: Cascade)
  lease      Lease?      @relation(fields: [leaseId], references: [id], onDelete: Cascade)
  inspection Inspection? @relation(fields: [inspectionId], references: [id], onDelete: Cascade)
}

// ─── Payment ────────────────────────────────────────────

enum PaymentStatus {
  PENDING
  COMPLETED
  FAILED
  REFUNDED
}

enum PaymentMethod {
  STRIPE
  CASH
  CHECK
  BANK_TRANSFER
}

model Payment {
  id               String        @id @default(cuid())
  contractId       String?
  rentEntryId      String?
  amount           Float
  status           PaymentStatus @default(PENDING)
  method           PaymentMethod @default(STRIPE)
  stripeSessionId  String?
  stripePaymentId  String?
  paidAt           DateTime?
  createdAt        DateTime      @default(now())
  updatedAt        DateTime      @updatedAt

  contract  Contract?  @relation(fields: [contractId], references: [id], onDelete: SetNull)
  rentEntry RentEntry? @relation(fields: [rentEntryId], references: [id], onDelete: SetNull)
  documents Document[]
}

// ─── Deposit ────────────────────────────────────────────

enum DepositStatus {
  PENDING
  RECEIVED
  HELD
  PARTIALLY_RETURNED
  FULLY_RETURNED
  HOLD_ACTIVE
  HOLD_RELEASED
  HOLD_CAPTURED
}

enum DepositType {
  CASH
  BANK_TRANSFER
  CREDIT_CARD_HOLD
}

model Deposit {
  id                String       @id @default(cuid())
  contractId        String?      @unique
  leaseId           String?      @unique
  amount            Float
  type              DepositType
  status            DepositStatus @default(PENDING)
  receivedAt        DateTime?
  returnedAt        DateTime?
  returnAmount      Float?
  deductions        String?
  stripeSetupIntentId String?
  stripePaymentIntentId String?
  createdAt         DateTime     @default(now())
  updatedAt         DateTime     @updatedAt

  contract Contract? @relation(fields: [contractId], references: [id], onDelete: SetNull)
  lease    Lease?    @relation(fields: [leaseId], references: [id], onDelete: SetNull)
}

// ─── Rent Entry ─────────────────────────────────────────

enum RentStatus {
  PENDING
  PAID
  PARTIAL
  OVERDUE
}

model RentEntry {
  id          String     @id @default(cuid())
  leaseId     String
  month       DateTime
  amountDue   Float
  amountPaid  Float      @default(0)
  status      RentStatus @default(PENDING)
  dueDate     DateTime
  paidAt      DateTime?
  createdAt   DateTime   @default(now())
  updatedAt   DateTime   @updatedAt

  lease     Lease     @relation(fields: [leaseId], references: [id], onDelete: Cascade)
  payments  Payment[]
  documents Document[]
}

// ─── Document ───────────────────────────────────────────

enum DocumentType {
  CONTRACT_PDF
  LEASE_PDF
  INSPECTION_PDF
  RENT_NOTICE
  RENT_RECEIPT
  PAYMENT_RECEIPT
  LETTER
  ANNUAL_SUMMARY
  DEPOSIT_RETURN
}

model Document {
  id           String       @id @default(cuid())
  type         DocumentType
  name         String
  url          String
  propertyId   String?
  contractId   String?
  leaseId      String?
  inspectionId String?
  paymentId    String?
  rentEntryId  String?
  createdAt    DateTime     @default(now())

  property   Property?   @relation(fields: [propertyId], references: [id], onDelete: SetNull)
  contract   Contract?   @relation(fields: [contractId], references: [id], onDelete: SetNull)
  lease      Lease?      @relation(fields: [leaseId], references: [id], onDelete: SetNull)
  inspection Inspection? @relation(fields: [inspectionId], references: [id], onDelete: SetNull)
  payment    Payment?    @relation(fields: [paymentId], references: [id], onDelete: SetNull)
  rentEntry  RentEntry?  @relation(fields: [rentEntryId], references: [id], onDelete: SetNull)
}

// ─── Calendar ───────────────────────────────────────────

enum CalendarEventType {
  BLOCKED
  EXTERNAL
}

model CalendarEvent {
  id         String            @id @default(cuid())
  propertyId String
  type       CalendarEventType
  startDate  DateTime
  endDate    DateTime
  reason     String?
  sourceLabel String?
  externalUid String?
  createdAt  DateTime          @default(now())
  updatedAt  DateTime          @updatedAt

  property Property @relation(fields: [propertyId], references: [id], onDelete: Cascade)
}

model ExternalCalendar {
  id         String   @id @default(cuid())
  propertyId String
  name       String
  url        String
  lastSyncAt DateTime?
  lastError  String?
  createdAt  DateTime @default(now())
  updatedAt  DateTime @updatedAt

  property Property @relation(fields: [propertyId], references: [id], onDelete: Cascade)
}

```

### `prisma/seed.ts`

```typescript
import { PrismaClient } from "../src/generated/prisma/client";
import { PrismaPg } from "@prisma/adapter-pg";

const adapter = new PrismaPg({ connectionString: process.env.DATABASE_URL! });
const prisma = new PrismaClient({ adapter });

async function main() {
  console.log("Seeding database...");

  const user = await prisma.user.upsert({
    where: { email: "demo@ls-immo.fr" },
    update: {},
    create: {
      email: "demo@ls-immo.fr",
      name: "Propriétaire Demo",
      emailVerified: new Date(),
    },
  });

  const seasonal = await prisma.property.create({
    data: {
      userId: user.id,
      name: "Villa Bord de Mer",
      rentalType: "SEASONAL",
      streetNumber: "12",
      streetName: "Rue des Flots",
      postalCode: "06400",
      city: "Cannes",
    },
  });

  const furnished = await prisma.property.create({
    data: {
      userId: user.id,
      name: "Studio Parisien",
      rentalType: "CLASSIC_FURNISHED",
      streetNumber: "8",
      streetName: "Rue de Rivoli",
      postalCode: "75001",
      city: "Paris",
    },
  });

  await prisma.property.create({
    data: {
      userId: user.id,
      name: "Appartement T3 Lyon",
      rentalType: "CLASSIC_UNFURNISHED",
      streetNumber: "45",
      streetName: "Rue de la République",
      postalCode: "69002",
      city: "Lyon",
    },
  });

  await prisma.furnitureItem.createMany({
    data: [
      { propertyId: seasonal.id, name: "Canapé 3 places", quantity: 1, condition: "GOOD" },
      { propertyId: seasonal.id, name: "Chaise de jardin", quantity: 6, condition: "GOOD" },
      { propertyId: seasonal.id, name: "Lit double", quantity: 2, condition: "NEW" },
      { propertyId: furnished.id, name: "Lit simple", quantity: 1, condition: "GOOD" },
      { propertyId: furnished.id, name: "Bureau", quantity: 1, condition: "FAIR" },
    ],
  });

  console.log("Seed complete.");
}

main()
  .then(async () => {
    await prisma.$disconnect();
  })
  .catch(async (e) => {
    console.error(e);
    await prisma.$disconnect();
    process.exit(1);
  });

```

### `src/middleware.ts`

```typescript
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";
import { getToken } from "next-auth/jwt";

/**
 * Routes that do NOT require authentication.
 * All other routes redirect unauthenticated users to /login.
 */
const publicPaths = [
  "/login",
  "/register",
  "/verify-email",
  "/reset-password",
  "/api/auth",
  "/api/webhooks",
];

/**
 * Prefix patterns that should be treated as public.
 * Matched with `startsWith` after normalising the pathname.
 */
const publicPrefixes = [
  "/sign/",
  "/api/auth/",
  "/api/webhooks/",
  "/reset-password/",
  "/verify-email/",
];

function isPublicPath(pathname: string): boolean {
  // Exact match against known public paths
  if (publicPaths.includes(pathname)) {
    return true;
  }

  // Prefix match (e.g. /sign/abc, /api/auth/callback/google)
  return publicPrefixes.some((prefix) => pathname.startsWith(prefix));
}

// Static assets and Next.js internals that should never be intercepted
const IGNORED_PATTERN =
  /^\/(_next\/|favicon\.ico|.*\.(?:svg|png|jpg|jpeg|gif|webp|ico|css|js|woff2?|ttf|eot|map)$)/;

export async function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl;

  // Skip static assets and Next.js internals
  if (IGNORED_PATTERN.test(pathname)) {
    return NextResponse.next();
  }

  // Allow public paths through without authentication
  if (isPublicPath(pathname)) {
    return NextResponse.next();
  }

  // Verify the JWT token
  const token = await getToken({
    req: request,
    secret: process.env.AUTH_SECRET,
  });

  if (!token) {
    const loginUrl = new URL("/login", request.url);
    loginUrl.searchParams.set("callbackUrl", pathname);
    return NextResponse.redirect(loginUrl);
  }

  return NextResponse.next();
}

export const config = {
  matcher: [
    /*
     * Match all request paths except:
     * - _next/static (static files)
     * - _next/image (image optimisation)
     * - favicon.ico, sitemap.xml, robots.txt (metadata files)
     */
    "/((?!_next/static|_next/image|favicon\\.ico|sitemap\\.xml|robots\\.txt).*)",
  ],
};

```

### `src/lib/prisma.ts`

```typescript
import { PrismaClient } from "@/generated/prisma/client";
import { PrismaPg } from "@prisma/adapter-pg";

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

function createPrismaClient() {
  const adapter = new PrismaPg({ connectionString: process.env.DATABASE_URL! });
  return new PrismaClient({ adapter });
}

export const prisma = globalForPrisma.prisma ?? createPrismaClient();

if (process.env.NODE_ENV !== "production") globalForPrisma.prisma = prisma;

```

### `src/lib/auth.ts`

```typescript
import NextAuth from "next-auth";
import type { NextAuthConfig } from "next-auth";
import Credentials from "next-auth/providers/credentials";
import Google from "next-auth/providers/google";
import { PrismaAdapter } from "@auth/prisma-adapter";
import bcrypt from "bcryptjs";
import { prisma } from "@/lib/prisma";

declare module "next-auth" {
  interface Session {
    user: {
      id: string;
      email: string;
      name?: string | null;
      image?: string | null;
    };
  }
}

declare module "@auth/core/jwt" {
  interface JWT {
    userId: string;
    email: string;
  }
}

const authConfig: NextAuthConfig = {
  // The PrismaAdapter type expects @prisma/client's PrismaClient,
  // but our generated client is structurally compatible.
  // eslint-disable-next-line @typescript-eslint/no-explicit-any
  adapter: PrismaAdapter(prisma as any),

  providers: [
    Google({
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
      allowDangerousEmailAccountLinking: true,
    }),

    Credentials({
      name: "credentials",
      credentials: {
        email: { label: "Email", type: "email" },
        password: { label: "Password", type: "password" },
      },
      async authorize(credentials) {
        if (!credentials?.email || !credentials?.password) {
          return null;
        }

        const email = credentials.email as string;
        const password = credentials.password as string;

        const user = await prisma.user.findUnique({
          where: { email },
        });

        if (!user || !user.passwordHash) {
          return null;
        }

        const isValid = await bcrypt.compare(password, user.passwordHash);

        if (!isValid) {
          return null;
        }

        return {
          id: user.id,
          email: user.email,
          name: user.name,
          image: user.image,
        };
      },
    }),
  ],

  session: {
    strategy: "jwt",
    maxAge: 7 * 24 * 60 * 60, // 7 days
  },

  pages: {
    signIn: "/login",
    // signUp is not a built-in NextAuth page config key; new users
    // are directed to /register via the application's own UI.
    error: "/login",
  },

  callbacks: {
    async jwt({ token, user }) {
      if (user) {
        token.userId = user.id as string;
        token.email = user.email as string;
      }
      return token;
    },

    async session({ session, token }) {
      if (token) {
        session.user.id = token.userId as string;
        session.user.email = token.email as string;
      }
      return session;
    },
  },
};

export const { handlers, auth, signIn, signOut } = NextAuth(authConfig);

```

### `src/lib/auth-helpers.ts`

```typescript
import bcrypt from "bcryptjs";
import crypto from "crypto";
import { auth } from "@/lib/auth";

const BCRYPT_ROUNDS = 12;

/**
 * Hash a plain-text password using bcryptjs.
 */
export async function hashPassword(password: string): Promise<string> {
  return bcrypt.hash(password, BCRYPT_ROUNDS);
}

/**
 * Compare a plain-text password against a bcrypt hash.
 */
export async function verifyPassword(
  password: string,
  hash: string
): Promise<boolean> {
  return bcrypt.compare(password, hash);
}

/**
 * Validate a password against security rules.
 * Rules: at least 8 characters, one uppercase letter,
 * one lowercase letter, and one digit.
 */
export function validatePassword(password: string): {
  valid: boolean;
  errors: string[];
} {
  const errors: string[] = [];

  if (password.length < 8) {
    errors.push("Le mot de passe doit contenir au moins 8 caractères.");
  }

  if (!/[A-Z]/.test(password)) {
    errors.push("Le mot de passe doit contenir au moins une lettre majuscule.");
  }

  if (!/[a-z]/.test(password)) {
    errors.push("Le mot de passe doit contenir au moins une lettre minuscule.");
  }

  if (!/\d/.test(password)) {
    errors.push("Le mot de passe doit contenir au moins un chiffre.");
  }

  return {
    valid: errors.length === 0,
    errors,
  };
}

/**
 * Generate a cryptographically random token (UUID v4).
 */
export function generateToken(): string {
  return crypto.randomUUID();
}

/**
 * Get the current server-side NextAuth session.
 * Returns `null` if the user is not authenticated.
 */
export async function getServerSession() {
  return auth();
}

```

### `src/lib/email.ts`

```typescript
import { Resend } from "resend";

export const resend = new Resend(process.env.RESEND_API_KEY);

const FROM = process.env.EMAIL_FROM || "LS Immo <noreply@ls-immo.fr>";

export async function sendEmail({
  to,
  subject,
  html,
}: {
  to: string;
  subject: string;
  html: string;
}) {
  return resend.emails.send({
    from: FROM,
    to,
    subject,
    html,
  });
}

export function verificationEmailHtml(url: string) {
  return `
    <h2>Vérifiez votre adresse email</h2>
    <p>Cliquez sur le lien ci-dessous pour activer votre compte :</p>
    <a href="${url}" style="display:inline-block;padding:12px 24px;background:#2563eb;color:white;text-decoration:none;border-radius:6px;">Vérifier mon email</a>
    <p>Ce lien expire dans 24 heures.</p>
  `;
}

export function resetPasswordEmailHtml(url: string) {
  return `
    <h2>Réinitialisation de mot de passe</h2>
    <p>Cliquez sur le lien ci-dessous pour réinitialiser votre mot de passe :</p>
    <a href="${url}" style="display:inline-block;padding:12px 24px;background:#2563eb;color:white;text-decoration:none;border-radius:6px;">Réinitialiser</a>
    <p>Ce lien expire dans 1 heure.</p>
  `;
}

export function signingRequestEmailHtml(documentName: string, signingUrl: string) {
  return `
    <h2>Document à signer</h2>
    <p>Vous avez un document à signer : <strong>${documentName}</strong></p>
    <a href="${signingUrl}" style="display:inline-block;padding:12px 24px;background:#2563eb;color:white;text-decoration:none;border-radius:6px;">Signer le document</a>
  `;
}

export function paymentRequestEmailHtml(amount: number, paymentUrl: string) {
  return `
    <h2>Paiement en attente</h2>
    <p>Un paiement de <strong>${amount.toFixed(2)} €</strong> est en attente.</p>
    <a href="${paymentUrl}" style="display:inline-block;padding:12px 24px;background:#2563eb;color:white;text-decoration:none;border-radius:6px;">Payer maintenant</a>
  `;
}

export function rentNoticeEmailHtml(tenantName: string, amount: number, dueDate: string) {
  return `
    <h2>Avis d'échéance</h2>
    <p>Bonjour ${tenantName},</p>
    <p>Votre loyer de <strong>${amount.toFixed(2)} €</strong> est dû le <strong>${dueDate}</strong>.</p>
    <p>Veuillez procéder au paiement dans les délais.</p>
  `;
}

```

### `src/lib/r2.ts`

```typescript
import { S3Client, PutObjectCommand, GetObjectCommand, DeleteObjectCommand } from "@aws-sdk/client-s3";
import { getSignedUrl } from "@aws-sdk/s3-request-presigner";

export const r2 = new S3Client({
  region: "auto",
  endpoint: process.env.R2_ENDPOINT!,
  credentials: {
    accessKeyId: process.env.R2_ACCESS_KEY_ID!,
    secretAccessKey: process.env.R2_SECRET_ACCESS_KEY!,
  },
});

const BUCKET = process.env.R2_BUCKET_NAME!;

export async function uploadFile(key: string, body: Buffer | Uint8Array, contentType: string) {
  await r2.send(
    new PutObjectCommand({
      Bucket: BUCKET,
      Key: key,
      Body: body,
      ContentType: contentType,
    })
  );
  return key;
}

export async function getPresignedDownloadUrl(key: string, expiresIn = 3600) {
  return getSignedUrl(
    r2,
    new GetObjectCommand({ Bucket: BUCKET, Key: key }),
    { expiresIn }
  );
}

export async function getPresignedUploadUrl(key: string, contentType: string, expiresIn = 3600) {
  return getSignedUrl(
    r2,
    new PutObjectCommand({ Bucket: BUCKET, Key: key, ContentType: contentType }),
    { expiresIn }
  );
}

export async function deleteFile(key: string) {
  await r2.send(
    new DeleteObjectCommand({ Bucket: BUCKET, Key: key })
  );
}

```

### `src/lib/stripe.ts`

```typescript
import Stripe from "stripe";

export const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: "2026-01-28.clover",
});

```

### `src/lib/ical-sync.ts`

```typescript
import { prisma } from "@/lib/prisma";

/**
 * Sync a single external iCal calendar.
 * Fetches the iCal URL, parses events, and upserts them into CalendarEvent.
 */
export async function syncExternalCalendar(calendarId: string): Promise<void> {
  const calendar = await prisma.externalCalendar.findUnique({
    where: { id: calendarId },
  });

  if (!calendar) return;

  try {
    const response = await fetch(calendar.url, {
      headers: { "User-Agent": "LS-Immo/1.0" },
    });

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }

    const text = await response.text();
    const events = parseIcal(text);

    // Remove old external events from this source
    await prisma.calendarEvent.deleteMany({
      where: {
        propertyId: calendar.propertyId,
        type: "EXTERNAL",
        sourceLabel: calendar.name,
      },
    });

    // Insert new events
    if (events.length > 0) {
      await prisma.calendarEvent.createMany({
        data: events.map((e) => ({
          propertyId: calendar.propertyId,
          type: "EXTERNAL" as const,
          startDate: e.start,
          endDate: e.end,
          reason: e.summary || null,
          sourceLabel: calendar.name,
          externalUid: e.uid || null,
        })),
      });
    }

    await prisma.externalCalendar.update({
      where: { id: calendarId },
      data: { lastSyncAt: new Date(), lastError: null },
    });
  } catch (error) {
    const errorMessage = error instanceof Error ? error.message : "Unknown error";
    await prisma.externalCalendar.update({
      where: { id: calendarId },
      data: { lastError: errorMessage },
    });
  }
}

/**
 * Sync all external calendars for all properties.
 * Called periodically (e.g. every 15 minutes via cron).
 */
export async function syncAllExternalCalendars(): Promise<void> {
  const calendars = await prisma.externalCalendar.findMany();

  for (const calendar of calendars) {
    await syncExternalCalendar(calendar.id);
  }
}

export interface ParsedEvent {
  uid?: string;
  summary?: string;
  start: Date;
  end: Date;
}

/**
 * Simple iCal parser for VEVENT blocks.
 */
export function parseIcal(text: string): ParsedEvent[] {
  const events: ParsedEvent[] = [];
  const blocks = text.split("BEGIN:VEVENT");

  for (let i = 1; i < blocks.length; i++) {
    const block = blocks[i].split("END:VEVENT")[0];
    const lines = block.split(/\r?\n/);

    let uid: string | undefined;
    let summary: string | undefined;
    let start: Date | undefined;
    let end: Date | undefined;

    for (const line of lines) {
      if (line.startsWith("UID:")) uid = line.slice(4).trim();
      if (line.startsWith("SUMMARY:")) summary = line.slice(8).trim();
      if (line.startsWith("DTSTART")) {
        const val = line.split(":").pop()?.trim();
        if (val) start = parseIcalDate(val);
      }
      if (line.startsWith("DTEND")) {
        const val = line.split(":").pop()?.trim();
        if (val) end = parseIcalDate(val);
      }
    }

    if (start && end) {
      events.push({ uid, summary, start, end });
    }
  }

  return events;
}

function parseIcalDate(val: string): Date {
  // Handle formats: 20240115, 20240115T120000Z, 20240115T120000
  if (val.length === 8) {
    return new Date(`${val.slice(0, 4)}-${val.slice(4, 6)}-${val.slice(6, 8)}`);
  }
  const d = val.replace(/(\d{4})(\d{2})(\d{2})T(\d{2})(\d{2})(\d{2})Z?/, "$1-$2-$3T$4:$5:$6Z");
  return new Date(d);
}

```

### `src/lib/pdf-signature.ts`

```typescript
import { PDFDocument } from "pdf-lib";
import crypto from "crypto";

/**
 * Embed a signature image (PNG data URL) into a PDF at the last page bottom.
 * Returns the modified PDF bytes and the SHA-256 hash of the signed document.
 */
export async function embedSignatureInPdf(
  pdfBytes: Uint8Array,
  signatureDataUrl: string,
  signerName: string
): Promise<{ signedPdfBytes: Uint8Array; documentHash: string }> {
  const pdfDoc = await PDFDocument.load(pdfBytes);

  // Extract PNG bytes from data URL
  const base64Data = signatureDataUrl.replace(/^data:image\/png;base64,/, "");
  const signatureBytes = Uint8Array.from(atob(base64Data), (c) => c.charCodeAt(0));
  const signatureImage = await pdfDoc.embedPng(signatureBytes);

  // Get the last page
  const pages = pdfDoc.getPages();
  const lastPage = pages[pages.length - 1];
  const { width } = lastPage.getSize();

  // Scale signature to fit (max 150px wide)
  const maxWidth = 150;
  const scale = Math.min(maxWidth / signatureImage.width, 1);
  const sigWidth = signatureImage.width * scale;
  const sigHeight = signatureImage.height * scale;

  // Draw signature on the last page (bottom area)
  lastPage.drawImage(signatureImage, {
    x: width / 2 - sigWidth / 2,
    y: 60,
    width: sigWidth,
    height: sigHeight,
  });

  // Add signer name below signature
  lastPage.drawText(`Signé par : ${signerName}`, {
    x: width / 2 - 60,
    y: 45,
    size: 8,
  });

  lastPage.drawText(`Date : ${new Date().toLocaleString("fr-FR")}`, {
    x: width / 2 - 60,
    y: 35,
    size: 8,
  });

  const signedPdfBytes = await pdfDoc.save();

  // Compute SHA-256 hash of the signed document
  const hash = crypto.createHash("sha256");
  hash.update(Buffer.from(signedPdfBytes));
  const documentHash = hash.digest("hex");

  return { signedPdfBytes: new Uint8Array(signedPdfBytes), documentHash };
}

/**
 * Compute the SHA-256 hash of a PDF document.
 */
export function hashDocument(pdfBytes: Uint8Array): string {
  const hash = crypto.createHash("sha256");
  hash.update(Buffer.from(pdfBytes));
  return hash.digest("hex");
}

```

### `src/lib/pdf-templates/base.ts`

```typescript
/**
 * Base PDF template utilities — shared styles and layout helpers.
 * Uses pdf-lib for server-side PDF generation.
 */
import { PDFDocument, StandardFonts, rgb, PDFPage, PDFFont } from "pdf-lib";

export interface PdfContext {
  doc: PDFDocument;
  page: PDFPage;
  font: PDFFont;
  boldFont: PDFFont;
  y: number;
  margin: number;
  pageWidth: number;
  pageHeight: number;
  logoUrl?: string | null;
  headerText?: string | null;
}

const PAGE_WIDTH = 595.28; // A4
const PAGE_HEIGHT = 841.89;
const MARGIN = 50;

export async function createPdfContext(opts?: {
  logoUrl?: string | null;
  headerText?: string | null;
}): Promise<PdfContext> {
  const doc = await PDFDocument.create();
  const font = await doc.embedFont(StandardFonts.Helvetica);
  const boldFont = await doc.embedFont(StandardFonts.HelveticaBold);
  const page = doc.addPage([PAGE_WIDTH, PAGE_HEIGHT]);

  return {
    doc,
    page,
    font,
    boldFont,
    y: PAGE_HEIGHT - MARGIN,
    margin: MARGIN,
    pageWidth: PAGE_WIDTH,
    pageHeight: PAGE_HEIGHT,
    logoUrl: opts?.logoUrl,
    headerText: opts?.headerText,
  };
}

export function addNewPage(ctx: PdfContext): PdfContext {
  const page = ctx.doc.addPage([ctx.pageWidth, ctx.pageHeight]);
  return { ...ctx, page, y: ctx.pageHeight - ctx.margin };
}

export function drawTitle(ctx: PdfContext, text: string): PdfContext {
  ctx.page.drawText(text, {
    x: ctx.margin,
    y: ctx.y,
    size: 16,
    font: ctx.boldFont,
    color: rgb(0.1, 0.1, 0.1),
  });
  return { ...ctx, y: ctx.y - 28 };
}

export function drawSubtitle(ctx: PdfContext, text: string): PdfContext {
  ctx.page.drawText(text, {
    x: ctx.margin,
    y: ctx.y,
    size: 12,
    font: ctx.boldFont,
    color: rgb(0.2, 0.2, 0.2),
  });
  return { ...ctx, y: ctx.y - 20 };
}

export function drawText(
  ctx: PdfContext,
  text: string,
  opts?: { size?: number; bold?: boolean; indent?: number }
): PdfContext {
  const size = opts?.size || 10;
  const font = opts?.bold ? ctx.boldFont : ctx.font;
  const x = ctx.margin + (opts?.indent || 0);

  // Wrap long text
  const maxWidth = ctx.pageWidth - ctx.margin * 2 - (opts?.indent || 0);
  const lines = wrapText(text, font, size, maxWidth);

  for (const line of lines) {
    if (ctx.y < ctx.margin + 40) {
      ctx = addNewPage(ctx);
    }
    ctx.page.drawText(line, { x, y: ctx.y, size, font, color: rgb(0.15, 0.15, 0.15) });
    ctx.y -= size + 4;
  }

  return ctx;
}

export function drawLine(ctx: PdfContext): PdfContext {
  ctx.page.drawLine({
    start: { x: ctx.margin, y: ctx.y },
    end: { x: ctx.pageWidth - ctx.margin, y: ctx.y },
    thickness: 0.5,
    color: rgb(0.8, 0.8, 0.8),
  });
  return { ...ctx, y: ctx.y - 12 };
}

export function drawSpacing(ctx: PdfContext, px: number = 12): PdfContext {
  return { ...ctx, y: ctx.y - px };
}

export function drawKeyValue(ctx: PdfContext, key: string, value: string): PdfContext {
  ctx.page.drawText(`${key} :`, {
    x: ctx.margin,
    y: ctx.y,
    size: 10,
    font: ctx.boldFont,
    color: rgb(0.3, 0.3, 0.3),
  });
  ctx.page.drawText(value, {
    x: ctx.margin + 160,
    y: ctx.y,
    size: 10,
    font: ctx.font,
    color: rgb(0.15, 0.15, 0.15),
  });
  return { ...ctx, y: ctx.y - 16 };
}

export function drawSignatureBlock(ctx: PdfContext, label: string): PdfContext {
  if (ctx.y < ctx.margin + 100) {
    ctx = addNewPage(ctx);
  }

  ctx = drawSpacing(ctx, 20);
  ctx.page.drawText(label, {
    x: ctx.margin,
    y: ctx.y,
    size: 10,
    font: ctx.boldFont,
    color: rgb(0.2, 0.2, 0.2),
  });
  ctx.y -= 16;

  // Signature box
  ctx.page.drawRectangle({
    x: ctx.margin,
    y: ctx.y - 60,
    width: 200,
    height: 60,
    borderColor: rgb(0.7, 0.7, 0.7),
    borderWidth: 0.5,
  });
  ctx.page.drawText("Signature", {
    x: ctx.margin + 70,
    y: ctx.y - 40,
    size: 8,
    font: ctx.font,
    color: rgb(0.6, 0.6, 0.6),
  });

  return { ...ctx, y: ctx.y - 80 };
}

export function drawHeader(ctx: PdfContext): PdfContext {
  if (ctx.headerText) {
    ctx.page.drawText(ctx.headerText, {
      x: ctx.margin,
      y: ctx.y,
      size: 8,
      font: ctx.font,
      color: rgb(0.5, 0.5, 0.5),
    });
    ctx.y -= 14;
  }
  return ctx;
}

export function drawFooter(ctx: PdfContext, pageNumber: number): void {
  const text = `Page ${pageNumber}`;
  const width = ctx.font.widthOfTextAtSize(text, 8);
  ctx.page.drawText(text, {
    x: ctx.pageWidth / 2 - width / 2,
    y: 25,
    size: 8,
    font: ctx.font,
    color: rgb(0.6, 0.6, 0.6),
  });
}

function wrapText(text: string, font: PDFFont, size: number, maxWidth: number): string[] {
  const words = text.split(" ");
  const lines: string[] = [];
  let currentLine = "";

  for (const word of words) {
    const testLine = currentLine ? `${currentLine} ${word}` : word;
    const width = font.widthOfTextAtSize(testLine, size);

    if (width > maxWidth && currentLine) {
      lines.push(currentLine);
      currentLine = word;
    } else {
      currentLine = testLine;
    }
  }

  if (currentLine) lines.push(currentLine);
  return lines.length > 0 ? lines : [""];
}

export function formatDate(date: Date | string): string {
  return new Date(date).toLocaleDateString("fr-FR", {
    day: "2-digit",
    month: "long",
    year: "numeric",
  });
}

export function formatCurrency(amount: number): string {
  return new Intl.NumberFormat("fr-FR", { style: "currency", currency: "EUR" }).format(amount);
}

```

### `src/lib/pdf-templates/index.ts`

```typescript
export { generateSeasonalContractPdf } from "./seasonal-contract";
export { generateLeasePdf } from "./lease";
export { generateInspectionPdf } from "./inspection";
export { generateRentNoticePdf, generateRentReceiptPdf, generatePaymentReceiptPdf } from "./rent-documents";
export { generateLetterPdf } from "./letters";
export type { LetterType } from "./letters";

```

### `src/lib/pdf-templates/inspection.ts`

```typescript
import {
  createPdfContext,
  addNewPage,
  drawTitle,
  drawSubtitle,
  drawText,
  drawLine,
  drawSpacing,
  drawKeyValue,
  drawSignatureBlock,
  drawFooter,
  formatDate,
  PdfContext,
} from "./base";
import { rgb } from "pdf-lib";

interface RoomData {
  name: string;
  walls: string;
  floor: string;
  ceiling: string;
  fixtures: string;
  notes?: string;
}

interface InspectionData {
  type: "CHECK_IN" | "CHECK_OUT";
  date: Date | string;
  propertyName: string;
  propertyAddress: string;
  ownerName: string;
  tenantName: string;
  rooms: RoomData[];
  meterWater?: string;
  meterElectricity?: string;
  meterGas?: string;
  // For check-out comparison
  checkInRooms?: RoomData[];
  logoUrl?: string | null;
  headerText?: string | null;
}

const conditionLabels: Record<string, string> = {
  NEW: "Neuf",
  GOOD: "Bon état",
  FAIR: "État correct",
  POOR: "Mauvais état",
};

function conditionColor(condition: string) {
  switch (condition) {
    case "POOR": return rgb(0.8, 0.2, 0.2);
    case "FAIR": return rgb(0.8, 0.6, 0.1);
    default: return rgb(0.15, 0.15, 0.15);
  }
}

function drawRoomTable(ctx: PdfContext, rooms: RoomData[], checkInRooms?: RoomData[]): PdfContext {
  for (let i = 0; i < rooms.length; i++) {
    const room = rooms[i];
    const checkInRoom = checkInRooms?.[i];

    if (ctx.y < ctx.margin + 120) {
      ctx = addNewPage(ctx);
    }

    ctx = drawSubtitle(ctx, room.name);

    const items: [string, string, string?][] = [
      ["Murs", room.walls, checkInRoom?.walls],
      ["Sol", room.floor, checkInRoom?.floor],
      ["Plafond", room.ceiling, checkInRoom?.ceiling],
      ["Équipements", room.fixtures, checkInRoom?.fixtures],
    ];

    for (const [label, current, previous] of items) {
      const currentLabel = conditionLabels[current] || current;
      const color = conditionColor(current);

      ctx.page.drawText(`${label} :`, {
        x: ctx.margin + 10,
        y: ctx.y,
        size: 10,
        font: ctx.font,
        color: rgb(0.4, 0.4, 0.4),
      });

      ctx.page.drawText(currentLabel, {
        x: ctx.margin + 120,
        y: ctx.y,
        size: 10,
        font: ctx.boldFont,
        color,
      });

      if (previous && previous !== current) {
        const degraded = isDegraded(previous, current);
        const compText = `(entrée : ${conditionLabels[previous] || previous})`;
        ctx.page.drawText(compText, {
          x: ctx.margin + 250,
          y: ctx.y,
          size: 9,
          font: ctx.font,
          color: degraded ? rgb(0.8, 0.2, 0.2) : rgb(0.5, 0.5, 0.5),
        });
      }

      ctx.y -= 16;
    }

    if (room.notes) {
      ctx = drawText(ctx, `Notes : ${room.notes}`, { indent: 10, size: 9 });
    }

    ctx = drawSpacing(ctx, 8);
    ctx = drawLine(ctx);
  }

  return ctx;
}

function isDegraded(before: string, after: string): boolean {
  const order = ["NEW", "GOOD", "FAIR", "POOR"];
  return order.indexOf(after) > order.indexOf(before);
}

export async function generateInspectionPdf(data: InspectionData): Promise<Uint8Array> {
  const isCheckOut = data.type === "CHECK_OUT";
  const title = isCheckOut ? "ÉTAT DES LIEUX DE SORTIE" : "ÉTAT DES LIEUX D'ENTRÉE";

  let ctx = await createPdfContext({ logoUrl: data.logoUrl, headerText: data.headerText });

  ctx = drawTitle(ctx, title);
  ctx = drawSpacing(ctx, 8);
  ctx = drawKeyValue(ctx, "Date", formatDate(data.date));
  ctx = drawSpacing(ctx, 4);
  ctx = drawLine(ctx);

  // Property
  ctx = drawSubtitle(ctx, "LOGEMENT");
  ctx = drawKeyValue(ctx, "Bien", data.propertyName);
  ctx = drawKeyValue(ctx, "Adresse", data.propertyAddress);
  ctx = drawSpacing(ctx, 4);

  // Parties
  ctx = drawSubtitle(ctx, "PARTIES");
  ctx = drawKeyValue(ctx, "Bailleur", data.ownerName);
  ctx = drawKeyValue(ctx, "Locataire", data.tenantName);
  ctx = drawSpacing(ctx, 4);
  ctx = drawLine(ctx);

  // Meters
  if (data.meterWater || data.meterElectricity || data.meterGas) {
    ctx = drawSubtitle(ctx, "RELEVÉS DES COMPTEURS");
    if (data.meterWater) ctx = drawKeyValue(ctx, "Eau", data.meterWater);
    if (data.meterElectricity) ctx = drawKeyValue(ctx, "Électricité", data.meterElectricity);
    if (data.meterGas) ctx = drawKeyValue(ctx, "Gaz", data.meterGas);
    ctx = drawSpacing(ctx, 4);
    ctx = drawLine(ctx);
  }

  // Rooms
  ctx = drawSubtitle(ctx, "ÉTAT PIÈCE PAR PIÈCE");
  ctx = drawRoomTable(ctx, data.rooms, isCheckOut ? data.checkInRooms : undefined);

  // Signatures
  ctx = drawSubtitle(ctx, "SIGNATURES");
  ctx = drawText(ctx, "Les parties reconnaissent l'exactitude du présent état des lieux.");
  ctx = drawSignatureBlock(ctx, "Le Bailleur");
  ctx = drawSignatureBlock(ctx, "Le Locataire");

  const pages = ctx.doc.getPages();
  pages.forEach((_, i) => {
    drawFooter({ ...ctx, page: pages[i] }, i + 1);
  });

  return new Uint8Array(await ctx.doc.save());
}

```

### `src/lib/pdf-templates/lease.ts`

```typescript
import {
  createPdfContext,
  addNewPage,
  drawTitle,
  drawSubtitle,
  drawText,
  drawLine,
  drawSpacing,
  drawKeyValue,
  drawSignatureBlock,
  drawHeader,
  drawFooter,
  formatDate,
  formatCurrency,
} from "./base";

interface LeaseData {
  ownerName: string;
  ownerEmail: string;
  propertyName: string;
  propertyAddress: string;
  rentalType: "CLASSIC_FURNISHED" | "CLASSIC_UNFURNISHED";
  tenantName: string;
  tenantEmail: string;
  tenantPhone?: string;
  tenantBirthDate?: Date | string;
  startDate: Date | string;
  endDate?: Date | string;
  rentAmount: number;
  chargesAmount: number;
  chargeType: "FIXED" | "PROVISION";
  depositAmount: number;
  paymentDueDay: number;
  logoUrl?: string | null;
  headerText?: string | null;
}

export async function generateLeasePdf(data: LeaseData): Promise<Uint8Array> {
  const isFurnished = data.rentalType === "CLASSIC_FURNISHED";
  const minDuration = isFurnished ? "1 an" : "3 ans";
  const title = isFurnished
    ? "CONTRAT DE LOCATION MEUBLÉE"
    : "CONTRAT DE LOCATION NON MEUBLÉE";

  let ctx = await createPdfContext({ logoUrl: data.logoUrl, headerText: data.headerText });
  ctx = drawHeader(ctx);
  ctx = drawTitle(ctx, title);
  ctx = drawText(ctx, `(Loi n° 89-462 du 6 juillet 1989 — Loi ALUR du 24 mars 2014)`);
  ctx = drawSpacing(ctx, 8);
  ctx = drawText(ctx, `Établi le ${formatDate(new Date())}`);
  ctx = drawSpacing(ctx, 8);
  ctx = drawLine(ctx);

  // --- PARTIES ---
  ctx = drawSubtitle(ctx, "ARTICLE 1 — LES PARTIES");
  ctx = drawSpacing(ctx, 4);
  ctx = drawText(ctx, "Le Bailleur :", { bold: true });
  ctx = drawKeyValue(ctx, "Nom", data.ownerName);
  ctx = drawKeyValue(ctx, "Email", data.ownerEmail);
  ctx = drawSpacing(ctx, 8);

  ctx = drawText(ctx, "Le Locataire :", { bold: true });
  ctx = drawKeyValue(ctx, "Nom", data.tenantName);
  ctx = drawKeyValue(ctx, "Email", data.tenantEmail);
  if (data.tenantPhone) ctx = drawKeyValue(ctx, "Téléphone", data.tenantPhone);
  if (data.tenantBirthDate) ctx = drawKeyValue(ctx, "Date de naissance", formatDate(data.tenantBirthDate));
  ctx = drawSpacing(ctx, 8);
  ctx = drawLine(ctx);

  // --- LOGEMENT ---
  ctx = drawSubtitle(ctx, "ARTICLE 2 — OBJET DU CONTRAT");
  ctx = drawKeyValue(ctx, "Bien", data.propertyName);
  ctx = drawKeyValue(ctx, "Adresse", data.propertyAddress);
  ctx = drawKeyValue(ctx, "Type", isFurnished ? "Meublé" : "Non meublé");
  ctx = drawText(
    ctx,
    "Le logement est loué à usage exclusif d'habitation principale du locataire."
  );
  ctx = drawSpacing(ctx, 8);
  ctx = drawLine(ctx);

  // --- DURÉE ---
  ctx = drawSubtitle(ctx, "ARTICLE 3 — DURÉE");
  ctx = drawKeyValue(ctx, "Date de prise d'effet", formatDate(data.startDate));
  if (data.endDate) {
    ctx = drawKeyValue(ctx, "Date de fin", formatDate(data.endDate));
  }
  ctx = drawText(
    ctx,
    `Le bail est consenti pour une durée minimale de ${minDuration}, renouvelable par tacite reconduction.`
  );
  ctx = drawSpacing(ctx, 8);
  ctx = drawLine(ctx);

  // --- LOYER & CHARGES ---
  ctx = drawSubtitle(ctx, "ARTICLE 4 — LOYER ET CHARGES");
  ctx = drawKeyValue(ctx, "Loyer mensuel (hors charges)", formatCurrency(data.rentAmount));
  ctx = drawKeyValue(ctx, "Charges mensuelles", formatCurrency(data.chargesAmount));
  ctx = drawKeyValue(
    ctx,
    "Type de charges",
    data.chargeType === "FIXED" ? "Forfaitaires" : "Provisions sur charges (régularisation annuelle)"
  );
  ctx = drawKeyValue(ctx, "Total mensuel", formatCurrency(data.rentAmount + data.chargesAmount));
  ctx = drawKeyValue(ctx, "Jour de paiement", `Le ${data.paymentDueDay} de chaque mois`);
  ctx = drawSpacing(ctx, 8);
  ctx = drawLine(ctx);

  // --- DÉPÔT DE GARANTIE ---
  ctx = drawSubtitle(ctx, "ARTICLE 5 — DÉPÔT DE GARANTIE");
  ctx = drawKeyValue(ctx, "Montant", formatCurrency(data.depositAmount));
  const maxDeposit = isFurnished ? "deux mois" : "un mois";
  ctx = drawText(
    ctx,
    `Le dépôt de garantie ne peut excéder ${maxDeposit} de loyer hors charges. ` +
    "Il est restitué dans un délai maximal de un mois (si état des lieux conforme) " +
    "ou deux mois (si dégradations) après la remise des clés, déduction faite des sommes dues."
  );
  ctx = drawSpacing(ctx, 8);
  ctx = drawLine(ctx);

  // New page for additional articles
  ctx = addNewPage(ctx);
  ctx = drawHeader(ctx);

  // --- OBLIGATIONS ---
  ctx = drawSubtitle(ctx, "ARTICLE 6 — OBLIGATIONS DU BAILLEUR");
  ctx = drawText(ctx, "Le bailleur est tenu de :");
  ctx = drawText(ctx, "- Délivrer au locataire le logement en bon état d'usage et de réparation", { indent: 10 });
  ctx = drawText(ctx, "- Assurer la jouissance paisible du logement", { indent: 10 });
  ctx = drawText(ctx, "- Entretenir les locaux en état de servir à l'usage prévu", { indent: 10 });
  ctx = drawText(ctx, "- Ne pas s'opposer aux aménagements réalisés par le locataire (sous conditions)", { indent: 10 });
  ctx = drawText(ctx, "- Remettre gratuitement une quittance de loyer au locataire qui en fait la demande", { indent: 10 });
  ctx = drawSpacing(ctx, 8);

  ctx = drawSubtitle(ctx, "ARTICLE 7 — OBLIGATIONS DU LOCATAIRE");
  ctx = drawText(ctx, "Le locataire est tenu de :");
  ctx = drawText(ctx, "- Payer le loyer et les charges aux termes convenus", { indent: 10 });
  ctx = drawText(ctx, "- User paisiblement des locaux suivant la destination prévue au contrat", { indent: 10 });
  ctx = drawText(ctx, "- Répondre des dégradations survenues pendant la durée du bail", { indent: 10 });
  ctx = drawText(ctx, "- Prendre à sa charge l'entretien courant et les menues réparations", { indent: 10 });
  ctx = drawText(ctx, "- Souscrire une assurance habitation et en justifier annuellement", { indent: 10 });
  ctx = drawText(ctx, "- Ne pas transformer les locaux sans accord écrit du bailleur", { indent: 10 });
  ctx = drawSpacing(ctx, 8);
  ctx = drawLine(ctx);

  // --- RÉSILIATION ---
  ctx = drawSubtitle(ctx, "ARTICLE 8 — RÉSILIATION");
  const noticePeriod = isFurnished ? "1 mois" : "3 mois";
  const ownerNotice = isFurnished ? "3 mois" : "6 mois";
  ctx = drawText(
    ctx,
    `Le locataire peut résilier le bail à tout moment avec un préavis de ${noticePeriod} ` +
    "(réduit à 1 mois en zone tendue, mutation, perte d'emploi, ou bénéficiaire du RSA/AAH)."
  );
  ctx = drawText(
    ctx,
    `Le bailleur peut donner congé avec un préavis de ${ownerNotice} avant l'échéance du bail, ` +
    "pour motif légitime et sérieux, reprise pour habiter, ou vente du logement."
  );
  ctx = drawSpacing(ctx, 8);
  ctx = drawLine(ctx);

  // --- ÉTAT DES LIEUX ---
  ctx = drawSubtitle(ctx, "ARTICLE 9 — ÉTAT DES LIEUX");
  ctx = drawText(
    ctx,
    "Un état des lieux d'entrée et un état des lieux de sortie seront établis contradictoirement " +
    "et annexés au présent bail, conformément à la loi ALUR."
  );
  ctx = drawSpacing(ctx, 8);
  ctx = drawLine(ctx);

  // --- DIAGNOSTICS ---
  ctx = drawSubtitle(ctx, "ARTICLE 10 — DIAGNOSTICS");
  ctx = drawText(
    ctx,
    "Sont annexés au présent bail les diagnostics obligatoires : DPE (diagnostic de performance énergétique), " +
    "CREP (constat de risque d'exposition au plomb), diagnostic amiante (si applicable), " +
    "état des risques naturels et technologiques (ERNT), diagnostic électricité et gaz (si installations > 15 ans)."
  );
  ctx = drawSpacing(ctx, 12);
  ctx = drawLine(ctx);

  // --- SIGNATURES ---
  ctx = drawSubtitle(ctx, "SIGNATURES");
  ctx = drawText(
    ctx,
    'Fait en deux exemplaires originaux, dont un pour chaque partie. Mention manuscrite "Lu et approuvé" :'
  );
  ctx = drawSignatureBlock(ctx, "Le Bailleur");
  ctx = drawSignatureBlock(ctx, "Le Locataire");

  // Page numbers
  const pages = ctx.doc.getPages();
  pages.forEach((_, i) => {
    drawFooter({ ...ctx, page: pages[i] }, i + 1);
  });

  return new Uint8Array(await ctx.doc.save());
}

```

### `src/lib/pdf-templates/letters.ts`

```typescript
import {
  createPdfContext,
  drawTitle,
  drawText,
  drawSpacing,
  drawKeyValue,
  drawHeader,
  drawFooter,
  formatDate,
  formatCurrency,
} from "./base";

export type LetterType =
  | "MISE_EN_DEMEURE"
  | "CONGE_BAILLEUR"
  | "CONGE_LOCATAIRE"
  | "REGULARISATION_CHARGES"
  | "REVISION_LOYER";

interface LetterData {
  type: LetterType;
  ownerName: string;
  ownerAddress: string;
  tenantName: string;
  propertyAddress: string;
  // Contextual fields
  amountDue?: number;
  newRent?: number;
  oldRent?: number;
  effectiveDate?: Date | string;
  chargesDetail?: string;
  reason?: string;
  logoUrl?: string | null;
  headerText?: string | null;
}

const letterTitles: Record<LetterType, string> = {
  MISE_EN_DEMEURE: "MISE EN DEMEURE DE PAYER",
  CONGE_BAILLEUR: "CONGÉ DONNÉ PAR LE BAILLEUR",
  CONGE_LOCATAIRE: "CONGÉ DONNÉ PAR LE LOCATAIRE",
  REGULARISATION_CHARGES: "RÉGULARISATION DES CHARGES LOCATIVES",
  REVISION_LOYER: "RÉVISION DU LOYER",
};

export async function generateLetterPdf(data: LetterData): Promise<Uint8Array> {
  let ctx = await createPdfContext({ logoUrl: data.logoUrl, headerText: data.headerText });
  ctx = drawHeader(ctx);

  // Sender
  ctx = drawText(ctx, data.ownerName, { bold: true });
  ctx = drawText(ctx, data.ownerAddress);
  ctx = drawSpacing(ctx, 16);

  // Recipient
  ctx = drawText(ctx, data.tenantName, { bold: true });
  ctx = drawText(ctx, data.propertyAddress);
  ctx = drawSpacing(ctx, 16);

  // Date
  ctx = drawText(ctx, `Le ${formatDate(new Date())}`);
  ctx = drawSpacing(ctx, 16);

  // Title
  ctx = drawTitle(ctx, letterTitles[data.type]);
  ctx = drawSpacing(ctx, 12);

  // Body
  switch (data.type) {
    case "MISE_EN_DEMEURE":
      ctx = drawText(ctx, "Madame, Monsieur,");
      ctx = drawSpacing(ctx, 8);
      ctx = drawText(
        ctx,
        `Par la présente, je vous mets en demeure de régler la somme de ${formatCurrency(data.amountDue || 0)} ` +
        "correspondant aux loyers et charges impayés à ce jour."
      );
      ctx = drawSpacing(ctx, 8);
      ctx = drawText(
        ctx,
        "À défaut de règlement dans un délai de 8 jours à compter de la réception de ce courrier, " +
        "je me verrai contraint(e) d'engager les poursuites prévues par la loi."
      );
      ctx = drawSpacing(ctx, 8);
      ctx = drawText(
        ctx,
        "Conformément à l'article 24 de la loi du 6 juillet 1989, je vous rappelle que vous pouvez " +
        "saisir le Fonds de Solidarité pour le Logement (FSL) de votre département."
      );
      break;

    case "CONGE_BAILLEUR":
      ctx = drawText(ctx, "Madame, Monsieur,");
      ctx = drawSpacing(ctx, 8);
      ctx = drawText(
        ctx,
        "Par la présente, je vous informe de ma décision de ne pas renouveler le bail vous concernant " +
        `pour le logement situé ${data.propertyAddress}.`
      );
      ctx = drawSpacing(ctx, 8);
      if (data.reason) {
        ctx = drawKeyValue(ctx, "Motif", data.reason);
        ctx = drawSpacing(ctx, 8);
      }
      if (data.effectiveDate) {
        ctx = drawKeyValue(ctx, "Date d'effet", formatDate(data.effectiveDate));
        ctx = drawSpacing(ctx, 8);
      }
      ctx = drawText(
        ctx,
        "Conformément aux dispositions de la loi du 6 juillet 1989, je vous notifie le présent congé " +
        "dans les délais légaux. Vous disposez du droit de contester ce congé devant le tribunal judiciaire."
      );
      break;

    case "CONGE_LOCATAIRE":
      ctx = drawText(ctx, "Madame, Monsieur,");
      ctx = drawSpacing(ctx, 8);
      ctx = drawText(
        ctx,
        "Par la présente, je vous informe de ma décision de quitter le logement situé " +
        `${data.propertyAddress}.`
      );
      ctx = drawSpacing(ctx, 8);
      if (data.effectiveDate) {
        ctx = drawKeyValue(ctx, "Date de départ souhaitée", formatDate(data.effectiveDate));
        ctx = drawSpacing(ctx, 8);
      }
      ctx = drawText(
        ctx,
        "Le préavis court à compter de la réception du présent courrier. " +
        "Je m'engage à restituer le logement en bon état, conformément à l'état des lieux d'entrée."
      );
      break;

    case "REGULARISATION_CHARGES":
      ctx = drawText(ctx, "Madame, Monsieur,");
      ctx = drawSpacing(ctx, 8);
      ctx = drawText(
        ctx,
        "Conformément à l'article 23 de la loi du 6 juillet 1989, je procède à la régularisation " +
        "annuelle des charges locatives."
      );
      ctx = drawSpacing(ctx, 8);
      if (data.chargesDetail) {
        ctx = drawText(ctx, data.chargesDetail);
        ctx = drawSpacing(ctx, 8);
      }
      if (data.amountDue !== undefined) {
        const label = data.amountDue >= 0 ? "Solde à votre charge" : "Trop-perçu à vous restituer";
        ctx = drawKeyValue(ctx, label, formatCurrency(Math.abs(data.amountDue)));
      }
      break;

    case "REVISION_LOYER":
      ctx = drawText(ctx, "Madame, Monsieur,");
      ctx = drawSpacing(ctx, 8);
      ctx = drawText(
        ctx,
        "Conformément aux dispositions du bail et à l'article 17-1 de la loi du 6 juillet 1989, " +
        "je vous informe de la révision du loyer."
      );
      ctx = drawSpacing(ctx, 8);
      if (data.oldRent !== undefined) ctx = drawKeyValue(ctx, "Loyer actuel", formatCurrency(data.oldRent));
      if (data.newRent !== undefined) ctx = drawKeyValue(ctx, "Nouveau loyer", formatCurrency(data.newRent));
      if (data.effectiveDate) ctx = drawKeyValue(ctx, "Date d'effet", formatDate(data.effectiveDate));
      break;
  }

  ctx = drawSpacing(ctx, 16);
  ctx = drawText(ctx, "Veuillez agréer, Madame, Monsieur, l'expression de mes salutations distinguées.");
  ctx = drawSpacing(ctx, 24);
  ctx = drawText(ctx, data.ownerName, { bold: true });

  drawFooter(ctx, 1);
  return new Uint8Array(await ctx.doc.save());
}

```

### `src/lib/pdf-templates/rent-documents.ts`

```typescript
import {
  createPdfContext,
  drawTitle,
  drawSubtitle,
  drawText,
  drawLine,
  drawSpacing,
  drawKeyValue,
  drawHeader,
  drawFooter,
  formatDate,
  formatCurrency,
} from "./base";

interface RentNoticeData {
  ownerName: string;
  tenantName: string;
  propertyAddress: string;
  month: Date | string;
  rentAmount: number;
  chargesAmount: number;
  dueDate: Date | string;
  logoUrl?: string | null;
  headerText?: string | null;
}

interface RentReceiptData {
  ownerName: string;
  tenantName: string;
  propertyAddress: string;
  month: Date | string;
  rentAmount: number;
  chargesAmount: number;
  paidAmount: number;
  paidAt: Date | string;
  receiptNumber?: string;
  logoUrl?: string | null;
  headerText?: string | null;
}

interface PaymentReceiptData {
  ownerName: string;
  payerName: string;
  propertyAddress: string;
  amount: number;
  purpose: string;
  paidAt: Date | string;
  receiptNumber: string;
  logoUrl?: string | null;
  headerText?: string | null;
}

export async function generateRentNoticePdf(data: RentNoticeData): Promise<Uint8Array> {
  let ctx = await createPdfContext({ logoUrl: data.logoUrl, headerText: data.headerText });
  ctx = drawHeader(ctx);

  ctx = drawTitle(ctx, "AVIS D'ÉCHÉANCE");
  ctx = drawSpacing(ctx, 8);

  const monthLabel = new Date(data.month).toLocaleDateString("fr-FR", { month: "long", year: "numeric" });

  ctx = drawKeyValue(ctx, "Période", monthLabel);
  ctx = drawKeyValue(ctx, "Date d'émission", formatDate(new Date()));
  ctx = drawSpacing(ctx, 8);
  ctx = drawLine(ctx);

  ctx = drawSubtitle(ctx, "BAILLEUR");
  ctx = drawText(ctx, data.ownerName);
  ctx = drawSpacing(ctx, 8);

  ctx = drawSubtitle(ctx, "LOCATAIRE");
  ctx = drawText(ctx, data.tenantName);
  ctx = drawSpacing(ctx, 8);

  ctx = drawSubtitle(ctx, "LOGEMENT");
  ctx = drawText(ctx, data.propertyAddress);
  ctx = drawSpacing(ctx, 8);
  ctx = drawLine(ctx);

  ctx = drawSubtitle(ctx, "DÉTAIL");
  ctx = drawKeyValue(ctx, "Loyer", formatCurrency(data.rentAmount));
  ctx = drawKeyValue(ctx, "Charges", formatCurrency(data.chargesAmount));
  ctx = drawLine(ctx);
  ctx = drawKeyValue(ctx, "Total à payer", formatCurrency(data.rentAmount + data.chargesAmount));
  ctx = drawKeyValue(ctx, "Date d'échéance", formatDate(data.dueDate));
  ctx = drawSpacing(ctx, 16);

  ctx = drawText(
    ctx,
    "Ce document est un avis d'échéance et ne constitue pas une quittance de loyer. " +
    "La quittance sera émise après réception du paiement intégral."
  );

  drawFooter(ctx, 1);
  return new Uint8Array(await ctx.doc.save());
}

export async function generateRentReceiptPdf(data: RentReceiptData): Promise<Uint8Array> {
  let ctx = await createPdfContext({ logoUrl: data.logoUrl, headerText: data.headerText });
  ctx = drawHeader(ctx);

  const total = data.rentAmount + data.chargesAmount;
  const isPartial = data.paidAmount < total;
  const title = isPartial ? "REÇU DE PAIEMENT PARTIEL" : "QUITTANCE DE LOYER";

  ctx = drawTitle(ctx, title);
  ctx = drawSpacing(ctx, 8);

  const monthLabel = new Date(data.month).toLocaleDateString("fr-FR", { month: "long", year: "numeric" });

  if (data.receiptNumber) {
    ctx = drawKeyValue(ctx, "N° de quittance", data.receiptNumber);
  }
  ctx = drawKeyValue(ctx, "Période", monthLabel);
  ctx = drawSpacing(ctx, 8);
  ctx = drawLine(ctx);

  ctx = drawSubtitle(ctx, "BAILLEUR");
  ctx = drawText(ctx, data.ownerName);
  ctx = drawSpacing(ctx, 8);

  ctx = drawSubtitle(ctx, "LOCATAIRE");
  ctx = drawText(ctx, data.tenantName);
  ctx = drawSpacing(ctx, 8);

  ctx = drawSubtitle(ctx, "LOGEMENT");
  ctx = drawText(ctx, data.propertyAddress);
  ctx = drawSpacing(ctx, 8);
  ctx = drawLine(ctx);

  ctx = drawSubtitle(ctx, "DÉTAIL");
  ctx = drawKeyValue(ctx, "Loyer", formatCurrency(data.rentAmount));
  ctx = drawKeyValue(ctx, "Charges", formatCurrency(data.chargesAmount));
  ctx = drawKeyValue(ctx, "Total dû", formatCurrency(total));
  ctx = drawLine(ctx);
  ctx = drawKeyValue(ctx, "Montant reçu", formatCurrency(data.paidAmount));
  ctx = drawKeyValue(ctx, "Date du paiement", formatDate(data.paidAt));
  if (isPartial) {
    ctx = drawKeyValue(ctx, "Solde restant", formatCurrency(total - data.paidAmount));
  }
  ctx = drawSpacing(ctx, 16);

  if (!isPartial) {
    ctx = drawText(
      ctx,
      `Je soussigné(e) ${data.ownerName}, bailleur du logement désigné ci-dessus, ` +
      `déclare avoir reçu de ${data.tenantName} la somme de ${formatCurrency(data.paidAmount)} ` +
      `au titre du paiement du loyer et des charges pour la période susmentionnée, ` +
      "et lui en donne quittance, sous réserve de tous mes droits."
    );
  }

  ctx = drawSpacing(ctx, 16);
  ctx = drawText(
    ctx,
    "Conformément à l'article 21 de la loi n° 89-462 du 6 juillet 1989, " +
    "le bailleur est tenu de transmettre gratuitement une quittance au locataire qui en fait la demande."
  );

  drawFooter(ctx, 1);
  return new Uint8Array(await ctx.doc.save());
}

export async function generatePaymentReceiptPdf(data: PaymentReceiptData): Promise<Uint8Array> {
  let ctx = await createPdfContext({ logoUrl: data.logoUrl, headerText: data.headerText });
  ctx = drawHeader(ctx);

  ctx = drawTitle(ctx, "REÇU DE PAIEMENT");
  ctx = drawSpacing(ctx, 8);
  ctx = drawKeyValue(ctx, "N° de reçu", data.receiptNumber);
  ctx = drawKeyValue(ctx, "Date", formatDate(data.paidAt));
  ctx = drawSpacing(ctx, 8);
  ctx = drawLine(ctx);

  ctx = drawKeyValue(ctx, "Reçu de", data.payerName);
  ctx = drawKeyValue(ctx, "Par", data.ownerName);
  ctx = drawKeyValue(ctx, "Bien", data.propertyAddress);
  ctx = drawSpacing(ctx, 8);
  ctx = drawLine(ctx);

  ctx = drawKeyValue(ctx, "Objet", data.purpose);
  ctx = drawKeyValue(ctx, "Montant", formatCurrency(data.amount));
  ctx = drawSpacing(ctx, 16);

  ctx = drawText(
    ctx,
    `Je soussigné(e) ${data.ownerName} confirme avoir reçu de ${data.payerName} ` +
    `la somme de ${formatCurrency(data.amount)} au titre de : ${data.purpose}.`
  );

  drawFooter(ctx, 1);
  return new Uint8Array(await ctx.doc.save());
}

```

### `src/lib/pdf-templates/seasonal-contract.ts`

```typescript
import {
  createPdfContext,
  drawTitle,
  drawSubtitle,
  drawText,
  drawLine,
  drawSpacing,
  drawKeyValue,
  drawSignatureBlock,
  drawHeader,
  drawFooter,
  formatDate,
  formatCurrency,
} from "./base";

interface SeasonalContractData {
  ownerName: string;
  ownerEmail: string;
  propertyName: string;
  propertyAddress: string;
  tenantName: string;
  tenantEmail: string;
  tenantPhone?: string;
  checkIn: Date | string;
  checkOut: Date | string;
  guests: number;
  rentalPrice: number;
  cleaningFee: number;
  conditions?: string;
  logoUrl?: string | null;
  headerText?: string | null;
}

export async function generateSeasonalContractPdf(data: SeasonalContractData): Promise<Uint8Array> {
  let ctx = await createPdfContext({ logoUrl: data.logoUrl, headerText: data.headerText });
  ctx = drawHeader(ctx);

  ctx = drawTitle(ctx, "CONTRAT DE LOCATION SAISONNIÈRE");
  ctx = drawSpacing(ctx, 8);

  ctx = drawText(ctx, `Établi le ${formatDate(new Date())}`);
  ctx = drawSpacing(ctx, 12);
  ctx = drawLine(ctx);

  // Owner
  ctx = drawSubtitle(ctx, "LE BAILLEUR");
  ctx = drawKeyValue(ctx, "Nom", data.ownerName);
  ctx = drawKeyValue(ctx, "Email", data.ownerEmail);
  ctx = drawSpacing(ctx, 8);

  // Tenant
  ctx = drawSubtitle(ctx, "LE LOCATAIRE");
  ctx = drawKeyValue(ctx, "Nom", data.tenantName);
  ctx = drawKeyValue(ctx, "Email", data.tenantEmail);
  if (data.tenantPhone) {
    ctx = drawKeyValue(ctx, "Téléphone", data.tenantPhone);
  }
  ctx = drawSpacing(ctx, 8);
  ctx = drawLine(ctx);

  // Property
  ctx = drawSubtitle(ctx, "DÉSIGNATION DU LOGEMENT");
  ctx = drawKeyValue(ctx, "Nom", data.propertyName);
  ctx = drawKeyValue(ctx, "Adresse", data.propertyAddress);
  ctx = drawSpacing(ctx, 8);
  ctx = drawLine(ctx);

  // Dates and conditions
  ctx = drawSubtitle(ctx, "CONDITIONS DE LA LOCATION");
  ctx = drawKeyValue(ctx, "Date d'arrivée", formatDate(data.checkIn));
  ctx = drawKeyValue(ctx, "Date de départ", formatDate(data.checkOut));
  ctx = drawKeyValue(ctx, "Nombre d'occupants", String(data.guests));
  ctx = drawSpacing(ctx, 8);

  // Pricing
  ctx = drawSubtitle(ctx, "TARIFICATION");
  ctx = drawKeyValue(ctx, "Loyer", formatCurrency(data.rentalPrice));
  if (data.cleaningFee > 0) {
    ctx = drawKeyValue(ctx, "Frais de ménage", formatCurrency(data.cleaningFee));
  }
  ctx = drawKeyValue(ctx, "Total", formatCurrency(data.rentalPrice + data.cleaningFee));
  ctx = drawSpacing(ctx, 8);
  ctx = drawLine(ctx);

  // Custom conditions
  if (data.conditions) {
    ctx = drawSubtitle(ctx, "CONDITIONS PARTICULIÈRES");
    ctx = drawText(ctx, data.conditions);
    ctx = drawSpacing(ctx, 8);
    ctx = drawLine(ctx);
  }

  // Legal
  ctx = drawSubtitle(ctx, "DISPOSITIONS LÉGALES");
  ctx = drawText(
    ctx,
    "Ce contrat est régi par les articles 1713 et suivants du Code civil et la loi n° 70-9 du 2 janvier 1970 modifiée. " +
    "Le locataire déclare avoir pris connaissance du descriptif du logement et des conditions de la location. " +
    "Le présent contrat est conclu pour la durée indiquée ci-dessus et ne peut être renouvelé par tacite reconduction."
  );
  ctx = drawSpacing(ctx, 8);

  ctx = drawText(
    ctx,
    "Le locataire s'engage à : jouir paisiblement des lieux, ne pas sous-louer, restituer le logement " +
    "dans l'état dans lequel il l'a trouvé (sauf usure normale), signaler tout dysfonctionnement au bailleur."
  );
  ctx = drawSpacing(ctx, 12);
  ctx = drawLine(ctx);

  // Signatures
  ctx = drawSubtitle(ctx, "SIGNATURES");
  ctx = drawText(ctx, 'Fait en deux exemplaires, dont un pour chaque partie. Mention manuscrite "Lu et approuvé" :');
  ctx = drawSignatureBlock(ctx, "Le Bailleur");
  ctx = drawSignatureBlock(ctx, "Le Locataire");

  // Footer
  const pages = ctx.doc.getPages();
  pages.forEach((_, i) => {
    drawFooter({ ...ctx, page: pages[i] }, i + 1);
  });

  return new Uint8Array(await ctx.doc.save());
}

```

### `src/components/providers.tsx`

```typescript
"use client";

import { SessionProvider } from "next-auth/react";

export function Providers({ children }: { children: React.ReactNode }) {
  return <SessionProvider>{children}</SessionProvider>;
}

```

### `src/components/layout/sidebar.tsx`

```typescript
"use client";

import Link from "next/link";
import { usePathname } from "next/navigation";

const nav = [
  { href: "/dashboard", label: "Tableau de bord", icon: "M3 12l2-2m0 0l7-7 7 7M5 10v10a1 1 0 001 1h3m10-11l2 2m-2-2v10a1 1 0 01-1 1h-3m-6 0a1 1 0 001-1v-4a1 1 0 011-1h2a1 1 0 011 1v4a1 1 0 001 1m-6 0h6" },
  { href: "/properties", label: "Mes biens", icon: "M19 21V5a2 2 0 00-2-2H7a2 2 0 00-2 2v16m14 0h2m-2 0h-5m-9 0H3m2 0h5M9 7h1m-1 4h1m4-4h1m-1 4h1m-5 10v-5a1 1 0 011-1h2a1 1 0 011 1v5m-4 0h4" },
  { href: "/payments", label: "Paiements", icon: "M12 8c-1.657 0-3 .895-3 2s1.343 2 3 2 3 .895 3 2-1.343 2-3 2m0-8c1.11 0 2.08.402 2.599 1M12 8V7m0 1v8m0 0v1m0-1c-1.11 0-2.08-.402-2.599-1M21 12a9 9 0 11-18 0 9 9 0 0118 0z" },
  { href: "/profile", label: "Mon profil", icon: "M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z" },
];

export function Sidebar() {
  const pathname = usePathname();

  return (
    <aside className="w-64 bg-white border-r border-gray-200 min-h-screen p-4 hidden md:block">
      <div className="mb-8">
        <Link href="/dashboard" className="text-xl font-bold text-blue-600">
          LS Immo
        </Link>
      </div>
      <nav className="space-y-1">
        {nav.map((item) => {
          const active = pathname.startsWith(item.href);
          return (
            <Link
              key={item.href}
              href={item.href}
              className={`flex items-center gap-3 px-3 py-2 rounded-lg text-sm font-medium transition-colors ${
                active
                  ? "bg-blue-50 text-blue-700"
                  : "text-gray-700 hover:bg-gray-50"
              }`}
            >
              <svg className="w-5 h-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={1.5} d={item.icon} />
              </svg>
              {item.label}
            </Link>
          );
        })}
      </nav>
    </aside>
  );
}

```

### `src/components/signature/signature-pad.tsx`

```typescript
"use client";

import { useRef, useEffect, useCallback } from "react";
import SignaturePadLib from "signature_pad";
import { Button } from "@/components/ui/button";

interface SignaturePadProps {
  onSave: (dataUrl: string) => void;
  width?: number;
  height?: number;
}

export function SignaturePad({ onSave, width = 500, height = 200 }: SignaturePadProps) {
  const canvasRef = useRef<HTMLCanvasElement>(null);
  const padRef = useRef<SignaturePadLib | null>(null);

  useEffect(() => {
    if (canvasRef.current) {
      padRef.current = new SignaturePadLib(canvasRef.current, {
        backgroundColor: "rgb(255, 255, 255)",
        penColor: "rgb(0, 0, 0)",
      });
    }

    return () => {
      padRef.current?.off();
    };
  }, []);

  const handleClear = useCallback(() => {
    padRef.current?.clear();
  }, []);

  const handleSave = useCallback(() => {
    if (!padRef.current || padRef.current.isEmpty()) {
      return;
    }
    const dataUrl = padRef.current.toDataURL("image/png");
    onSave(dataUrl);
  }, [onSave]);

  return (
    <div className="space-y-3">
      <div className="border-2 border-dashed border-gray-300 rounded-lg overflow-hidden">
        <canvas ref={canvasRef} width={width} height={height} className="w-full touch-none" />
      </div>
      <div className="flex gap-3">
        <Button type="button" variant="secondary" size="sm" onClick={handleClear}>
          Effacer
        </Button>
        <Button type="button" size="sm" onClick={handleSave}>
          Valider la signature
        </Button>
      </div>
    </div>
  );
}

```

### `src/components/ui/index.ts`

```typescript
export { Button } from "./button";
export { Input } from "./input";
export { Select } from "./select";
export { Card, CardHeader, CardTitle } from "./card";
export { Modal } from "./modal";
export { ToastProvider, useToast } from "./toast";

```

### `src/components/ui/button.tsx`

```typescript
import { ButtonHTMLAttributes, forwardRef } from "react";

type Variant = "primary" | "secondary" | "danger" | "ghost";
type Size = "sm" | "md" | "lg";

const variantClasses: Record<Variant, string> = {
  primary: "bg-blue-600 text-white hover:bg-blue-700 focus:ring-blue-500",
  secondary: "bg-gray-100 text-gray-900 hover:bg-gray-200 focus:ring-gray-400",
  danger: "bg-red-600 text-white hover:bg-red-700 focus:ring-red-500",
  ghost: "bg-transparent text-gray-700 hover:bg-gray-100 focus:ring-gray-400",
};

const sizeClasses: Record<Size, string> = {
  sm: "px-3 py-1.5 text-sm",
  md: "px-4 py-2 text-sm",
  lg: "px-6 py-3 text-base",
};

interface ButtonProps extends ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: Variant;
  size?: Size;
  loading?: boolean;
}

export const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  ({ variant = "primary", size = "md", loading, className = "", children, disabled, ...props }, ref) => {
    return (
      <button
        ref={ref}
        disabled={disabled || loading}
        className={`inline-flex items-center justify-center font-medium rounded-lg transition-colors focus:outline-none focus:ring-2 focus:ring-offset-2 disabled:opacity-50 disabled:cursor-not-allowed ${variantClasses[variant]} ${sizeClasses[size]} ${className}`}
        {...props}
      >
        {loading && (
          <svg className="animate-spin -ml-1 mr-2 h-4 w-4" fill="none" viewBox="0 0 24 24">
            <circle className="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" strokeWidth="4" />
            <path className="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4z" />
          </svg>
        )}
        {children}
      </button>
    );
  }
);

Button.displayName = "Button";

```

### `src/components/ui/card.tsx`

```typescript
import { HTMLAttributes } from "react";

interface CardProps extends HTMLAttributes<HTMLDivElement> {
  padding?: boolean;
}

export function Card({ padding = true, className = "", children, ...props }: CardProps) {
  return (
    <div
      className={`bg-white rounded-xl border border-gray-200 shadow-sm ${padding ? "p-6" : ""} ${className}`}
      {...props}
    >
      {children}
    </div>
  );
}

export function CardHeader({ className = "", children, ...props }: HTMLAttributes<HTMLDivElement>) {
  return (
    <div className={`mb-4 ${className}`} {...props}>
      {children}
    </div>
  );
}

export function CardTitle({ className = "", children, ...props }: HTMLAttributes<HTMLHeadingElement>) {
  return (
    <h3 className={`text-lg font-semibold text-gray-900 ${className}`} {...props}>
      {children}
    </h3>
  );
}

```

### `src/components/ui/input.tsx`

```typescript
import { InputHTMLAttributes, forwardRef } from "react";

interface InputProps extends InputHTMLAttributes<HTMLInputElement> {
  label?: string;
  error?: string;
}

export const Input = forwardRef<HTMLInputElement, InputProps>(
  ({ label, error, className = "", id, ...props }, ref) => {
    const inputId = id || label?.toLowerCase().replace(/\s+/g, "-");
    return (
      <div className="w-full">
        {label && (
          <label htmlFor={inputId} className="block text-sm font-medium text-gray-700 mb-1">
            {label}
          </label>
        )}
        <input
          ref={ref}
          id={inputId}
          className={`block w-full rounded-lg border px-3 py-2 text-sm shadow-sm transition-colors focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-blue-500 ${
            error ? "border-red-500" : "border-gray-300"
          } ${className}`}
          {...props}
        />
        {error && <p className="mt-1 text-sm text-red-600">{error}</p>}
      </div>
    );
  }
);

Input.displayName = "Input";

```

### `src/components/ui/modal.tsx`

```typescript
"use client";

import { ReactNode, useEffect } from "react";

interface ModalProps {
  open: boolean;
  onClose: () => void;
  title?: string;
  children: ReactNode;
}

export function Modal({ open, onClose, title, children }: ModalProps) {
  useEffect(() => {
    if (!open) return;
    const handleEsc = (e: KeyboardEvent) => {
      if (e.key === "Escape") onClose();
    };
    document.addEventListener("keydown", handleEsc);
    return () => document.removeEventListener("keydown", handleEsc);
  }, [open, onClose]);

  if (!open) return null;

  return (
    <div className="fixed inset-0 z-50 flex items-center justify-center">
      <div className="fixed inset-0 bg-black/50" onClick={onClose} />
      <div className="relative z-10 bg-white rounded-xl shadow-xl max-w-lg w-full mx-4 p-6">
        {title && (
          <div className="flex items-center justify-between mb-4">
            <h2 className="text-lg font-semibold text-gray-900">{title}</h2>
            <button onClick={onClose} className="text-gray-400 hover:text-gray-600">
              <svg className="w-5 h-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M6 18L18 6M6 6l12 12" />
              </svg>
            </button>
          </div>
        )}
        {children}
      </div>
    </div>
  );
}

```

### `src/components/ui/select.tsx`

```typescript
import { SelectHTMLAttributes, forwardRef } from "react";

interface SelectProps extends SelectHTMLAttributes<HTMLSelectElement> {
  label?: string;
  error?: string;
  options: { value: string; label: string }[];
  placeholder?: string;
}

export const Select = forwardRef<HTMLSelectElement, SelectProps>(
  ({ label, error, options, placeholder, className = "", id, ...props }, ref) => {
    const selectId = id || label?.toLowerCase().replace(/\s+/g, "-");
    return (
      <div className="w-full">
        {label && (
          <label htmlFor={selectId} className="block text-sm font-medium text-gray-700 mb-1">
            {label}
          </label>
        )}
        <select
          ref={ref}
          id={selectId}
          className={`block w-full rounded-lg border px-3 py-2 text-sm shadow-sm transition-colors focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-blue-500 ${
            error ? "border-red-500" : "border-gray-300"
          } ${className}`}
          {...props}
        >
          {placeholder && (
            <option value="" disabled>
              {placeholder}
            </option>
          )}
          {options.map((opt) => (
            <option key={opt.value} value={opt.value}>
              {opt.label}
            </option>
          ))}
        </select>
        {error && <p className="mt-1 text-sm text-red-600">{error}</p>}
      </div>
    );
  }
);

Select.displayName = "Select";

```

### `src/components/ui/toast.tsx`

```typescript
"use client";

import { createContext, useContext, useState, useCallback, ReactNode } from "react";

type ToastType = "success" | "error" | "info";

interface Toast {
  id: string;
  message: string;
  type: ToastType;
}

interface ToastContextType {
  toast: (message: string, type?: ToastType) => void;
}

const ToastContext = createContext<ToastContextType>({ toast: () => {} });

export function useToast() {
  return useContext(ToastContext);
}

export function ToastProvider({ children }: { children: ReactNode }) {
  const [toasts, setToasts] = useState<Toast[]>([]);

  const toast = useCallback((message: string, type: ToastType = "info") => {
    const id = Math.random().toString(36).slice(2);
    setToasts((prev) => [...prev, { id, message, type }]);
    setTimeout(() => {
      setToasts((prev) => prev.filter((t) => t.id !== id));
    }, 4000);
  }, []);

  const typeClasses: Record<ToastType, string> = {
    success: "bg-green-600",
    error: "bg-red-600",
    info: "bg-blue-600",
  };

  return (
    <ToastContext.Provider value={{ toast }}>
      {children}
      <div className="fixed bottom-4 right-4 z-50 flex flex-col gap-2">
        {toasts.map((t) => (
          <div
            key={t.id}
            className={`${typeClasses[t.type]} text-white px-4 py-3 rounded-lg shadow-lg text-sm max-w-sm animate-fade-in`}
          >
            {t.message}
          </div>
        ))}
      </div>
    </ToastContext.Provider>
  );
}

```

### `src/app/globals.css`

```css
@import "tailwindcss";

@theme inline {
  --color-background: #f9fafb;
  --color-foreground: #111827;
  --font-sans: var(--font-geist-sans);
  --font-mono: var(--font-geist-mono);

  --animate-fade-in: fade-in 0.3s ease-out;
}

@keyframes fade-in {
  from { opacity: 0; transform: translateY(8px); }
  to { opacity: 1; transform: translateY(0); }
}

body {
  background: var(--color-background);
  color: var(--color-foreground);
}

```

### `src/app/layout.tsx`

```typescript
import type { Metadata } from "next";
import { Providers } from "@/components/providers";
import "./globals.css";

export const metadata: Metadata = {
  title: "LS Immo — Gestion locative",
  description: "Application SaaS de gestion locative pour propriétaires bailleurs",
};

export default function RootLayout({
  children,
}: Readonly<{
  children: React.ReactNode;
}>) {
  return (
    <html lang="fr">
      <body className="antialiased">
        <Providers>{children}</Providers>
      </body>
    </html>
  );
}

```

### `src/app/page.tsx`

```typescript
import { redirect } from "next/navigation";

export default function Home() {
  redirect("/dashboard");
}

```

### `src/app/(auth)/layout.tsx`

```typescript
export default function AuthLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div className="min-h-screen flex items-center justify-center bg-gray-50 px-4 py-12">
      {children}
    </div>
  );
}

```

### `src/app/(auth)/login/page.tsx`

```typescript
"use client";

import { useState } from "react";
import { signIn } from "next-auth/react";
import { useRouter } from "next/navigation";
import Link from "next/link";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Card, CardHeader, CardTitle } from "@/components/ui/card";

export default function LoginPage() {
  const router = useRouter();
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [error, setError] = useState("");
  const [loading, setLoading] = useState(false);

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    setError("");
    setLoading(true);

    try {
      const result = await signIn("credentials", {
        email,
        password,
        redirect: false,
      });

      if (result?.error) {
        setError("Identifiants invalides");
      } else {
        router.push("/dashboard");
      }
    } catch {
      setError("Identifiants invalides");
    } finally {
      setLoading(false);
    }
  }

  async function handleGoogleSignIn() {
    await signIn("google", { callbackUrl: "/dashboard" });
  }

  return (
    <Card className="w-full max-w-md">
      <CardHeader>
        <CardTitle className="text-center text-2xl">
          Connexion à LS Immo
        </CardTitle>
      </CardHeader>

      {error && (
        <div className="mb-4 rounded-lg bg-red-50 border border-red-200 p-3 text-sm text-red-700">
          {error}
        </div>
      )}

      <form onSubmit={handleSubmit} className="space-y-4">
        <Input
          label="Adresse email"
          type="email"
          placeholder="vous@exemple.fr"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          required
          autoComplete="email"
        />

        <Input
          label="Mot de passe"
          type="password"
          placeholder="Votre mot de passe"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
          required
          autoComplete="current-password"
        />

        <Button type="submit" loading={loading} className="w-full">
          Se connecter
        </Button>
      </form>

      <div className="relative my-6">
        <div className="absolute inset-0 flex items-center">
          <div className="w-full border-t border-gray-200" />
        </div>
        <div className="relative flex justify-center text-sm">
          <span className="bg-white px-2 text-gray-500">ou</span>
        </div>
      </div>

      <Button
        type="button"
        variant="secondary"
        className="w-full"
        onClick={handleGoogleSignIn}
      >
        <svg className="mr-2 h-4 w-4" viewBox="0 0 24 24">
          <path
            d="M22.56 12.25c0-.78-.07-1.53-.2-2.25H12v4.26h5.92a5.06 5.06 0 01-2.2 3.32v2.77h3.57c2.08-1.92 3.28-4.74 3.28-8.1z"
            fill="#4285F4"
          />
          <path
            d="M12 23c2.97 0 5.46-.98 7.28-2.66l-3.57-2.77c-.98.66-2.23 1.06-3.71 1.06-2.86 0-5.29-1.93-6.16-4.53H2.18v2.84C3.99 20.53 7.7 23 12 23z"
            fill="#34A853"
          />
          <path
            d="M5.84 14.09c-.22-.66-.35-1.36-.35-2.09s.13-1.43.35-2.09V7.07H2.18C1.43 8.55 1 10.22 1 12s.43 3.45 1.18 4.93l2.85-2.22.81-.62z"
            fill="#FBBC05"
          />
          <path
            d="M12 5.38c1.62 0 3.06.56 4.21 1.64l3.15-3.15C17.45 2.09 14.97 1 12 1 7.7 1 3.99 3.47 2.18 7.07l3.66 2.84c.87-2.6 3.3-4.53 6.16-4.53z"
            fill="#EA4335"
          />
        </svg>
        Continuer avec Google
      </Button>

      <div className="mt-6 space-y-2 text-center text-sm text-gray-600">
        <p>
          Pas encore de compte ?{" "}
          <Link
            href="/register"
            className="font-medium text-blue-600 hover:text-blue-500"
          >
            Créer un compte
          </Link>
        </p>
        <p>
          <Link
            href="/reset-password"
            className="font-medium text-blue-600 hover:text-blue-500"
          >
            Mot de passe oublié ?
          </Link>
        </p>
      </div>
    </Card>
  );
}

```

### `src/app/(auth)/register/page.tsx`

```typescript
"use client";

import { useState } from "react";
import Link from "next/link";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Card, CardHeader, CardTitle } from "@/components/ui/card";

interface FieldErrors {
  name?: string;
  email?: string;
  password?: string;
  confirmPassword?: string;
}

export default function RegisterPage() {
  const [name, setName] = useState("");
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [confirmPassword, setConfirmPassword] = useState("");
  const [error, setError] = useState("");
  const [fieldErrors, setFieldErrors] = useState<FieldErrors>({});
  const [success, setSuccess] = useState(false);
  const [loading, setLoading] = useState(false);

  function validateForm(): boolean {
    const errors: FieldErrors = {};

    if (!name.trim()) {
      errors.name = "Le nom est requis";
    }

    if (!email.trim()) {
      errors.email = "L'adresse email est requise";
    }

    if (password.length < 8) {
      errors.password = "Le mot de passe doit contenir au moins 8 caractères";
    } else if (!/[A-Z]/.test(password)) {
      errors.password =
        "Le mot de passe doit contenir au moins une majuscule";
    } else if (!/[a-z]/.test(password)) {
      errors.password =
        "Le mot de passe doit contenir au moins une minuscule";
    } else if (!/[0-9]/.test(password)) {
      errors.password = "Le mot de passe doit contenir au moins un chiffre";
    }

    if (password !== confirmPassword) {
      errors.confirmPassword = "Les mots de passe ne correspondent pas";
    }

    setFieldErrors(errors);
    return Object.keys(errors).length === 0;
  }

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    setError("");
    setFieldErrors({});

    if (!validateForm()) {
      return;
    }

    setLoading(true);

    try {
      const res = await fetch("/api/auth/register", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ name, email, password }),
      });

      const data = await res.json();

      if (!res.ok) {
        setError(data.error || "Une erreur est survenue");
        return;
      }

      setSuccess(true);
    } catch {
      setError("Une erreur est survenue. Veuillez réessayer.");
    } finally {
      setLoading(false);
    }
  }

  if (success) {
    return (
      <Card className="w-full max-w-md">
        <CardHeader>
          <CardTitle className="text-center text-2xl">
            Vérifiez votre email
          </CardTitle>
        </CardHeader>

        <div className="rounded-lg bg-green-50 border border-green-200 p-4 text-sm text-green-700">
          Un email de vérification a été envoyé à{" "}
          <strong>{email}</strong>. Cliquez sur le lien dans l&apos;email pour
          activer votre compte.
        </div>

        <div className="mt-6 text-center text-sm text-gray-600">
          <Link
            href="/login"
            className="font-medium text-blue-600 hover:text-blue-500"
          >
            Retour à la connexion
          </Link>
        </div>
      </Card>
    );
  }

  return (
    <Card className="w-full max-w-md">
      <CardHeader>
        <CardTitle className="text-center text-2xl">
          Créer un compte LS Immo
        </CardTitle>
      </CardHeader>

      {error && (
        <div className="mb-4 rounded-lg bg-red-50 border border-red-200 p-3 text-sm text-red-700">
          {error}
        </div>
      )}

      <form onSubmit={handleSubmit} className="space-y-4">
        <Input
          label="Nom complet"
          type="text"
          placeholder="Jean Dupont"
          value={name}
          onChange={(e) => setName(e.target.value)}
          error={fieldErrors.name}
          required
          autoComplete="name"
        />

        <Input
          label="Adresse email"
          type="email"
          placeholder="vous@exemple.fr"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          error={fieldErrors.email}
          required
          autoComplete="email"
        />

        <Input
          label="Mot de passe"
          type="password"
          placeholder="8 caractères min., majuscule, minuscule, chiffre"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
          error={fieldErrors.password}
          required
          autoComplete="new-password"
        />

        <Input
          label="Confirmer le mot de passe"
          type="password"
          placeholder="Confirmez votre mot de passe"
          value={confirmPassword}
          onChange={(e) => setConfirmPassword(e.target.value)}
          error={fieldErrors.confirmPassword}
          required
          autoComplete="new-password"
        />

        <Button type="submit" loading={loading} className="w-full">
          Créer mon compte
        </Button>
      </form>

      <div className="mt-6 text-center text-sm text-gray-600">
        Déjà un compte ?{" "}
        <Link
          href="/login"
          className="font-medium text-blue-600 hover:text-blue-500"
        >
          Se connecter
        </Link>
      </div>
    </Card>
  );
}

```

### `src/app/(auth)/reset-password/page.tsx`

```typescript
"use client";

import { Suspense, useState } from "react";
import { useSearchParams } from "next/navigation";
import Link from "next/link";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Card, CardHeader, CardTitle } from "@/components/ui/card";

function RequestResetForm() {
  const [email, setEmail] = useState("");
  const [loading, setLoading] = useState(false);
  const [submitted, setSubmitted] = useState(false);
  const [error, setError] = useState("");

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    setError("");
    setLoading(true);

    try {
      const res = await fetch("/api/auth/reset-password", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ email }),
      });

      if (!res.ok) {
        const data = await res.json();
        setError(data.error || "Une erreur est survenue");
        return;
      }

      setSubmitted(true);
    } catch {
      setError("Une erreur est survenue. Veuillez réessayer.");
    } finally {
      setLoading(false);
    }
  }

  if (submitted) {
    return (
      <Card className="w-full max-w-md">
        <CardHeader>
          <CardTitle className="text-center text-2xl">
            Email envoyé
          </CardTitle>
        </CardHeader>

        <div className="rounded-lg bg-green-50 border border-green-200 p-4 text-sm text-green-700">
          Si un compte existe avec cette adresse email, vous recevrez un lien
          pour réinitialiser votre mot de passe.
        </div>

        <div className="mt-6 text-center text-sm text-gray-600">
          <Link
            href="/login"
            className="font-medium text-blue-600 hover:text-blue-500"
          >
            Retour à la connexion
          </Link>
        </div>
      </Card>
    );
  }

  return (
    <Card className="w-full max-w-md">
      <CardHeader>
        <CardTitle className="text-center text-2xl">
          Mot de passe oublié
        </CardTitle>
      </CardHeader>

      <p className="mb-4 text-sm text-gray-600">
        Entrez votre adresse email et nous vous enverrons un lien pour
        réinitialiser votre mot de passe.
      </p>

      {error && (
        <div className="mb-4 rounded-lg bg-red-50 border border-red-200 p-3 text-sm text-red-700">
          {error}
        </div>
      )}

      <form onSubmit={handleSubmit} className="space-y-4">
        <Input
          label="Adresse email"
          type="email"
          placeholder="vous@exemple.fr"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          required
          autoComplete="email"
        />

        <Button type="submit" loading={loading} className="w-full">
          Envoyer le lien
        </Button>
      </form>

      <div className="mt-6 text-center text-sm text-gray-600">
        <Link
          href="/login"
          className="font-medium text-blue-600 hover:text-blue-500"
        >
          Retour à la connexion
        </Link>
      </div>
    </Card>
  );
}

function ResetPasswordForm({ token }: { token: string }) {
  const [password, setPassword] = useState("");
  const [confirmPassword, setConfirmPassword] = useState("");
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState("");
  const [success, setSuccess] = useState(false);
  const [passwordError, setPasswordError] = useState("");
  const [confirmError, setConfirmError] = useState("");

  function validateForm(): boolean {
    let valid = true;
    setPasswordError("");
    setConfirmError("");

    if (password.length < 8) {
      setPasswordError(
        "Le mot de passe doit contenir au moins 8 caractères"
      );
      valid = false;
    } else if (!/[A-Z]/.test(password)) {
      setPasswordError(
        "Le mot de passe doit contenir au moins une majuscule"
      );
      valid = false;
    } else if (!/[a-z]/.test(password)) {
      setPasswordError(
        "Le mot de passe doit contenir au moins une minuscule"
      );
      valid = false;
    } else if (!/[0-9]/.test(password)) {
      setPasswordError(
        "Le mot de passe doit contenir au moins un chiffre"
      );
      valid = false;
    }

    if (password !== confirmPassword) {
      setConfirmError("Les mots de passe ne correspondent pas");
      valid = false;
    }

    return valid;
  }

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    setError("");

    if (!validateForm()) {
      return;
    }

    setLoading(true);

    try {
      const res = await fetch("/api/auth/reset-password/confirm", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ token, password }),
      });

      const data = await res.json();

      if (!res.ok) {
        setError(data.error || "Une erreur est survenue");
        return;
      }

      setSuccess(true);
    } catch {
      setError("Une erreur est survenue. Veuillez réessayer.");
    } finally {
      setLoading(false);
    }
  }

  if (success) {
    return (
      <Card className="w-full max-w-md">
        <CardHeader>
          <CardTitle className="text-center text-2xl">
            Mot de passe réinitialisé
          </CardTitle>
        </CardHeader>

        <div className="rounded-lg bg-green-50 border border-green-200 p-4 text-sm text-green-700">
          Votre mot de passe a été modifié avec succès. Vous pouvez maintenant
          vous connecter.
        </div>

        <div className="mt-6 text-center">
          <Link href="/login">
            <Button variant="primary">Se connecter</Button>
          </Link>
        </div>
      </Card>
    );
  }

  return (
    <Card className="w-full max-w-md">
      <CardHeader>
        <CardTitle className="text-center text-2xl">
          Nouveau mot de passe
        </CardTitle>
      </CardHeader>

      {error && (
        <div className="mb-4 rounded-lg bg-red-50 border border-red-200 p-3 text-sm text-red-700">
          {error}
        </div>
      )}

      <form onSubmit={handleSubmit} className="space-y-4">
        <Input
          label="Nouveau mot de passe"
          type="password"
          placeholder="8 caractères min., majuscule, minuscule, chiffre"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
          error={passwordError}
          required
          autoComplete="new-password"
        />

        <Input
          label="Confirmer le mot de passe"
          type="password"
          placeholder="Confirmez votre mot de passe"
          value={confirmPassword}
          onChange={(e) => setConfirmPassword(e.target.value)}
          error={confirmError}
          required
          autoComplete="new-password"
        />

        <Button type="submit" loading={loading} className="w-full">
          Réinitialiser le mot de passe
        </Button>
      </form>

      <div className="mt-6 text-center text-sm text-gray-600">
        <Link
          href="/login"
          className="font-medium text-blue-600 hover:text-blue-500"
        >
          Retour à la connexion
        </Link>
      </div>
    </Card>
  );
}

function ResetPasswordContent() {
  const searchParams = useSearchParams();
  const token = searchParams.get("token");

  if (token) {
    return <ResetPasswordForm token={token} />;
  }

  return <RequestResetForm />;
}

export default function ResetPasswordPage() {
  return (
    <Suspense fallback={<div className="text-center py-8">Chargement...</div>}>
      <ResetPasswordContent />
    </Suspense>
  );
}

```

### `src/app/(auth)/verify-email/page.tsx`

```typescript
"use client";

import { Suspense, useEffect, useState, useCallback } from "react";
import { useSearchParams } from "next/navigation";
import Link from "next/link";
import { Card, CardHeader, CardTitle } from "@/components/ui/card";
import { Button } from "@/components/ui/button";

function VerifyEmailContent() {
  const searchParams = useSearchParams();
  const token = searchParams.get("token");

  const [status, setStatus] = useState<"loading" | "success" | "error">(
    "loading"
  );
  const [message, setMessage] = useState("");

  const verifyEmail = useCallback(async (verificationToken: string) => {
    try {
      const res = await fetch(
        `/api/auth/verify-email?token=${encodeURIComponent(verificationToken)}`
      );
      const data = await res.json();

      if (res.ok) {
        setStatus("success");
        setMessage("Votre adresse email a été vérifiée avec succès.");
      } else {
        setStatus("error");
        setMessage(data.error || "Le lien de vérification est invalide ou expiré.");
      }
    } catch {
      setStatus("error");
      setMessage("Une erreur est survenue. Veuillez réessayer.");
    }
  }, []);

  useEffect(() => {
    if (!token) {
      setStatus("error");
      setMessage("Aucun jeton de vérification fourni.");
      return;
    }

    verifyEmail(token);
  }, [token, verifyEmail]);

  return (
    <Card className="w-full max-w-md">
      <CardHeader>
        <CardTitle className="text-center text-2xl">
          Vérification de l&apos;email
        </CardTitle>
      </CardHeader>

      {status === "loading" && (
        <div className="flex flex-col items-center py-8">
          <svg
            className="animate-spin h-8 w-8 text-blue-600 mb-4"
            fill="none"
            viewBox="0 0 24 24"
          >
            <circle
              className="opacity-25"
              cx="12"
              cy="12"
              r="10"
              stroke="currentColor"
              strokeWidth="4"
            />
            <path
              className="opacity-75"
              fill="currentColor"
              d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4z"
            />
          </svg>
          <p className="text-sm text-gray-600">Vérification en cours...</p>
        </div>
      )}

      {status === "success" && (
        <div className="rounded-lg bg-green-50 border border-green-200 p-4 text-sm text-green-700">
          {message}
        </div>
      )}

      {status === "error" && (
        <div className="rounded-lg bg-red-50 border border-red-200 p-4 text-sm text-red-700">
          {message}
        </div>
      )}

      {status !== "loading" && (
        <div className="mt-6 text-center">
          <Link href="/login">
            <Button variant="primary">Se connecter</Button>
          </Link>
        </div>
      )}
    </Card>
  );
}

export default function VerifyEmailPage() {
  return (
    <Suspense fallback={<div className="text-center py-8">Chargement...</div>}>
      <VerifyEmailContent />
    </Suspense>
  );
}

```

### `src/app/(dashboard)/layout.tsx`

```typescript
import { Sidebar } from "@/components/layout/sidebar";

export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div className="flex min-h-screen bg-gray-50">
      <Sidebar />
      <main className="flex-1 p-6 md:p-8">{children}</main>
    </div>
  );
}

```

### `src/app/(dashboard)/dashboard/page.tsx`

```typescript
"use client";

import { useState, useEffect } from "react";
import Link from "next/link";
import { Button } from "@/components/ui/button";
import { Card } from "@/components/ui/card";

interface Property {
  id: string;
  name: string;
  rentalType: string;
  streetNumber: string | null;
  streetName: string;
  postalCode: string;
  city: string;
  contracts: { id: string }[];
  leases: { id: string }[];
}

const rentalTypeLabels: Record<string, string> = {
  SEASONAL: "Saisonnière",
  CLASSIC_FURNISHED: "Classique meublée",
  CLASSIC_UNFURNISHED: "Classique non meublée",
};

function PropertyCard({ property }: { property: Property }) {
  const occupied =
    property.contracts.length > 0 || property.leases.length > 0;

  const address = [
    property.streetNumber,
    property.streetName,
  ]
    .filter(Boolean)
    .join(" ");

  return (
    <Link href={`/properties/${property.id}`}>
      <Card className="hover:shadow-md transition-shadow cursor-pointer">
        <div className="p-5">
          <div className="flex items-start justify-between">
            <div>
              <h3 className="font-semibold text-gray-900">{property.name}</h3>
              <p className="text-sm text-gray-500 mt-1">
                {address}, {property.postalCode} {property.city}
              </p>
            </div>
            <span
              className={`inline-flex items-center rounded-full px-2.5 py-0.5 text-xs font-medium ${
                occupied
                  ? "bg-green-100 text-green-800"
                  : "bg-gray-100 text-gray-600"
              }`}
            >
              {occupied ? "Occupé" : "Libre"}
            </span>
          </div>
          <div className="mt-3">
            <span className="inline-flex items-center rounded-md bg-blue-50 px-2 py-1 text-xs font-medium text-blue-700">
              {rentalTypeLabels[property.rentalType] || property.rentalType}
            </span>
          </div>
        </div>
      </Card>
    </Link>
  );
}

function EmptyState() {
  return (
    <div className="text-center py-16">
      <svg
        className="mx-auto h-12 w-12 text-gray-400"
        fill="none"
        viewBox="0 0 24 24"
        stroke="currentColor"
      >
        <path
          strokeLinecap="round"
          strokeLinejoin="round"
          strokeWidth={1.5}
          d="M19 21V5a2 2 0 00-2-2H7a2 2 0 00-2 2v16m14 0h2m-2 0h-5m-9 0H3m2 0h5M9 7h1m-1 4h1m4-4h1m-1 4h1m-5 10v-5a1 1 0 011-1h2a1 1 0 011 1v5m-4 0h4"
        />
      </svg>
      <h3 className="mt-4 text-lg font-medium text-gray-900">
        Aucun bien enregistré
      </h3>
      <p className="mt-2 text-sm text-gray-500">
        Commencez par ajouter votre premier bien immobilier.
      </p>
      <div className="mt-6">
        <Link href="/properties/new">
          <Button>Ajouter un bien</Button>
        </Link>
      </div>
    </div>
  );
}

export default function DashboardPage() {
  const [properties, setProperties] = useState<Property[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch("/api/properties")
      .then((res) => res.json())
      .then((data) => setProperties(Array.isArray(data) ? data : []))
      .catch(() => setProperties([]))
      .finally(() => setLoading(false));
  }, []);

  if (loading) {
    return (
      <div className="flex items-center justify-center py-16">
        <p className="text-sm text-gray-500">Chargement...</p>
      </div>
    );
  }

  return (
    <div>
      <div className="flex items-center justify-between mb-6">
        <h1 className="text-2xl font-bold text-gray-900">Tableau de bord</h1>
        {properties.length > 0 && (
          <Link href="/properties/new">
            <Button>Ajouter un bien</Button>
          </Link>
        )}
      </div>

      {properties.length === 0 ? (
        <EmptyState />
      ) : (
        <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
          {properties.map((p) => (
            <PropertyCard key={p.id} property={p} />
          ))}
        </div>
      )}
    </div>
  );
}

```

### `src/app/(dashboard)/payments/page.tsx`

```typescript
"use client";

import { useState, useEffect } from "react";
import { Card, CardHeader, CardTitle } from "@/components/ui/card";
import { Button } from "@/components/ui/button";

interface Payment {
  id: string;
  amount: number;
  status: string;
  method: string;
  paidAt: string | null;
  createdAt: string;
  contract?: { tenantName: string; property: { name: string } } | null;
  rentEntry?: { lease: { tenantName: string; property: { name: string } }; month: string } | null;
}

const statusLabels: Record<string, string> = {
  PENDING: "En attente",
  COMPLETED: "Payé",
  FAILED: "Échoué",
  REFUNDED: "Remboursé",
};

const statusColors: Record<string, string> = {
  PENDING: "text-amber-600",
  COMPLETED: "text-green-600",
  FAILED: "text-red-600",
  REFUNDED: "text-gray-600",
};

const methodLabels: Record<string, string> = {
  STRIPE: "Carte bancaire",
  CASH: "Espèces",
  CHECK: "Chèque",
  BANK_TRANSFER: "Virement",
};

function fmt(d: string) {
  return new Date(d).toLocaleDateString("fr-FR", { day: "2-digit", month: "short", year: "numeric" });
}
function cur(n: number) {
  return new Intl.NumberFormat("fr-FR", { style: "currency", currency: "EUR" }).format(n);
}

export default function PaymentsPage() {
  const [payments, setPayments] = useState<Payment[]>([]);
  const [loading, setLoading] = useState(true);
  const [stripeReady, setStripeReady] = useState(false);

  useEffect(() => {
    Promise.all([
      fetch("/api/user/profile").then((r) => r.json()),
      fetch("/api/payments").then((r) => r.json()).catch(() => []),
    ]).then(([profile, paymentsData]) => {
      setStripeReady(!!profile.stripeOnboarded);
      setPayments(Array.isArray(paymentsData) ? paymentsData : []);
      setLoading(false);
    }).catch(() => setLoading(false));
  }, []);

  async function handleStripeOnboard() {
    const res = await fetch("/api/stripe/onboard", { method: "POST" });
    const data = await res.json();
    if (data.url) {
      window.location.href = data.url;
    }
  }

  // Compute stats
  const completed = payments.filter((p) => p.status === "COMPLETED");
  const totalCollected = completed.reduce((sum, p) => sum + p.amount, 0);
  const pending = payments.filter((p) => p.status === "PENDING");
  const totalPending = pending.reduce((sum, p) => sum + p.amount, 0);

  if (loading) {
    return <div className="flex items-center justify-center py-16"><p className="text-sm text-gray-500">Chargement...</p></div>;
  }

  return (
    <div>
      <h1 className="text-2xl font-bold text-gray-900 mb-6">Paiements</h1>

      {!stripeReady && (
        <div className="mb-6 rounded-lg bg-amber-50 border border-amber-200 p-4">
          <p className="text-sm text-amber-700 mb-3">
            Configurez votre compte Stripe pour recevoir des paiements en ligne.
          </p>
          <Button size="sm" onClick={handleStripeOnboard}>
            Configurer Stripe
          </Button>
        </div>
      )}

      {/* Stats */}
      <div className="grid grid-cols-1 md:grid-cols-3 gap-4 mb-6">
        <Card>
          <div className="p-4">
            <p className="text-sm text-gray-500">Total encaissé</p>
            <p className="text-2xl font-bold text-green-600">{cur(totalCollected)}</p>
          </div>
        </Card>
        <Card>
          <div className="p-4">
            <p className="text-sm text-gray-500">En attente</p>
            <p className="text-2xl font-bold text-amber-600">{cur(totalPending)}</p>
          </div>
        </Card>
        <Card>
          <div className="p-4">
            <p className="text-sm text-gray-500">Transactions</p>
            <p className="text-2xl font-bold text-gray-900">{payments.length}</p>
          </div>
        </Card>
      </div>

      {/* Transactions list */}
      <Card>
        <CardHeader><CardTitle>Historique des transactions</CardTitle></CardHeader>

        {payments.length === 0 ? (
          <p className="text-sm text-gray-500">Aucune transaction pour le moment.</p>
        ) : (
          <div className="overflow-x-auto">
            <table className="w-full text-sm">
              <thead>
                <tr className="border-b text-left text-gray-500">
                  <th className="pb-2 font-medium">Date</th>
                  <th className="pb-2 font-medium">Description</th>
                  <th className="pb-2 font-medium">Méthode</th>
                  <th className="pb-2 font-medium text-right">Montant</th>
                  <th className="pb-2 font-medium text-right">Statut</th>
                </tr>
              </thead>
              <tbody>
                {payments.map((p) => (
                  <tr key={p.id} className="border-b last:border-0">
                    <td className="py-3">{fmt(p.paidAt || p.createdAt)}</td>
                    <td className="py-3">
                      {p.contract
                        ? `${p.contract.tenantName} — ${p.contract.property.name}`
                        : p.rentEntry
                        ? `${p.rentEntry.lease.tenantName} — ${p.rentEntry.lease.property.name}`
                        : "Paiement"}
                    </td>
                    <td className="py-3">{methodLabels[p.method] || p.method}</td>
                    <td className="py-3 text-right font-medium">{cur(p.amount)}</td>
                    <td className={`py-3 text-right font-medium ${statusColors[p.status]}`}>
                      {statusLabels[p.status]}
                    </td>
                  </tr>
                ))}
              </tbody>
            </table>
          </div>
        )}
      </Card>
    </div>
  );
}

```

### `src/app/(dashboard)/profile/page.tsx`

```typescript
"use client";

import { useState, useEffect } from "react";
import { signOut } from "next-auth/react";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Card, CardHeader, CardTitle } from "@/components/ui/card";

interface UserProfile {
  id: string;
  name: string | null;
  email: string;
  phone: string | null;
  emailVerified: string | null;
}

export default function ProfilePage() {
  const [user, setUser] = useState<UserProfile | null>(null);
  const [name, setName] = useState("");
  const [email, setEmail] = useState("");
  const [phone, setPhone] = useState("");
  const [loading, setLoading] = useState(true);
  const [saving, setSaving] = useState(false);
  const [error, setError] = useState("");
  const [success, setSuccess] = useState("");

  useEffect(() => {
    fetch("/api/user/profile")
      .then((res) => res.json())
      .then((data) => {
        setUser(data);
        setName(data.name || "");
        setEmail(data.email || "");
        setPhone(data.phone || "");
      })
      .catch(() => setError("Impossible de charger le profil"))
      .finally(() => setLoading(false));
  }, []);

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    setError("");
    setSuccess("");
    setSaving(true);

    try {
      const res = await fetch("/api/user/profile", {
        method: "PATCH",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ name, email, phone }),
      });

      const data = await res.json();

      if (!res.ok) {
        setError(data.error || "Une erreur est survenue");
        return;
      }

      setUser(data);
      const emailChanged = data.email !== user?.email;
      setSuccess(
        emailChanged
          ? "Profil mis à jour. Un email de vérification a été envoyé à votre nouvelle adresse."
          : "Profil mis à jour avec succès."
      );
    } catch {
      setError("Une erreur est survenue. Veuillez réessayer.");
    } finally {
      setSaving(false);
    }
  }

  if (loading) {
    return (
      <div className="flex items-center justify-center py-16">
        <p className="text-sm text-gray-500">Chargement...</p>
      </div>
    );
  }

  return (
    <div className="max-w-2xl mx-auto">
      <h1 className="text-2xl font-bold text-gray-900 mb-6">Mon profil</h1>

      <Card>
        <CardHeader>
          <CardTitle>Informations personnelles</CardTitle>
        </CardHeader>

        {error && (
          <div className="mb-4 rounded-lg bg-red-50 border border-red-200 p-3 text-sm text-red-700">
            {error}
          </div>
        )}

        {success && (
          <div className="mb-4 rounded-lg bg-green-50 border border-green-200 p-3 text-sm text-green-700">
            {success}
          </div>
        )}

        <form onSubmit={handleSubmit} className="space-y-4">
          <Input
            label="Nom complet"
            type="text"
            value={name}
            onChange={(e) => setName(e.target.value)}
            required
            autoComplete="name"
          />

          <div>
            <Input
              label="Adresse email"
              type="email"
              value={email}
              onChange={(e) => setEmail(e.target.value)}
              required
              autoComplete="email"
            />
            {user && !user.emailVerified && (
              <p className="mt-1 text-xs text-amber-600">
                Email non vérifié — vérifiez votre boîte de réception.
              </p>
            )}
          </div>

          <Input
            label="Téléphone"
            type="tel"
            value={phone}
            onChange={(e) => setPhone(e.target.value)}
            placeholder="06 12 34 56 78"
            autoComplete="tel"
          />

          <Button type="submit" loading={saving} className="w-full">
            Enregistrer
          </Button>
        </form>
      </Card>

      <div className="mt-6">
        <Button
          variant="secondary"
          className="w-full"
          onClick={() => signOut({ callbackUrl: "/login" })}
        >
          Se déconnecter
        </Button>
      </div>
    </div>
  );
}

```

### `src/app/(dashboard)/properties/new/page.tsx`

```typescript
"use client";

import { useState } from "react";
import { useRouter } from "next/navigation";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Select } from "@/components/ui/select";
import { Card, CardHeader, CardTitle } from "@/components/ui/card";

const rentalTypes = [
  { value: "SEASONAL", label: "Location saisonnière" },
  { value: "CLASSIC_FURNISHED", label: "Location classique meublée" },
  { value: "CLASSIC_UNFURNISHED", label: "Location classique non meublée" },
];

export default function NewPropertyPage() {
  const router = useRouter();
  const [name, setName] = useState("");
  const [rentalType, setRentalType] = useState("SEASONAL");
  const [streetNumber, setStreetNumber] = useState("");
  const [streetName, setStreetName] = useState("");
  const [complement, setComplement] = useState("");
  const [postalCode, setPostalCode] = useState("");
  const [city, setCity] = useState("");
  const [error, setError] = useState("");
  const [loading, setLoading] = useState(false);

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    setError("");

    // Client-side postal code validation
    if (!/^\d{5}$/.test(postalCode)) {
      setError("Le code postal doit contenir exactement 5 chiffres");
      return;
    }

    setLoading(true);

    try {
      const res = await fetch("/api/properties", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          name,
          rentalType,
          streetNumber: streetNumber || null,
          streetName,
          complement: complement || null,
          postalCode,
          city,
        }),
      });

      const data = await res.json();

      if (!res.ok) {
        setError(data.error || "Une erreur est survenue");
        return;
      }

      router.push(`/properties/${data.id}`);
    } catch {
      setError("Une erreur est survenue. Veuillez réessayer.");
    } finally {
      setLoading(false);
    }
  }

  return (
    <div className="max-w-2xl mx-auto">
      <h1 className="text-2xl font-bold text-gray-900 mb-6">Ajouter un bien</h1>

      <Card>
        <CardHeader>
          <CardTitle>Informations du bien</CardTitle>
        </CardHeader>

        {error && (
          <div className="mb-4 rounded-lg bg-red-50 border border-red-200 p-3 text-sm text-red-700">
            {error}
          </div>
        )}

        <form onSubmit={handleSubmit} className="space-y-4">
          <Input
            label="Nom du bien"
            type="text"
            placeholder="Appartement Paris 11e"
            value={name}
            onChange={(e) => setName(e.target.value)}
            required
          />

          <Select
            label="Type de location"
            value={rentalType}
            onChange={(e) => setRentalType(e.target.value)}
            options={rentalTypes}
          />

          <div className="grid grid-cols-3 gap-3">
            <Input
              label="N° de rue"
              type="text"
              placeholder="12"
              value={streetNumber}
              onChange={(e) => setStreetNumber(e.target.value)}
            />
            <div className="col-span-2">
              <Input
                label="Nom de rue"
                type="text"
                placeholder="Rue de la Paix"
                value={streetName}
                onChange={(e) => setStreetName(e.target.value)}
                required
              />
            </div>
          </div>

          <Input
            label="Complément d'adresse"
            type="text"
            placeholder="Bât. A, 3e étage"
            value={complement}
            onChange={(e) => setComplement(e.target.value)}
          />

          <div className="grid grid-cols-2 gap-3">
            <Input
              label="Code postal"
              type="text"
              placeholder="75011"
              value={postalCode}
              onChange={(e) => setPostalCode(e.target.value)}
              required
              maxLength={5}
            />
            <Input
              label="Ville"
              type="text"
              placeholder="Paris"
              value={city}
              onChange={(e) => setCity(e.target.value)}
              required
            />
          </div>

          <div className="flex gap-3 pt-2">
            <Button
              type="button"
              variant="secondary"
              className="flex-1"
              onClick={() => router.back()}
            >
              Annuler
            </Button>
            <Button type="submit" loading={loading} className="flex-1">
              Créer le bien
            </Button>
          </div>
        </form>
      </Card>
    </div>
  );
}

```

### `src/app/(dashboard)/properties/[id]/page.tsx`

```typescript
"use client";

import { useState, useEffect, useCallback } from "react";
import { useRouter, useParams } from "next/navigation";
import Link from "next/link";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Select } from "@/components/ui/select";
import { Card, CardHeader, CardTitle } from "@/components/ui/card";
import { Modal } from "@/components/ui/modal";

interface FurnitureItem {
  id: string;
  name: string;
  quantity: number;
  condition: string;
  photoUrl: string | null;
}

interface Property {
  id: string;
  name: string;
  rentalType: string;
  streetNumber: string | null;
  streetName: string;
  complement: string | null;
  postalCode: string;
  city: string;
  furnitureItems: FurnitureItem[];
  contracts: { id: string; status: string }[];
  leases: { id: string; status: string }[];
}

const rentalTypeLabels: Record<string, string> = {
  SEASONAL: "Location saisonnière",
  CLASSIC_FURNISHED: "Location classique meublée",
  CLASSIC_UNFURNISHED: "Location classique non meublée",
};

const conditionLabels: Record<string, string> = {
  NEW: "Neuf",
  GOOD: "Bon",
  FAIR: "Correct",
  POOR: "Mauvais",
};

const conditionOptions = [
  { value: "NEW", label: "Neuf" },
  { value: "GOOD", label: "Bon" },
  { value: "FAIR", label: "Correct" },
  { value: "POOR", label: "Mauvais" },
];

export default function PropertyDetailPage() {
  const router = useRouter();
  const { id } = useParams<{ id: string }>();

  const [property, setProperty] = useState<Property | null>(null);
  const [loading, setLoading] = useState(true);
  const [editing, setEditing] = useState(false);
  const [saving, setSaving] = useState(false);
  const [error, setError] = useState("");
  const [success, setSuccess] = useState("");

  // Edit form state
  const [editName, setEditName] = useState("");
  const [editStreetNumber, setEditStreetNumber] = useState("");
  const [editStreetName, setEditStreetName] = useState("");
  const [editComplement, setEditComplement] = useState("");
  const [editPostalCode, setEditPostalCode] = useState("");
  const [editCity, setEditCity] = useState("");

  // Furniture modal
  const [showFurnitureModal, setShowFurnitureModal] = useState(false);
  const [furnitureName, setFurnitureName] = useState("");
  const [furnitureQty, setFurnitureQty] = useState("1");
  const [furnitureCondition, setFurnitureCondition] = useState("GOOD");
  const [addingFurniture, setAddingFurniture] = useState(false);

  // Delete confirmation
  const [showDeleteConfirm, setShowDeleteConfirm] = useState(false);
  const [deleting, setDeleting] = useState(false);

  const fetchProperty = useCallback(async () => {
    try {
      const res = await fetch(`/api/properties/${id}`);
      if (!res.ok) {
        setError("Bien introuvable");
        return;
      }
      const data = await res.json();
      setProperty(data);
      setEditName(data.name);
      setEditStreetNumber(data.streetNumber || "");
      setEditStreetName(data.streetName);
      setEditComplement(data.complement || "");
      setEditPostalCode(data.postalCode);
      setEditCity(data.city);
    } catch {
      setError("Impossible de charger le bien");
    } finally {
      setLoading(false);
    }
  }, [id]);

  useEffect(() => {
    fetchProperty();
  }, [fetchProperty]);

  async function handleSave(e: React.FormEvent) {
    e.preventDefault();
    setError("");
    setSuccess("");
    setSaving(true);

    try {
      const res = await fetch(`/api/properties/${id}`, {
        method: "PATCH",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          name: editName,
          streetNumber: editStreetNumber || null,
          streetName: editStreetName,
          complement: editComplement || null,
          postalCode: editPostalCode,
          city: editCity,
        }),
      });

      const data = await res.json();

      if (!res.ok) {
        setError(data.error || "Une erreur est survenue");
        return;
      }

      setProperty((prev) => (prev ? { ...prev, ...data } : prev));
      setEditing(false);
      setSuccess("Bien mis à jour avec succès");
    } catch {
      setError("Une erreur est survenue");
    } finally {
      setSaving(false);
    }
  }

  async function handleAddFurniture(e: React.FormEvent) {
    e.preventDefault();
    setAddingFurniture(true);

    try {
      const res = await fetch(`/api/properties/${id}/furniture`, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          name: furnitureName,
          quantity: parseInt(furnitureQty) || 1,
          condition: furnitureCondition,
        }),
      });

      if (!res.ok) {
        const data = await res.json();
        setError(data.error || "Erreur lors de l'ajout");
        return;
      }

      setShowFurnitureModal(false);
      setFurnitureName("");
      setFurnitureQty("1");
      setFurnitureCondition("GOOD");
      fetchProperty();
    } catch {
      setError("Une erreur est survenue");
    } finally {
      setAddingFurniture(false);
    }
  }

  async function handleDelete() {
    setDeleting(true);
    try {
      const res = await fetch(`/api/properties/${id}`, { method: "DELETE" });
      const data = await res.json();

      if (!res.ok) {
        setError(data.error || "Impossible de supprimer");
        setShowDeleteConfirm(false);
        setDeleting(false);
        return;
      }

      router.push("/dashboard");
    } catch {
      setError("Une erreur est survenue");
      setDeleting(false);
    }
  }

  if (loading) {
    return (
      <div className="flex items-center justify-center py-16">
        <p className="text-sm text-gray-500">Chargement...</p>
      </div>
    );
  }

  if (!property) {
    return (
      <div className="text-center py-16">
        <p className="text-gray-500">{error || "Bien introuvable"}</p>
        <Button variant="secondary" className="mt-4" onClick={() => router.push("/dashboard")}>
          Retour au tableau de bord
        </Button>
      </div>
    );
  }

  const hasFurniture = property.rentalType !== "CLASSIC_UNFURNISHED";

  return (
    <div className="max-w-3xl mx-auto">
      <div className="flex items-center justify-between mb-6">
        <h1 className="text-2xl font-bold text-gray-900">{property.name}</h1>
        <div className="flex gap-2">
          {!editing && (
            <Button variant="secondary" onClick={() => setEditing(true)}>
              Modifier
            </Button>
          )}
          <Button variant="danger" onClick={() => setShowDeleteConfirm(true)}>
            Supprimer
          </Button>
        </div>
      </div>

      {error && (
        <div className="mb-4 rounded-lg bg-red-50 border border-red-200 p-3 text-sm text-red-700">
          {error}
        </div>
      )}

      {success && (
        <div className="mb-4 rounded-lg bg-green-50 border border-green-200 p-3 text-sm text-green-700">
          {success}
        </div>
      )}

      {/* Property info / edit */}
      <Card className="mb-6">
        <CardHeader>
          <CardTitle>Informations</CardTitle>
        </CardHeader>

        {editing ? (
          <form onSubmit={handleSave} className="space-y-4">
            <Input label="Nom" value={editName} onChange={(e) => setEditName(e.target.value)} required />
            <div className="grid grid-cols-3 gap-3">
              <Input label="N°" value={editStreetNumber} onChange={(e) => setEditStreetNumber(e.target.value)} />
              <div className="col-span-2">
                <Input label="Rue" value={editStreetName} onChange={(e) => setEditStreetName(e.target.value)} required />
              </div>
            </div>
            <Input label="Complément" value={editComplement} onChange={(e) => setEditComplement(e.target.value)} />
            <div className="grid grid-cols-2 gap-3">
              <Input label="Code postal" value={editPostalCode} onChange={(e) => setEditPostalCode(e.target.value)} required maxLength={5} />
              <Input label="Ville" value={editCity} onChange={(e) => setEditCity(e.target.value)} required />
            </div>
            <div className="flex gap-3">
              <Button type="button" variant="secondary" onClick={() => setEditing(false)}>
                Annuler
              </Button>
              <Button type="submit" loading={saving}>
                Enregistrer
              </Button>
            </div>
          </form>
        ) : (
          <div className="space-y-2 text-sm">
            <div className="flex justify-between">
              <span className="text-gray-500">Type</span>
              <span className="font-medium">
                {rentalTypeLabels[property.rentalType]}
              </span>
            </div>
            <div className="flex justify-between">
              <span className="text-gray-500">Adresse</span>
              <span className="font-medium text-right">
                {[property.streetNumber, property.streetName].filter(Boolean).join(" ")}
                {property.complement && <>, {property.complement}</>}
                <br />
                {property.postalCode} {property.city}
              </span>
            </div>
          </div>
        )}
      </Card>

      {/* Quick actions */}
      <Card className="mb-6">
        <CardHeader><CardTitle>Actions rapides</CardTitle></CardHeader>
        <div className="flex flex-wrap gap-3">
          {property.rentalType === "SEASONAL" ? (
            <>
              <Link href={`/properties/${id}/contracts/new`}>
                <Button size="sm">Nouveau contrat</Button>
              </Link>
              <Link href={`/properties/${id}/calendar`}>
                <Button size="sm" variant="secondary">Calendrier</Button>
              </Link>
            </>
          ) : (
            <Link href={`/properties/${id}/leases/new`}>
              <Button size="sm">Nouveau bail</Button>
            </Link>
          )}
        </div>

        {/* Active contracts */}
        {property.contracts.length > 0 && (
          <div className="mt-4">
            <p className="text-sm font-medium text-gray-700 mb-2">Contrats actifs</p>
            {property.contracts.map((c) => (
              <Link key={c.id} href={`/properties/${id}/contracts/${c.id}`} className="block text-sm text-blue-600 hover:text-blue-500 mb-1">
                Contrat {c.status} →
              </Link>
            ))}
          </div>
        )}

        {/* Active leases */}
        {property.leases.length > 0 && (
          <div className="mt-4">
            <p className="text-sm font-medium text-gray-700 mb-2">Baux actifs</p>
            {property.leases.map((l) => (
              <Link key={l.id} href={`/properties/${id}/leases/${l.id}`} className="block text-sm text-blue-600 hover:text-blue-500 mb-1">
                Bail {l.status} →
              </Link>
            ))}
          </div>
        )}
      </Card>

      {/* Furniture inventory */}
      {hasFurniture && (
        <Card>
          <CardHeader>
            <div className="flex items-center justify-between">
              <CardTitle>Inventaire mobilier</CardTitle>
              <Button size="sm" onClick={() => setShowFurnitureModal(true)}>
                Ajouter
              </Button>
            </div>
          </CardHeader>

          {property.furnitureItems.length === 0 ? (
            <p className="text-sm text-gray-500">Aucun élément dans l&apos;inventaire.</p>
          ) : (
            <div className="overflow-x-auto">
              <table className="w-full text-sm">
                <thead>
                  <tr className="border-b text-left text-gray-500">
                    <th className="pb-2 font-medium">Élément</th>
                    <th className="pb-2 font-medium">Qté</th>
                    <th className="pb-2 font-medium">État</th>
                  </tr>
                </thead>
                <tbody>
                  {property.furnitureItems.map((item) => (
                    <tr key={item.id} className="border-b last:border-0">
                      <td className="py-2">{item.name}</td>
                      <td className="py-2">{item.quantity}</td>
                      <td className="py-2">{conditionLabels[item.condition] || item.condition}</td>
                    </tr>
                  ))}
                </tbody>
              </table>
            </div>
          )}
        </Card>
      )}

      {/* Add furniture modal */}
      <Modal open={showFurnitureModal} onClose={() => setShowFurnitureModal(false)} title="Ajouter un élément">
        <form onSubmit={handleAddFurniture} className="space-y-4">
          <Input
            label="Nom de l'élément"
            placeholder="Table, Lit, Canapé..."
            value={furnitureName}
            onChange={(e) => setFurnitureName(e.target.value)}
            required
          />
          <Input
            label="Quantité"
            type="number"
            min="1"
            value={furnitureQty}
            onChange={(e) => setFurnitureQty(e.target.value)}
          />
          <Select
            label="État"
            value={furnitureCondition}
            onChange={(e) => setFurnitureCondition(e.target.value)}
            options={conditionOptions}
          />
          <div className="flex gap-3 justify-end">
            <Button type="button" variant="secondary" onClick={() => setShowFurnitureModal(false)}>
              Annuler
            </Button>
            <Button type="submit" loading={addingFurniture}>
              Ajouter
            </Button>
          </div>
        </form>
      </Modal>

      {/* Delete confirmation modal */}
      <Modal open={showDeleteConfirm} onClose={() => setShowDeleteConfirm(false)} title="Confirmer la suppression">
        <p className="text-sm text-gray-600 mb-4">
          Êtes-vous sûr de vouloir supprimer <strong>{property.name}</strong> ? Cette action est irréversible.
        </p>
        <div className="flex gap-3 justify-end">
          <Button variant="secondary" onClick={() => setShowDeleteConfirm(false)}>
            Annuler
          </Button>
          <Button variant="danger" loading={deleting} onClick={handleDelete}>
            Supprimer
          </Button>
        </div>
      </Modal>
    </div>
  );
}

```

### `src/app/(dashboard)/properties/[id]/calendar/page.tsx`

```typescript
"use client";

import { useState, useEffect, useCallback } from "react";
import { useParams } from "next/navigation";
import dynamic from "next/dynamic";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Card, CardHeader, CardTitle } from "@/components/ui/card";
import { Modal } from "@/components/ui/modal";

// Dynamic import FullCalendar to avoid SSR issues
const FullCalendar = dynamic(
  () => import("@fullcalendar/react").then((mod) => mod.default),
  { ssr: false }
);

import dayGridPlugin from "@fullcalendar/daygrid";
import interactionPlugin from "@fullcalendar/interaction";

interface CalEvent {
  id: string;
  title: string;
  start: string;
  end: string;
  type: string;
  status?: string;
  color: string;
  meta?: Record<string, unknown>;
}

interface ExternalCal {
  id: string;
  name: string;
  url: string;
  lastSyncAt: string | null;
  lastError: string | null;
}

function fmt(d: string) {
  return new Date(d).toLocaleDateString("fr-FR", { day: "2-digit", month: "short", year: "numeric", hour: "2-digit", minute: "2-digit" });
}

export default function CalendarPage() {
  const { id: propertyId } = useParams<{ id: string }>();

  const [events, setEvents] = useState<CalEvent[]>([]);
  const [externals, setExternals] = useState<ExternalCal[]>([]);
  const [loading, setLoading] = useState(true);

  // Block dates modal
  const [showBlockModal, setShowBlockModal] = useState(false);
  const [blockStart, setBlockStart] = useState("");
  const [blockEnd, setBlockEnd] = useState("");
  const [blockReason, setBlockReason] = useState("");
  const [blocking, setBlocking] = useState(false);

  // Add external calendar modal
  const [showAddExternal, setShowAddExternal] = useState(false);
  const [extName, setExtName] = useState("");
  const [extUrl, setExtUrl] = useState("");
  const [addingExt, setAddingExt] = useState(false);

  // Event details popover
  const [selectedEvent, setSelectedEvent] = useState<CalEvent | null>(null);

  const [error, setError] = useState("");

  // iCal export URL
  const baseUrl = typeof window !== "undefined" ? window.location.origin : "";
  const icalUrl = `${baseUrl}/api/calendar/ical?propertyId=${propertyId}`;

  const fetchData = useCallback(async () => {
    try {
      const [eventsRes, extRes] = await Promise.all([
        fetch(`/api/calendar/events?propertyId=${propertyId}`),
        fetch(`/api/calendar/external?propertyId=${propertyId}`),
      ]);
      setEvents(await eventsRes.json());
      setExternals(await extRes.json());
    } catch {
      setError("Impossible de charger le calendrier");
    } finally {
      setLoading(false);
    }
  }, [propertyId]);

  useEffect(() => {
    fetchData();
  }, [fetchData]);

  async function handleBlockDates(e: React.FormEvent) {
    e.preventDefault();
    setBlocking(true);
    setError("");

    try {
      const res = await fetch("/api/calendar/events", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ propertyId, startDate: blockStart, endDate: blockEnd, reason: blockReason }),
      });
      if (!res.ok) {
        const data = await res.json();
        setError(data.error);
        return;
      }
      setShowBlockModal(false);
      setBlockStart("");
      setBlockEnd("");
      setBlockReason("");
      fetchData();
    } catch {
      setError("Erreur");
    } finally {
      setBlocking(false);
    }
  }

  async function handleAddExternal(e: React.FormEvent) {
    e.preventDefault();
    setAddingExt(true);
    setError("");

    try {
      const res = await fetch("/api/calendar/external", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ propertyId, name: extName, url: extUrl }),
      });
      if (!res.ok) {
        const data = await res.json();
        setError(data.error);
        return;
      }
      setShowAddExternal(false);
      setExtName("");
      setExtUrl("");
      fetchData();
    } catch {
      setError("Erreur");
    } finally {
      setAddingExt(false);
    }
  }

  async function handleSync(calendarId: string) {
    await fetch(`/api/calendar/sync?calendarId=${calendarId}`, { method: "POST" });
    fetchData();
  }

  async function handleDeleteExternal(calendarId: string) {
    await fetch(`/api/calendar/external?id=${calendarId}`, { method: "DELETE" });
    fetchData();
  }

  function handleDateSelect(info: { startStr: string; endStr: string }) {
    setBlockStart(info.startStr);
    setBlockEnd(info.endStr);
    setShowBlockModal(true);
  }

  function handleEventClick(info: { event: { id: string } }) {
    const ev = events.find((e) => e.id === info.event.id);
    if (ev) setSelectedEvent(ev);
  }

  if (loading) {
    return <div className="flex items-center justify-center py-16"><p className="text-sm text-gray-500">Chargement...</p></div>;
  }

  return (
    <div>
      <div className="flex items-center justify-between mb-6">
        <h1 className="text-2xl font-bold text-gray-900">Calendrier</h1>
        <div className="flex gap-2">
          <Button size="sm" onClick={() => setShowBlockModal(true)}>Bloquer des dates</Button>
          <Button size="sm" variant="secondary" onClick={() => setShowAddExternal(true)}>
            Ajouter un calendrier
          </Button>
        </div>
      </div>

      {error && <div className="mb-4 rounded-lg bg-red-50 border border-red-200 p-3 text-sm text-red-700">{error}</div>}

      {/* Legend */}
      <div className="flex flex-wrap gap-4 mb-4 text-xs">
        <span className="flex items-center gap-1"><span className="w-3 h-3 rounded bg-green-500" /> Confirmé</span>
        <span className="flex items-center gap-1"><span className="w-3 h-3 rounded bg-amber-500" /> En attente</span>
        <span className="flex items-center gap-1"><span className="w-3 h-3 rounded bg-gray-400" /> Brouillon</span>
        <span className="flex items-center gap-1"><span className="w-3 h-3 rounded bg-red-500" /> Bloqué</span>
        <span className="flex items-center gap-1"><span className="w-3 h-3 rounded bg-purple-500" /> Externe</span>
      </div>

      {/* Calendar */}
      <Card className="mb-6">
        <FullCalendar
          plugins={[dayGridPlugin, interactionPlugin]}
          initialView="dayGridMonth"
          locale="fr"
          selectable={true}
          select={handleDateSelect}
          eventClick={handleEventClick}
          events={events.map((e) => ({
            id: e.id,
            title: e.title,
            start: e.start,
            end: e.end,
            backgroundColor: e.color,
            borderColor: e.color,
          }))}
          headerToolbar={{
            left: "prev,next today",
            center: "title",
            right: "dayGridMonth,dayGridWeek",
          }}
          height="auto"
          buttonText={{ today: "Aujourd'hui", month: "Mois", week: "Semaine" }}
        />
      </Card>

      {/* External calendars */}
      {externals.length > 0 && (
        <Card className="mb-6">
          <CardHeader><CardTitle>Calendriers externes</CardTitle></CardHeader>
          <div className="space-y-3">
            {externals.map((ext) => (
              <div key={ext.id} className="flex items-center justify-between text-sm">
                <div>
                  <p className="font-medium">{ext.name}</p>
                  <p className="text-xs text-gray-500">
                    {ext.lastSyncAt ? `Dernière synchro : ${fmt(ext.lastSyncAt)}` : "Jamais synchronisé"}
                    {ext.lastError && <span className="text-red-500 ml-2">Erreur: {ext.lastError}</span>}
                  </p>
                </div>
                <div className="flex gap-2">
                  <Button size="sm" variant="ghost" onClick={() => handleSync(ext.id)}>Synchroniser</Button>
                  <Button size="sm" variant="ghost" onClick={() => handleDeleteExternal(ext.id)}>Supprimer</Button>
                </div>
              </div>
            ))}
          </div>
        </Card>
      )}

      {/* iCal export */}
      <Card>
        <CardHeader><CardTitle>Export iCal</CardTitle></CardHeader>
        <p className="text-sm text-gray-600 mb-2">
          Utilisez ce lien pour synchroniser votre calendrier avec Airbnb, Booking ou autre :
        </p>
        <code className="block text-xs bg-gray-100 p-2 rounded break-all">{icalUrl}</code>
      </Card>

      {/* Block dates modal */}
      <Modal open={showBlockModal} onClose={() => setShowBlockModal(false)} title="Bloquer des dates">
        <form onSubmit={handleBlockDates} className="space-y-4">
          <div className="grid grid-cols-2 gap-3">
            <Input label="Début" type="date" value={blockStart} onChange={(e) => setBlockStart(e.target.value)} required />
            <Input label="Fin" type="date" value={blockEnd} onChange={(e) => setBlockEnd(e.target.value)} required />
          </div>
          <Input label="Raison (optionnel)" value={blockReason} onChange={(e) => setBlockReason(e.target.value)} placeholder="Travaux, usage personnel..." />
          <div className="flex gap-3 justify-end">
            <Button type="button" variant="secondary" onClick={() => setShowBlockModal(false)}>Annuler</Button>
            <Button type="submit" loading={blocking}>Bloquer</Button>
          </div>
        </form>
      </Modal>

      {/* Add external modal */}
      <Modal open={showAddExternal} onClose={() => setShowAddExternal(false)} title="Ajouter un calendrier iCal">
        <form onSubmit={handleAddExternal} className="space-y-4">
          <Input label="Nom" value={extName} onChange={(e) => setExtName(e.target.value)} placeholder="Airbnb, Booking..." required />
          <Input label="URL iCal" value={extUrl} onChange={(e) => setExtUrl(e.target.value)} placeholder="https://..." required />
          <div className="flex gap-3 justify-end">
            <Button type="button" variant="secondary" onClick={() => setShowAddExternal(false)}>Annuler</Button>
            <Button type="submit" loading={addingExt}>Ajouter</Button>
          </div>
        </form>
      </Modal>

      {/* Event detail popover */}
      <Modal open={!!selectedEvent} onClose={() => setSelectedEvent(null)} title={selectedEvent?.title || "Détails"}>
        {selectedEvent && (
          <div className="space-y-2 text-sm">
            <div className="flex justify-between"><span className="text-gray-500">Type</span><span>{selectedEvent.type === "contract" ? "Réservation" : selectedEvent.type === "blocked" ? "Bloqué" : "Externe"}</span></div>
            <div className="flex justify-between"><span className="text-gray-500">Début</span><span>{new Date(selectedEvent.start).toLocaleDateString("fr-FR")}</span></div>
            <div className="flex justify-between"><span className="text-gray-500">Fin</span><span>{new Date(selectedEvent.end).toLocaleDateString("fr-FR")}</span></div>
            {selectedEvent.status && <div className="flex justify-between"><span className="text-gray-500">Statut</span><span>{selectedEvent.status}</span></div>}
          </div>
        )}
      </Modal>
    </div>
  );
}

```

### `src/app/(dashboard)/properties/[id]/contracts/new/page.tsx`

```typescript
"use client";

import { useState, useEffect } from "react";
import { useRouter, useParams } from "next/navigation";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Card, CardHeader, CardTitle } from "@/components/ui/card";

interface ConditionTemplate {
  id: string;
  name: string;
  content: string;
}

export default function NewContractPage() {
  const router = useRouter();
  const { id: propertyId } = useParams<{ id: string }>();

  const [checkIn, setCheckIn] = useState("");
  const [checkOut, setCheckOut] = useState("");
  const [guests, setGuests] = useState("2");
  const [rentalPrice, setRentalPrice] = useState("");
  const [cleaningFee, setCleaningFee] = useState("0");
  const [tenantName, setTenantName] = useState("");
  const [tenantEmail, setTenantEmail] = useState("");
  const [tenantPhone, setTenantPhone] = useState("");
  const [conditions, setConditions] = useState("");
  const [templates, setTemplates] = useState<ConditionTemplate[]>([]);
  const [error, setError] = useState("");
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    fetch("/api/condition-templates")
      .then((r) => r.json())
      .then((data) => setTemplates(Array.isArray(data) ? data : []))
      .catch(() => {});
  }, []);

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    setError("");
    setLoading(true);

    try {
      const res = await fetch("/api/contracts", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          propertyId,
          checkIn,
          checkOut,
          guests: parseInt(guests),
          rentalPrice: parseFloat(rentalPrice),
          cleaningFee: parseFloat(cleaningFee) || 0,
          tenantName,
          tenantEmail,
          tenantPhone: tenantPhone || null,
          conditions: conditions || null,
        }),
      });

      const data = await res.json();

      if (!res.ok) {
        setError(data.error || "Une erreur est survenue");
        return;
      }

      router.push(`/properties/${propertyId}/contracts/${data.id}`);
    } catch {
      setError("Une erreur est survenue. Veuillez réessayer.");
    } finally {
      setLoading(false);
    }
  }

  function applyTemplate(template: ConditionTemplate) {
    setConditions((prev) => (prev ? `${prev}\n\n${template.content}` : template.content));
  }

  return (
    <div className="max-w-2xl mx-auto">
      <h1 className="text-2xl font-bold text-gray-900 mb-6">Nouveau contrat saisonnier</h1>

      {error && (
        <div className="mb-4 rounded-lg bg-red-50 border border-red-200 p-3 text-sm text-red-700">
          {error}
        </div>
      )}

      <Card>
        <CardHeader>
          <CardTitle>Informations du séjour</CardTitle>
        </CardHeader>

        <form onSubmit={handleSubmit} className="space-y-4">
          <div className="grid grid-cols-2 gap-3">
            <Input
              label="Date d'arrivée"
              type="date"
              value={checkIn}
              onChange={(e) => setCheckIn(e.target.value)}
              required
            />
            <Input
              label="Date de départ"
              type="date"
              value={checkOut}
              onChange={(e) => setCheckOut(e.target.value)}
              required
            />
          </div>

          <Input
            label="Nombre d'occupants"
            type="number"
            min="1"
            value={guests}
            onChange={(e) => setGuests(e.target.value)}
            required
          />

          <div className="grid grid-cols-2 gap-3">
            <Input
              label="Prix de la location (€)"
              type="number"
              step="0.01"
              min="0"
              value={rentalPrice}
              onChange={(e) => setRentalPrice(e.target.value)}
              required
            />
            <Input
              label="Frais de ménage (€)"
              type="number"
              step="0.01"
              min="0"
              value={cleaningFee}
              onChange={(e) => setCleaningFee(e.target.value)}
            />
          </div>

          <CardHeader>
            <CardTitle>Locataire</CardTitle>
          </CardHeader>

          <Input
            label="Nom complet"
            type="text"
            value={tenantName}
            onChange={(e) => setTenantName(e.target.value)}
            required
          />

          <div className="grid grid-cols-2 gap-3">
            <Input
              label="Email"
              type="email"
              value={tenantEmail}
              onChange={(e) => setTenantEmail(e.target.value)}
              required
            />
            <Input
              label="Téléphone"
              type="tel"
              value={tenantPhone}
              onChange={(e) => setTenantPhone(e.target.value)}
            />
          </div>

          <CardHeader>
            <CardTitle>Conditions particulières</CardTitle>
          </CardHeader>

          {templates.length > 0 && (
            <div className="flex flex-wrap gap-2 mb-2">
              {templates.map((t) => (
                <button
                  key={t.id}
                  type="button"
                  onClick={() => applyTemplate(t)}
                  className="text-xs px-2 py-1 rounded-full bg-blue-50 text-blue-700 hover:bg-blue-100"
                >
                  + {t.name}
                </button>
              ))}
            </div>
          )}

          <textarea
            value={conditions}
            onChange={(e) => setConditions(e.target.value)}
            rows={6}
            className="block w-full rounded-lg border border-gray-300 px-3 py-2 text-sm shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
            placeholder="Conditions particulières de la location (optionnel)..."
          />

          <div className="flex gap-3 pt-2">
            <Button type="button" variant="secondary" className="flex-1" onClick={() => router.back()}>
              Annuler
            </Button>
            <Button type="submit" loading={loading} className="flex-1">
              Créer le contrat
            </Button>
          </div>
        </form>
      </Card>
    </div>
  );
}

```

### `src/app/(dashboard)/properties/[id]/contracts/[contractId]/page.tsx`

```typescript
"use client";

import { useState, useEffect, useCallback } from "react";
import { useRouter, useParams } from "next/navigation";
import { Button } from "@/components/ui/button";
import { Card, CardHeader, CardTitle } from "@/components/ui/card";

interface Signature {
  id: string;
  role: string;
  signedAt: string | null;
  token: string;
}

interface Contract {
  id: string;
  status: string;
  checkIn: string;
  checkOut: string;
  guests: number;
  rentalPrice: number;
  cleaningFee: number;
  tenantName: string;
  tenantEmail: string;
  tenantPhone: string | null;
  conditions: string | null;
  signatures: Signature[];
  payments: { id: string; status: string; amount: number }[];
  documents: { id: string; type: string; name: string; url: string }[];
  property: { name: string };
}

const statusLabels: Record<string, string> = {
  DRAFT: "Brouillon",
  PENDING_SIGNATURE: "En attente de signature",
  SIGNED: "Signé",
  CONFIRMED: "Confirmé",
  IN_PROGRESS: "En cours",
  COMPLETED: "Terminé",
  CANCELLED: "Annulé",
  ARCHIVED: "Archivé",
};

const statusColors: Record<string, string> = {
  DRAFT: "bg-gray-100 text-gray-700",
  PENDING_SIGNATURE: "bg-amber-100 text-amber-700",
  SIGNED: "bg-blue-100 text-blue-700",
  CONFIRMED: "bg-green-100 text-green-700",
  IN_PROGRESS: "bg-green-100 text-green-800",
  COMPLETED: "bg-gray-200 text-gray-700",
  CANCELLED: "bg-red-100 text-red-700",
  ARCHIVED: "bg-gray-100 text-gray-500",
};

function formatDate(d: string) {
  return new Date(d).toLocaleDateString("fr-FR", { day: "2-digit", month: "long", year: "numeric" });
}

function formatCurrency(n: number) {
  return new Intl.NumberFormat("fr-FR", { style: "currency", currency: "EUR" }).format(n);
}

export default function ContractDetailPage() {
  const router = useRouter();
  const { id: propertyId, contractId } = useParams<{ id: string; contractId: string }>();

  const [contract, setContract] = useState<Contract | null>(null);
  const [loading, setLoading] = useState(true);
  const [actionLoading, setActionLoading] = useState(false);
  const [error, setError] = useState("");
  const [success, setSuccess] = useState("");

  const fetchContract = useCallback(async () => {
    try {
      const res = await fetch(`/api/contracts/${contractId}`);
      if (!res.ok) {
        setError("Contrat introuvable");
        return;
      }
      setContract(await res.json());
    } catch {
      setError("Impossible de charger le contrat");
    } finally {
      setLoading(false);
    }
  }, [contractId]);

  useEffect(() => {
    fetchContract();
  }, [fetchContract]);

  async function sendForSignature() {
    setActionLoading(true);
    setError("");
    setSuccess("");
    try {
      const res = await fetch(`/api/contracts/${contractId}/send-signature`, { method: "POST" });
      const data = await res.json();
      if (!res.ok) {
        setError(data.error || "Erreur");
        return;
      }
      setSuccess("Le contrat a été envoyé au locataire pour signature.");
      fetchContract();
    } catch {
      setError("Une erreur est survenue");
    } finally {
      setActionLoading(false);
    }
  }

  async function updateStatus(newStatus: string) {
    setActionLoading(true);
    setError("");
    try {
      const res = await fetch(`/api/contracts/${contractId}`, {
        method: "PATCH",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ status: newStatus }),
      });
      const data = await res.json();
      if (!res.ok) {
        setError(data.error || "Erreur");
        return;
      }
      setContract(data);
    } catch {
      setError("Une erreur est survenue");
    } finally {
      setActionLoading(false);
    }
  }

  if (loading) {
    return <div className="flex items-center justify-center py-16"><p className="text-sm text-gray-500">Chargement...</p></div>;
  }

  if (!contract) {
    return (
      <div className="text-center py-16">
        <p className="text-gray-500">{error || "Contrat introuvable"}</p>
        <Button variant="secondary" className="mt-4" onClick={() => router.push(`/properties/${propertyId}`)}>Retour</Button>
      </div>
    );
  }

  return (
    <div className="max-w-3xl mx-auto">
      <div className="flex items-center justify-between mb-6">
        <div>
          <h1 className="text-2xl font-bold text-gray-900">Contrat saisonnier</h1>
          <p className="text-sm text-gray-500">{contract.tenantName} — {contract.property.name}</p>
        </div>
        <span className={`inline-flex items-center rounded-full px-3 py-1 text-sm font-medium ${statusColors[contract.status]}`}>
          {statusLabels[contract.status]}
        </span>
      </div>

      {error && (
        <div className="mb-4 rounded-lg bg-red-50 border border-red-200 p-3 text-sm text-red-700">{error}</div>
      )}
      {success && (
        <div className="mb-4 rounded-lg bg-green-50 border border-green-200 p-3 text-sm text-green-700">{success}</div>
      )}

      {/* Details */}
      <Card className="mb-6">
        <CardHeader><CardTitle>Détails du séjour</CardTitle></CardHeader>
        <div className="space-y-2 text-sm">
          <div className="flex justify-between"><span className="text-gray-500">Arrivée</span><span>{formatDate(contract.checkIn)}</span></div>
          <div className="flex justify-between"><span className="text-gray-500">Départ</span><span>{formatDate(contract.checkOut)}</span></div>
          <div className="flex justify-between"><span className="text-gray-500">Occupants</span><span>{contract.guests}</span></div>
          <div className="flex justify-between"><span className="text-gray-500">Location</span><span>{formatCurrency(contract.rentalPrice)}</span></div>
          {contract.cleaningFee > 0 && (
            <div className="flex justify-between"><span className="text-gray-500">Ménage</span><span>{formatCurrency(contract.cleaningFee)}</span></div>
          )}
          <div className="flex justify-between font-medium"><span>Total</span><span>{formatCurrency(contract.rentalPrice + contract.cleaningFee)}</span></div>
        </div>
      </Card>

      {/* Tenant */}
      <Card className="mb-6">
        <CardHeader><CardTitle>Locataire</CardTitle></CardHeader>
        <div className="space-y-2 text-sm">
          <div className="flex justify-between"><span className="text-gray-500">Nom</span><span>{contract.tenantName}</span></div>
          <div className="flex justify-between"><span className="text-gray-500">Email</span><span>{contract.tenantEmail}</span></div>
          {contract.tenantPhone && <div className="flex justify-between"><span className="text-gray-500">Téléphone</span><span>{contract.tenantPhone}</span></div>}
        </div>
      </Card>

      {/* Signatures */}
      {contract.signatures.length > 0 && (
        <Card className="mb-6">
          <CardHeader><CardTitle>Signatures</CardTitle></CardHeader>
          <div className="space-y-2 text-sm">
            {contract.signatures.map((sig) => (
              <div key={sig.id} className="flex justify-between">
                <span className="text-gray-500">{sig.role === "TENANT" ? "Locataire" : "Propriétaire"}</span>
                <span className={sig.signedAt ? "text-green-600" : "text-amber-600"}>
                  {sig.signedAt ? `Signé le ${formatDate(sig.signedAt)}` : "En attente"}
                </span>
              </div>
            ))}
          </div>
        </Card>
      )}

      {/* Actions */}
      <Card>
        <CardHeader><CardTitle>Actions</CardTitle></CardHeader>
        <div className="flex flex-wrap gap-3">
          {contract.status === "DRAFT" && (
            <Button loading={actionLoading} onClick={sendForSignature}>
              Envoyer pour signature
            </Button>
          )}
          {contract.status === "SIGNED" && (
            <Button loading={actionLoading} onClick={() => updateStatus("CONFIRMED")}>
              Confirmer la réservation
            </Button>
          )}
          {["DRAFT", "PENDING_SIGNATURE", "SIGNED", "CONFIRMED"].includes(contract.status) && (
            <Button variant="danger" loading={actionLoading} onClick={() => updateStatus("CANCELLED")}>
              Annuler
            </Button>
          )}
          {contract.status === "COMPLETED" && (
            <Button variant="secondary" loading={actionLoading} onClick={() => updateStatus("ARCHIVED")}>
              Archiver
            </Button>
          )}
          <Button variant="secondary" onClick={() => router.push(`/properties/${propertyId}`)}>
            Retour au bien
          </Button>
        </div>
      </Card>
    </div>
  );
}

```

### `src/app/(dashboard)/properties/[id]/leases/new/page.tsx`

```typescript
"use client";

import { useState } from "react";
import { useRouter, useParams } from "next/navigation";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Select } from "@/components/ui/select";
import { Card, CardHeader, CardTitle } from "@/components/ui/card";

const chargeTypes = [
  { value: "FIXED", label: "Forfaitaires" },
  { value: "PROVISION", label: "Provisions sur charges" },
];

export default function NewLeasePage() {
  const router = useRouter();
  const { id: propertyId } = useParams<{ id: string }>();

  const [tenantName, setTenantName] = useState("");
  const [tenantEmail, setTenantEmail] = useState("");
  const [tenantPhone, setTenantPhone] = useState("");
  const [tenantBirthDate, setTenantBirthDate] = useState("");
  const [startDate, setStartDate] = useState("");
  const [endDate, setEndDate] = useState("");
  const [rentAmount, setRentAmount] = useState("");
  const [chargesAmount, setChargesAmount] = useState("");
  const [chargeType, setChargeType] = useState("FIXED");
  const [depositAmount, setDepositAmount] = useState("");
  const [paymentDueDay, setPaymentDueDay] = useState("1");
  const [error, setError] = useState("");
  const [loading, setLoading] = useState(false);

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    setError("");
    setLoading(true);

    try {
      const res = await fetch("/api/leases", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          propertyId,
          tenantName,
          tenantEmail,
          tenantPhone: tenantPhone || null,
          tenantBirthDate: tenantBirthDate || null,
          startDate,
          endDate: endDate || null,
          rentAmount: parseFloat(rentAmount),
          chargesAmount: parseFloat(chargesAmount),
          chargeType,
          depositAmount: parseFloat(depositAmount),
          paymentDueDay: parseInt(paymentDueDay),
        }),
      });

      const data = await res.json();

      if (!res.ok) {
        setError(data.error || "Une erreur est survenue");
        return;
      }

      router.push(`/properties/${propertyId}/leases/${data.id}`);
    } catch {
      setError("Une erreur est survenue. Veuillez réessayer.");
    } finally {
      setLoading(false);
    }
  }

  return (
    <div className="max-w-2xl mx-auto">
      <h1 className="text-2xl font-bold text-gray-900 mb-6">Nouveau bail</h1>

      {error && (
        <div className="mb-4 rounded-lg bg-red-50 border border-red-200 p-3 text-sm text-red-700">{error}</div>
      )}

      <Card>
        <form onSubmit={handleSubmit} className="space-y-4">
          <CardHeader><CardTitle>Locataire</CardTitle></CardHeader>

          <Input label="Nom complet" value={tenantName} onChange={(e) => setTenantName(e.target.value)} required />
          <div className="grid grid-cols-2 gap-3">
            <Input label="Email" type="email" value={tenantEmail} onChange={(e) => setTenantEmail(e.target.value)} required />
            <Input label="Téléphone" type="tel" value={tenantPhone} onChange={(e) => setTenantPhone(e.target.value)} />
          </div>
          <Input label="Date de naissance" type="date" value={tenantBirthDate} onChange={(e) => setTenantBirthDate(e.target.value)} />

          <CardHeader><CardTitle>Durée du bail</CardTitle></CardHeader>

          <div className="grid grid-cols-2 gap-3">
            <Input label="Date de début" type="date" value={startDate} onChange={(e) => setStartDate(e.target.value)} required />
            <Input label="Date de fin (optionnel)" type="date" value={endDate} onChange={(e) => setEndDate(e.target.value)} />
          </div>

          <CardHeader><CardTitle>Loyer et charges</CardTitle></CardHeader>

          <div className="grid grid-cols-2 gap-3">
            <Input label="Loyer mensuel (€)" type="number" step="0.01" min="0" value={rentAmount} onChange={(e) => setRentAmount(e.target.value)} required />
            <Input label="Charges (€)" type="number" step="0.01" min="0" value={chargesAmount} onChange={(e) => setChargesAmount(e.target.value)} required />
          </div>

          <div className="grid grid-cols-2 gap-3">
            <Select label="Type de charges" value={chargeType} onChange={(e) => setChargeType(e.target.value)} options={chargeTypes} />
            <Input label="Jour de paiement" type="number" min="1" max="28" value={paymentDueDay} onChange={(e) => setPaymentDueDay(e.target.value)} />
          </div>

          <Input label="Dépôt de garantie (€)" type="number" step="0.01" min="0" value={depositAmount} onChange={(e) => setDepositAmount(e.target.value)} required />

          <div className="flex gap-3 pt-2">
            <Button type="button" variant="secondary" className="flex-1" onClick={() => router.back()}>Annuler</Button>
            <Button type="submit" loading={loading} className="flex-1">Créer le bail</Button>
          </div>
        </form>
      </Card>
    </div>
  );
}

```

### `src/app/(dashboard)/properties/[id]/leases/[leaseId]/page.tsx`

```typescript
"use client";

import { useState, useEffect, useCallback } from "react";
import { useRouter, useParams } from "next/navigation";
import Link from "next/link";
import { Button } from "@/components/ui/button";
import { Card, CardHeader, CardTitle } from "@/components/ui/card";

interface Lease {
  id: string;
  status: string;
  tenantName: string;
  tenantEmail: string;
  tenantPhone: string | null;
  startDate: string;
  endDate: string | null;
  rentAmount: number;
  chargesAmount: number;
  chargeType: string;
  depositAmount: number;
  paymentDueDay: number;
  terminationDate: string | null;
  property: { name: string; rentalType: string };
  inspections: { id: string; type: string; date: string; validated: boolean }[];
  signatures: { id: string; role: string; signedAt: string | null }[];
  documents: { id: string; type: string; name: string }[];
}

const statusLabels: Record<string, string> = {
  DRAFT: "Brouillon",
  PENDING_SIGNATURE: "En attente de signature",
  ACTIVE: "Actif",
  TERMINATION_NOTICE: "Préavis en cours",
  ENDED: "Terminé",
  ARCHIVED: "Archivé",
};

const statusColors: Record<string, string> = {
  DRAFT: "bg-gray-100 text-gray-700",
  PENDING_SIGNATURE: "bg-amber-100 text-amber-700",
  ACTIVE: "bg-green-100 text-green-700",
  TERMINATION_NOTICE: "bg-orange-100 text-orange-700",
  ENDED: "bg-gray-200 text-gray-700",
  ARCHIVED: "bg-gray-100 text-gray-500",
};

function fmt(d: string) {
  return new Date(d).toLocaleDateString("fr-FR", { day: "2-digit", month: "long", year: "numeric" });
}
function cur(n: number) {
  return new Intl.NumberFormat("fr-FR", { style: "currency", currency: "EUR" }).format(n);
}

export default function LeaseDetailPage() {
  const router = useRouter();
  const { id: propertyId, leaseId } = useParams<{ id: string; leaseId: string }>();

  const [lease, setLease] = useState<Lease | null>(null);
  const [loading, setLoading] = useState(true);
  const [actionLoading, setActionLoading] = useState(false);
  const [error, setError] = useState("");
  const [success, setSuccess] = useState("");

  const fetchLease = useCallback(async () => {
    try {
      const res = await fetch(`/api/leases/${leaseId}`);
      if (!res.ok) { setError("Bail introuvable"); return; }
      setLease(await res.json());
    } catch { setError("Impossible de charger le bail"); }
    finally { setLoading(false); }
  }, [leaseId]);

  useEffect(() => { fetchLease(); }, [fetchLease]);

  async function sendForSignature() {
    setActionLoading(true); setError(""); setSuccess("");
    try {
      const res = await fetch(`/api/leases/${leaseId}/send-signature`, { method: "POST" });
      const data = await res.json();
      if (!res.ok) { setError(data.error); return; }
      setSuccess("Le bail a été envoyé au locataire pour signature.");
      fetchLease();
    } catch { setError("Erreur"); }
    finally { setActionLoading(false); }
  }

  async function updateStatus(newStatus: string) {
    setActionLoading(true); setError("");
    try {
      const res = await fetch(`/api/leases/${leaseId}`, {
        method: "PATCH",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ status: newStatus }),
      });
      const data = await res.json();
      if (!res.ok) { setError(data.error); return; }
      setLease(data);
    } catch { setError("Erreur"); }
    finally { setActionLoading(false); }
  }

  if (loading) return <div className="flex items-center justify-center py-16"><p className="text-sm text-gray-500">Chargement...</p></div>;
  if (!lease) return <div className="text-center py-16"><p className="text-gray-500">{error}</p></div>;

  return (
    <div className="max-w-3xl mx-auto">
      <div className="flex items-center justify-between mb-6">
        <div>
          <h1 className="text-2xl font-bold text-gray-900">Bail — {lease.tenantName}</h1>
          <p className="text-sm text-gray-500">{lease.property.name}</p>
        </div>
        <span className={`inline-flex items-center rounded-full px-3 py-1 text-sm font-medium ${statusColors[lease.status]}`}>
          {statusLabels[lease.status]}
        </span>
      </div>

      {error && <div className="mb-4 rounded-lg bg-red-50 border border-red-200 p-3 text-sm text-red-700">{error}</div>}
      {success && <div className="mb-4 rounded-lg bg-green-50 border border-green-200 p-3 text-sm text-green-700">{success}</div>}

      <Card className="mb-6">
        <CardHeader><CardTitle>Détails</CardTitle></CardHeader>
        <div className="space-y-2 text-sm">
          <div className="flex justify-between"><span className="text-gray-500">Début</span><span>{fmt(lease.startDate)}</span></div>
          {lease.endDate && <div className="flex justify-between"><span className="text-gray-500">Fin</span><span>{fmt(lease.endDate)}</span></div>}
          <div className="flex justify-between"><span className="text-gray-500">Loyer</span><span>{cur(lease.rentAmount)}</span></div>
          <div className="flex justify-between"><span className="text-gray-500">Charges</span><span>{cur(lease.chargesAmount)} ({lease.chargeType === "FIXED" ? "forfaitaires" : "provisions"})</span></div>
          <div className="flex justify-between"><span className="text-gray-500">Total mensuel</span><span className="font-medium">{cur(lease.rentAmount + lease.chargesAmount)}</span></div>
          <div className="flex justify-between"><span className="text-gray-500">Dépôt de garantie</span><span>{cur(lease.depositAmount)}</span></div>
          <div className="flex justify-between"><span className="text-gray-500">Jour de paiement</span><span>Le {lease.paymentDueDay} du mois</span></div>
          {lease.terminationDate && <div className="flex justify-between"><span className="text-gray-500">Date de fin de préavis</span><span>{fmt(lease.terminationDate)}</span></div>}
        </div>
      </Card>

      {/* Inspections */}
      <Card className="mb-6">
        <CardHeader>
          <div className="flex items-center justify-between">
            <CardTitle>États des lieux</CardTitle>
            {lease.status === "ACTIVE" && (
              <Link href={`/properties/${propertyId}/leases/${leaseId}/inspection`}>
                <Button size="sm">Créer un état des lieux</Button>
              </Link>
            )}
          </div>
        </CardHeader>
        {lease.inspections.length === 0 ? (
          <p className="text-sm text-gray-500">Aucun état des lieux.</p>
        ) : (
          <div className="space-y-2 text-sm">
            {lease.inspections.map((i) => (
              <div key={i.id} className="flex justify-between">
                <span>{i.type === "CHECK_IN" ? "Entrée" : "Sortie"} — {fmt(i.date)}</span>
                <span className={i.validated ? "text-green-600" : "text-amber-600"}>
                  {i.validated ? "Validé" : "En attente"}
                </span>
              </div>
            ))}
          </div>
        )}
      </Card>

      {/* Actions */}
      <Card>
        <CardHeader><CardTitle>Actions</CardTitle></CardHeader>
        <div className="flex flex-wrap gap-3">
          {lease.status === "DRAFT" && (
            <Button loading={actionLoading} onClick={sendForSignature}>Envoyer pour signature</Button>
          )}
          {lease.status === "ACTIVE" && (
            <Button variant="secondary" loading={actionLoading} onClick={() => updateStatus("TERMINATION_NOTICE")}>
              Donner congé
            </Button>
          )}
          {lease.status === "TERMINATION_NOTICE" && (
            <Button loading={actionLoading} onClick={() => updateStatus("ENDED")}>Clôturer le bail</Button>
          )}
          {lease.status === "ENDED" && (
            <Button variant="secondary" loading={actionLoading} onClick={() => updateStatus("ARCHIVED")}>Archiver</Button>
          )}
          <Button variant="secondary" onClick={() => router.push(`/properties/${propertyId}`)}>Retour au bien</Button>
        </div>
      </Card>
    </div>
  );
}

```

### `src/app/(dashboard)/properties/[id]/leases/[leaseId]/inspection/page.tsx`

```typescript
"use client";

import { useState } from "react";
import { useRouter, useParams } from "next/navigation";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Select } from "@/components/ui/select";
import { Card, CardHeader, CardTitle } from "@/components/ui/card";

interface Room {
  name: string;
  walls: string;
  floor: string;
  ceiling: string;
  fixtures: string;
  notes: string;
}

const conditionOptions = [
  { value: "NEW", label: "Neuf" },
  { value: "GOOD", label: "Bon état" },
  { value: "FAIR", label: "État correct" },
  { value: "POOR", label: "Mauvais état" },
];

const typeOptions = [
  { value: "CHECK_IN", label: "État des lieux d'entrée" },
  { value: "CHECK_OUT", label: "État des lieux de sortie" },
];

const defaultRooms = ["Entrée", "Séjour", "Cuisine", "Chambre 1", "Salle de bain", "WC"];

export default function InspectionPage() {
  const router = useRouter();
  const { id: propertyId, leaseId } = useParams<{ id: string; leaseId: string }>();

  const [type, setType] = useState("CHECK_IN");
  const [rooms, setRooms] = useState<Room[]>(
    defaultRooms.map((name) => ({ name, walls: "GOOD", floor: "GOOD", ceiling: "GOOD", fixtures: "GOOD", notes: "" }))
  );
  const [meterWater, setMeterWater] = useState("");
  const [meterElectricity, setMeterElectricity] = useState("");
  const [meterGas, setMeterGas] = useState("");
  const [newRoomName, setNewRoomName] = useState("");
  const [error, setError] = useState("");
  const [loading, setLoading] = useState(false);

  function updateRoom(index: number, field: keyof Room, value: string) {
    setRooms((prev) => prev.map((r, i) => (i === index ? { ...r, [field]: value } : r)));
  }

  function addRoom() {
    if (!newRoomName.trim()) return;
    setRooms((prev) => [...prev, { name: newRoomName.trim(), walls: "GOOD", floor: "GOOD", ceiling: "GOOD", fixtures: "GOOD", notes: "" }]);
    setNewRoomName("");
  }

  function removeRoom(index: number) {
    setRooms((prev) => prev.filter((_, i) => i !== index));
  }

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    setError("");
    setLoading(true);

    try {
      const res = await fetch(`/api/leases/${leaseId}/inspections`, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          type,
          rooms,
          meterWater: meterWater || null,
          meterElectricity: meterElectricity || null,
          meterGas: meterGas || null,
        }),
      });

      const data = await res.json();
      if (!res.ok) { setError(data.error); return; }

      router.push(`/properties/${propertyId}/leases/${leaseId}`);
    } catch {
      setError("Une erreur est survenue");
    } finally {
      setLoading(false);
    }
  }

  return (
    <div className="max-w-3xl mx-auto">
      <h1 className="text-2xl font-bold text-gray-900 mb-6">Nouvel état des lieux</h1>

      {error && <div className="mb-4 rounded-lg bg-red-50 border border-red-200 p-3 text-sm text-red-700">{error}</div>}

      <form onSubmit={handleSubmit}>
        <Card className="mb-6">
          <CardHeader><CardTitle>Type et relevés</CardTitle></CardHeader>
          <div className="space-y-4">
            <Select label="Type" value={type} onChange={(e) => setType(e.target.value)} options={typeOptions} />
            <div className="grid grid-cols-3 gap-3">
              <Input label="Compteur eau" value={meterWater} onChange={(e) => setMeterWater(e.target.value)} placeholder="m³" />
              <Input label="Compteur électricité" value={meterElectricity} onChange={(e) => setMeterElectricity(e.target.value)} placeholder="kWh" />
              <Input label="Compteur gaz" value={meterGas} onChange={(e) => setMeterGas(e.target.value)} placeholder="m³" />
            </div>
          </div>
        </Card>

        {rooms.map((room, i) => (
          <Card key={i} className="mb-4">
            <CardHeader>
              <div className="flex items-center justify-between">
                <CardTitle>{room.name}</CardTitle>
                <button type="button" onClick={() => removeRoom(i)} className="text-sm text-red-500 hover:text-red-700">Supprimer</button>
              </div>
            </CardHeader>
            <div className="grid grid-cols-2 gap-3">
              <Select label="Murs" value={room.walls} onChange={(e) => updateRoom(i, "walls", e.target.value)} options={conditionOptions} />
              <Select label="Sol" value={room.floor} onChange={(e) => updateRoom(i, "floor", e.target.value)} options={conditionOptions} />
              <Select label="Plafond" value={room.ceiling} onChange={(e) => updateRoom(i, "ceiling", e.target.value)} options={conditionOptions} />
              <Select label="Équipements" value={room.fixtures} onChange={(e) => updateRoom(i, "fixtures", e.target.value)} options={conditionOptions} />
            </div>
            <div className="mt-3">
              <Input label="Notes" value={room.notes} onChange={(e) => updateRoom(i, "notes", e.target.value)} placeholder="Observations..." />
            </div>
          </Card>
        ))}

        <Card className="mb-6">
          <div className="flex gap-3">
            <Input placeholder="Nom de la pièce" value={newRoomName} onChange={(e) => setNewRoomName(e.target.value)} />
            <Button type="button" variant="secondary" onClick={addRoom}>Ajouter une pièce</Button>
          </div>
        </Card>

        <div className="flex gap-3">
          <Button type="button" variant="secondary" className="flex-1" onClick={() => router.back()}>Annuler</Button>
          <Button type="submit" loading={loading} className="flex-1">Créer et envoyer pour signature</Button>
        </div>
      </form>
    </div>
  );
}

```

### `src/app/sign/[token]/page.tsx`

```typescript
"use client";

import { useState, useEffect, useCallback } from "react";
import { useParams } from "next/navigation";
import { SignaturePad } from "@/components/signature/signature-pad";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Card, CardHeader, CardTitle } from "@/components/ui/card";

interface SignatureInfo {
  id: string;
  role: string;
  propertyName: string;
  documentType: string;
  documentUrl: string;
}

export default function SigningPage() {
  const { token } = useParams<{ token: string }>();
  const [info, setInfo] = useState<SignatureInfo | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState("");
  const [signerName, setSignerName] = useState("");
  const [signatureData, setSignatureData] = useState<string | null>(null);
  const [submitting, setSubmitting] = useState(false);
  const [success, setSuccess] = useState(false);

  const fetchInfo = useCallback(async () => {
    try {
      const res = await fetch(`/api/signatures/${token}`);
      const data = await res.json();
      if (!res.ok) {
        setError(data.error || "Lien invalide");
        return;
      }
      setInfo(data);
    } catch {
      setError("Impossible de charger le document");
    } finally {
      setLoading(false);
    }
  }, [token]);

  useEffect(() => {
    fetchInfo();
  }, [fetchInfo]);

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    if (!signatureData) {
      setError("Veuillez dessiner votre signature");
      return;
    }
    if (!signerName.trim()) {
      setError("Veuillez saisir votre nom");
      return;
    }

    setError("");
    setSubmitting(true);

    try {
      const res = await fetch(`/api/signatures/${token}`, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ signatureDataUrl: signatureData, signerName: signerName.trim() }),
      });

      const data = await res.json();
      if (!res.ok) {
        setError(data.error || "Une erreur est survenue");
        return;
      }

      setSuccess(true);
    } catch {
      setError("Une erreur est survenue. Veuillez réessayer.");
    } finally {
      setSubmitting(false);
    }
  }

  if (loading) {
    return (
      <div className="min-h-screen flex items-center justify-center bg-gray-50">
        <p className="text-sm text-gray-500">Chargement du document...</p>
      </div>
    );
  }

  if (success) {
    return (
      <div className="min-h-screen flex items-center justify-center bg-gray-50 px-4">
        <Card className="w-full max-w-md">
          <CardHeader>
            <CardTitle className="text-center text-2xl">Document signé</CardTitle>
          </CardHeader>
          <div className="rounded-lg bg-green-50 border border-green-200 p-4 text-sm text-green-700 text-center">
            Votre signature a été enregistrée avec succès. Vous pouvez fermer cette page.
          </div>
        </Card>
      </div>
    );
  }

  if (error && !info) {
    return (
      <div className="min-h-screen flex items-center justify-center bg-gray-50 px-4">
        <Card className="w-full max-w-md">
          <CardHeader>
            <CardTitle className="text-center text-2xl">Signature</CardTitle>
          </CardHeader>
          <div className="rounded-lg bg-red-50 border border-red-200 p-4 text-sm text-red-700">
            {error}
          </div>
        </Card>
      </div>
    );
  }

  return (
    <div className="min-h-screen bg-gray-50 px-4 py-8">
      <div className="max-w-2xl mx-auto">
        <Card>
          <CardHeader>
            <CardTitle>Signature de document</CardTitle>
          </CardHeader>

          {info && (
            <div className="mb-6 space-y-2 text-sm">
              <div className="flex justify-between">
                <span className="text-gray-500">Document</span>
                <span className="font-medium">{info.documentType}</span>
              </div>
              <div className="flex justify-between">
                <span className="text-gray-500">Bien</span>
                <span className="font-medium">{info.propertyName}</span>
              </div>
              <div className="flex justify-between">
                <span className="text-gray-500">Rôle</span>
                <span className="font-medium">
                  {info.role === "TENANT" ? "Locataire" : "Propriétaire"}
                </span>
              </div>

              {info.documentUrl && (
                <div className="pt-2">
                  <a
                    href={info.documentUrl}
                    target="_blank"
                    rel="noopener noreferrer"
                    className="text-blue-600 hover:text-blue-500 text-sm font-medium"
                  >
                    Consulter le document (PDF)
                  </a>
                </div>
              )}
            </div>
          )}

          {error && (
            <div className="mb-4 rounded-lg bg-red-50 border border-red-200 p-3 text-sm text-red-700">
              {error}
            </div>
          )}

          <form onSubmit={handleSubmit} className="space-y-6">
            <Input
              label="Votre nom complet"
              value={signerName}
              onChange={(e) => setSignerName(e.target.value)}
              required
              placeholder="Jean Dupont"
            />

            <div>
              <label className="block text-sm font-medium text-gray-700 mb-2">
                Votre signature
              </label>
              <SignaturePad onSave={(data) => setSignatureData(data)} />
              {signatureData && (
                <p className="mt-2 text-sm text-green-600">Signature capturée</p>
              )}
            </div>

            <p className="text-xs text-gray-500">
              En signant ce document, vous certifiez avoir pris connaissance de son contenu
              et acceptez les termes qui y sont décrits. Votre signature, votre adresse IP
              et l&apos;horodatage seront enregistrés comme preuve.
            </p>

            <Button type="submit" loading={submitting} className="w-full">
              Signer le document
            </Button>
          </form>
        </Card>
      </div>
    </div>
  );
}

```

### `src/app/api/auth/[...nextauth]/route.ts`

```typescript
import { handlers } from "@/lib/auth";

export const { GET, POST } = handlers;

```

### `src/app/api/auth/register/route.ts`

```typescript
import { NextRequest, NextResponse } from "next/server";
import { prisma } from "@/lib/prisma";
import {
  hashPassword,
  validatePassword,
  generateToken,
} from "@/lib/auth-helpers";
import {
  sendEmail,
  verificationEmailHtml,
} from "@/lib/email";

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { name, email, password } = body;

    if (!name || !email || !password) {
      return NextResponse.json(
        { error: "Tous les champs sont requis" },
        { status: 400 }
      );
    }

    // Validate email format
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(email)) {
      return NextResponse.json(
        { error: "Adresse email invalide" },
        { status: 400 }
      );
    }

    // Validate password strength
    const { valid, errors: passwordErrors } = validatePassword(password);
    if (!valid) {
      return NextResponse.json(
        { error: passwordErrors[0] },
        { status: 400 }
      );
    }

    // Check if email already exists - return generic error to avoid email enumeration
    const existingUser = await prisma.user.findUnique({
      where: { email: email.toLowerCase() },
    });

    if (existingUser) {
      return NextResponse.json(
        { error: "Impossible de créer le compte. Veuillez réessayer." },
        { status: 400 }
      );
    }

    // Hash password
    const passwordHash = await hashPassword(password);

    // Create user
    const user = await prisma.user.create({
      data: {
        name,
        email: email.toLowerCase(),
        passwordHash,
      },
    });

    // Create verification token (24h expiry)
    const token = generateToken();
    const expires = new Date(Date.now() + 24 * 60 * 60 * 1000); // 24 hours

    await prisma.verificationToken.create({
      data: {
        identifier: user.email,
        token,
        expires,
      },
    });

    // Send verification email
    const baseUrl = process.env.NEXTAUTH_URL || process.env.NEXT_PUBLIC_APP_URL || "http://localhost:3000";
    const verifyUrl = `${baseUrl}/verify-email?token=${token}`;

    await sendEmail({
      to: user.email,
      subject: "Vérifiez votre adresse email — LS Immo",
      html: verificationEmailHtml(verifyUrl),
    });

    return NextResponse.json({ success: true }, { status: 201 });
  } catch (error) {
    console.error("Registration error:", error);
    return NextResponse.json(
      { error: "Une erreur interne est survenue" },
      { status: 500 }
    );
  }
}

```

### `src/app/api/auth/reset-password/route.ts`

```typescript
import { NextRequest, NextResponse } from "next/server";
import { prisma } from "@/lib/prisma";
import { generateToken } from "@/lib/auth-helpers";
import { sendEmail, resetPasswordEmailHtml } from "@/lib/email";

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { email } = body;

    if (!email) {
      return NextResponse.json(
        { error: "L'adresse email est requise" },
        { status: 400 }
      );
    }

    // Always return success to avoid revealing whether the email exists
    const successResponse = NextResponse.json({ success: true });

    // Look up the user
    const user = await prisma.user.findUnique({
      where: { email: email.toLowerCase() },
    });

    if (!user) {
      // Return success even if user doesn't exist (security: prevent email enumeration)
      return successResponse;
    }

    // Create a password reset token (1h expiry)
    const token = generateToken();
    const expires = new Date(Date.now() + 60 * 60 * 1000); // 1 hour

    await prisma.verificationToken.create({
      data: {
        identifier: user.email,
        token,
        expires,
      },
    });

    // Send reset password email
    const baseUrl =
      process.env.NEXTAUTH_URL ||
      process.env.NEXT_PUBLIC_APP_URL ||
      "http://localhost:3000";
    const resetUrl = `${baseUrl}/reset-password?token=${token}`;

    await sendEmail({
      to: user.email,
      subject: "Réinitialisation de votre mot de passe — LS Immo",
      html: resetPasswordEmailHtml(resetUrl),
    });

    return successResponse;
  } catch (error) {
    console.error("Reset password error:", error);
    return NextResponse.json(
      { error: "Une erreur interne est survenue" },
      { status: 500 }
    );
  }
}

```

### `src/app/api/auth/reset-password/confirm/route.ts`

```typescript
import { NextRequest, NextResponse } from "next/server";
import { prisma } from "@/lib/prisma";
import { hashPassword, validatePassword } from "@/lib/auth-helpers";

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { token, password } = body;

    if (!token || !password) {
      return NextResponse.json(
        { error: "Jeton et mot de passe requis" },
        { status: 400 }
      );
    }

    // Validate password strength
    const { valid, errors: passwordErrors } = validatePassword(password);
    if (!valid) {
      return NextResponse.json(
        { error: passwordErrors[0] },
        { status: 400 }
      );
    }

    // Find the verification token
    const verificationToken = await prisma.verificationToken.findUnique({
      where: { token },
    });

    if (!verificationToken) {
      return NextResponse.json(
        { error: "Le lien de réinitialisation est invalide ou expiré" },
        { status: 400 }
      );
    }

    // Check if token has expired
    if (new Date() > verificationToken.expires) {
      // Clean up expired token
      await prisma.verificationToken.delete({
        where: { token },
      });

      return NextResponse.json(
        { error: "Le lien de réinitialisation a expiré" },
        { status: 400 }
      );
    }

    // Find the user
    const user = await prisma.user.findUnique({
      where: { email: verificationToken.identifier },
    });

    if (!user) {
      return NextResponse.json(
        { error: "Utilisateur introuvable" },
        { status: 400 }
      );
    }

    // Hash the new password
    const passwordHash = await hashPassword(password);

    // Update password, delete all sessions, and delete the token in a transaction
    await prisma.$transaction([
      prisma.user.update({
        where: { id: user.id },
        data: { passwordHash },
      }),
      prisma.session.deleteMany({
        where: { userId: user.id },
      }),
      prisma.verificationToken.delete({
        where: { token },
      }),
    ]);

    return NextResponse.json({
      success: true,
      message: "Mot de passe réinitialisé avec succès",
    });
  } catch (error) {
    console.error("Reset password confirm error:", error);
    return NextResponse.json(
      { error: "Une erreur interne est survenue" },
      { status: 500 }
    );
  }
}

```

### `src/app/api/auth/verify-email/route.ts`

```typescript
import { NextRequest, NextResponse } from "next/server";
import { prisma } from "@/lib/prisma";

export async function GET(request: NextRequest) {
  try {
    const { searchParams } = new URL(request.url);
    const token = searchParams.get("token");

    if (!token) {
      return NextResponse.json(
        { error: "Jeton de vérification manquant" },
        { status: 400 }
      );
    }

    // Find the verification token
    const verificationToken = await prisma.verificationToken.findUnique({
      where: { token },
    });

    if (!verificationToken) {
      return NextResponse.json(
        { error: "Le lien de vérification est invalide ou expiré" },
        { status: 400 }
      );
    }

    // Check if token has expired
    if (new Date() > verificationToken.expires) {
      // Clean up expired token
      await prisma.verificationToken.delete({
        where: { token },
      });

      return NextResponse.json(
        { error: "Le lien de vérification a expiré" },
        { status: 400 }
      );
    }

    // Find the user by email (identifier)
    const user = await prisma.user.findUnique({
      where: { email: verificationToken.identifier },
    });

    if (!user) {
      return NextResponse.json(
        { error: "Utilisateur introuvable" },
        { status: 400 }
      );
    }

    // Update user emailVerified and delete the token in a transaction
    await prisma.$transaction([
      prisma.user.update({
        where: { id: user.id },
        data: { emailVerified: new Date() },
      }),
      prisma.verificationToken.delete({
        where: { token },
      }),
    ]);

    return NextResponse.json({
      success: true,
      message: "Adresse email vérifiée avec succès",
    });
  } catch (error) {
    console.error("Email verification error:", error);
    return NextResponse.json(
      { error: "Une erreur interne est survenue" },
      { status: 500 }
    );
  }
}

```

### `src/app/api/properties/route.ts`

```typescript
import { NextRequest, NextResponse } from "next/server";
import { prisma } from "@/lib/prisma";
import { auth } from "@/lib/auth";

export async function GET() {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ error: "Non autorisé" }, { status: 401 });
  }

  const properties = await prisma.property.findMany({
    where: { userId: session.user.id, deletedAt: null },
    orderBy: { createdAt: "desc" },
    include: {
      contracts: {
        where: { status: { in: ["CONFIRMED", "IN_PROGRESS"] } },
        select: { id: true },
      },
      leases: {
        where: { status: "ACTIVE" },
        select: { id: true },
      },
    },
  });

  return NextResponse.json(properties);
}

export async function POST(request: NextRequest) {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ error: "Non autorisé" }, { status: 401 });
  }

  try {
    const body = await request.json();
    const { name, rentalType, streetNumber, streetName, complement, postalCode, city } = body;

    if (!name || !rentalType || !streetName || !postalCode || !city) {
      return NextResponse.json(
        { error: "Les champs obligatoires sont : nom, type, rue, code postal, ville" },
        { status: 400 }
      );
    }

    // Validate rental type
    if (!["SEASONAL", "CLASSIC_FURNISHED", "CLASSIC_UNFURNISHED"].includes(rentalType)) {
      return NextResponse.json({ error: "Type de location invalide" }, { status: 400 });
    }

    // Validate French postal code (5 digits)
    if (!/^\d{5}$/.test(postalCode)) {
      return NextResponse.json(
        { error: "Le code postal doit contenir exactement 5 chiffres" },
        { status: 400 }
      );
    }

    const property = await prisma.property.create({
      data: {
        userId: session.user.id,
        name: name.trim(),
        rentalType,
        streetNumber: streetNumber?.trim() || null,
        streetName: streetName.trim(),
        complement: complement?.trim() || null,
        postalCode,
        city: city.trim(),
      },
    });

    return NextResponse.json(property, { status: 201 });
  } catch (error) {
    console.error("Property creation error:", error);
    return NextResponse.json({ error: "Une erreur interne est survenue" }, { status: 500 });
  }
}

```

### `src/app/api/properties/[id]/route.ts`

```typescript
import { NextRequest, NextResponse } from "next/server";
import { prisma } from "@/lib/prisma";
import { auth } from "@/lib/auth";

async function getProperty(id: string, userId: string) {
  return prisma.property.findFirst({
    where: { id, userId, deletedAt: null },
    include: {
      furnitureItems: { orderBy: { createdAt: "asc" } },
      contracts: {
        where: { status: { in: ["CONFIRMED", "IN_PROGRESS", "PENDING_SIGNATURE", "SIGNED"] } },
        select: { id: true, status: true },
      },
      leases: {
        where: { status: { in: ["ACTIVE", "PENDING_SIGNATURE"] } },
        select: { id: true, status: true },
      },
    },
  });
}

export async function GET(
  _request: NextRequest,
  { params }: { params: Promise<{ id: string }> }
) {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ error: "Non autorisé" }, { status: 401 });
  }

  const { id } = await params;
  const property = await getProperty(id, session.user.id);

  if (!property) {
    return NextResponse.json({ error: "Bien introuvable" }, { status: 404 });
  }

  return NextResponse.json(property);
}

export async function PATCH(
  request: NextRequest,
  { params }: { params: Promise<{ id: string }> }
) {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ error: "Non autorisé" }, { status: 401 });
  }

  const { id } = await params;
  const property = await getProperty(id, session.user.id);

  if (!property) {
    return NextResponse.json({ error: "Bien introuvable" }, { status: 404 });
  }

  try {
    const body = await request.json();
    const { name, streetNumber, streetName, complement, postalCode, city } = body;

    const data: Record<string, string | null> = {};

    if (name !== undefined) {
      if (!name.trim()) {
        return NextResponse.json({ error: "Le nom est requis" }, { status: 400 });
      }
      data.name = name.trim();
    }

    if (streetName !== undefined) {
      if (!streetName.trim()) {
        return NextResponse.json({ error: "La rue est requise" }, { status: 400 });
      }
      data.streetName = streetName.trim();
    }

    if (postalCode !== undefined) {
      if (!/^\d{5}$/.test(postalCode)) {
        return NextResponse.json(
          { error: "Le code postal doit contenir exactement 5 chiffres" },
          { status: 400 }
        );
      }
      data.postalCode = postalCode;
    }

    if (city !== undefined) {
      if (!city.trim()) {
        return NextResponse.json({ error: "La ville est requise" }, { status: 400 });
      }
      data.city = city.trim();
    }

    if (streetNumber !== undefined) data.streetNumber = streetNumber?.trim() || null;
    if (complement !== undefined) data.complement = complement?.trim() || null;

    // rentalType is intentionally NOT editable after creation

    const updated = await prisma.property.update({
      where: { id },
      data,
    });

    return NextResponse.json(updated);
  } catch (error) {
    console.error("Property update error:", error);
    return NextResponse.json({ error: "Une erreur interne est survenue" }, { status: 500 });
  }
}

export async function DELETE(
  _request: NextRequest,
  { params }: { params: Promise<{ id: string }> }
) {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ error: "Non autorisé" }, { status: 401 });
  }

  const { id } = await params;
  const property = await getProperty(id, session.user.id);

  if (!property) {
    return NextResponse.json({ error: "Bien introuvable" }, { status: 404 });
  }

  // Block deletion if active contracts or leases exist
  const activeContracts = property.contracts.length;
  const activeLeases = property.leases.length;

  if (activeContracts > 0 || activeLeases > 0) {
    return NextResponse.json(
      {
        error:
          "Impossible de supprimer ce bien : des contrats ou baux actifs y sont associés. Terminez-les d'abord.",
      },
      { status: 409 }
    );
  }

  // Soft delete
  await prisma.property.update({
    where: { id },
    data: { deletedAt: new Date() },
  });

  return NextResponse.json({ success: true });
}

```

### `src/app/api/properties/[id]/furniture/route.ts`

```typescript
import { NextRequest, NextResponse } from "next/server";
import { prisma } from "@/lib/prisma";
import { auth } from "@/lib/auth";

export async function GET(
  _request: NextRequest,
  { params }: { params: Promise<{ id: string }> }
) {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ error: "Non autorisé" }, { status: 401 });
  }

  const { id } = await params;
  const property = await prisma.property.findFirst({
    where: { id, userId: session.user.id, deletedAt: null },
  });

  if (!property) {
    return NextResponse.json({ error: "Bien introuvable" }, { status: 404 });
  }

  const items = await prisma.furnitureItem.findMany({
    where: { propertyId: id },
    orderBy: { createdAt: "asc" },
  });

  return NextResponse.json(items);
}

export async function POST(
  request: NextRequest,
  { params }: { params: Promise<{ id: string }> }
) {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ error: "Non autorisé" }, { status: 401 });
  }

  const { id } = await params;
  const property = await prisma.property.findFirst({
    where: { id, userId: session.user.id, deletedAt: null },
  });

  if (!property) {
    return NextResponse.json({ error: "Bien introuvable" }, { status: 404 });
  }

  // Only seasonal or furnished properties have furniture inventory
  if (property.rentalType === "CLASSIC_UNFURNISHED") {
    return NextResponse.json(
      { error: "L'inventaire mobilier n'est pas disponible pour les locations non meublées" },
      { status: 400 }
    );
  }

  try {
    const body = await request.json();
    const { name, quantity, condition, photoUrl } = body;

    if (!name?.trim()) {
      return NextResponse.json({ error: "Le nom de l'élément est requis" }, { status: 400 });
    }

    if (condition && !["NEW", "GOOD", "FAIR", "POOR"].includes(condition)) {
      return NextResponse.json({ error: "État invalide" }, { status: 400 });
    }

    const item = await prisma.furnitureItem.create({
      data: {
        propertyId: id,
        name: name.trim(),
        quantity: quantity && quantity > 0 ? quantity : 1,
        condition: condition || "GOOD",
        photoUrl: photoUrl || null,
      },
    });

    return NextResponse.json(item, { status: 201 });
  } catch (error) {
    console.error("Furniture creation error:", error);
    return NextResponse.json({ error: "Une erreur interne est survenue" }, { status: 500 });
  }
}

```

### `src/app/api/contracts/route.ts`

```typescript
import { NextRequest, NextResponse } from "next/server";
import { prisma } from "@/lib/prisma";
import { auth } from "@/lib/auth";

export async function GET(request: NextRequest) {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ error: "Non autorisé" }, { status: 401 });
  }

  const { searchParams } = new URL(request.url);
  const propertyId = searchParams.get("propertyId");
  const status = searchParams.get("status");

  const where: Record<string, unknown> = {
    property: { userId: session.user.id },
  };
  if (propertyId) where.propertyId = propertyId;
  if (status) where.status = status;

  const contracts = await prisma.contract.findMany({
    where,
    include: {
      property: { select: { name: true } },
      signatures: { select: { id: true, role: true, signedAt: true } },
      payments: { select: { id: true, status: true, amount: true } },
    },
    orderBy: { checkIn: "desc" },
  });

  return NextResponse.json(contracts);
}

export async function POST(request: NextRequest) {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ error: "Non autorisé" }, { status: 401 });
  }

  try {
    const body = await request.json();
    const {
      propertyId,
      checkIn,
      checkOut,
      guests,
      rentalPrice,
      cleaningFee,
      tenantName,
      tenantEmail,
      tenantPhone,
      conditions,
    } = body;

    if (!propertyId || !checkIn || !checkOut || !guests || !rentalPrice || !tenantName || !tenantEmail) {
      return NextResponse.json({ error: "Champs obligatoires manquants" }, { status: 400 });
    }

    // Verify ownership & seasonal type
    const property = await prisma.property.findFirst({
      where: { id: propertyId, userId: session.user.id, deletedAt: null },
    });

    if (!property) {
      return NextResponse.json({ error: "Bien introuvable" }, { status: 404 });
    }

    if (property.rentalType !== "SEASONAL") {
      return NextResponse.json(
        { error: "Les contrats saisonniers sont réservés aux biens de type saisonnière" },
        { status: 400 }
      );
    }

    const start = new Date(checkIn);
    const end = new Date(checkOut);

    if (end <= start) {
      return NextResponse.json({ error: "La date de départ doit être après la date d'arrivée" }, { status: 400 });
    }

    // Check date overlap with confirmed/in-progress contracts and blocked calendar events
    const overlapping = await prisma.contract.findFirst({
      where: {
        propertyId,
        status: { in: ["CONFIRMED", "IN_PROGRESS", "SIGNED", "PENDING_SIGNATURE"] },
        OR: [
          { checkIn: { lt: end }, checkOut: { gt: start } },
        ],
      },
    });

    if (overlapping) {
      return NextResponse.json(
        { error: "Les dates sélectionnées chevauchent une réservation existante" },
        { status: 409 }
      );
    }

    // Check blocked dates
    const blockedOverlap = await prisma.calendarEvent.findFirst({
      where: {
        propertyId,
        type: "BLOCKED",
        startDate: { lt: end },
        endDate: { gt: start },
      },
    });

    if (blockedOverlap) {
      return NextResponse.json(
        { error: "Les dates sélectionnées chevauchent une période bloquée" },
        { status: 409 }
      );
    }

    const contract = await prisma.contract.create({
      data: {
        propertyId,
        checkIn: start,
        checkOut: end,
        guests,
        rentalPrice,
        cleaningFee: cleaningFee || 0,
        tenantName,
        tenantEmail,
        tenantPhone: tenantPhone || null,
        conditions: conditions || null,
        status: "DRAFT",
      },
    });

    return NextResponse.json(contract, { status: 201 });
  } catch (error) {
    console.error("Contract creation error:", error);
    return NextResponse.json({ error: "Une erreur interne est survenue" }, { status: 500 });
  }
}

```

### `src/app/api/contracts/[id]/route.ts`

```typescript
import { NextRequest, NextResponse } from "next/server";
import { prisma } from "@/lib/prisma";
import { auth } from "@/lib/auth";

export async function GET(
  _request: NextRequest,
  { params }: { params: Promise<{ id: string }> }
) {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ error: "Non autorisé" }, { status: 401 });
  }

  const { id } = await params;

  const contract = await prisma.contract.findFirst({
    where: { id, property: { userId: session.user.id } },
    include: {
      property: true,
      signatures: true,
      payments: true,
      deposit: true,
      documents: true,
    },
  });

  if (!contract) {
    return NextResponse.json({ error: "Contrat introuvable" }, { status: 404 });
  }

  return NextResponse.json(contract);
}

export async function PATCH(
  request: NextRequest,
  { params }: { params: Promise<{ id: string }> }
) {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ error: "Non autorisé" }, { status: 401 });
  }

  const { id } = await params;

  const contract = await prisma.contract.findFirst({
    where: { id, property: { userId: session.user.id } },
  });

  if (!contract) {
    return NextResponse.json({ error: "Contrat introuvable" }, { status: 404 });
  }

  try {
    const body = await request.json();
    const { status } = body;

    // Status transition validation
    const validTransitions: Record<string, string[]> = {
      DRAFT: ["PENDING_SIGNATURE", "CANCELLED"],
      PENDING_SIGNATURE: ["SIGNED", "CANCELLED"],
      SIGNED: ["CONFIRMED", "CANCELLED"],
      CONFIRMED: ["IN_PROGRESS", "CANCELLED"],
      IN_PROGRESS: ["COMPLETED"],
      COMPLETED: ["ARCHIVED"],
    };

    if (status) {
      const allowed = validTransitions[contract.status] || [];
      if (!allowed.includes(status)) {
        return NextResponse.json(
          { error: `Transition de ${contract.status} vers ${status} non autorisée` },
          { status: 400 }
        );
      }

      await prisma.contract.update({
        where: { id },
        data: { status },
      });
    }

    const updated = await prisma.contract.findUnique({
      where: { id },
      include: { signatures: true, payments: true, documents: true },
    });

    return NextResponse.json(updated);
  } catch (error) {
    console.error("Contract update error:", error);
    return NextResponse.json({ error: "Une erreur interne est survenue" }, { status: 500 });
  }
}

```

### `src/app/api/contracts/[id]/send-signature/route.ts`

```typescript
import { NextRequest, NextResponse } from "next/server";
import { prisma } from "@/lib/prisma";
import { auth } from "@/lib/auth";
import { generateSeasonalContractPdf } from "@/lib/pdf-templates";
import { uploadFile } from "@/lib/r2";
import { sendEmail, signingRequestEmailHtml } from "@/lib/email";
import crypto from "crypto";

export async function POST(
  _request: NextRequest,
  { params }: { params: Promise<{ id: string }> }
) {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ error: "Non autorisé" }, { status: 401 });
  }

  const { id } = await params;

  const contract = await prisma.contract.findFirst({
    where: { id, property: { userId: session.user.id } },
    include: { property: true },
  });

  if (!contract) {
    return NextResponse.json({ error: "Contrat introuvable" }, { status: 404 });
  }

  if (contract.status !== "DRAFT") {
    return NextResponse.json(
      { error: "Le contrat doit être en brouillon pour envoyer la signature" },
      { status: 400 }
    );
  }

  try {
    const user = await prisma.user.findUnique({
      where: { id: session.user.id },
      select: { name: true, email: true },
    });

    const address = [
      contract.property.streetNumber,
      contract.property.streetName,
      contract.property.postalCode,
      contract.property.city,
    ].filter(Boolean).join(" ");

    // Generate PDF
    const pdfBytes = await generateSeasonalContractPdf({
      ownerName: user?.name || "Propriétaire",
      ownerEmail: user?.email || "",
      propertyName: contract.property.name,
      propertyAddress: address,
      tenantName: contract.tenantName,
      tenantEmail: contract.tenantEmail,
      tenantPhone: contract.tenantPhone || undefined,
      checkIn: contract.checkIn,
      checkOut: contract.checkOut,
      guests: contract.guests,
      rentalPrice: contract.rentalPrice,
      cleaningFee: contract.cleaningFee,
      conditions: contract.conditions || undefined,
      logoUrl: contract.property.logoUrl,
      headerText: contract.property.headerText,
    });

    // Upload PDF to R2
    const pdfKey = `documents/${session.user.id}/${crypto.randomUUID()}.pdf`;
    await uploadFile(pdfKey, pdfBytes, "application/pdf");

    // Create document record
    await prisma.document.create({
      data: {
        type: "CONTRACT_PDF",
        name: `Contrat saisonnier — ${contract.tenantName}`,
        url: pdfKey,
        propertyId: contract.propertyId,
        contractId: contract.id,
      },
    });

    // Create signature records (tenant + owner)
    const tenantSig = await prisma.signature.create({
      data: {
        contractId: contract.id,
        role: "TENANT",
      },
    });

    const ownerSig = await prisma.signature.create({
      data: {
        contractId: contract.id,
        role: "OWNER",
      },
    });

    // Update contract status
    await prisma.contract.update({
      where: { id },
      data: { status: "PENDING_SIGNATURE" },
    });

    // Send email to tenant
    const baseUrl = process.env.NEXTAUTH_URL || process.env.NEXT_PUBLIC_APP_URL || "http://localhost:3000";

    await sendEmail({
      to: contract.tenantEmail,
      subject: `Document à signer — ${contract.property.name}`,
      html: signingRequestEmailHtml(
        `Contrat saisonnier — ${contract.property.name}`,
        `${baseUrl}/sign/${tenantSig.token}`
      ),
    });

    return NextResponse.json({
      success: true,
      tenantSignToken: tenantSig.token,
      ownerSignToken: ownerSig.token,
    });
  } catch (error) {
    console.error("Send signature error:", error);
    return NextResponse.json({ error: "Une erreur interne est survenue" }, { status: 500 });
  }
}

```

### `src/app/api/condition-templates/route.ts`

```typescript
import { NextRequest, NextResponse } from "next/server";
import { prisma } from "@/lib/prisma";
import { auth } from "@/lib/auth";

export async function GET() {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ error: "Non autorisé" }, { status: 401 });
  }

  const templates = await prisma.conditionTemplate.findMany({
    where: { userId: session.user.id },
    orderBy: { createdAt: "desc" },
  });

  return NextResponse.json(templates);
}

export async function POST(request: NextRequest) {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ error: "Non autorisé" }, { status: 401 });
  }

  try {
    const body = await request.json();
    const { name, content } = body;

    if (!name?.trim() || !content?.trim()) {
      return NextResponse.json({ error: "Nom et contenu requis" }, { status: 400 });
    }

    const template = await prisma.conditionTemplate.create({
      data: {
        userId: session.user.id,
        name: name.trim(),
        content: content.trim(),
      },
    });

    return NextResponse.json(template, { status: 201 });
  } catch (error) {
    console.error("Condition template error:", error);
    return NextResponse.json({ error: "Une erreur interne est survenue" }, { status: 500 });
  }
}

```

### `src/app/api/leases/route.ts`

```typescript
import { NextRequest, NextResponse } from "next/server";
import { prisma } from "@/lib/prisma";
import { auth } from "@/lib/auth";

export async function GET(request: NextRequest) {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ error: "Non autorisé" }, { status: 401 });
  }

  const { searchParams } = new URL(request.url);
  const propertyId = searchParams.get("propertyId");
  const status = searchParams.get("status");

  const where: Record<string, unknown> = {
    property: { userId: session.user.id },
  };
  if (propertyId) where.propertyId = propertyId;
  if (status) where.status = status;

  const leases = await prisma.lease.findMany({
    where,
    include: {
      property: { select: { name: true, rentalType: true } },
      signatures: { select: { id: true, role: true, signedAt: true } },
    },
    orderBy: { createdAt: "desc" },
  });

  return NextResponse.json(leases);
}

export async function POST(request: NextRequest) {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ error: "Non autorisé" }, { status: 401 });
  }

  try {
    const body = await request.json();
    const {
      propertyId, tenantName, tenantEmail, tenantPhone, tenantBirthDate,
      startDate, endDate, rentAmount, chargesAmount, chargeType,
      depositAmount, paymentDueDay, gracePeriodDays,
    } = body;

    if (!propertyId || !tenantName || !tenantEmail || !startDate || rentAmount === undefined || chargesAmount === undefined || depositAmount === undefined) {
      return NextResponse.json({ error: "Champs obligatoires manquants" }, { status: 400 });
    }

    const property = await prisma.property.findFirst({
      where: { id: propertyId, userId: session.user.id, deletedAt: null },
    });

    if (!property) {
      return NextResponse.json({ error: "Bien introuvable" }, { status: 404 });
    }

    if (property.rentalType === "SEASONAL") {
      return NextResponse.json(
        { error: "Les baux classiques ne sont pas disponibles pour les locations saisonnières" },
        { status: 400 }
      );
    }

    const isFurnished = property.rentalType === "CLASSIC_FURNISHED";

    // Validate deposit max (1 month unfurnished, 2 months furnished)
    const maxDeposit = isFurnished ? rentAmount * 2 : rentAmount;
    if (depositAmount > maxDeposit) {
      const label = isFurnished ? "deux mois" : "un mois";
      return NextResponse.json(
        { error: `Le dépôt de garantie ne peut excéder ${label} de loyer (${maxDeposit} €)` },
        { status: 400 }
      );
    }

    const lease = await prisma.lease.create({
      data: {
        propertyId,
        tenantName,
        tenantEmail,
        tenantPhone: tenantPhone || null,
        tenantBirthDate: tenantBirthDate ? new Date(tenantBirthDate) : null,
        startDate: new Date(startDate),
        endDate: endDate ? new Date(endDate) : null,
        rentAmount,
        chargesAmount,
        chargeType: chargeType || "FIXED",
        depositAmount,
        paymentDueDay: paymentDueDay || 1,
        gracePeriodDays: gracePeriodDays || 5,
        status: "DRAFT",
      },
    });

    return NextResponse.json(lease, { status: 201 });
  } catch (error) {
    console.error("Lease creation error:", error);
    return NextResponse.json({ error: "Une erreur interne est survenue" }, { status: 500 });
  }
}

```

### `src/app/api/leases/[id]/route.ts`

```typescript
import { NextRequest, NextResponse } from "next/server";
import { prisma } from "@/lib/prisma";
import { auth } from "@/lib/auth";

export async function GET(
  _request: NextRequest,
  { params }: { params: Promise<{ id: string }> }
) {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ error: "Non autorisé" }, { status: 401 });
  }

  const { id } = await params;

  const lease = await prisma.lease.findFirst({
    where: { id, property: { userId: session.user.id } },
    include: {
      property: true,
      inspections: {
        include: { rooms: true, signatures: true },
        orderBy: { date: "desc" },
      },
      rentEntries: { orderBy: { month: "desc" } },
      signatures: true,
      deposit: true,
      documents: { orderBy: { createdAt: "desc" } },
    },
  });

  if (!lease) {
    return NextResponse.json({ error: "Bail introuvable" }, { status: 404 });
  }

  return NextResponse.json(lease);
}

export async function PATCH(
  request: NextRequest,
  { params }: { params: Promise<{ id: string }> }
) {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ error: "Non autorisé" }, { status: 401 });
  }

  const { id } = await params;

  const lease = await prisma.lease.findFirst({
    where: { id, property: { userId: session.user.id } },
    include: { property: true },
  });

  if (!lease) {
    return NextResponse.json({ error: "Bail introuvable" }, { status: 404 });
  }

  try {
    const body = await request.json();
    const { status, terminationDate } = body;

    const validTransitions: Record<string, string[]> = {
      DRAFT: ["PENDING_SIGNATURE", "ARCHIVED"],
      PENDING_SIGNATURE: ["ACTIVE", "ARCHIVED"],
      ACTIVE: ["TERMINATION_NOTICE", "ENDED"],
      TERMINATION_NOTICE: ["ENDED"],
      ENDED: ["ARCHIVED"],
    };

    const data: Record<string, unknown> = {};

    if (status) {
      const allowed = validTransitions[lease.status] || [];
      if (!allowed.includes(status)) {
        return NextResponse.json(
          { error: `Transition de ${lease.status} vers ${status} non autorisée` },
          { status: 400 }
        );
      }
      data.status = status;

      // Validate notice period for termination
      if (status === "TERMINATION_NOTICE") {
        const isFurnished = lease.property.rentalType === "CLASSIC_FURNISHED";
        const noticeMonths = isFurnished ? 3 : 6;
        const minTermination = new Date();
        minTermination.setMonth(minTermination.getMonth() + noticeMonths);

        if (terminationDate) {
          const termDate = new Date(terminationDate);
          if (termDate < minTermination) {
            return NextResponse.json(
              { error: `Le préavis minimum est de ${noticeMonths} mois` },
              { status: 400 }
            );
          }
          data.terminationDate = termDate;
        } else {
          data.terminationDate = minTermination;
        }
      }
    }

    const updated = await prisma.lease.update({
      where: { id },
      data,
      include: { signatures: true, inspections: true, documents: true },
    });

    return NextResponse.json(updated);
  } catch (error) {
    console.error("Lease update error:", error);
    return NextResponse.json({ error: "Une erreur interne est survenue" }, { status: 500 });
  }
}

```

### `src/app/api/leases/[id]/inspections/route.ts`

```typescript
import { NextRequest, NextResponse } from "next/server";
import { prisma } from "@/lib/prisma";
import { auth } from "@/lib/auth";
import { generateInspectionPdf } from "@/lib/pdf-templates";
import { uploadFile } from "@/lib/r2";
import { sendEmail, signingRequestEmailHtml } from "@/lib/email";
import crypto from "crypto";

export async function POST(
  request: NextRequest,
  { params }: { params: Promise<{ id: string }> }
) {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ error: "Non autorisé" }, { status: 401 });
  }

  const { id: leaseId } = await params;

  const lease = await prisma.lease.findFirst({
    where: { id: leaseId, property: { userId: session.user.id } },
    include: {
      property: true,
      inspections: {
        where: { type: "CHECK_IN" },
        include: { rooms: true },
        take: 1,
      },
    },
  });

  if (!lease) {
    return NextResponse.json({ error: "Bail introuvable" }, { status: 404 });
  }

  try {
    const body = await request.json();
    const { type, rooms, meterWater, meterElectricity, meterGas } = body;

    if (!type || !rooms?.length) {
      return NextResponse.json({ error: "Type et pièces requis" }, { status: 400 });
    }

    if (!["CHECK_IN", "CHECK_OUT"].includes(type)) {
      return NextResponse.json({ error: "Type invalide" }, { status: 400 });
    }

    // Create inspection + rooms
    const inspection = await prisma.inspection.create({
      data: {
        leaseId,
        type,
        meterWater: meterWater || null,
        meterElectricity: meterElectricity || null,
        meterGas: meterGas || null,
        rooms: {
          create: rooms.map((r: { name: string; walls: string; floor: string; ceiling: string; fixtures: string; notes?: string; photos?: string[] }) => ({
            name: r.name,
            walls: r.walls || "GOOD",
            floor: r.floor || "GOOD",
            ceiling: r.ceiling || "GOOD",
            fixtures: r.fixtures || "GOOD",
            notes: r.notes || null,
            photos: r.photos || [],
          })),
        },
      },
      include: { rooms: true },
    });

    // Generate PDF
    const user = await prisma.user.findUnique({
      where: { id: session.user.id },
      select: { name: true },
    });

    const address = [
      lease.property.streetNumber,
      lease.property.streetName,
      lease.property.postalCode,
      lease.property.city,
    ].filter(Boolean).join(" ");

    const checkInRooms = type === "CHECK_OUT" ? lease.inspections[0]?.rooms : undefined;

    const pdfBytes = await generateInspectionPdf({
      type,
      date: inspection.date,
      propertyName: lease.property.name,
      propertyAddress: address,
      ownerName: user?.name || "Propriétaire",
      tenantName: lease.tenantName,
      rooms: inspection.rooms.map((r) => ({
        name: r.name,
        walls: r.walls,
        floor: r.floor,
        ceiling: r.ceiling,
        fixtures: r.fixtures,
        notes: r.notes || undefined,
      })),
      meterWater: meterWater,
      meterElectricity: meterElectricity,
      meterGas: meterGas,
      checkInRooms: checkInRooms?.map((r) => ({
        name: r.name,
        walls: r.walls,
        floor: r.floor,
        ceiling: r.ceiling,
        fixtures: r.fixtures,
        notes: r.notes || undefined,
      })),
      logoUrl: lease.property.logoUrl,
      headerText: lease.property.headerText,
    });

    const pdfKey = `documents/${session.user.id}/${crypto.randomUUID()}.pdf`;
    await uploadFile(pdfKey, pdfBytes, "application/pdf");

    await prisma.document.create({
      data: {
        type: "INSPECTION_PDF",
        name: `État des lieux ${type === "CHECK_IN" ? "d'entrée" : "de sortie"} — ${lease.tenantName}`,
        url: pdfKey,
        propertyId: lease.propertyId,
        leaseId,
        inspectionId: inspection.id,
      },
    });

    // Create signature records
    const tenantSig = await prisma.signature.create({
      data: { inspectionId: inspection.id, role: "TENANT" },
    });

    await prisma.signature.create({
      data: { inspectionId: inspection.id, role: "OWNER" },
    });

    // Send for signature
    const baseUrl = process.env.NEXTAUTH_URL || process.env.NEXT_PUBLIC_APP_URL || "http://localhost:3000";

    await sendEmail({
      to: lease.tenantEmail,
      subject: `État des lieux à signer — ${lease.property.name}`,
      html: signingRequestEmailHtml(
        `État des lieux ${type === "CHECK_IN" ? "d'entrée" : "de sortie"}`,
        `${baseUrl}/sign/${tenantSig.token}`
      ),
    });

    return NextResponse.json(inspection, { status: 201 });
  } catch (error) {
    console.error("Inspection creation error:", error);
    return NextResponse.json({ error: "Une erreur interne est survenue" }, { status: 500 });
  }
}

```

### `src/app/api/leases/[id]/letters/route.ts`

```typescript
import { NextRequest, NextResponse } from "next/server";
import { prisma } from "@/lib/prisma";
import { auth } from "@/lib/auth";
import { generateLetterPdf, type LetterType } from "@/lib/pdf-templates";
import { uploadFile } from "@/lib/r2";
import crypto from "crypto";

export async function POST(
  request: NextRequest,
  { params }: { params: Promise<{ id: string }> }
) {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ error: "Non autorisé" }, { status: 401 });
  }

  const { id: leaseId } = await params;

  const lease = await prisma.lease.findFirst({
    where: { id: leaseId, property: { userId: session.user.id } },
    include: { property: true },
  });

  if (!lease) {
    return NextResponse.json({ error: "Bail introuvable" }, { status: 404 });
  }

  try {
    const body = await request.json();
    const { type, amountDue, newRent, oldRent, effectiveDate, chargesDetail, reason } = body;

    const validTypes: LetterType[] = [
      "MISE_EN_DEMEURE",
      "CONGE_BAILLEUR",
      "CONGE_LOCATAIRE",
      "REGULARISATION_CHARGES",
      "REVISION_LOYER",
    ];

    if (!validTypes.includes(type)) {
      return NextResponse.json({ error: "Type de courrier invalide" }, { status: 400 });
    }

    const user = await prisma.user.findUnique({
      where: { id: session.user.id },
      select: { name: true },
    });

    const address = [
      lease.property.streetNumber,
      lease.property.streetName,
      lease.property.postalCode,
      lease.property.city,
    ].filter(Boolean).join(" ");

    const pdfBytes = await generateLetterPdf({
      type,
      ownerName: user?.name || "Propriétaire",
      ownerAddress: address,
      tenantName: lease.tenantName,
      propertyAddress: address,
      amountDue,
      newRent,
      oldRent,
      effectiveDate: effectiveDate ? new Date(effectiveDate) : undefined,
      chargesDetail,
      reason,
      logoUrl: lease.property.logoUrl,
      headerText: lease.property.headerText,
    });

    const pdfKey = `documents/${session.user.id}/${crypto.randomUUID()}.pdf`;
    await uploadFile(pdfKey, pdfBytes, "application/pdf");

    const letterNames: Record<LetterType, string> = {
      MISE_EN_DEMEURE: "Mise en demeure",
      CONGE_BAILLEUR: "Congé bailleur",
      CONGE_LOCATAIRE: "Congé locataire",
      REGULARISATION_CHARGES: "Régularisation de charges",
      REVISION_LOYER: "Révision de loyer",
    };

    const doc = await prisma.document.create({
      data: {
        type: "LETTER",
        name: `${letterNames[type as LetterType]} — ${lease.tenantName}`,
        url: pdfKey,
        propertyId: lease.propertyId,
        leaseId,
      },
    });

    return NextResponse.json(doc, { status: 201 });
  } catch (error) {
    console.error("Letter generation error:", error);
    return NextResponse.json({ error: "Une erreur interne est survenue" }, { status: 500 });
  }
}

```

### `src/app/api/leases/[id]/send-signature/route.ts`

```typescript
import { NextRequest, NextResponse } from "next/server";
import { prisma } from "@/lib/prisma";
import { auth } from "@/lib/auth";
import { generateLeasePdf } from "@/lib/pdf-templates";
import { uploadFile } from "@/lib/r2";
import { sendEmail, signingRequestEmailHtml } from "@/lib/email";
import crypto from "crypto";

export async function POST(
  _request: NextRequest,
  { params }: { params: Promise<{ id: string }> }
) {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ error: "Non autorisé" }, { status: 401 });
  }

  const { id } = await params;

  const lease = await prisma.lease.findFirst({
    where: { id, property: { userId: session.user.id } },
    include: { property: true },
  });

  if (!lease) {
    return NextResponse.json({ error: "Bail introuvable" }, { status: 404 });
  }

  if (lease.status !== "DRAFT") {
    return NextResponse.json({ error: "Le bail doit être en brouillon" }, { status: 400 });
  }

  try {
    const user = await prisma.user.findUnique({
      where: { id: session.user.id },
      select: { name: true, email: true },
    });

    const address = [
      lease.property.streetNumber,
      lease.property.streetName,
      lease.property.postalCode,
      lease.property.city,
    ].filter(Boolean).join(" ");

    const pdfBytes = await generateLeasePdf({
      ownerName: user?.name || "Propriétaire",
      ownerEmail: user?.email || "",
      propertyName: lease.property.name,
      propertyAddress: address,
      rentalType: lease.property.rentalType as "CLASSIC_FURNISHED" | "CLASSIC_UNFURNISHED",
      tenantName: lease.tenantName,
      tenantEmail: lease.tenantEmail,
      tenantPhone: lease.tenantPhone || undefined,
      tenantBirthDate: lease.tenantBirthDate || undefined,
      startDate: lease.startDate,
      endDate: lease.endDate || undefined,
      rentAmount: lease.rentAmount,
      chargesAmount: lease.chargesAmount,
      chargeType: lease.chargeType as "FIXED" | "PROVISION",
      depositAmount: lease.depositAmount,
      paymentDueDay: lease.paymentDueDay,
      logoUrl: lease.property.logoUrl,
      headerText: lease.property.headerText,
    });

    const pdfKey = `documents/${session.user.id}/${crypto.randomUUID()}.pdf`;
    await uploadFile(pdfKey, pdfBytes, "application/pdf");

    await prisma.document.create({
      data: {
        type: "LEASE_PDF",
        name: `Bail — ${lease.tenantName}`,
        url: pdfKey,
        propertyId: lease.propertyId,
        leaseId: lease.id,
      },
    });

    const tenantSig = await prisma.signature.create({
      data: { leaseId: lease.id, role: "TENANT" },
    });

    await prisma.signature.create({
      data: { leaseId: lease.id, role: "OWNER" },
    });

    await prisma.lease.update({
      where: { id },
      data: { status: "PENDING_SIGNATURE" },
    });

    const baseUrl = process.env.NEXTAUTH_URL || process.env.NEXT_PUBLIC_APP_URL || "http://localhost:3000";

    await sendEmail({
      to: lease.tenantEmail,
      subject: `Bail à signer — ${lease.property.name}`,
      html: signingRequestEmailHtml(
        `Bail — ${lease.property.name}`,
        `${baseUrl}/sign/${tenantSig.token}`
      ),
    });

    return NextResponse.json({ success: true });
  } catch (error) {
    console.error("Lease send-signature error:", error);
    return NextResponse.json({ error: "Une erreur interne est survenue" }, { status: 500 });
  }
}

```

### `src/app/api/signatures/[token]/route.ts`

```typescript
import { NextRequest, NextResponse } from "next/server";
import { prisma } from "@/lib/prisma";
import { embedSignatureInPdf } from "@/lib/pdf-signature";
import { uploadFile, getPresignedDownloadUrl } from "@/lib/r2";

/**
 * GET /api/signatures/:token — Get signature info + document download URL
 */
export async function GET(
  _request: NextRequest,
  { params }: { params: Promise<{ token: string }> }
) {
  const { token } = await params;

  const signature = await prisma.signature.findUnique({
    where: { token },
    include: {
      contract: {
        include: {
          property: { select: { name: true, streetName: true, city: true } },
          documents: { where: { type: "CONTRACT_PDF" }, take: 1 },
        },
      },
      lease: {
        include: {
          property: { select: { name: true, streetName: true, city: true } },
          documents: { where: { type: "LEASE_PDF" }, take: 1 },
        },
      },
      inspection: {
        include: {
          lease: {
            include: {
              property: { select: { name: true, streetName: true, city: true } },
            },
          },
          documents: { where: { type: "INSPECTION_PDF" }, take: 1 },
        },
      },
    },
  });

  if (!signature) {
    return NextResponse.json({ error: "Lien de signature invalide" }, { status: 404 });
  }

  if (signature.signedAt) {
    return NextResponse.json({ error: "Ce document a déjà été signé" }, { status: 400 });
  }

  // Determine the document and context
  let propertyName = "";
  let documentUrl = "";
  let documentType = "";

  if (signature.contract) {
    propertyName = signature.contract.property.name;
    documentType = "Contrat saisonnier";
    const doc = signature.contract.documents[0];
    if (doc) documentUrl = await getPresignedDownloadUrl(doc.url);
  } else if (signature.lease) {
    propertyName = signature.lease.property.name;
    documentType = "Bail";
    const doc = signature.lease.documents[0];
    if (doc) documentUrl = await getPresignedDownloadUrl(doc.url);
  } else if (signature.inspection) {
    propertyName = signature.inspection.lease.property.name;
    documentType = "État des lieux";
    const doc = signature.inspection.documents[0];
    if (doc) documentUrl = await getPresignedDownloadUrl(doc.url);
  }

  return NextResponse.json({
    id: signature.id,
    role: signature.role,
    propertyName,
    documentType,
    documentUrl,
  });
}

/**
 * POST /api/signatures/:token — Submit a signature
 */
export async function POST(
  request: NextRequest,
  { params }: { params: Promise<{ token: string }> }
) {
  const { token } = await params;

  const signature = await prisma.signature.findUnique({
    where: { token },
    include: {
      contract: { include: { documents: { where: { type: "CONTRACT_PDF" }, take: 1 } } },
      lease: { include: { documents: { where: { type: "LEASE_PDF" }, take: 1 } } },
      inspection: { include: { documents: { where: { type: "INSPECTION_PDF" }, take: 1 } } },
    },
  });

  if (!signature) {
    return NextResponse.json({ error: "Lien de signature invalide" }, { status: 404 });
  }

  if (signature.signedAt) {
    return NextResponse.json({ error: "Ce document a déjà été signé" }, { status: 400 });
  }

  try {
    const body = await request.json();
    const { signatureDataUrl, signerName } = body;

    if (!signatureDataUrl || !signerName) {
      return NextResponse.json(
        { error: "Signature et nom du signataire requis" },
        { status: 400 }
      );
    }

    // Get the current PDF document
    let documentKey = "";
    if (signature.contract?.documents[0]) {
      documentKey = signature.contract.documents[0].url;
    } else if (signature.lease?.documents[0]) {
      documentKey = signature.lease.documents[0].url;
    } else if (signature.inspection?.documents[0]) {
      documentKey = signature.inspection.documents[0].url;
    }

    // Get signer's IP
    const signerIp =
      request.headers.get("x-forwarded-for")?.split(",")[0]?.trim() ||
      request.headers.get("x-real-ip") ||
      "unknown";

    let documentHash = "";
    let signatureUrl = "";

    if (documentKey) {
      // Download current PDF from R2
      const pdfUrl = await getPresignedDownloadUrl(documentKey);
      const pdfResponse = await fetch(pdfUrl);
      const pdfBytes = new Uint8Array(await pdfResponse.arrayBuffer());

      // Embed signature in PDF
      const result = await embedSignatureInPdf(pdfBytes, signatureDataUrl, signerName);
      documentHash = result.documentHash;

      // Upload signed PDF
      const signedKey = documentKey.replace(".pdf", `-signed-${signature.role.toLowerCase()}.pdf`);
      await uploadFile(signedKey, result.signedPdfBytes, "application/pdf");
      signatureUrl = signedKey;
    }

    // Store signature image
    const sigImageBase64 = signatureDataUrl.replace(/^data:image\/png;base64,/, "");
    const sigImageBytes = Uint8Array.from(atob(sigImageBase64), (c) => c.charCodeAt(0));
    const sigImageKey = `signatures/${signature.id}.png`;
    await uploadFile(sigImageKey, sigImageBytes, "image/png");

    // Update signature record
    await prisma.signature.update({
      where: { id: signature.id },
      data: {
        signedAt: new Date(),
        signerIp,
        documentHash,
        signatureUrl: sigImageKey,
      },
    });

    // Check if all signatures for this document are complete → update status
    if (signature.contractId) {
      const allSigs = await prisma.signature.findMany({
        where: { contractId: signature.contractId },
      });
      const allSigned = allSigs.every((s) => s.id === signature.id || s.signedAt !== null);
      if (allSigned) {
        await prisma.contract.update({
          where: { id: signature.contractId },
          data: { status: "SIGNED" },
        });
      }
    } else if (signature.leaseId) {
      const allSigs = await prisma.signature.findMany({
        where: { leaseId: signature.leaseId },
      });
      const allSigned = allSigs.every((s) => s.id === signature.id || s.signedAt !== null);
      if (allSigned) {
        await prisma.lease.update({
          where: { id: signature.leaseId },
          data: { status: "ACTIVE" },
        });
      }
    } else if (signature.inspectionId) {
      const allSigs = await prisma.signature.findMany({
        where: { inspectionId: signature.inspectionId },
      });
      const allSigned = allSigs.every((s) => s.id === signature.id || s.signedAt !== null);
      if (allSigned) {
        await prisma.inspection.update({
          where: { id: signature.inspectionId },
          data: { validated: true },
        });
      }
    }

    return NextResponse.json({ success: true });
  } catch (error) {
    console.error("Signature error:", error);
    return NextResponse.json({ error: "Une erreur interne est survenue" }, { status: 500 });
  }
}

```

### `src/app/api/payments/route.ts`

```typescript
import { NextResponse } from "next/server";
import { prisma } from "@/lib/prisma";
import { auth } from "@/lib/auth";

export async function GET() {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ error: "Non autorisé" }, { status: 401 });
  }

  const payments = await prisma.payment.findMany({
    where: {
      OR: [
        { contract: { property: { userId: session.user.id } } },
        { rentEntry: { lease: { property: { userId: session.user.id } } } },
      ],
    },
    include: {
      contract: {
        select: { tenantName: true, property: { select: { name: true } } },
      },
      rentEntry: {
        select: {
          month: true,
          lease: {
            select: { tenantName: true, property: { select: { name: true } } },
          },
        },
      },
    },
    orderBy: { createdAt: "desc" },
  });

  return NextResponse.json(payments);
}

```

### `src/app/api/deposits/route.ts`

```typescript
import { NextRequest, NextResponse } from "next/server";
import { prisma } from "@/lib/prisma";
import { auth } from "@/lib/auth";
import { stripe } from "@/lib/stripe";

export async function POST(request: NextRequest) {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ error: "Non autorisé" }, { status: 401 });
  }

  try {
    const body = await request.json();
    const { contractId, leaseId, amount, type } = body;

    if (!amount || amount <= 0) {
      return NextResponse.json({ error: "Montant invalide" }, { status: 400 });
    }

    if (!["CASH", "BANK_TRANSFER", "CREDIT_CARD_HOLD"].includes(type)) {
      return NextResponse.json({ error: "Type de dépôt invalide" }, { status: 400 });
    }

    // Verify ownership
    if (contractId) {
      const contract = await prisma.contract.findFirst({
        where: { id: contractId, property: { userId: session.user.id } },
      });
      if (!contract) return NextResponse.json({ error: "Contrat introuvable" }, { status: 404 });
    }

    if (leaseId) {
      const lease = await prisma.lease.findFirst({
        where: { id: leaseId, property: { userId: session.user.id } },
        include: { property: true },
      });
      if (!lease) return NextResponse.json({ error: "Bail introuvable" }, { status: 404 });

      // Validate legal max
      const isFurnished = lease.property.rentalType === "CLASSIC_FURNISHED";
      const maxDeposit = isFurnished ? lease.rentAmount * 2 : lease.rentAmount;
      if (amount > maxDeposit) {
        return NextResponse.json(
          { error: `Le dépôt ne peut excéder ${maxDeposit.toFixed(2)} €` },
          { status: 400 }
        );
      }
    }

    let stripeSetupIntentId: string | undefined;
    let depositStatus = "PENDING";

    // For credit card holds on seasonal, create Stripe SetupIntent
    if (type === "CREDIT_CARD_HOLD" && contractId) {
      const user = await prisma.user.findUnique({
        where: { id: session.user.id },
        select: { stripeAccountId: true },
      });

      if (user?.stripeAccountId) {
        const setupIntent = await stripe.setupIntents.create({
          payment_method_types: ["card"],
          on_behalf_of: user.stripeAccountId,
        });
        stripeSetupIntentId = setupIntent.id;
        depositStatus = "HOLD_ACTIVE";
      }
    }

    const deposit = await prisma.deposit.create({
      data: {
        amount,
        type: type as "CASH" | "BANK_TRANSFER" | "CREDIT_CARD_HOLD",
        status: depositStatus as "PENDING" | "HOLD_ACTIVE",
        ...(contractId ? { contract: { connect: { id: contractId } } } : {}),
        ...(leaseId ? { lease: { connect: { id: leaseId } } } : {}),
        ...(stripeSetupIntentId ? { stripeSetupIntentId } : {}),
      },
    });

    return NextResponse.json(deposit, { status: 201 });
  } catch (error) {
    console.error("Deposit creation error:", error);
    return NextResponse.json({ error: "Une erreur interne est survenue" }, { status: 500 });
  }
}

```

### `src/app/api/rent-entries/route.ts`

```typescript
import { NextRequest, NextResponse } from "next/server";
import { prisma } from "@/lib/prisma";
import { auth } from "@/lib/auth";

export async function GET(request: NextRequest) {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ error: "Non autorisé" }, { status: 401 });
  }

  const { searchParams } = new URL(request.url);
  const leaseId = searchParams.get("leaseId");

  if (!leaseId) {
    return NextResponse.json({ error: "leaseId requis" }, { status: 400 });
  }

  const lease = await prisma.lease.findFirst({
    where: { id: leaseId, property: { userId: session.user.id } },
  });

  if (!lease) {
    return NextResponse.json({ error: "Bail introuvable" }, { status: 404 });
  }

  const entries = await prisma.rentEntry.findMany({
    where: { leaseId },
    include: { payments: true },
    orderBy: { month: "desc" },
  });

  return NextResponse.json(entries);
}

```

### `src/app/api/rent-entries/[id]/route.ts`

```typescript
import { NextRequest, NextResponse } from "next/server";
import { prisma } from "@/lib/prisma";
import { auth } from "@/lib/auth";
import { generateRentReceiptPdf } from "@/lib/pdf-templates";
import { uploadFile } from "@/lib/r2";
import crypto from "crypto";

/**
 * PATCH /api/rent-entries/:id — Record manual payment
 */
export async function PATCH(
  request: NextRequest,
  { params }: { params: Promise<{ id: string }> }
) {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ error: "Non autorisé" }, { status: 401 });
  }

  const { id } = await params;

  const rentEntry = await prisma.rentEntry.findUnique({
    where: { id },
    include: {
      lease: {
        include: { property: { include: { user: { select: { name: true, id: true } } } } },
      },
    },
  });

  if (!rentEntry) {
    return NextResponse.json({ error: "Entrée introuvable" }, { status: 404 });
  }

  if (rentEntry.lease.property.user.id !== session.user.id) {
    return NextResponse.json({ error: "Non autorisé" }, { status: 403 });
  }

  try {
    const body = await request.json();
    const { amount, method } = body;

    if (!amount || amount <= 0) {
      return NextResponse.json({ error: "Montant invalide" }, { status: 400 });
    }

    if (!["CASH", "CHECK", "BANK_TRANSFER"].includes(method)) {
      return NextResponse.json({ error: "Méthode de paiement invalide" }, { status: 400 });
    }

    // Create payment record
    await prisma.payment.create({
      data: {
        rentEntryId: id,
        amount,
        method,
        status: "COMPLETED",
        paidAt: new Date(),
      },
    });

    // Update rent entry
    const newPaid = rentEntry.amountPaid + amount;
    const status = newPaid >= rentEntry.amountDue ? "PAID" : "PARTIAL";

    const updated = await prisma.rentEntry.update({
      where: { id },
      data: {
        amountPaid: newPaid,
        status,
        paidAt: status === "PAID" ? new Date() : null,
      },
    });

    // Generate receipt PDF
    const address = [
      rentEntry.lease.property.streetNumber,
      rentEntry.lease.property.streetName,
      rentEntry.lease.property.postalCode,
      rentEntry.lease.property.city,
    ].filter(Boolean).join(" ");

    const receiptNumber = `Q-${Date.now()}`;

    const pdfBytes = await generateRentReceiptPdf({
      ownerName: rentEntry.lease.property.user.name || "Propriétaire",
      tenantName: rentEntry.lease.tenantName,
      propertyAddress: address,
      month: rentEntry.month,
      rentAmount: rentEntry.lease.rentAmount,
      chargesAmount: rentEntry.lease.chargesAmount,
      paidAmount: amount,
      paidAt: new Date(),
      receiptNumber,
      logoUrl: rentEntry.lease.property.logoUrl,
      headerText: rentEntry.lease.property.headerText,
    });

    const pdfKey = `documents/${session.user.id}/${crypto.randomUUID()}.pdf`;
    await uploadFile(pdfKey, pdfBytes, "application/pdf");

    const docType = status === "PAID" ? "RENT_RECEIPT" : "PAYMENT_RECEIPT";
    await prisma.document.create({
      data: {
        type: docType,
        name: `${docType === "RENT_RECEIPT" ? "Quittance" : "Reçu"} — ${receiptNumber}`,
        url: pdfKey,
        propertyId: rentEntry.lease.propertyId,
        leaseId: rentEntry.leaseId,
        rentEntryId: id,
      },
    });

    return NextResponse.json(updated);
  } catch (error) {
    console.error("Rent entry update error:", error);
    return NextResponse.json({ error: "Une erreur interne est survenue" }, { status: 500 });
  }
}

```

### `src/app/api/documents/route.ts`

```typescript
import { NextRequest, NextResponse } from "next/server";
import { prisma } from "@/lib/prisma";
import { auth } from "@/lib/auth";
import { uploadFile, getPresignedDownloadUrl } from "@/lib/r2";
import crypto from "crypto";

/**
 * GET /api/documents?propertyId=...&contractId=...&leaseId=...
 * List documents filtered by context.
 */
export async function GET(request: NextRequest) {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ error: "Non autorisé" }, { status: 401 });
  }

  const { searchParams } = new URL(request.url);
  const propertyId = searchParams.get("propertyId");
  const contractId = searchParams.get("contractId");
  const leaseId = searchParams.get("leaseId");

  const where: Record<string, string | { userId: string }> = {};
  if (propertyId) where.propertyId = propertyId;
  if (contractId) where.contractId = contractId;
  if (leaseId) where.leaseId = leaseId;

  // Security: ensure the user owns the property
  if (propertyId) {
    const property = await prisma.property.findFirst({
      where: { id: propertyId, userId: session.user.id },
    });
    if (!property) {
      return NextResponse.json({ error: "Non autorisé" }, { status: 403 });
    }
  }

  const documents = await prisma.document.findMany({
    where,
    orderBy: { createdAt: "desc" },
  });

  return NextResponse.json(documents);
}

/**
 * POST /api/documents — Store a generated PDF document
 * Expects { pdfBytes (base64), type, name, propertyId, contractId?, leaseId?, inspectionId?, paymentId?, rentEntryId? }
 */
export async function POST(request: NextRequest) {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ error: "Non autorisé" }, { status: 401 });
  }

  try {
    const body = await request.json();
    const { pdfBase64, type, name, propertyId, contractId, leaseId, inspectionId, paymentId, rentEntryId } = body;

    if (!pdfBase64 || !type || !name) {
      return NextResponse.json({ error: "Données manquantes" }, { status: 400 });
    }

    // Verify ownership
    if (propertyId) {
      const property = await prisma.property.findFirst({
        where: { id: propertyId, userId: session.user.id },
      });
      if (!property) {
        return NextResponse.json({ error: "Non autorisé" }, { status: 403 });
      }
    }

    // Upload to R2
    const pdfBytes = Uint8Array.from(Buffer.from(pdfBase64, "base64"));
    const key = `documents/${session.user.id}/${crypto.randomUUID()}.pdf`;
    await uploadFile(key, pdfBytes, "application/pdf");

    // Store in database
    const document = await prisma.document.create({
      data: {
        type,
        name,
        url: key,
        propertyId: propertyId || null,
        contractId: contractId || null,
        leaseId: leaseId || null,
        inspectionId: inspectionId || null,
        paymentId: paymentId || null,
        rentEntryId: rentEntryId || null,
      },
    });

    return NextResponse.json(document, { status: 201 });
  } catch (error) {
    console.error("Document creation error:", error);
    return NextResponse.json({ error: "Une erreur interne est survenue" }, { status: 500 });
  }
}

/**
 * Helper used by other API routes to save a generated PDF.
 */
export async function saveGeneratedPdf(
  userId: string,
  pdfBytes: Uint8Array,
  opts: {
    type: string;
    name: string;
    propertyId?: string;
    contractId?: string;
    leaseId?: string;
    inspectionId?: string;
    paymentId?: string;
    rentEntryId?: string;
  }
) {
  const key = `documents/${userId}/${crypto.randomUUID()}.pdf`;
  await uploadFile(key, pdfBytes, "application/pdf");

  return prisma.document.create({
    data: {
      type: opts.type as import("@/generated/prisma/enums").DocumentType,
      name: opts.name,
      url: key,
      propertyId: opts.propertyId || null,
      contractId: opts.contractId || null,
      leaseId: opts.leaseId || null,
      inspectionId: opts.inspectionId || null,
      paymentId: opts.paymentId || null,
      rentEntryId: opts.rentEntryId || null,
    },
  });
}

/**
 * GET /api/documents/download?key=...
 */
export async function generateDownloadUrl(key: string): Promise<string> {
  return getPresignedDownloadUrl(key);
}

```

### `src/app/api/upload/route.ts`

```typescript
import { NextRequest, NextResponse } from "next/server";
import { auth } from "@/lib/auth";
import { getPresignedUploadUrl } from "@/lib/r2";
import crypto from "crypto";

export async function POST(request: NextRequest) {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ error: "Non autorisé" }, { status: 401 });
  }

  try {
    const body = await request.json();
    const { contentType, folder } = body;

    if (!contentType) {
      return NextResponse.json({ error: "Content type requis" }, { status: 400 });
    }

    const allowedTypes = ["image/jpeg", "image/png", "image/webp", "application/pdf"];
    if (!allowedTypes.includes(contentType)) {
      return NextResponse.json({ error: "Type de fichier non autorisé" }, { status: 400 });
    }

    const ext = contentType.split("/")[1] === "jpeg" ? "jpg" : contentType.split("/")[1];
    const key = `${folder || "uploads"}/${session.user.id}/${crypto.randomUUID()}.${ext}`;

    const uploadUrl = await getPresignedUploadUrl(key, contentType);

    return NextResponse.json({ uploadUrl, key });
  } catch (error) {
    console.error("Upload URL generation error:", error);
    return NextResponse.json({ error: "Une erreur interne est survenue" }, { status: 500 });
  }
}

```

### `src/app/api/calendar/events/route.ts`

```typescript
import { NextRequest, NextResponse } from "next/server";
import { prisma } from "@/lib/prisma";
import { auth } from "@/lib/auth";

/**
 * GET /api/calendar/events?propertyId=...
 * Returns all events (contracts + blocked + external) for a property.
 */
export async function GET(request: NextRequest) {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ error: "Non autorisé" }, { status: 401 });
  }

  const { searchParams } = new URL(request.url);
  const propertyId = searchParams.get("propertyId");

  if (!propertyId) {
    return NextResponse.json({ error: "propertyId requis" }, { status: 400 });
  }

  const property = await prisma.property.findFirst({
    where: { id: propertyId, userId: session.user.id, deletedAt: null },
  });

  if (!property) {
    return NextResponse.json({ error: "Bien introuvable" }, { status: 404 });
  }

  // Get contracts
  const contracts = await prisma.contract.findMany({
    where: {
      propertyId,
      status: { notIn: ["CANCELLED", "ARCHIVED"] },
    },
    select: {
      id: true,
      checkIn: true,
      checkOut: true,
      tenantName: true,
      status: true,
      rentalPrice: true,
    },
  });

  // Get calendar events (blocked + external)
  const calendarEvents = await prisma.calendarEvent.findMany({
    where: { propertyId },
    orderBy: { startDate: "asc" },
  });

  // Format as unified events
  type Event = {
    id: string;
    title: string;
    start: string;
    end: string;
    type: string;
    status?: string;
    color: string;
    meta?: Record<string, unknown>;
  };

  const events: Event[] = [];

  // Contract events
  const statusColors: Record<string, string> = {
    DRAFT: "#9CA3AF",
    PENDING_SIGNATURE: "#F59E0B",
    SIGNED: "#3B82F6",
    CONFIRMED: "#10B981",
    IN_PROGRESS: "#059669",
    COMPLETED: "#6B7280",
  };

  for (const c of contracts) {
    events.push({
      id: `contract-${c.id}`,
      title: c.tenantName,
      start: c.checkIn.toISOString(),
      end: c.checkOut.toISOString(),
      type: "contract",
      status: c.status,
      color: statusColors[c.status] || "#6B7280",
      meta: { contractId: c.id, price: c.rentalPrice },
    });
  }

  // Blocked events
  for (const e of calendarEvents.filter((ev) => ev.type === "BLOCKED")) {
    events.push({
      id: `blocked-${e.id}`,
      title: e.reason || "Bloqué",
      start: e.startDate.toISOString(),
      end: e.endDate.toISOString(),
      type: "blocked",
      color: "#EF4444",
    });
  }

  // External events
  for (const e of calendarEvents.filter((ev) => ev.type === "EXTERNAL")) {
    events.push({
      id: `external-${e.id}`,
      title: e.sourceLabel || "Externe",
      start: e.startDate.toISOString(),
      end: e.endDate.toISOString(),
      type: "external",
      color: "#8B5CF6",
    });
  }

  return NextResponse.json(events);
}

/**
 * POST /api/calendar/events — Block dates
 */
export async function POST(request: NextRequest) {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ error: "Non autorisé" }, { status: 401 });
  }

  try {
    const body = await request.json();
    const { propertyId, startDate, endDate, reason } = body;

    if (!propertyId || !startDate || !endDate) {
      return NextResponse.json({ error: "Champs requis manquants" }, { status: 400 });
    }

    const property = await prisma.property.findFirst({
      where: { id: propertyId, userId: session.user.id, deletedAt: null },
    });

    if (!property) {
      return NextResponse.json({ error: "Bien introuvable" }, { status: 404 });
    }

    const event = await prisma.calendarEvent.create({
      data: {
        propertyId,
        type: "BLOCKED",
        startDate: new Date(startDate),
        endDate: new Date(endDate),
        reason: reason || null,
      },
    });

    return NextResponse.json(event, { status: 201 });
  } catch (error) {
    console.error("Block dates error:", error);
    return NextResponse.json({ error: "Une erreur interne est survenue" }, { status: 500 });
  }
}

/**
 * DELETE /api/calendar/events?id=... — Remove a blocked date
 */
export async function DELETE(request: NextRequest) {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ error: "Non autorisé" }, { status: 401 });
  }

  const { searchParams } = new URL(request.url);
  const eventId = searchParams.get("id");

  if (!eventId) {
    return NextResponse.json({ error: "id requis" }, { status: 400 });
  }

  const event = await prisma.calendarEvent.findUnique({
    where: { id: eventId },
    include: { property: { select: { userId: true } } },
  });

  if (!event || event.property.userId !== session.user.id) {
    return NextResponse.json({ error: "Événement introuvable" }, { status: 404 });
  }

  await prisma.calendarEvent.delete({ where: { id: eventId } });

  return NextResponse.json({ success: true });
}

```

### `src/app/api/calendar/external/route.ts`

```typescript
import { NextRequest, NextResponse } from "next/server";
import { prisma } from "@/lib/prisma";
import { auth } from "@/lib/auth";

/**
 * GET /api/calendar/external?propertyId=... — List external calendars
 */
export async function GET(request: NextRequest) {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ error: "Non autorisé" }, { status: 401 });
  }

  const { searchParams } = new URL(request.url);
  const propertyId = searchParams.get("propertyId");

  if (!propertyId) {
    return NextResponse.json({ error: "propertyId requis" }, { status: 400 });
  }

  const property = await prisma.property.findFirst({
    where: { id: propertyId, userId: session.user.id },
  });

  if (!property) {
    return NextResponse.json({ error: "Bien introuvable" }, { status: 404 });
  }

  const calendars = await prisma.externalCalendar.findMany({
    where: { propertyId },
    orderBy: { createdAt: "desc" },
  });

  return NextResponse.json(calendars);
}

/**
 * POST /api/calendar/external — Add external iCal calendar
 */
export async function POST(request: NextRequest) {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ error: "Non autorisé" }, { status: 401 });
  }

  try {
    const body = await request.json();
    const { propertyId, name, url } = body;

    if (!propertyId || !name || !url) {
      return NextResponse.json({ error: "Champs requis manquants" }, { status: 400 });
    }

    const property = await prisma.property.findFirst({
      where: { id: propertyId, userId: session.user.id },
    });

    if (!property) {
      return NextResponse.json({ error: "Bien introuvable" }, { status: 404 });
    }

    const calendar = await prisma.externalCalendar.create({
      data: { propertyId, name, url },
    });

    return NextResponse.json(calendar, { status: 201 });
  } catch (error) {
    console.error("External calendar error:", error);
    return NextResponse.json({ error: "Une erreur interne est survenue" }, { status: 500 });
  }
}

/**
 * DELETE /api/calendar/external?id=... — Remove external calendar
 */
export async function DELETE(request: NextRequest) {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ error: "Non autorisé" }, { status: 401 });
  }

  const { searchParams } = new URL(request.url);
  const calendarId = searchParams.get("id");

  if (!calendarId) {
    return NextResponse.json({ error: "id requis" }, { status: 400 });
  }

  const calendar = await prisma.externalCalendar.findUnique({
    where: { id: calendarId },
    include: { property: { select: { userId: true } } },
  });

  if (!calendar || calendar.property.userId !== session.user.id) {
    return NextResponse.json({ error: "Calendrier introuvable" }, { status: 404 });
  }

  // Remove associated external events
  await prisma.calendarEvent.deleteMany({
    where: {
      propertyId: calendar.propertyId,
      type: "EXTERNAL",
      sourceLabel: calendar.name,
    },
  });

  await prisma.externalCalendar.delete({ where: { id: calendarId } });

  return NextResponse.json({ success: true });
}

```

### `src/app/api/calendar/ical/route.ts`

```typescript
import { NextRequest, NextResponse } from "next/server";
import { prisma } from "@/lib/prisma";
import icalGenerator from "ical-generator";

/**
 * GET /api/calendar/ical?propertyId=... — Export iCal feed for a property
 * This is a public URL (unique per property) that external services can consume.
 */
export async function GET(request: NextRequest) {
  const { searchParams } = new URL(request.url);
  const propertyId = searchParams.get("propertyId");

  if (!propertyId) {
    return new NextResponse("Missing propertyId", { status: 400 });
  }

  const property = await prisma.property.findUnique({
    where: { id: propertyId },
    select: { id: true, name: true },
  });

  if (!property) {
    return new NextResponse("Property not found", { status: 404 });
  }

  // Get confirmed contracts and blocked dates
  const contracts = await prisma.contract.findMany({
    where: {
      propertyId,
      status: { in: ["CONFIRMED", "IN_PROGRESS", "COMPLETED"] },
    },
    select: { id: true, checkIn: true, checkOut: true, tenantName: true, status: true },
  });

  const blockedDates = await prisma.calendarEvent.findMany({
    where: { propertyId, type: "BLOCKED" },
    select: { id: true, startDate: true, endDate: true, reason: true },
  });

  const cal = icalGenerator({
    name: `LS Immo — ${property.name}`,
    prodId: { company: "LS Immo", product: "Calendar" },
  });

  for (const c of contracts) {
    cal.createEvent({
      id: c.id,
      start: c.checkIn,
      end: c.checkOut,
      summary: `Réservation — ${c.tenantName}`,
      description: `Statut: ${c.status}`,
      allDay: true,
    });
  }

  for (const b of blockedDates) {
    cal.createEvent({
      id: b.id,
      start: b.startDate,
      end: b.endDate,
      summary: b.reason || "Bloqué",
      allDay: true,
    });
  }

  return new NextResponse(cal.toString(), {
    headers: {
      "Content-Type": "text/calendar; charset=utf-8",
      "Content-Disposition": `attachment; filename="${property.name}.ics"`,
    },
  });
}

```

### `src/app/api/calendar/sync/route.ts`

```typescript
import { NextRequest, NextResponse } from "next/server";
import { auth } from "@/lib/auth";
import { prisma } from "@/lib/prisma";
import { syncExternalCalendar } from "@/lib/ical-sync";

/**
 * POST /api/calendar/sync?calendarId=... — Trigger manual sync
 */
export async function POST(request: NextRequest) {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ error: "Non autorisé" }, { status: 401 });
  }

  const { searchParams } = new URL(request.url);
  const calendarId = searchParams.get("calendarId");

  if (!calendarId) {
    return NextResponse.json({ error: "calendarId requis" }, { status: 400 });
  }

  const calendar = await prisma.externalCalendar.findUnique({
    where: { id: calendarId },
    include: { property: { select: { userId: true } } },
  });

  if (!calendar || calendar.property.userId !== session.user.id) {
    return NextResponse.json({ error: "Calendrier introuvable" }, { status: 404 });
  }

  await syncExternalCalendar(calendarId);

  const updated = await prisma.externalCalendar.findUnique({
    where: { id: calendarId },
  });

  return NextResponse.json(updated);
}

```

### `src/app/api/stripe/checkout/route.ts`

```typescript
import { NextRequest, NextResponse } from "next/server";
import { auth } from "@/lib/auth";
import { prisma } from "@/lib/prisma";
import { stripe } from "@/lib/stripe";

export async function POST(request: NextRequest) {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ error: "Non autorisé" }, { status: 401 });
  }

  try {
    const body = await request.json();
    const { contractId, rentEntryId, amount, description } = body;

    if (!amount || amount <= 0) {
      return NextResponse.json({ error: "Montant invalide" }, { status: 400 });
    }

    // Get owner's Stripe connected account
    const user = await prisma.user.findUnique({
      where: { id: session.user.id },
      select: { stripeAccountId: true, stripeOnboarded: true },
    });

    if (!user?.stripeAccountId || !user.stripeOnboarded) {
      return NextResponse.json(
        { error: "Veuillez d'abord configurer votre compte Stripe" },
        { status: 400 }
      );
    }

    const baseUrl = process.env.NEXTAUTH_URL || process.env.NEXT_PUBLIC_APP_URL || "http://localhost:3000";

    const checkoutSession = await stripe.checkout.sessions.create({
      mode: "payment",
      line_items: [
        {
          price_data: {
            currency: "eur",
            unit_amount: Math.round(amount * 100),
            product_data: { name: description || "Paiement locatif" },
          },
          quantity: 1,
        },
      ],
      payment_intent_data: {
        application_fee_amount: 0,
        transfer_data: { destination: user.stripeAccountId },
      },
      success_url: `${baseUrl}/payments?success=true`,
      cancel_url: `${baseUrl}/payments?cancelled=true`,
      metadata: {
        contractId: contractId || "",
        rentEntryId: rentEntryId || "",
        userId: session.user.id,
      },
    });

    // Create pending payment record
    await prisma.payment.create({
      data: {
        contractId: contractId || null,
        rentEntryId: rentEntryId || null,
        amount,
        status: "PENDING",
        method: "STRIPE",
        stripeSessionId: checkoutSession.id,
      },
    });

    return NextResponse.json({ url: checkoutSession.url });
  } catch (error) {
    console.error("Checkout error:", error);
    return NextResponse.json({ error: "Une erreur interne est survenue" }, { status: 500 });
  }
}

```

### `src/app/api/stripe/onboard/route.ts`

```typescript
import { NextResponse } from "next/server";
import { auth } from "@/lib/auth";
import { prisma } from "@/lib/prisma";
import { stripe } from "@/lib/stripe";

export async function POST() {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ error: "Non autorisé" }, { status: 401 });
  }

  try {
    const user = await prisma.user.findUnique({
      where: { id: session.user.id },
      select: { stripeAccountId: true, email: true },
    });

    let accountId = user?.stripeAccountId;

    if (!accountId) {
      const account = await stripe.accounts.create({
        type: "standard",
        email: user?.email || undefined,
      });
      accountId = account.id;

      await prisma.user.update({
        where: { id: session.user.id },
        data: { stripeAccountId: accountId },
      });
    }

    const baseUrl = process.env.NEXTAUTH_URL || process.env.NEXT_PUBLIC_APP_URL || "http://localhost:3000";

    const accountLink = await stripe.accountLinks.create({
      account: accountId,
      refresh_url: `${baseUrl}/profile`,
      return_url: `${baseUrl}/profile?stripe=success`,
      type: "account_onboarding",
    });

    return NextResponse.json({ url: accountLink.url });
  } catch (error) {
    console.error("Stripe onboard error:", error);
    return NextResponse.json({ error: "Une erreur interne est survenue" }, { status: 500 });
  }
}

```

### `src/app/api/webhooks/stripe/route.ts`

```typescript
import { NextRequest, NextResponse } from "next/server";
import { prisma } from "@/lib/prisma";
import { stripe } from "@/lib/stripe";

export async function POST(request: NextRequest) {
  const body = await request.text();
  const sig = request.headers.get("stripe-signature");

  if (!sig) {
    return NextResponse.json({ error: "Missing signature" }, { status: 400 });
  }

  let event;
  try {
    event = stripe.webhooks.constructEvent(
      body,
      sig,
      process.env.STRIPE_WEBHOOK_SECRET!
    );
  } catch (err) {
    console.error("Webhook signature verification failed:", err);
    return NextResponse.json({ error: "Invalid signature" }, { status: 400 });
  }

  try {
    switch (event.type) {
      case "checkout.session.completed": {
        const session = event.data.object;
        const stripeSessionId = session.id;
        const paymentIntentId = typeof session.payment_intent === "string"
          ? session.payment_intent
          : session.payment_intent?.id;

        const payment = await prisma.payment.findFirst({
          where: { stripeSessionId },
        });

        if (payment) {
          await prisma.payment.update({
            where: { id: payment.id },
            data: {
              status: "COMPLETED",
              stripePaymentId: paymentIntentId || null,
              paidAt: new Date(),
            },
          });

          // Update rent entry if linked
          if (payment.rentEntryId) {
            const rentEntry = await prisma.rentEntry.findUnique({
              where: { id: payment.rentEntryId },
            });

            if (rentEntry) {
              const newPaid = rentEntry.amountPaid + payment.amount;
              const status = newPaid >= rentEntry.amountDue ? "PAID" : "PARTIAL";

              await prisma.rentEntry.update({
                where: { id: payment.rentEntryId },
                data: {
                  amountPaid: newPaid,
                  status,
                  paidAt: status === "PAID" ? new Date() : null,
                },
              });
            }
          }

          // Update contract status if needed
          if (payment.contractId) {
            const contract = await prisma.contract.findUnique({
              where: { id: payment.contractId },
            });
            if (contract?.status === "SIGNED") {
              await prisma.contract.update({
                where: { id: payment.contractId },
                data: { status: "CONFIRMED" },
              });
            }
          }
        }
        break;
      }

      case "payment_intent.payment_failed": {
        const paymentIntent = event.data.object;
        const failedPayment = await prisma.payment.findFirst({
          where: { stripePaymentId: paymentIntent.id },
        });

        if (failedPayment) {
          await prisma.payment.update({
            where: { id: failedPayment.id },
            data: { status: "FAILED" },
          });
        }
        break;
      }

      case "account.updated": {
        const account = event.data.object;
        if (account.charges_enabled && account.payouts_enabled) {
          await prisma.user.updateMany({
            where: { stripeAccountId: account.id },
            data: { stripeOnboarded: true },
          });
        }
        break;
      }
    }
  } catch (error) {
    console.error("Webhook processing error:", error);
  }

  return NextResponse.json({ received: true });
}

```

### `src/app/api/user/profile/route.ts`

```typescript
import { NextRequest, NextResponse } from "next/server";
import { prisma } from "@/lib/prisma";
import { auth } from "@/lib/auth";
import { generateToken } from "@/lib/auth-helpers";
import { sendEmail, verificationEmailHtml } from "@/lib/email";

export async function GET() {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ error: "Non autorisé" }, { status: 401 });
  }

  const user = await prisma.user.findUnique({
    where: { id: session.user.id },
    select: { id: true, name: true, email: true, phone: true, image: true, emailVerified: true, stripeAccountId: true, stripeOnboarded: true },
  });

  if (!user) {
    return NextResponse.json({ error: "Utilisateur introuvable" }, { status: 404 });
  }

  return NextResponse.json(user);
}

export async function PATCH(request: NextRequest) {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ error: "Non autorisé" }, { status: 401 });
  }

  try {
    const body = await request.json();
    const { name, phone, email } = body;

    const data: Record<string, string | null | Date> = {};

    if (name !== undefined) {
      if (!name.trim()) {
        return NextResponse.json({ error: "Le nom est requis" }, { status: 400 });
      }
      data.name = name.trim();
    }

    if (phone !== undefined) {
      data.phone = phone ? phone.trim() : null;
    }

    // Email change triggers re-verification
    if (email !== undefined) {
      const newEmail = email.toLowerCase().trim();
      if (!newEmail) {
        return NextResponse.json({ error: "L'email est requis" }, { status: 400 });
      }

      if (newEmail !== session.user.email) {
        // Check if email is already taken
        const existing = await prisma.user.findUnique({ where: { email: newEmail } });
        if (existing) {
          return NextResponse.json(
            { error: "Cette adresse email est déjà utilisée" },
            { status: 400 }
          );
        }

        data.email = newEmail;
        data.emailVerified = null as unknown as Date;

        // Send verification email for the new address
        const token = generateToken();
        const expires = new Date(Date.now() + 24 * 60 * 60 * 1000);

        await prisma.verificationToken.create({
          data: { identifier: newEmail, token, expires },
        });

        const baseUrl =
          process.env.NEXTAUTH_URL || process.env.NEXT_PUBLIC_APP_URL || "http://localhost:3000";
        const verifyUrl = `${baseUrl}/verify-email?token=${token}`;

        await sendEmail({
          to: newEmail,
          subject: "Vérifiez votre nouvelle adresse email — LS Immo",
          html: verificationEmailHtml(verifyUrl),
        });
      }
    }

    if (Object.keys(data).length === 0) {
      return NextResponse.json({ error: "Aucune modification" }, { status: 400 });
    }

    const updatedUser = await prisma.user.update({
      where: { id: session.user.id },
      data,
      select: { id: true, name: true, email: true, phone: true, image: true, emailVerified: true },
    });

    return NextResponse.json(updatedUser);
  } catch (error) {
    console.error("Profile update error:", error);
    return NextResponse.json({ error: "Une erreur interne est survenue" }, { status: 500 });
  }
}

```

### `src/__tests__/auth-helpers.test.ts`

```typescript
import { describe, it, expect, vi } from "vitest";

// Mock the auth module to prevent NextAuth import chain
vi.mock("@/lib/auth", () => ({
  auth: vi.fn().mockResolvedValue(null),
}));

const { hashPassword, verifyPassword, validatePassword, generateToken } =
  await import("@/lib/auth-helpers");

describe("validatePassword", () => {
  it("rejects passwords shorter than 8 characters", () => {
    const result = validatePassword("Ab1");
    expect(result.valid).toBe(false);
    expect(result.errors).toContain(
      "Le mot de passe doit contenir au moins 8 caractères."
    );
  });

  it("rejects passwords without uppercase letters", () => {
    const result = validatePassword("abcdefg1");
    expect(result.valid).toBe(false);
    expect(result.errors).toContain(
      "Le mot de passe doit contenir au moins une lettre majuscule."
    );
  });

  it("rejects passwords without lowercase letters", () => {
    const result = validatePassword("ABCDEFG1");
    expect(result.valid).toBe(false);
    expect(result.errors).toContain(
      "Le mot de passe doit contenir au moins une lettre minuscule."
    );
  });

  it("rejects passwords without digits", () => {
    const result = validatePassword("Abcdefgh");
    expect(result.valid).toBe(false);
    expect(result.errors).toContain(
      "Le mot de passe doit contenir au moins un chiffre."
    );
  });

  it("accepts valid passwords", () => {
    const result = validatePassword("SecureP4ss");
    expect(result.valid).toBe(true);
    expect(result.errors).toHaveLength(0);
  });

  it("returns multiple errors for very weak passwords", () => {
    const result = validatePassword("abc");
    expect(result.valid).toBe(false);
    expect(result.errors.length).toBeGreaterThanOrEqual(2);
  });
});

describe("hashPassword / verifyPassword", () => {
  it("hashes and verifies a password correctly", async () => {
    const password = "SecureP4ss";
    const hash = await hashPassword(password);

    expect(hash).not.toBe(password);
    expect(hash.startsWith("$2")).toBe(true); // bcrypt prefix

    const isValid = await verifyPassword(password, hash);
    expect(isValid).toBe(true);
  });

  it("rejects wrong password", async () => {
    const hash = await hashPassword("SecureP4ss");
    const isValid = await verifyPassword("WrongP4ss", hash);
    expect(isValid).toBe(false);
  });
});

describe("generateToken", () => {
  it("generates a UUID-format token", () => {
    const token = generateToken();
    expect(token).toMatch(
      /^[0-9a-f]{8}-[0-9a-f]{4}-4[0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$/i
    );
  });

  it("generates unique tokens", () => {
    const tokens = new Set(Array.from({ length: 100 }, () => generateToken()));
    expect(tokens.size).toBe(100);
  });
});

```

### `src/__tests__/ical-parser.test.ts`

```typescript
import { describe, it, expect } from "vitest";
import { parseIcal } from "@/lib/ical-sync";

const SAMPLE_ICAL = `BEGIN:VCALENDAR
VERSION:2.0
PRODID:-//Test//Test//EN
BEGIN:VEVENT
UID:event-1@test
DTSTART:20240615
DTEND:20240620
SUMMARY:Réservation Airbnb
END:VEVENT
BEGIN:VEVENT
UID:event-2@test
DTSTART:20240701T140000Z
DTEND:20240715T100000Z
SUMMARY:Booking.com reservation
END:VEVENT
END:VCALENDAR`;

describe("parseIcal", () => {
  it("parses VEVENT blocks with date-only format", () => {
    const events = parseIcal(SAMPLE_ICAL);
    expect(events).toHaveLength(2);

    const first = events[0];
    expect(first.uid).toBe("event-1@test");
    expect(first.summary).toBe("Réservation Airbnb");
    expect(first.start).toEqual(new Date("2024-06-15"));
    expect(first.end).toEqual(new Date("2024-06-20"));
  });

  it("parses VEVENT blocks with datetime format (UTC)", () => {
    const events = parseIcal(SAMPLE_ICAL);
    const second = events[1];
    expect(second.uid).toBe("event-2@test");
    expect(second.summary).toBe("Booking.com reservation");
    expect(second.start).toEqual(new Date("2024-07-01T14:00:00Z"));
    expect(second.end).toEqual(new Date("2024-07-15T10:00:00Z"));
  });

  it("returns empty array for empty input", () => {
    const events = parseIcal("");
    expect(events).toHaveLength(0);
  });

  it("returns empty array for calendar with no events", () => {
    const ical = `BEGIN:VCALENDAR
VERSION:2.0
PRODID:-//Test//Test//EN
END:VCALENDAR`;
    const events = parseIcal(ical);
    expect(events).toHaveLength(0);
  });

  it("skips events without valid start/end dates", () => {
    const ical = `BEGIN:VCALENDAR
BEGIN:VEVENT
UID:incomplete@test
SUMMARY:No dates
END:VEVENT
BEGIN:VEVENT
UID:valid@test
DTSTART:20240801
DTEND:20240805
SUMMARY:Valid
END:VEVENT
END:VCALENDAR`;
    const events = parseIcal(ical);
    expect(events).toHaveLength(1);
    expect(events[0].uid).toBe("valid@test");
  });

  it("handles datetime without Z suffix", () => {
    const ical = `BEGIN:VCALENDAR
BEGIN:VEVENT
UID:local@test
DTSTART:20240301T090000
DTEND:20240301T180000
SUMMARY:Local time event
END:VEVENT
END:VCALENDAR`;
    const events = parseIcal(ical);
    expect(events).toHaveLength(1);
    expect(events[0].start.getFullYear()).toBe(2024);
    expect(events[0].start.getMonth()).toBe(2); // March = 2
    expect(events[0].start.getDate()).toBe(1);
  });

  it("handles Windows-style line endings (CRLF)", () => {
    const ical =
      "BEGIN:VCALENDAR\r\nBEGIN:VEVENT\r\nUID:crlf@test\r\nDTSTART:20240101\r\nDTEND:20240102\r\nSUMMARY:CRLF Test\r\nEND:VEVENT\r\nEND:VCALENDAR";
    const events = parseIcal(ical);
    expect(events).toHaveLength(1);
    expect(events[0].summary).toBe("CRLF Test");
  });
});

```

### `src/__tests__/pdf-signature.test.ts`

```typescript
import { describe, it, expect } from "vitest";
import { PDFDocument } from "pdf-lib";
import { hashDocument, embedSignatureInPdf } from "@/lib/pdf-signature";

async function createTestPdf(): Promise<Uint8Array> {
  const doc = await PDFDocument.create();
  const page = doc.addPage([595, 842]); // A4
  page.drawText("Test Document", { x: 50, y: 750, size: 20 });
  return new Uint8Array(await doc.save());
}

// Create a minimal 1x1 white PNG as a data URL
function createTestSignatureDataUrl(): string {
  // 1x1 white pixel PNG in base64
  const png =
    "iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mP8/5+hHgAHggJ/PchI7wAAAABJRU5ErkJggg==";
  return `data:image/png;base64,${png}`;
}

describe("hashDocument", () => {
  it("returns a 64-character hex SHA-256 hash", async () => {
    const pdf = await createTestPdf();
    const hash = hashDocument(pdf);
    expect(hash).toMatch(/^[0-9a-f]{64}$/);
  });

  it("returns the same hash for the same input", async () => {
    const pdf = await createTestPdf();
    const hash1 = hashDocument(pdf);
    const hash2 = hashDocument(pdf);
    expect(hash1).toBe(hash2);
  });

  it("returns different hashes for different inputs", async () => {
    const pdf1 = await createTestPdf();

    const doc2 = await PDFDocument.create();
    doc2.addPage([595, 842]);
    const pdf2 = new Uint8Array(await doc2.save());

    const hash1 = hashDocument(pdf1);
    const hash2 = hashDocument(pdf2);
    expect(hash1).not.toBe(hash2);
  });
});

describe("embedSignatureInPdf", () => {
  it("embeds a signature and returns valid PDF bytes + hash", async () => {
    const pdfBytes = await createTestPdf();
    const signatureDataUrl = createTestSignatureDataUrl();

    const { signedPdfBytes, documentHash } = await embedSignatureInPdf(
      pdfBytes,
      signatureDataUrl,
      "Jean Dupont"
    );

    // Result should be valid PDF bytes
    expect(signedPdfBytes).toBeInstanceOf(Uint8Array);
    expect(signedPdfBytes.length).toBeGreaterThan(pdfBytes.length);

    // Hash should be 64-char hex
    expect(documentHash).toMatch(/^[0-9a-f]{64}$/);

    // Signed PDF should be loadable
    const doc = await PDFDocument.load(signedPdfBytes);
    expect(doc.getPageCount()).toBe(1);
  });

  it("produces different hashes for different signatures", async () => {
    const pdfBytes = await createTestPdf();
    const sig = createTestSignatureDataUrl();

    const result1 = await embedSignatureInPdf(pdfBytes, sig, "Alice");
    const result2 = await embedSignatureInPdf(pdfBytes, sig, "Bob");

    // Different signer names should produce different PDFs/hashes
    expect(result1.documentHash).not.toBe(result2.documentHash);
  });
});

```
---

## Reconstruction Instructions

1. Create a new Next.js project: `npx create-next-app@latest ls-immo --typescript --tailwind --app --src-dir`
2. Replace `package.json` dependencies and scripts with the versions above
3. Run `npm install` to install all dependencies
4. Create every file listed above at its exact path
5. Copy `.env.example` to `.env` and fill in real values
6. Run `npx prisma generate` to generate the Prisma client
7. Run `npx prisma db push` to create database tables
8. Optionally run `npm run db:seed` to seed test data
9. Run `npm run dev` to start the development server

### Key Architecture Notes

- **Prisma Client** is generated to `src/generated/prisma/` (configured in schema.prisma `output`)
- **Prisma config** uses `@prisma/adapter-pg` with a raw `pg.Pool` connection (see `prisma.config.ts`)
- **Auth middleware** protects all routes except `/api/auth`, `/sign`, `/api/webhooks`, and static assets
- **API routes** use `getServerSession` from next-auth for authentication and return JSON responses
- **PDF generation** uses `pdf-lib` (not @react-pdf/renderer which is only in package.json but not used for server PDFs)
- **Signature flow**: Owner creates contract/lease → system generates PDF → sends email with signing link → tenant signs via `/sign/[token]` → signature embedded in PDF → document stored in R2
- **Calendar sync**: External iCal URLs (Airbnb, Booking) are parsed with `node-ical` and stored as `CalendarEvent` records with `type: EXTERNAL`
- **Stripe Connect**: Owners onboard via Stripe Connect, tenants pay via Stripe Checkout sessions linked to the owner's connected account
- **All French UI**: The entire interface is in French (labels, messages, legal text, PDF content)
- **Soft delete**: Properties use `deletedAt` field for soft deletion
- **Document types**: Each document type (CONTRACT_PDF, LEASE_PDF, etc.) has dedicated generation logic in `src/lib/pdf-templates/`

### Environment Variables Required

| Variable | Description |
|----------|-------------|
| `DATABASE_URL` | PostgreSQL connection string |
| `NEXTAUTH_URL` | App URL for NextAuth (e.g., `http://localhost:3000`) |
| `AUTH_SECRET` | NextAuth secret for JWT signing |
| `GOOGLE_CLIENT_ID` | Google OAuth client ID |
| `GOOGLE_CLIENT_SECRET` | Google OAuth client secret |
| `STRIPE_SECRET_KEY` | Stripe secret API key |
| `STRIPE_PUBLISHABLE_KEY` | Stripe publishable key |
| `STRIPE_WEBHOOK_SECRET` | Stripe webhook endpoint secret |
| `R2_ENDPOINT` | Cloudflare R2 endpoint URL |
| `R2_ACCESS_KEY_ID` | R2 access key |
| `R2_SECRET_ACCESS_KEY` | R2 secret key |
| `R2_BUCKET_NAME` | R2 bucket name (default: `ls-immo`) |
| `RESEND_API_KEY` | Resend API key for emails |
| `EMAIL_FROM` | Sender email address |
| `NEXT_PUBLIC_APP_URL` | Public-facing app URL |
