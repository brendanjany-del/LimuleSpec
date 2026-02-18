# Rental Management SaaS

Application web SaaS pour gérer ses locations immobilières de A à Z — saisonnières et classiques.

## Vue d'ensemble

| | |
|---|---|
| **Schema** | spec-driven |
| **Statut** | 4/4 artifacts complétés |
| **Capabilities** | 9 |
| **Tasks** | 87 |

## Artifacts

| Document | Description |
|----------|-------------|
| [Proposal](Proposal) | Pourquoi ce projet, quelles fonctionnalités, quel impact |
| [Design](Design) | Architecture technique, stack, décisions |
| [Tasks](Tasks) | Liste des 87 tâches d'implémentation |

## Spécifications fonctionnelles

| Capability | Description |
|------------|-------------|
| [User Auth](Spec‐User‐Auth) | Authentification, inscription, OAuth, sessions |
| [Property Management](Spec‐Property‐Management) | Gestion des biens immobiliers |
| [Seasonal Rental](Spec‐Seasonal‐Rental) | Location saisonnière — contrats, signature, cycle de vie |
| [Classic Rental](Spec‐Classic‐Rental) | Location classique — bail, états des lieux, charges |
| [Deposit and Guarantee](Spec‐Deposit‐and‐Guarantee) | Cautions, dépôts, Visale/Locapass |
| [Payment Processing](Spec‐Payment‐Processing) | Paiements Stripe Connect |
| [Rent Management](Spec‐Rent‐Management) | Échéances, quittances, suivi |
| [Calendar and Sync](Spec‐Calendar‐and‐Sync) | Calendrier, blocage, iCal |
| [Document Generation](Spec‐Document‐Generation) | Génération PDF, templates |

## Stack technique

| Service | Choix | Coût |
|---------|-------|------|
| Framework | Next.js (App Router) | Gratuit |
| BDD | Supabase PostgreSQL | Free tier |
| Auth | NextAuth.js | Gratuit |
| Paiements | Stripe Connect | Commissions uniquement |
| Signature | SignaturePad.js + pdf-lib | Gratuit |
| Calendrier | FullCalendar + iCal | Gratuit |
| Stockage | Cloudflare R2 | Free tier (10 Go) |
| Emails | Resend | Free tier (3 000/mois) |
| Hébergement | Vercel | Free tier |
| Documentation | GitHub Wiki | Gratuit (repo public) |
