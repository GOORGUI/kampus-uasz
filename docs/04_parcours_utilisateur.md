# Parcours utilisateur — Campus Connect

Ce document décrit les parcours principaux des utilisateurs dans Campus Connect. Pour chaque parcours, nous détaillons les préconditions, actions, résultats et cas d'erreur.

---

## 1. Parcours : Inscription

### Description

Un nouvel utilisateur crée un compte Campus Connect en fournissant ses informations de base.

### Préconditions

- Utilisateur non encore inscrit
- Accès à Internet
- Adresse e-mail valide

### Actions utilisateur

1. Clique sur "S'inscrire"
2. Entre son adresse e-mail
3. Entre un mot de passe (confirmation)
4. Accepte les conditions d'utilisation
5. Confirme être âgé d'au moins 18 ans
6. Clique sur "Créer mon compte"

### Actions système

1. Valide l'e-mail (format valide)
2. Valide le mot de passe (critères de sécurité)
3. Enregistre l'utilisateur en base de données avec statut `pending_email`
4. Envoie un e-mail de confirmation
5. Affiche un écran de confirmation

### Résultat attendu

- ✅ Compte créé
- ✅ E-mail de confirmation envoyé
- ✅ Utilisateur redirigé vers "Confirmer votre e-mail"
- ✅ État du compte : `pending_email`

### Cas d'erreur

| Erreur | Action système |
|--------|-----------------||
| E-mail déjà utilisé | Affiche "Cet e-mail est déjà inscrit" |
| E-mail invalide | Affiche "Format d'e-mail invalide" |
| Mot de passe faible | Affiche "Mot de passe trop simple" |
| Conditions non acceptées | Désactive le bouton "Créer mon compte" |
| Âge non confirmé | Désactive le bouton "Créer mon compte" |
| Erreur réseau | Affiche "Connexion impossible, réessayez" |

---

## 2. Parcours : Vérification par e-mail et justificatif étudiant

### Description

L'utilisateur confirme son adresse e-mail, puis soumet un justificatif étudiant (carte ou certificat) pour vérification manuelle.

### Préconditions

- Compte créé et statut `pending_email`
- E-mail de confirmation reçu
- Accès au justificatif étudiant

### Actions utilisateur — Étape 1 : Confirmation d'e-mail

1. Ouvre l'e-mail de confirmation
2. Clique sur le lien de confirmation
3. Revient sur l'application

### Actions système — Étape 1

1. Valide le token du lien
2. Met à jour le statut du compte à `pending_verification`
3. Affiche l'écran de soumission du justificatif

### Résultat — Étape 1

- ✅ E-mail confirmé
- ✅ Accès à l'écran de soumission du justificatif
- ✅ État du compte : `pending_verification`

---

### Actions utilisateur — Étape 2 : Soumission du justificatif

1. Clique sur "Télécharger mon justificatif"
2. Sélectionne une photo de sa carte d'étudiant (recto) ou certificat d'inscription
3. Vérifie que le document est lisible
4. Clique sur "Soumettre pour vérification"

### Actions système — Étape 2

1. Valide le format du fichier (JPG, PNG, PDF)
2. Valide la taille (< 5 MB)
3. Stocke le document dans Supabase Storage (privé)
4. Crée un enregistrement `verification` avec statut `pending_review`
5. Envoie une notification à l'administrateur
6. Affiche un écran "En attente de vérification"

### Résultat — Étape 2

- ✅ Justificatif téléchargé et stocké
- ✅ Administrateur notifié
- ✅ État du compte : `pending_verification`
- ✅ État du justificatif : `pending_review`

---

### Actions administrateur — Étape 3 : Validation manuelle

1. Accède au tableau de bord d'administration
2. Consulte la demande de vérification
3. Examine le document téléchargé
4. Accepte ou refuse la vérification

#### Cas : Acceptation
1. Clique sur "Approuver la vérification"
2. Le justificatif est marqué `verified`
3. L'utilisateur reçoit une notification
4. L'utilisateur peut maintenant créer son profil

#### Cas : Refus
1. Clique sur "Rejeter la vérification"
2. Sélectionne une raison (document flou, ne correspond pas à l'âge, etc.)
3. Le justificatif est marqué `rejected`
4. L'utilisateur reçoit une notification avec la raison
5. L'utilisateur peut réessayer avec un nouveau document

### Actions système — Étape 3

1. Met à jour le statut de vérification
2. Envoie un e-mail à l'utilisateur
3. Si accepté : met à jour le statut du compte à `verified`
4. Si refusé : remet le statut à `pending_verification`
5. Le document est conservé selon la politique de conservation des données

### Résultat — Étape 3

- ✅ Justificatif validé ou rejeté
- ✅ Utilisateur notifié
- ✅ État du compte : `verified` (si accepté) ou `pending_verification` (si refusé)

### Cas d'erreur

| Erreur | Action système |
|--------|-----------------||
| Document flou ou illisible | Admin rejette avec raison |
| Document expiré | Admin rejette avec raison |
| Document ne correspond pas à l'âge | Admin rejette avec raison |
| Utilisateur n'apparaît pas sur le document | Admin rejette avec raison |
| Fichier corrompu | Affiche erreur et demande re-téléchargement |
| Token de confirmation expiré | Envoie un nouveau lien |
| Délai de traitement dépassé (> 48h) | Relance administrateur |

---

## 3. Parcours : Création du profil

### Description

Après vérification, l'utilisateur crée son profil en remplissant ses informations personnelles.

### Préconditions

- Compte vérifié (statut `verified`)
- Utilisateur connecté
- Pas de profil existant

### Actions utilisateur

1. Clique sur "Créer mon profil"
2. Entre son prénom ou pseudonyme
3. Entre son âge
4. Sélectionne son genre (homme/femme/autre)
5. Sélectionne sa faculté
6. Sélectionne sa filière
7. Sélectionne son niveau (L1, L2, L3, M1, M2, etc.)
8. Télécharge une photo de profil
9. Écrit une courte biographie (max 200 caractères)
10. Sélectionne ses centres d'intérêt (checkbox) : musique, cinéma, sport, étude, etc.
11. Sélectionne ses objectifs (checkbox) : amitié, étude, réseau, relation sérieuse
12. Clique sur "Valider mon profil"

### Actions système

1. Valide tous les champs obligatoires
2. Vérifie que l'âge >= 18 ans
3. Redimensionne et compresse la photo
4. Stocke la photo dans Supabase Storage (privé)
5. Crée l'enregistrement `profile` en base de données avec statut `pending_review`
6. Crée les enregistrements `preferences` correspondants
7. Envoie une notification pour modération légère (vérification photo/contenu)
8. Affiche un écran "Profil en attente de validation"

### Actions système — Validation du profil

1. **Validation automatisée** :
   - Vérifie l'absence de contenu explicite (détection basique)
   - Vérifie que la photo est au bon format et non corrompue
   - Si validations passées : statut → `active`

2. **Si détection de contenu suspect** :
   - Affiche au modérateur pour revue manuelle
   - Modérateur approuve ou rejette

3. **Une fois approuvé** :
   - Statut du profil → `active`
   - Utilisateur reçoit notification : "Profil activé !"
   - Utilisateur peut commencer à découvrir d'autres profils

### Résultat attendu

- ✅ Profil créé
- ✅ Photo stockée
- ✅ En attente de validation (5-30 min généralement)
- ✅ Devient visible après approbation
- ✅ État du compte : `verified` (pas de changement jusqu'à activation du profil)
- ✅ État du profil : `pending_review` → `active`

### Cas d'erreur

| Erreur | Action système |
|--------|-----------------||
| Champ obligatoire vide | Affiche message d'erreur sous le champ |
| Âge < 18 ans | Affiche "Vous devez être majeur" et désactive la validation |
| Photo trop volumineux | Affiche "Fichier trop lourd (max 5 MB)" |
| Photo au mauvais format | Affiche "Format accepté : JPG, PNG" |
| Aucun objectif sélectionné | Affiche "Sélectionnez au moins un objectif" |
| Biographie trop longue | Affiche "Max 200 caractères" |

---

## 4. Parcours : Modification du profil

### Description

L'utilisateur peut modifier son profil après création.

### Préconditions

- Compte actif
- Profil existant
- Utilisateur connecté

### Actions utilisateur

1. Accède à "Mon profil"
2. Clique sur "Modifier le profil"
3. Modifie les champs souhaités :
   - Biographie
   - Photo (remplacement complet)
   - Intérêts
   - Objectifs
   - Prénom/pseudonyme (audit obligatoire)
   - Âge/genre/filière (audit obligatoire)
4. Clique sur "Enregistrer les modifications"

### Actions système

1. **Pour les champs simples** (bio, intérêts, objectifs) :
   - Validation basique
   - Mise à jour immédiate
   - Enregistrement du changement dans `audit_logs`

2. **Pour les champs sensibles** (photo, prénom, âge, genre, filière) :
   - Validation complète
   - Enregistrement du changement dans `audit_logs` avec raison
   - Photo : re-stockage, ancien fichier supprimé
   - Autres : mise à jour immédiate mais avec timestamp et ancien/nouveau disponible

3. **Pour la photo** :
   - Si nouveau upload : remet le profil en `pending_review` (revalidation rapide)
   - Une fois validée : profil redevient `active`

### Résultat attendu

- ✅ Modifications enregistrées
- ✅ Audit log créé
- ✅ Photo remplacée si applicable
- ✅ Profil revalidé si photo modifiée

### Cas d'erreur

| Erreur | Action système |
|--------|-----------------||
| Champ vide | Affiche message d'erreur |
| Photo invalide | Affiche "Format non valide" |
| Tentative spam (> 10 modifs/jour) | Limite à 5 modifications par jour |

---

## 5. Parcours : Découverte de profils

### Description

L'utilisateur parcourt les profils d'autres étudiants vérifiés pour découvrir d'autres utilisateurs.

### Préconditions

- Compte statut `verified`
- Profil statut `active`
- Au moins 1 profil éligible existe dans la base de données
- Utilisateur non bloqué

### Actions utilisateur

1. Accède à l'écran "Découverte"
2. Parcourt les profils affichés (mode "pile de cartes" ou "liste")
3. Applique des filtres optionnels (filière, niveau, objectif, intérêts)
4. Clique sur un profil pour voir plus de détails
5. Peut masquer temporairement son profil

### Actions système — Affichage des profils

1. Récupère les profils éligibles (statut `active`, non bloqués par l'utilisateur)
2. Exclut les profils que l'utilisateur a déjà likés ou passés (7 derniers jours)
3. Exclut les profils qui l'ont bloqué
4. Recommande par ordre de pertinence (objectifs communs, intérêts)
5. Affiche le prénom, la photo, l'âge, la filière, les objectifs
6. Affiche la biographie au clic

### Actions système — Filtrage

1. Applique les filtres sélectionnés
2. Re-récupère les profils pertinents
3. Met à jour l'affichage en temps réel

### Actions système — Masquage temporaire

1. Met le profil de l'utilisateur à statut `active` avec `hidden_until` = timestamp (24h)
2. Aucun autre utilisateur ne peut voir ce profil
3. Affiche un message "Votre profil est masqué jusqu'à demain"

### Résultat attendu

- ✅ Profils affichés correctement
- ✅ Filtres appliqués
- ✅ Aucun profil bloqué ou supprimé visible
- ✅ Profil peut être masqué temporairement

### Cas d'erreur

| Erreur | Action système |
|--------|-----------------||
| Aucun profil disponible | Affiche "Pas de profils à découvrir pour l'instant" |
| Utilisateur bloqué | Affiche "Vous n'avez pas accès à cette fonctionnalité" |
| Profil supprimé | N'affiche pas le profil, passe au suivant |
| Filtres trop restrictifs | Affiche "Aucun résultat avec ces filtres" |
| Erreur de chargement | Affiche "Erreur lors du chargement des profils" |

---

## 6. Parcours : Like et Match

### Description

L'utilisateur like un profil. S'il y a un like réciproque, un match est automatiquement créé.

### Préconditions

- Compte statut `verified`
- Profil statut `active`
- Visualisation d'un profil autre
- Non bloqué par ce profil
- Pas d'action antérieure sur ce profil (like ou pass) dans les 7 derniers jours

### Actions utilisateur

1. Visualise un profil
2. Clique sur "J'aime" ou fait un swipe vers la droite

### Actions système

1. Valide que l'utilisateur n'a pas déjà aimé ce profil (dans les 7 jours)
2. Vérifie que l'utilisateur n'est pas bloqué
3. Enregistre le like en base de données (table `likes`) — **le like est conservé**
4. Vérifie l'existence d'un like réciproque
5. **Si like réciproque existe** :
   - Crée un match (table `matches`)
   - **Conserve les likes** (utile pour analytics, statistiques, recommandations futures)
   - Envoie une notification à l'autre utilisateur : "Match avec [Prénom] ! 🎉"
   - Envoie une notification à l'utilisateur courant : "Match avec [Prénom] ! 🎉"
   - Redirige vers la conversation ou l'écran de match
6. **Si pas de like réciproque** :
   - Affiche un message de confirmation : "Vous avez aimé ce profil"
   - Continue la découverte

### Résultat attendu

- ✅ Like enregistré
- ✅ Match créé si réciproque
- ✅ Notifications envoyées
- ✅ Accès à la messagerie si match
- ✅ Likes conservés pour analytics

### Cas d'erreur

| Erreur | Action système |
|--------|-----------------||
| Utilisateur bloqué | Affiche "Ce profil n'est pas disponible" |
| Profil supprimé | Affiche "Ce profil n'existe plus" |
| Like déjà envoyé (7 jours) | Affiche "Vous avez déjà aimé ce profil" |
| Limite quotidienne dépassée | Affiche "Vous avez atteint la limite de likes (50/jour)" |
| Compte suspendu | Accès refusé à la fonctionnalité |

### Capacité de modification

- **Un like peut être retiré tant qu'aucun match n'existe** pour corriger une erreur ou un changement d'avis
- Une fois un match créé, le like devient irrévocable (suppression du match uniquement possible)
- Un like expire après 30 jours s'il n'y a pas de réciprocité (soft delete)

---

## 7. Parcours : Conversation et messagerie

### Description

Deux utilisateurs matchés peuvent échanger des messages privés.

### Préconditions

- Match existant entre deux utilisateurs
- Les deux comptes statut `verified` (pas `suspended`)
- Aucun des deux n'a bloqué l'autre

### Actions utilisateur

1. Accède à la liste des matchs
2. Clique sur un match
3. Visualise l'historique de la conversation
4. Tape un message (max 500 caractères)
5. Clique sur "Envoyer" ou appuie sur Entrée
6. Peut visualiser les messages précédents
7. Peut supprimer sa conversation

### Actions système — Envoi de message

1. Valide que le match existe toujours
2. Valide que les deux utilisateurs ne se sont pas bloqués
3. Enregistre le message en base de données (table `messages`)
4. Envoie une notification poussée à l'autre utilisateur
5. Affiche le message en temps réel (Supabase Realtime)
6. Marque le message comme `sent` puis `delivered`

### Actions système — Historique

1. Récupère les messages du match (trié par date)
2. Affiche les messages avec timestamp
3. Indique qui a envoyé chaque message

### Actions système — Suppression de conversation

1. Supprime tous les messages du match pour cet utilisateur (soft delete)
2. Le match reste existant pour l'autre utilisateur
3. Affiche un message de confirmation

### Résultat attendu

- ✅ Message envoyé et reçu
- ✅ Notification envoyée à l'autre utilisateur
- ✅ Historique visible
- ✅ Conversation supprimable

### Cas d'erreur

| Erreur | Action système |
|--------|-----------------||
| Un utilisateur a bloqué l'autre | Message non envoyé, affiche "Ce profil a bloqué" |
| Match supprimé | Affiche "La conversation n'existe plus" |
| Message vide | Désactive le bouton "Envoyer" |
| Message trop long | Affiche "Message limité à 500 caractères" |
| Utilisateur suspendu | Accès refusé à la messagerie |
| Erreur réseau | Affiche "Message non envoyé" et propose de réessayer |

---

## 8. Parcours : Blocage et déblocage

### Description

Un utilisateur peut bloquer un autre profil et le débloquer ultérieurement.

### Préconditions

- Visualisation d'un profil ou conversation active
- Compte actif

### Actions utilisateur — Blocage

1. Visualise un profil ou une conversation
2. Clique sur le menu (⋮) ou bouton "Bloquer"
3. Clique sur "Bloquer cet utilisateur"
4. Confirme l'action

### Actions système — Blocage

1. Crée un enregistrement `blocks` (utilisateur A bloque utilisateur B)
2. Met à jour les politiques RLS Supabase
3. Utilisateur B ne voit plus le profil de A
4. Utilisateur A ne voit plus le profil de B
5. Conversation existante devient invisible pour les deux
6. Nouveaux matchs entre A et B sont impossibles
7. Affiche un message "Cet utilisateur a été bloqué"

### Résultat attendu — Blocage

- ✅ Utilisateur bloqué
- ✅ Profil masqué
- ✅ Messagerie supprimée de l'affichage
- ✅ Blocage réversible (déblocage possible)

---

### Actions utilisateur — Déblocage

1. Accède à Paramètres → Confidentialité
2. Clique sur "Liste des utilisateurs bloqués"
3. Trouve l'utilisateur à débloquer
4. Clique sur "Débloquer"

### Actions système — Déblocage

1. Supprime l'enregistrement `blocks`
2. Met à jour les politiques RLS Supabase
3. Les profils redeviennent visibles mutuellement
4. Les nouveaux matchs deviennent à nouveau possibles
5. Les conversations précédentes restent supprimées (ne sont pas restaurées)

### Résultat attendu — Déblocage

- ✅ Utilisateur débloqué
- ✅ Profils à nouveau visibles
- ✅ Nouveaux matchs possibles

### Cas d'erreur

| Erreur | Action système |
|--------|-----------------||
| Blocage d'un utilisateur déjà bloqué | Affiche "Cet utilisateur est déjà bloqué" |
| Déblocage d'un utilisateur non bloqué | Affiche "Cet utilisateur n'est pas bloqué" |

---

## 9. Parcours : Signalement et modération

### Description

Un utilisateur peut signaler un autre profil en cas de comportement inapproprié. L'équipe d'administration traite les signalements.

### Préconditions

- Visualisation d'un profil ou conversation active
- Compte actif

### Actions utilisateur — Signalement

1. Visualise un profil ou une conversation
2. Clique sur le menu (⋮) ou bouton "Signaler"
3. Sélectionne une raison : 
   - Spam ou messages offensants
   - Contenu explicite ou pornographique
   - Harcèlement ou menaces
   - Faux profil ou arnaque
   - Autre
4. Peut ajouter un commentaire optionnel
5. Clique sur "Signaler"

### Actions système — Signalement

1. Enregistre le signalement en base de données (table `reports`)
2. Attribue le statut `pending_review`
3. Affiche un message "Merci de votre signalement"
4. Envoie une notification à l'administrateur
5. Si signalements multiples du même utilisateur :
   - Des signalements multiples augmentent la priorité de revue humaine
   - Aucune sanction n'est appliquée automatiquement sans validation humaine

### Résultat attendu — Signalement

- ✅ Signalement enregistré
- ✅ Admin notifié
- ✅ Utilisateur reçoit confirmation
- ✅ Signalement conservé pour audit

### Actions administrateur

1. Accède au tableau de bord des signalements
2. Examine les détails du signalement
3. Visualise le profil et les messages en question
4. Valide si le signalement est fondé
5. Prend une action (basée sur validation humaine) :
   - **Classer sans suite** : signalement non fondé
   - **Avertir l'utilisateur** : envoie un message d'avertissement
   - **Suspendre temporairement** : compte inaccessible pendant 7 jours
   - **Suspendre définitivement** : compte inaccessible
   - **Supprimer le compte** : suppression complète des données

### Résultat — Action administrative

- ✅ Signalement traité sous 48h
- ✅ Utilisateur signalé notifié de l'action
- ✅ Historique conservé pour audit
- ✅ Décision reste humaine (pas d'escalade automatique)

### Cas d'erreur

| Erreur | Action système |
|--------|-----------------||
| Signalement d'un utilisateur déjà signalé | Affiche "Vous avez déjà signalé cet utilisateur" |
| Raison de signalement vide | Affiche "Sélectionnez une raison" |

---

## 10. Parcours : Suspension de compte

### Description

Un administrateur suspend le compte d'un utilisateur pour violation des règles.

### Préconditions

- Signalement confirmé ou violation détectée
- Compte actif ou en état vérifiable

### Actions administrateur

1. Accède au profil utilisateur
2. Clique sur "Suspendre le compte"
3. Sélectionne une durée (7 jours, 30 jours, permanent)
4. Saisit une raison
5. Clique sur "Confirmer la suspension"

### Actions système

1. Met à jour le statut du compte à `suspended`
2. Définit la date de réactivation (si temporaire) dans `suspended_until`
3. Envoie un e-mail à l'utilisateur avec :
   - La raison de la suspension
   - La durée
   - La date de réactivation (si applicable)
4. Rend le compte inaccessible
5. Masque tous ses profils et conversations
6. Conserve les données (pas de suppression)

### Résultat attendu

- ✅ Compte inaccessible
- ✅ Utilisateur notifié
- ✅ Données conservées
- ✅ Date de réactivation enregistrée (si temporaire)

### Cas d'erreur

| Erreur | Action système |
|--------|-----------------||
| Compte déjà suspendu | Affiche "Ce compte est déjà suspendu" |
| Compte administrateur | Affiche "Action non autorisée" |

---

## 11. Parcours : Réactivation après suspension temporaire

### Description

Un compte suspendu temporairement est réactivé automatiquement après la durée écoulée.

### Préconditions

- Compte suspendu avec `suspended_until` défini
- Date de `suspended_until` atteinte

### Actions système

1. Détecte qu'un compte doit être réactivé (cronjob quotidien)
2. Met à jour le statut du compte à `verified`
3. Efface `suspended_until`
4. Envoie un e-mail à l'utilisateur : "Votre compte a été réactivé"
5. Utilisateur peut se reconnecter

### Résultat attendu

- ✅ Compte réactivé automatiquement
- ✅ Utilisateur notifié
- ✅ Peut reprendre l'utilisation

### Cas d'erreur

| Erreur | Action système |
|--------|-----------------||
| Suspension permanente | Aucune réactivation automatique |
| Erreur du cronjob | Alerte l'administrateur |

---

## 12. Parcours : Suppression de compte

### Description

Un utilisateur demande la suppression complète de son compte.

### Préconditions

- Compte actif
- Utilisateur connecté

### Actions utilisateur

1. Accède à Paramètres → Confidentialité
2. Clique sur "Supprimer mon compte"
3. Lit l'avertissement (données supprimées, processus irréversible)
4. Confirme en tapant "SUPPRIMER"
5. Clique sur "Confirmer la suppression"

### Actions système — Étape 1 : Demande de suppression

1. Met à jour le statut du compte à `deletion_pending`
2. Stocke une date limite (30 jours + 1 jour = 31 jours)
3. Envoie un e-mail de confirmation
4. Affiche un message : "Votre compte sera supprimé dans 30 jours"
5. Utilisateur reçoit un lien pour annuler la suppression

### Actions système — Étape 2 : Annulation possible

- Pendant les 30 jours, l'utilisateur peut :
  - Se reconnecter normalement
  - Clique sur "Annuler la suppression" dans l'e-mail
  - Statut redevient `verified` (ou `active` si profil existant)
  - Compte fonctionne normalement

### Actions système — Étape 3 : Suppression logique (après 30 jours)

1. Cronjob détecte les demandes de suppression expirées
2. **Anonymisation des données personnelles** :
   - Prénom → "[Utilisateur supprimé]"
   - Biographie → supprimée
   - Photo → supprimée
   - E-mail → hachée + suffixe `_deleted` (reste unique pour audit)
3. **Conservation des données pour audit et sécurité** :
   - Messages : conservés mais anonymisés
   - Matches : marqués comme `deleted_by_user`
   - Likes : conservés (sans lien vers le profil, utile pour analytics)
4. **Mise à jour du profil** :
   - Profil marqué `deleted` avec `deleted_at` timestamp
   - Visible en tant que "[Utilisateur supprimé]"
5. **Audit et conformité** :
   - Enregistrement de suppression dans logs d'audit
   - Logs de sécurité conservés (RGPD compliant)

### Résultat attendu

- ✅ Demande de suppression enregistrée
- ✅ Délai d'annulation de 30 jours
- ✅ Suppression logique (soft delete) après délai
- ✅ Données anonymisées
- ✅ Conformité RGPD
- ✅ Données de sécurité conservées

### Cas d'erreur

| Erreur | Action système |
|--------|-----------------||
| Compte déjà en cours de suppression | Affiche "Suppression déjà en cours" |
| Confirmation échouée | Affiche "Vous devez taper 'SUPPRIMER'" |
| Token d'annulation expiré | Affiche "Impossible d'annuler, délai dépassé" |

---

## 13. Cas d'erreur et exceptions globaux

### Authentification et session

| Cas | Action système |
|-----|-----------------||
| Token expiré | Redirige vers connexion |
| Session inactive > 30 min | Déconnexion automatique |
| Accès sans authentification | Redirige vers inscription |
| Tentatives de login répétées | Blocage temporaire après 5 tentatives (15 min) |

### Données incohérentes

| Cas | Action système |
|-----|-----------------||
| Profil supprimé mais visible | Le masque de l'affichage |
| Match avec compte suspendu | Masque de la liste des matchs |
| Utilisateur bloqué dans un match existant | Masque la conversation |
| Like orphelin | Supprime le like orphelin après 90 jours |

### Limites et quotas

| Limite | Valeur | Raison |
|--------|--------|--------|
| Likes par jour | 50 | Éviter le spam |
| Messages par heure | 30 | Éviter le spam |
| Nouveaux profils visibles | 100 | Éviter la surcharge |
| Taille de photo | 5 MB | Stockage optimisé |
| Longueur de message | 500 caractères | Conversations saines |
| Modifications de profil par jour | 5 | Éviter le spam |

---

## États des comptes (users.status)

```
pending_email
    ↓
pending_verification
    ↓
verified
    ↓
    ├→ suspended (temporaire ou permanent)
    │   └→ verified (réactivation automatique si temporaire)
    └→ deletion_pending
        └→ deleted (après 30 jours, soft delete)
```

## États des profils (profiles.status)

```
pending_review
    ↓
active
    ↓
    ├→ active + hidden_until [timestamp] (masquage temporaire)
    └→ deleted (soft delete)
```

---

## Diagramme de flux global

```
Inscription
    ↓
Vérification e-mail
    ↓
Soumission justificatif
    ↓
Approbation admin
    ↓
Création de profil
    ↓
Validation profil (pending_review → active)
    ↓
Découverte de profils
    ↓
Like → Match (si réciproque, likes conservés)
    ↓
Conversation
    ↓
    ├→ Blocage/Déblocage
    ├→ Signalement → Modération admin
    └→ Suspension (si nécessaire)
        └→ Réactivation (après durée)
    
Suppression de compte (optionnel)
    ↓
deletion_pending (30 jours)
    ↓
deleted (soft delete + anonymisation)
```

---

## Conservation des données

- **Documents de vérification** : selon politique de conservation des données (à définir légalement)
- **Messages** : conservés indéfiniment, anonymisés après suppression de compte
- **Likes** : conservés indéfiniment (utiles pour analytics même après suppression)
- **Logs d'audit** : conservés 1 an minimum
- **Données personnelles** : supprimées (anonymisées) selon RGPD après demande

---

**Document rédigé** : 26 septembre 2026
**Version** : 1.2
**Statut** : ✅ Validé