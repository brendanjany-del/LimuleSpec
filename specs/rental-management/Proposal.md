## Why

Les propriétaires qui gèrent eux-mêmes leurs locations (saisonnières ou classiques) jonglent entre de multiples outils : tableurs, documents Word, virements manuels, courriers papier. Il n'existe pas de solution simple et unifiée, en mode SaaS, qui couvre l'intégralité du cycle de vie d'une location — du contrat au paiement, en passant par les états des lieux et le calendrier. L'objectif est de fournir une application web tout-en-un permettant de gérer ses biens et ses locataires de A à Z, quel que soit le mode de location.

## What Changes

- **Gestion multi-biens** : l'utilisateur authentifié peut créer et sélectionner ses biens immobiliers, chacun associé à un mode de location (saisonnière, classique meublée, classique non meublée)
- **Fiche bien** : gestion de l'adresse, du mobilier et accessoires (pour les biens meublés ou saisonniers), des caractéristiques du logement
- **Location saisonnière** :
  - Contrats de location courte durée avec signature électronique (dates, nombre de personnes, prix, ménage, conditions et annexes personnalisables)
  - Gestion de la caution / dépôt de garantie / empreinte de carte bancaire
  - Encaissement des paiements locataires et récupération des fonds
  - Calendrier de réservation avec blocage de dates et synchronisation externe (Airbnb, Booking, etc. via iCal)
- **Location classique** :
  - Bail de location avec signature électronique
  - États des lieux d'entrée et de sortie
  - Gestion des avis d'échéance et quittances de loyer
  - Gestion des charges locatives
  - Dépôt de garantie sécurisé (non récupérable par le bailleur sauf fin de location avec retenue justifiée)
  - Support Visale / Locapass : mode opératoire, liens utiles, rappels
  - Encaissement des loyers
  - Génération de courriers / lettres types (mise en demeure, congé, régularisation de charges, etc.)
- **Authentification utilisateur** : inscription, connexion, gestion de compte

## Capabilities

### New Capabilities

- `user-auth` : authentification, inscription et gestion de compte utilisateur
- `property-management` : création, sélection et configuration des biens immobiliers (adresse, type de location, mobilier/accessoires)
- `seasonal-rental` : gestion complète d'une location saisonnière — contrats, conditions personnalisables, signature électronique
- `classic-rental` : gestion complète d'une location classique — bail, états des lieux, charges, courriers types
- `deposit-and-guarantee` : gestion des cautions, dépôts de garantie, empreintes CB, et dispositifs Visale/Locapass
- `payment-processing` : encaissement des paiements locataires (loyers, séjours) et récupération des fonds par le propriétaire
- `rent-management` : avis d'échéance, quittances de loyer, suivi des paiements récurrents
- `calendar-and-sync` : calendrier de réservation, blocage de dates, synchronisation iCal avec plateformes externes (Airbnb, Booking)
- `document-generation` : génération de contrats, courriers types et lettres avec modèles personnalisables

### Modified Capabilities

_Aucune — il s'agit d'un nouveau projet._

## Impact

- **Frontend** : application web SaaS (responsive) avec dashboard multi-biens, vues calendrier, formulaires de contrats et états des lieux
- **Backend / API** : API REST ou GraphQL pour la gestion des biens, locataires, contrats, paiements, documents
- **Services tiers** :
  - Signature électronique (ex : Yousign, DocuSign ou équivalent)
  - Paiement en ligne (ex : Stripe, GoCardless) pour encaissement et empreinte CB
  - Synchronisation calendrier via iCal
  - Visale / Locapass : intégration des liens et informations
- **Base de données** : modèle relationnel couvrant biens, locataires, contrats, paiements, documents, réservations
- **Sécurité** : données personnelles sensibles (RGPD), données bancaires (PCI DSS via provider), gestion des rôles
- **Infrastructure** : hébergement cloud, stockage de documents, envoi d'emails transactionnels
