# Campus Connect

Le réseau social étudiant vérifié de l'Université Assane Seck de Ziguinchor.

## Vision

Campus Connect est une plateforme communautaire réservée aux étudiants vérifiés de l'UASZ. Elle crée un espace de confiance où les étudiants peuvent :

- **développer leur réseau** auprès d'autres étudiants vérifiés ;
- **trouver des partenaires d'étude** pour préparer les examens ou les projets ;
- **rencontrer de nouveaux amis** dans un environnement modéré et sécurisé ;
- **créer des relations authentiques** basées sur l'intérêt mutuel.

## Valeurs fondamentales

🔐 **Vérification des étudiants** — Seuls les utilisateurs vérifiés comme étudiants à l'UASZ peuvent accéder à la plateforme.

🛡️ **Sécurité d'abord** — Blocage instantané, signalement, modération active et invisibilité temporaire.

✋ **Respect mutuel** — Aucun message sans accord préalable. Les conversations ne commencent qu'après un match réciproque.

🔒 **Confidentialité** — Les données personnelles ne sont jamais partagées. Les justificatifs étudiants restent privés.

🚨 **Modération** — L'équipe administrateur surveille les comportements abusifs et agit rapidement.

## Statut du projet

🚧 **MVP en cours de conception**

- Documentation stratégique : semaine 1
- Schéma de base de données : semaine 2
- Maquettes UI/UX : semaine 2-3
- Développement : après validation de la conception
- Bêta fermée UASZ : objectif à définir

## Objectifs de l'utilisateur

À l'inscription, chaque étudiant choisit ses objectifs :

- 💬 Amitié — Faire connaissance avec d'autres étudiants
- 📚 Étude — Trouver un partenaire pour préparer un examen ou un projet
- 🤝 Réseau — Développer son réseau professionnel et académique
- 💕 Relation sérieuse — Chercher une relation amoureuse dans un environnement de confiance

Un utilisateur peut avoir plusieurs objectifs actifs simultanément.

## Fonctionnalités du MVP

### Inscription et vérification

- Création de compte par e-mail universitaire
- Confirmation d'identité par justificatif étudiant
- Validation manuelle par un administrateur
- Confirmation de majorité (18 ans minimum)

### Profil

- Prénom, âge, genre, filière, niveau d'étude
- Photo principale
- Biographie personnelle
- Sélection des objectifs
- Centres d'intérêt

### Découverte

- Parcours les profils des étudiants vérifiés
- Filtres par faculté, filière, niveau, objectif
- Action « J'aime » ou « Passer »
- Masquage temporaire du profil

### Matching

- Match automatique lorsque deux utilisateurs s'aiment mutuellement
- Notification de match
- Accès à la messagerie

### Messagerie

- Conversations 1-à-1 entre utilisateurs matchés
- Blocage instantané
- Signalement en un clic
- Suppression de conversation

### Administration

- Validation des demandes de vérification
- Traitement des signalements
- Suspension ou suppression de comptes
- Dashboard des statistiques principales

## Architecture technique

- **Frontend** : Next.js + TypeScript + Tailwind CSS
- **Backend** : Supabase (PostgreSQL, Auth, Realtime)
- **Base de données** : PostgreSQL avec Row Level Security
- **Stockage** : Supabase Storage (documents privés)
- **Déploiement** : Vercel (frontend) + Supabase (backend)

## Extensibilité

Bien que le MVP cible uniquement l'UASZ, l'architecture est conçue pour supporter d'autres universités sénégalaises et africaines.

## Risques et défis

⚠️ **Équilibre de la communauté** — Les applications de rencontre souffrent souvent d'un déséquilibre hommes/femmes. La sécurité perçue des étudiantes sera critique.

⚠️ **Adoption initiale** — Le succès dépend des 100 premiers utilisateurs. Une stratégie de recrutement auprès des associations étudiantes est essentielle.

⚠️ **Modération** — Sans modération active et visible, la confiance s'érode rapidement.

## Roadmap post-MVP

### v1.1

- Notifications en temps réel
- Système de réputation (trust score)
- Recommandations intelligentes
- Amélioration du filtrage
- Galerie de photos

### v1.2

- Extension à une deuxième université
- Recherche de partenaires d'étude par sujet
- Événements étudiants

### v2.0

- Application mobile native
- Groupes thématiques
- Intégration avec les calendriers académiques

## Contribution

Ce projet est développé par une équipe étudiante de l'UASZ. Les retours des utilisateurs et des testeurs sont essentiels à son succès.

## Licence

À définir (probablement MIT pour l'open source)

## Contact

Pour toute question : [À définir]

---

**Statut** : 🟢 Documentation validée | Prochaine étape : Personas et règles métier
