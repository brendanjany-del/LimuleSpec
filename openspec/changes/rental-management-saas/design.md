## Context

Nouveau projet SaaS à destination des propriétaires bailleurs qui gèrent eux-mêmes leurs locations. Aucune base de code existante — tout est à construire. L'application doit couvrir deux modes de location (saisonnière et classique) avec des workflows distincts mais un socle technique commun (auth, biens, paiements, documents). Le marché cible est français, ce qui implique des contraintes juridiques spécifiques (loi ALUR, loi Hoguet, RGPD) et des intégrations locales (Visale, Locapass).

## Goals / Non-Goals

**Goals :**

- Fournir une application web responsive permettant de gérer l'intégralité du cycle de vie d'une location (saisonnière ou classique)
- Proposer la signature électronique des contrats et documents
- Permettre l'encaissement des paiements en ligne et la gestion des dépôts de garantie
- Offrir un calendrier avec synchronisation externe (iCal / Airbnb / Booking)
- Générer automatiquement les documents légaux (baux, quittances, états des lieux, courriers types)
- Respecter la réglementation française (RGPD, obligations légales du bailleur)

**Non-Goals :**

- Application mobile native (le responsive web suffit en V1)
- Gestion comptable complète ou déclaration fiscale
- Marketplace ou mise en relation propriétaire/locataire
- Gestion de copropriété
- Multi-pays (France uniquement en V1)
- Gestion de parc immobilier professionnel (agences, administrateurs de biens)

## Decisions

### 1. Stack technique : Next.js (App Router) + PostgreSQL

**Choix** : Next.js (React, App Router, Server Components) avec PostgreSQL.

**Pourquoi** :
- Next.js offre SSR/SSG, API routes intégrées, et un écosystème riche — un seul framework pour le front et le back
- PostgreSQL est robuste pour les données relationnelles complexes (biens, contrats, paiements, locataires)
- Prisma comme ORM pour le typage fort et les migrations

**Alternatives considérées** :
- SvelteKit : moins d'écosystème et de communauté pour les intégrations tierces
- Backend séparé (NestJS/Express) : complexité inutile en V1, Next.js API routes suffisent
- MongoDB : mal adapté aux relations complexes entre entités (biens ↔ contrats ↔ locataires ↔ paiements)

### 2. Authentification : NextAuth.js (Auth.js)

**Choix** : NextAuth.js avec providers email/password + OAuth (Google).

**Pourquoi** :
- Intégration native avec Next.js
- Gestion des sessions, JWT, et refresh tokens incluse
- Extensible pour ajouter d'autres providers plus tard

**Alternatives considérées** :
- Clerk / Auth0 : coût récurrent, dépendance externe pour une fonctionnalité critique
- Supabase Auth : lierait trop à l'écosystème Supabase

### 3. Paiements : Stripe Connect

**Choix** : Stripe Connect en mode "destination charges".

**Pourquoi** :
- Permet au propriétaire de recevoir les paiements directement sur son compte Stripe connecté
- Gère l'empreinte CB (Setup Intents) pour les cautions saisonnières
- Gère les virements programmés pour les loyers
- Conformité PCI DSS déléguée à Stripe
- API mature et bien documentée

**Alternatives considérées** :
- GoCardless : meilleur pour les prélèvements SEPA mais pas d'empreinte CB ni de paiement par carte
- Mangopay : plus complexe à intégrer, orienté marketplace
- Approche hybride Stripe + GoCardless : complexité non justifiée en V1

### 4. Signature électronique : solution intégrée gratuite

**Choix** : Signature intégrée à l'application — SignaturePad.js pour la capture de signature manuscrite + pdf-lib pour l'incrustation dans le PDF + workflow de consentement par email (lien unique avec token, horodatage, IP, hash du document).

**Pourquoi** :
- **Zéro coût** : pas de service tiers payant à l'acte
- En droit français, la signature électronique "simple" (article 1367 du Code civil) est recevable — il suffit de prouver l'identité du signataire et l'intégrité du document
- Le workflow (email + lien unique + capture signature + horodatage + hash SHA-256 du PDF) constitue un faisceau de preuves suffisant pour les baux et contrats de location
- Conservé en interne : le "dossier de preuve" (signature, IP, timestamp, hash) est stocké en base

**Alternatives considérées** :
- Yousign : solution française eIDAS conforme, mais coût par signature (à partir de ~3€/signature) — non viable pour un SaaS gratuit/low-cost avec beaucoup de contrats saisonniers
- DocuSign / HelloSign : encore plus cher, overkill
- **Évolution possible** : proposer Yousign en option premium pour les utilisateurs qui veulent une signature électronique avancée (eIDAS), tout en gardant la solution intégrée gratuite par défaut

### 5. Calendrier et synchronisation : FullCalendar + iCal

**Choix** : FullCalendar (composant React) pour l'affichage, import/export iCal pour la synchronisation.

**Pourquoi** :
- FullCalendar est le standard de facto pour les calendriers interactifs en React
- Le format iCal est universel — Airbnb, Booking, Abritel exportent tous en iCal
- Synchronisation par polling périodique (cron) des URLs iCal externes

**Alternatives considérées** :
- API directe Airbnb/Booking : pas d'API publique pour les petits acteurs, iCal est le standard
- Développement calendrier custom : effort disproportionné

### 6. Génération de documents : Templates MDX → PDF

**Choix** : Templates MDX/HTML avec variables dynamiques, convertis en PDF via Puppeteer ou react-pdf.

**Pourquoi** :
- Flexibilité totale sur le rendu (mise en page, logo, mentions légales)
- Les templates sont versionnables et personnalisables par l'utilisateur
- react-pdf pour les documents simples (quittances), Puppeteer pour les documents complexes (baux, états des lieux)

**Alternatives considérées** :
- Librairies DOCX (docxtemplater) : moins de contrôle sur le rendu, format moins universel que le PDF
- Service tiers (Carbone.io) : dépendance externe pour une fonctionnalité core

### 7. Architecture de la base de données : multi-tenant par colonne

**Choix** : Multi-tenant logique avec colonne `userId` sur chaque table principale.

**Pourquoi** :
- Simple à implémenter et à maintenir en V1
- Un seul schéma de base de données
- Les requêtes sont filtrées automatiquement par middleware Prisma

**Alternatives considérées** :
- Schema par tenant (PostgreSQL schemas) : overhead opérationnel non justifié pour la V1
- Base de données par tenant : idem, trop complexe

### 8. Stockage de fichiers : Cloudflare R2

**Choix** : Cloudflare R2 pour les documents (contrats signés, photos état des lieux, pièces jointes).

**Pourquoi** :
- **Free tier généreux** : 10 Go de stockage + 10 millions de requêtes de lecture/mois gratuites
- **Zéro frais d'egress** (contrairement à AWS S3 qui facture la bande passante sortante)
- API compatible S3 — migration facile vers un autre provider si besoin
- Pré-signed URLs pour l'accès sécurisé aux documents

**Alternatives considérées** :
- AWS S3 : frais d'egress qui peuvent grimper rapidement
- Supabase Storage : free tier plus limité (1 Go), mais option viable si on utilise déjà Supabase pour la BDD

## Risks / Trade-offs

- **Complexité juridique des documents** → Travailler avec un juriste pour valider les templates de contrats (bail, conditions générales saisonnières). Les templates seront fournis "à titre indicatif" en V1 avec disclaimer.

- **Dépendance à Stripe pour les paiements** → Stripe est mature et fiable, mais représente un point de défaillance unique. Mitigation : architecture avec couche d'abstraction paiement pour faciliter l'ajout d'un provider alternatif plus tard.

- **Synchronisation iCal non temps-réel** → Le polling iCal introduit un délai (5-15 min). Risque de double réservation. Mitigation : afficher clairement la date de dernière synchronisation, permettre un refresh manuel, avertir l'utilisateur des limites.

- **Valeur juridique de la signature intégrée** → La signature électronique simple est recevable en France mais offre une force probante inférieure à une signature avancée (eIDAS). Mitigation : conserver un dossier de preuve complet (hash SHA-256 du document, horodatage, adresse IP, capture de signature) et proposer une option Yousign en upgrade pour les utilisateurs qui veulent une garantie juridique renforcée.

- **RGPD et données sensibles** → Données personnelles des locataires, pièces d'identité, coordonnées bancaires. Mitigation : chiffrement au repos, politique de rétention, droit à l'effacement implémenté dès la V1, pas de stockage de données CB (délégué à Stripe).

- **Scope ambitieux pour une V1** → 9 capabilities identifiées. Risque de livrer un produit incomplet. Mitigation : prioriser un MVP fonctionnel par mode de location. Saisonnière d'abord (contrat + paiement + calendrier), puis classique (bail + quittances + états des lieux).

## Migration Plan

Nouveau projet — pas de migration de données existantes.

**Déploiement (maximiser les free tiers)** :
1. Vercel pour le frontend/backend Next.js (free tier : 100 Go de bande passante/mois)
2. Supabase PostgreSQL (free tier : 500 Mo, 2 projets)
3. Cloudflare R2 pour le stockage de documents (free tier : 10 Go)
4. Resend pour les emails transactionnels (free tier : 3 000 emails/mois)
5. Variables d'environnement pour les clés API (Stripe)
6. CI/CD via GitHub Actions (free pour les repos publics, 2 000 min/mois pour les privés)

**Rollback** : déploiements immutables via Vercel, rollback en un clic sur la version précédente.

## Open Questions

- **Pricing model** : freemium (1 bien gratuit) ? Abonnement mensuel par bien ? À définir.
- **Visale / Locapass** : intégration réelle (API) ou simplement mode opératoire avec liens et rappels ? À confirmer selon la disponibilité des APIs.
- **État des lieux** : formulaire interactif avec photos intégrées ou simple document PDF à remplir ? Impact sur la complexité du frontend.
- **Emails transactionnels** : Resend retenu (free tier 3 000/mois). Suffisant pour démarrer, à réévaluer si le volume augmente.
- **Internationalisation future** : structurer le code pour i18n dès le départ ou uniquement en français ?
