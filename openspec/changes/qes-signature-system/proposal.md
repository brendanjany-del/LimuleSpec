## Why

Les solutions de signature électronique existantes (DocuSign, Yousign, etc.) sont propriétaires, coûteuses à l'échelle, et difficilement intégrables en profondeur dans des plateformes métier custom. Il manque un **noyau open-source de signature électronique** modulaire, pensé dès le départ pour être embarqué dans n'importe quelle application (CRM, plateforme immobilière type LS-Immo, ERP, etc.) tout en restant utilisable de manière autonome via un frontend minimal.

## What Changes

- Création d'un **core library** (SDK/API) de signature électronique exposant des modules indépendants : gestion de documents PDF, positionnement de champs de signature, workflow multi-signataires, application cryptographique des signatures.
- Création d'une **API REST** servant de couche d'intégration pour toute application tierce.
- Création d'un **frontend minimal** couvrant trois modes d'utilisation :
  - **Envoi pour signature** : import PDF, positionnement des zones de signature via drag & drop, envoi par email aux destinataires.
  - **Signature directe multi-signataires** : signature immédiate par plusieurs personnes présentes (pas d'envoi mail).
  - **Signature par lien partagé** : génération d'une URL de signature intégrable dans un iframe ou partageable, sans notification email.
- Système de **notification configurable** (email par défaut, désactivable pour les intégrations headless).
- Architecture en **modules découplés** pour permettre l'intégration sélective dans des plateformes tierces.

## Capabilities

### New Capabilities

- `pdf-document-management`: Import, stockage et manipulation de documents PDF (upload, preview, metadata extraction).
- `signature-field-placement`: Positionnement visuel des champs de signature sur un document PDF (coordonnées, pages, assignation aux signataires).
- `signing-workflow`: Orchestration du processus de signature multi-signataires (création de session, suivi du statut par signataire, ordre de signature optionnel, complétion).
- `crypto-signing`: Application cryptographique de la signature sur le PDF (hash du document, signature numérique, intégrité, horodatage).
- `notification-service`: Envoi de notifications aux signataires (email avec lien de signature), configurable et désactivable.
- `shareable-signing-link`: Génération d'URLs de signature autonomes, intégrables en iframe ou partageables directement sans dépendance email.
- `signing-api`: API REST exposant l'ensemble des fonctionnalités du core pour intégration dans des applications tierces (LS-Immo, CRM, etc.).
- `signing-frontend`: Interface utilisateur minimale couvrant les trois modes de signature (envoi, direct, lien partagé).

### Modified Capabilities

_Aucune — projet nouveau, pas de capabilities existantes._

## Impact

- **Nouveau projet complet** : core library, API REST, frontend.
- **Stack technique** : Node.js/TypeScript (backend), React ou équivalent (frontend), PostgreSQL (persistence), bibliothèque PDF (pdf-lib ou équivalent).
- **Dépendances externes** : service SMTP pour les notifications email, librairie crypto pour les signatures numériques.
- **Intégrations futures** : le core et l'API sont conçus pour être consommés par LS-Immo, CRM, et toute plateforme métier via l'API REST ou en import direct des modules.
- **Sécurité** : manipulation de documents sensibles, signatures cryptographiques — nécessite une attention particulière à l'authentification, l'intégrité des documents, et le stockage sécurisé.
