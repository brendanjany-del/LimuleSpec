## Context

Ce projet crée un noyau de signature électronique open-source from scratch. Il n'existe pas de codebase existante — tout est à construire. Le noyau doit être **modulaire** et **intégrable** : il sera consommé à la fois par son propre frontend minimal et par des plateformes tierces (LS-Immo, CRM, etc.) via API REST ou import direct des modules TypeScript.

Contraintes principales :
- Monorepo TypeScript pour partager les types entre core, API et frontend.
- Chaque module du core doit être utilisable indépendamment.
- Le frontend est volontairement minimaliste — c'est un démonstrateur, pas le produit final.
- Pas d'authentification utilisateur dans le core (déléguée aux plateformes intégratrices). Le core gère l'identité des signataires via tokens de session.

## Goals / Non-Goals

**Goals:**

- Fournir un core library TypeScript avec des modules découplés (PDF, placement, workflow, crypto, notifications).
- Exposer une API REST complète permettant l'intégration dans n'importe quelle plateforme.
- Livrer un frontend minimal fonctionnel couvrant les 3 modes de signature.
- Permettre l'intégration en iframe via des URLs de signature autonomes.
- Stocker les documents et métadonnées dans PostgreSQL avec stockage fichiers sur disque (ou S3-compatible).

**Non-Goals:**

- Pas d'authentification utilisateur / gestion de comptes (hors scope, délégué aux intégrateurs).
- Pas de conformité eIDAS QES complète dans la v1 (signature avancée d'abord, QES dans une itération future avec TSP).
- Pas d'éditeur PDF avancé (annotation, modification de contenu).
- Pas de gestion de templates de documents.
- Pas de mobile-native app.

## Decisions

### 1. Architecture monorepo avec packages séparés

**Choix** : Monorepo avec workspaces (pnpm workspaces ou Turborepo).

```
packages/
  core/              # Logique métier pure, zero framework dependency
    src/
      pdf/           # pdf-document-management
      placement/     # signature-field-placement
      workflow/      # signing-workflow
      crypto/        # crypto-signing
      notification/  # notification-service
      link/          # shareable-signing-link
  api/               # Express/Fastify REST API
  frontend/          # React SPA minimal
  shared/            # Types partagés, DTOs
```

**Rationale** : Le core reste une lib pure TypeScript sans dépendance framework. L'API et le frontend sont des consommateurs du core, tout comme le seront les plateformes tierces. Cela garantit que le core est réellement intégrable par import.

**Alternative rejetée** : Repos séparés — trop de friction pour le développement initial et la cohérence des types.

### 2. Stack technique

**Choix** :
- **Runtime** : Node.js + TypeScript strict
- **API** : Fastify (performant, schema validation native avec JSON Schema)
- **Frontend** : React + Vite (léger, rapide)
- **PDF** : pdf-lib (manipulation PDF pure JS, pas de dépendance native)
- **BDD** : PostgreSQL + Prisma ORM
- **Stockage fichiers** : Abstraction avec implémentation locale (disque) et S3-compatible
- **Email** : Nodemailer avec transport configurable (SMTP, désactivable)

**Alternative rejetée** : Next.js fullstack — trop couplé, le core doit être framework-agnostic. Express — moins performant que Fastify et pas de validation de schema intégrée.

### 3. Gestion de l'identité des signataires sans authentification

**Choix** : Chaque session de signature génère des **tokens JWT à usage unique** par signataire. Le signataire accède à sa page de signature via un lien contenant ce token. Pas de compte utilisateur, pas de login.

**Rationale** : Le core ne doit pas imposer un système d'auth. Les plateformes intégratrices (LS-Immo, CRM) gèrent leurs propres utilisateurs. Le core identifie les signataires par email + token de session.

**Alternative rejetée** : OAuth2 / sessions classiques — ajouterait une dépendance auth que les intégrateurs n'utilisent pas.

### 4. Workflow de signature avec machine à états

**Choix** : Chaque session de signature suit une machine à états explicite :

```
DRAFT → PENDING → IN_PROGRESS → COMPLETED
                → EXPIRED
                → CANCELLED
```

Chaque signataire a son propre statut : `PENDING → SIGNED | DECLINED`.

**Rationale** : Machine à états explicite = comportement prévisible, facile à persister, facile à requêter.

### 5. Signature cryptographique

**Choix** : Signature PDF avec pdf-lib + node:crypto.
- Hash SHA-256 du document.
- Signature numérique RSA ou ECDSA (clé serveur dans la v1).
- Ajout d'un champ signature PDF standard (PAdES-compatible baseline).
- Horodatage serveur (TSA externe en v2 pour conformité).

**Rationale** : La v1 fournit une signature avancée vérifiable. La conformité QES complète (certificat qualifié, TSA qualifié) est un non-goal explicite de la v1 mais l'architecture le permet.

**Alternative rejetée** : Intégration HSM dès la v1 — complexité prématurée.

### 6. Stockage des documents

**Choix** : Interface `StorageProvider` avec deux implémentations :
- `LocalStorageProvider` : stockage sur disque (développement, self-host simple).
- `S3StorageProvider` : stockage S3-compatible (production, scalable).

Les métadonnées (sessions, signataires, statuts) sont dans PostgreSQL. Les fichiers PDF (original + signé) sont dans le storage provider. La BDD stocke uniquement les références (paths/keys).

### 7. Frontend minimal avec 3 modes

**Choix** : SPA React avec 3 routes principales :
- `/send` : Upload PDF → placement des champs → saisie des emails → envoi.
- `/sign/:sessionId` : Vue de signature directe (multi-signataires présents).
- `/s/:token` : Page de signature autonome (lien partageable, intégrable en iframe).

La page `/s/:token` est conçue pour être embedable : pas de header/nav, responsive, `postMessage` API pour communiquer avec le parent iframe.

**Rationale** : Le mode iframe est critique pour l'intégration dans LS-Immo et autres plateformes.

## Risks / Trade-offs

**[Sécurité des tokens de signature]** → Les tokens JWT à usage unique expirent (TTL configurable, défaut 7 jours). Rate limiting sur l'API. Les tokens sont invalidés après signature. Le lien seul donne accès à la signature — acceptable car c'est le modèle standard (DocuSign, Yousign fonctionnent pareil).

**[Conformité juridique limitée en v1]** → La v1 fournit une signature avancée (pas qualifiée). Suffisant pour beaucoup d'usages mais pas pour les actes notariés par exemple. Mitigation : l'architecture est extensible pour ajouter TSA qualifié et certificats qualifiés en v2.

**[Performance PDF côté serveur]** → pdf-lib est pur JS, potentiellement lent sur de gros PDF. Mitigation : limiter la taille des uploads (configurable, défaut 20MB), traitement async pour les gros fichiers.

**[Dépendance à PostgreSQL]** → Imposer Postgres peut freiner certaines intégrations. Mitigation : Prisma permet de supporter d'autres BDD à terme, mais Postgres est le seul target v1.

**[Complexité du placement visuel]** → Le drag & drop de champs sur un rendu PDF est techniquement complexe côté frontend. Mitigation : utiliser react-pdf pour le rendu et une lib de drag & drop éprouvée (dnd-kit). C'est le composant frontend le plus risqué.
