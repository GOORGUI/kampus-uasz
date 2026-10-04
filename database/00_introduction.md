# Introduction à l'architecture de données — Campus Connect

Ce dossier contient l'architecture de données officielle de Campus Connect.

Les modèles décrits ici sont **dérivés directement** des documents de conception :

- `01_vision.md` — Vision stratégique
- `03_personas.md` — Profils utilisateur
- `04_parcours_utilisateur.md` — Flux utilisateur
- `05_regles_metier.md` — Règles métier officielles

Cette architecture constitue la **référence technique de haut niveau** du projet. La véritable source de vérité reste `05_regles_metier.md`, car c'est le métier qui pilote la technique.

---

## Principes fondamentaux

### Architecture globale

```
Supabase Auth (authentification)
        ↓
Campus Connect (métier)
        ↓
       PostgreSQL
```

### Principes clés

- **PostgreSQL** comme base de données principale
- **Supabase Auth** pour l'authentification (séparé du modèle métier)
- **UUID** comme identifiant unique partout
- **Soft delete** par défaut (jamais de suppression physique)
- **Audit obligatoire** pour les actions sensibles
- **Row Level Security** pour les permissions
- **Indexation stratégique** pour la performance
- **Séparation claire** entre authentification et données métier

### Sécurité

- Toutes les données personnelles restent privées
- Aucune URL publique pour les documents sensibles
- Les justificatifs ne sont accessibles que par l'admin autorisé
- Les emails ne sortent jamais de l'espace d'authentification
- Les messages privés ne sont visibles que par les participants

---

## Convention de nommage

### Tables

- **Pluriel** en minuscules
- **snake_case**

**Exemples** : `users`, `profiles`, `matches`, `messages`, `blocks`, `reports`, `verifications`, `admin_roles`, `audit_logs`, `universities`

### Colonnes

- **snake_case** en minuscules
- Pas d'abréviations inutiles

**Exemples** : `id`, `user_id`, `created_at`, `updated_at`, `deleted_at`, `suspended_until`, `email`, `first_name`

### Identifiants étrangers

- Format : `{table_singulier}_id`

**Exemples** : `user_id`, `profile_id`, `university_id`

---

## Identifiants

### UUID partout

Toutes les tables principales utilisent **UUID v4** comme clé primaire.

**Avantages** :
- ✅ Sécurité (non séquentiel, non prévisible)
- ✅ URLs non énumérables
- ✅ Compatible Supabase
- ✅ Évolutivité (distribution et sharding faciles)
- ✅ Pas de collisions

### Identifiants composites

Pour les relations N-N (paires, triplets), utiliser des contraintes UNIQUE sur les colonnes concernées.

**Exemple** : Un utilisateur ne peut bloquer qu'une fois un autre utilisateur.

---

## Suppression logique (Soft Delete)

### Principe

Aucune suppression physique par défaut. Toutes les suppressions sont **logiques**.

### Convention

Chaque table sensible possède une colonne `deleted_at` (TIMESTAMP NULL).

- `deleted_at IS NULL` → enregistrement actif
- `deleted_at IS NOT NULL` → enregistrement supprimé

### Avantages

- ✅ Audit complet préservé
- ✅ Récupération possible
- ✅ Historique accessible
- ✅ Conformité RGPD (anonymisation, pas suppression définitive)
- ✅ Analytics conservées

### Implémentation

Les composants techniques de la plateforme doivent tenir compte des enregistrements supprimés logiquement. Les modalités d'implémentation seront définies dans `schema.sql` et `row_level_security.md`.

---

## Audit et traçabilité

### Principe

Toute action sensible doit être enregistrée avec :
- **Qui** a fait l'action (admin_id ou user_id)
- **Quoi** a changé (resource_type)
- **Quand** (timestamp)
- **Avant/Après** (old_value, new_value)

### Actions tracées obligatoirement

- ✅ Approbation / rejet de vérification (ACC-003)
- ✅ Modification de données sensibles (PROF-002)
- ✅ Suspension / réactivation de compte (SUSPEND-001, SUSPEND-002)
- ✅ Suppression de compte (DELETE-002)
- ✅ Accès admin aux données sensibles (ADM-002, ADM-003)
- ✅ Signalements approuvés / rejetés (REPORT-003)
- ✅ Actions de modération (AUDIT-001, AUDIT-002, AUDIT-003, AUDIT-004)

### Implémentation

Une table `audit_logs` centralisée enregistre toutes ces actions. La structure détaillée sera définie dans `schema.sql`.

---

## États métier

### États utilisateur (`users.status`)

Référence directe : `05_regles_metier.md` — Section "États utilisateur"

| État | Description | Accès plateforme |
|--------|-------------|-----------------|
| `pending_email` | E-mail non confirmé | ❌ Non |
| `pending_verification` | Justificatif en attente de validation | ❌ Non |
| `verified` | Vérifié et autorisé | ✅ Oui (si profil actif) |
| `suspended` | Suspendu temporairement ou définitivement | ❌ Non |
| `deletion_pending` | Suppression demandée (délai RGPD 30j) | ❌ Non |
| `deleted` | Compte anonymisé (soft delete) | ❌ Non |

### États profil (`profiles.status`)

Référence directe : `05_regles_metier.md` — Section "États profil"

| État | Visible | Likeable | Actif |
|--------|---------|----------|-------|
| `draft` | ❌ | ❌ | ❌ |
| `pending_review` | ❌ | ❌ | ❌ |
| `active` | ✅ | ✅ | ✅ |
| `hidden` | ❌ | ❌ | ✅ |
| `deleted` | ❌ | ❌ | ❌ |

### Transitions d'état autorisées

**Utilisateur** :
```
pending_email → pending_verification → verified → {suspended, deletion_pending} → deleted
```

**Profil** :
```
draft → pending_review → active ↔ hidden → deleted
```

Les règles de transition sont définies dans `05_regles_metier.md`.

---

## Séparation Auth / Métier

### Architecture Supabase

```
Authentification (Supabase Auth)
        ↓
      auth.users
(emails, passwords, tokens JWT)
        ↓
Métier (Campus Connect)
        ↓
      users
(profil métier, statuts, université)
        ↓
      profiles
(données sociales publiques)
```

### Responsabilités

- **`auth.users`** (Supabase)
  - Authentification et autorisation
  - Gestion des sessions
  - Encryption des mots de passe

- **`users`** (notre schéma)
  - Profil métier de l'utilisateur
  - Statut de vérification
  - Université d'appartenance
  - Informations de suspension

- **`profiles`** (notre schéma)
  - Données publiques/sociales
  - Profil visible (ou non) aux autres utilisateurs

### Avantage de cette séparation

- Authentification gérée par Supabase
- Données métier gérées par notre application
- Chacun peut évoluer indépendamment

---

## Multi-universités

### Contexte

Campus Connect démarre avec l'**UASZ** au MVP, mais l'architecture doit supporter plusieurs universités à terme.

**Références métier** :
- UNIV-001 : Un utilisateur appartient à une seule université active
- UNIV-002 : Découverte limitée à la même université au MVP
- UNIV-003 : Universités partenaires en v2.0

### Modèle de données

```
universities (1)
    ↓ (1-N)
users (N)
    ↓ (1-1)
profiles (1)
```

Chaque utilisateur appartient à **exactement une université**, déterminée lors de la vérification du justificatif.

### Université : dimension de sécurité

L'université n'est pas un simple champ optionnel. C'est une **dimension de sécurité, de segmentation et de gouvernance**.

- **Sécurité** : vérification d'un vrai contexte universitaire
- **Segmentation** : communautés isolées au MVP
- **Gouvernance** : meilleurs outils de modération
- **Évolution** : préparation des partenariats futurs

### MVP vs v2.0

**MVP** : découverte limitée à l'université de l'utilisateur

**v2.0** : choix de visibilité dans universités partenaires (opt-in)

---

## Performance et indexation

### Stratégie générale

Les colonnes suivantes devront être indexées pour assurer une bonne performance :

- **Clés étrangères** : `user_id`, `profile_id`, `university_id`, `match_id`
- **Filtres fréquents** : `status`, `deleted_at`
- **Tri courant** : `created_at`
- **Découverte** : combinaisons `(university_id, status, deleted_at)`

### Principes

- Indexer avant de requêter, pas après
- Préférer les index composites pour les requêtes multi-colonnes
- Monitorer les performances avec des outils de profiling
- Réévaluer les index régulièrement

### Implémentation

La définition détaillée de tous les index sera dans `schema.sql`.

---

## Entités principales

### Vue globale des 12 tables cœur

| Table | Responsabilité |
|--------|-----------------|
| `universities` | Universités partenaires |
| `users` | Comptes vérifiés et authentifiés |
| `profiles` | Profils sociaux publics |
| `profile_photos` | Photos de profil (1 seule au MVP) |
| `likes` | Actions "j'aime" entre utilisateurs |
| `matches` | Matchs mutuels (résultant de likes réciproques) |
| `messages` | Messages privés entre matched users |
| `blocks` | Blocages réciproques |
| `reports` | Signalements d'utilisateurs |
| `verifications` | Justificatifs étudiants et leur statut |
| `admin_roles` | Rôles et permissions administrateur |
| `audit_logs` | Historique de toutes les actions sensibles |

### Dépendances entre entités

```
universities
    ↓
users
    ├→ profiles
    │   ├→ profile_photos
    │   └→ likes
    ├→ likes
    ├→ matches
    │   └→ messages
    ├→ blocks
    ├→ reports
    ├→ verifications
    ├→ admin_roles
    └→ audit_logs
```

### Remarques

- `likes` et `matches` sont deux tables **distinctes** (un like n'est pas automatiquement un match)
- `matches` résulte de deux likes réciproques
- `blocks` sont **symétriques** en comportement (si A bloque B, B ne voit pas A)
- `audit_logs` n'a pas de dépendance directe de suppression (soft delete obligatoire)

---

## Row Level Security (RLS)

### Principe

Supabase RLS applique une politique d'accès **par utilisateur**. Chaque utilisateur ne voit que :

- Ses propres données privées
- Les données publiques autorisées selon le contexte
- Les données dont il/elle a accès (université, pas bloqué, etc.)

### Contextes de sécurité

**Profils visibles** :
- Gestion de la visibilité en fonction de l'université et du statut

**Messages privés** :
- Gestion de l'accès uniquement aux participants du match et admin

**Documents de vérification** :
- Gestion de la confidentialité absolue des justificatifs

### Implémentation

La définition complète des politiques RLS sera dans `row_level_security.md`.

---

## Traçabilité avec les règles métier

### Liaison avec les règles métier

Chaque élément de l'architecture peut être tracé vers une ou plusieurs règles métier :

**Exemple** :
```
ACC-001 (Accès à la plateforme)
    ↓
Gestion des accès et vérifications

UNIV-002 (Découverte limitée à l'université)
    ↓
Segmentation par université
```

Cette traçabilité garantit que le schéma respecte les règles métier et permet une validation facile de la cohérence.

---

## Prochaines étapes

### Phase 2.1 : Diagramme ERD

**Fichier** : `database/erd_diagram.md`

Visualisation complète de toutes les tables et leurs relations.

**Contenu** : Diagramme textuel ou Mermaid montrant les entités et associations.

### Phase 2.2 : Schéma SQL

**Fichier** : `database/schema.sql`

Définition complète des tables, colonnes, contraintes, triggers, indexes.

**Contenu** : Code SQL créant l'intégralité du schéma PostgreSQL.

### Phase 2.3 : Politiques Row Level Security

**Fichier** : `database/row_level_security.md`

Politiques Supabase pour chaque table et chaque cas d'usage.

**Contenu** : Définition des politiques RLS et justification.

### Phase 2.4 : Migrations

**Fichier** : `database/migrations/001_initial_schema.sql`

Script de migration créant la base de données.

**Contenu** : Version exécutable et versionnée du schéma.

---

## Liaison avec la documentation produit

### Flux de conception

```
Vision (01_vision.md)
    ↓
Personas (03_personas.md)
    ↓
Parcours (04_parcours_utilisateur.md)
    ↓
Règles métier (05_regles_metier.md) ← SOURCE DE VÉRITÉ
    ↓
Architecture données (database/00_introduction.md)
    ↓
Schéma SQL (database/schema.sql)
    ↓
RLS (database/row_level_security.md)
    ↓
Code (backend + frontend)
```

### Principe

À chaque étape, la phase suivante **dérive** de la phase précédente, pas l'inverse. Si le code révèle une faille, on remonte aux règles métier pour clarifier.

---

**Document rédigé** : 26 septembre 2026
**Version** : 1.0
**Statut** : ✅ Architecture conceptuelle
