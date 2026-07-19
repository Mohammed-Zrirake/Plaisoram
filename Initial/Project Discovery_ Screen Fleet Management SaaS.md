
---
### Phase 1: The Core Engine & Onboarding (Le MVP)
*Objectif : Un utilisateur s'inscrit en toute sécurité, paramètre son compte, associe un écran physique ou virtuel, ajoute un média et le publie.*

**1. Authentification & Sécurité de base :**
- [x] Inscription / Connexion (Email + Mot de passe avec icône Œil).
- [ ] Connexion sans friction (Google SSO).
- [x] Formulaire Lead Gen (Prénom, Nom, Téléphone avec indicatif pays).
- [x] Conformité RGPD & CGU (Cases à cocher obligatoires).
- [ ] Bouclier Anti-Spam (Intégration reCAPTCHA).
- [ ] Autonomie de récupération (Mot de passe oublié par email).

**2. Configuration du "Workspace" :**
- [x] Création automatique de l'Espace Entreprise à l'inscription.
- [x] Renseignement de l'identité de l'entreprise et du **Fuseau Horaire (Time Zone)**.

**3. Gestion des Médias Basique :**
- [x] Import de fichiers (Drag & Drop) avec limites de taille selon l'abonnement.
- [x] Stockage Cloud (DigitalOcean Spaces / S3) + CDN.
- [x] Édition de base (Renommer, Supprimer avec confirmation).

**4. Le Moteur de Diffusion / Player :**
- [x] Appairage via le code PIN 6 chiffres (`/pair` -> `LA9610`).
- [x] Compatibilité de l'application Kotlin (Offline-first, téléchargement local).
- [x] **Simulateur d'Écran Virtuel** (Faux écran dans le navigateur pour tester sans matériel).

**5. Séquençage & Playlists Basiques :**
- [x] CRUD Playlists (Créer, lister, supprimer, et **Dupliquer**).
- [x] Boucle de contenu (Ajout des médias + définition de la durée d'affichage).

**6. Le "Bouton Rouge" / Publication :**
- [x] Déploiement (Envoi du JSON manifest) avec choix de la cible (Écran spécifique) et diffusion "Immédiate".

**7. Onboarding & Support :**
- [x] Guide de démarrage rapide ("Let's get started" + animation démo).
- [ ] Liens vers le Support Accessible (Tél +212, Email, FAQ, Version du logiciel).

---

### Phase 2: Fleet Management & Advanced Layouts (L'Outil Pro)
*Objectif : Le client possède désormais plusieurs pharmacies/boutiques et veut des playlists complexes et un suivi de sa "flotte" d'écrans.*

**8. Gestion Avancée du Parc d'Écrans :**
- [ ] Fiche détaillée de l'écran (Adresse MAC, Numéro de série, Orientation Portrait/Paysage, Nom du Manager local).
- [ ] Structuration par Groupes (Hiérarchie Parent/Enfant : Maroc -> Rabat -> Pharmacie).
- [x] Tableau de bord de la flotte (Statut Vert/Rouge, Contenu en cours).
- [ ] Vue Cartographique (Pins Maps pour géolocaliser les écrans).
- [ ] Guide d'Achat Matériel (Liens recommandés Amazon/Jumia).

**9. Contrôle à Distance Temps Réel :**
- [x] Actions distantes (Redémarrer la TV, Afficher le code PIN).

**10. Playlists & Calendriers Avancés :**
- [x] Écrans Multi-Zones (Layouts divisés / Split screen).
- [x] Planification Intelligente (Scheduling pour une date/heure dans le futur).
- [x] Vue Calendrier Globale (Mois / Semaine / Jour).
- [ ] Filtrage du calendrier par Groupe d'écrans.
- [x] Rangement des médias par **Dossiers et Sous-dossiers**.

---

### Phase 3: SaaS Business & Security (La Monétisation)
*Objectif : Permettre le travail en équipe (collaborateurs) et activer la facturation.*

**11. Équipe & Permissions - ACL :**
- [ ] Inviter, lister, modifier et supprimer des membres de l'équipe.
- [ ] Création de rôles sur mesure (Manager, Graphiste, etc.).
- [ ] Matrice de permissions très granulaire (Cases à cocher par module : Peut voir ? Peut éditer ?).
- [ ] Affichage d'erreurs d'autorisation claires (UX Sécurité).

**12. Pack "Sécurité Entreprise" :**
- [ ] Blocage du compte après tentatives échouées.
- [ ] Déconnexion automatique (ex: le week-end).
- [x] **Branding & Marque Blanche** (Upload du logo de l'entreprise client).
- [ ] Génération de la **Secret Key** (Clé API) dans le profil utilisateur.

**13. Facturation & Upsell :**
- [ ] Synthèse Financière (Visibilité du plan actuel : "TRIAL S", expiration, date de facture).
- [ ] Jauges de consommation (1/1 Écran, 0.13/1000 Mo).
- [ ] Moteur d'Upsell In-App (Bouton "View plans", Starter/Essential/Business).
- [ ] Évolutivité du Parc (Ajouter/Retirer des licences à la volée avec recalcul du prix).
- [ ] Gestion Multi-Devises (MAD, EUR, USD).
- [ ] Intégration du Paiement (Stripe pour l'international, CMI pour le Maroc).

---

### Phase 4: Integrations, Widgets & Sobrus (L'Ultra-Premium)
*Objectif : Rendre le contenu "Vivant" (Prix dynamiques, Météo) et intégrer les outils de création.*

**14. L'Intégration ERP Sobrus :**
- [ ] Création de champs personnalisés Key/Value (Nom, Valeur, Description).
- [ ] Création du Webhook récepteur pour que l'ERP puisse mettre à jour un prix (ex: prix_doliprane) en temps réel.

**15. App Store & Widgets :**
- [ ] Catalogue de Widgets (YouTube, Météo, Horloge, Google Slides, RSS).
- [ ] Formulaires de paramètres dynamiques (Cover/Contain, Center/Top, Verrouillage).
- [ ] Sauvegarde de modèles (Instances - ex: "Météo Agadir").
- [ ] Ajout de ces widgets directement dans le séquençage de la playlist.

**16. Outils de Création Graphique :**
- [ ] Éditeur Graphique intégré (Canevas 1920x1080, Texte, Formes).
- [ ] Galerie de Templates (Menus de restaurant, Pharmacies).
- [ ] Bibliothèque "Starter Pack" (Fonds d'écran fournis).
- [ ] Moteur de recherche d'Images Stock (Unsplash / Pexels API).

**17. Outils IT Avancés :**
- [ ] Intégration des champs **Anydesk ID et Password** directement dans la fiche de l'écran pour la prise en main à distance par le support IT.