# AmongUS

Ce projet a été réalisé en groupe dans le cadre du cours **Programmation réseau multi-joueurs en ligne**, offert à l’**Université du Québec à Chicoutimi (UQAC)**.  

---

- **Titre du projet :** AmongUS
- **Date de réalisation :** 20 novembre 2025  
- **Type de projet :** Travail individuel  
- **Moteur de développement :** Unreal Engine

---

## Objectif du projet

Ce projet consiste à développer un jeu multijoueur de type *Among Us* sous **Unreal Engine 5.6**, en appliquant la réplication réseau, la synchronisation de temps et un système avancé de rewind pour la validation des tirs.

---

## Objectifs techniques

### Structure du projet
- Le fichier `.uproject` doit être placé **à la racine** du dépôt Perforce.
- Le jeu est en **multijoueur** et utilise la réplication d’Unreal.

### Rôles des joueurs (répliqués)
- Chaque joueur possède un état :  
  **GENTIL**, **MECHANT**, ou **MORT**  
- Cet état est **répliqué** et visible par **tous les joueurs**.

### Système de tâches (répliqué)
- Le niveau contient **N boutons** interactifs.
- Le compteur de tâches est **répliqué** et visible par tous.

**Règles :**
- Si un **Méchant** active un bouton → les tâches **augmentent**
- Si un **Gentil** active un bouton → les tâches **diminuent**

---

## Synchronisation du temps

- Le projet utilise une synchronisation temporelle basée sur un algorithme similaire au **NTP**, comme étudié en cours.

### Boucle Lobby → Partie → Lobby
- Dès que **2 joueurs** sont connectés au Lobby → début d’un **compte à rebours de 30s**
- Après 30s → chargement automatique du **niveau de jeu**
- La partie dure **X secondes** (ex : 120s)
- Une fois terminée → retour au **Lobby**
- Un **menu Pause** doit permettre de quitter le jeu ou retourner au menu principal

---

## Système de Rewind (validation des tirs)

### 1️ URewindableComponent
- Nouveau component dérivé de `UCapsuleComponent`
- Ajouté au personnage
- Capsule visible en jeu et couvrant le joueur
- Lors du `BeginPlay`, le composant **s’enregistre** auprès du RewindSubsystem  
  *(serveur uniquement)*

### 2️ URewindSubsystem (UTickableWorldSubsystem)
- Doit exister **uniquement sur le serveur** (via `ShouldCreateSubsystem`)
- Chaque tick, le serveur enregistre pendant **1 seconde** :
  - Le temps serveur
  - La liste des joueurs
  - Les positions de toutes les capsules `URewindableComponent`
- Débogage possible via l’affichage de capsules fantômes

### 3 Tir et validation serveur
**Client :**
- Effectue un Line Trace
- Si hit → envoie une RPC contenant :
  - Temps serveur du tir
  - Position d’origine
  - Rotation du tir
  - Joueur touché

**Serveur :**
- Récupère la frame correspondant au moment du tir  
  - Si manquante → interpolation linéaire entre deux frames
- Replace temporairement les capsules dans leur position historique
- Re-fait le Line Trace
- Confirme (ou non) le tir
- Restaure la position réelle des capsules

---

## Résultat attendu
Un prototype jouable intégrant :
- réplication complète des états joueurs,
- système de tâches interactif,
- synchronisation précise du temps,
- cycle lobby/partie automatisé,
- système de rewind garantissant une validation de tir juste malgré la latence.

---

## Structure du projet  
- `Code/Source/` : Contient le code C++ du projet  
- `Code/Content/` : Contient les éléments multimédias (Blueprints, textures, sons, etc.)  
- `Code/Config/` : Contient les fichiers de configuration du projet
- `Code/myproject.uproject` : Permet de lancer le jeux sur Unreal Engine
- `README.md` : Présentation et documentation du projet  

---

## Remarque  
Ce projet a été développé exclusivement dans un but académique et ne constitue pas un produit fini. Il s’agit d’une **preuve de concept** visant à mettre en application les principes la programmation de réseau multi-joueurs en ligne dans un contexte vidéoludique.  
