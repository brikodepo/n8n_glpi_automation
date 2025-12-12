# Prototype d'Assistance Technique Automatisée (Support V2)

Ce projet est un prototype développé dans le cadre de la modernisation du support interne. Il vise à réduire la charge du front-desk en automatisant la qualification, la priorisation et la création des tickets d'incidents dans l'outil ITSM (GLPI) à partir de la réception d'emails.

## Fonctionnalités

* **Réception des demandes** : Interception des emails envoyés au support via un serveur de messagerie conteneurisé.
* **Analyse par Intelligence Artificielle** : Utilisation du modèle Mistral AI pour analyser sémantiquement la demande, résumer le problème et déduire la catégorie métier.
* **Priorisation Automatique** : Classement de la criticité (Haute, Moyenne, Basse) selon une matrice de décision métier stricte.
* **Création de Ticket GLPI** : Génération automatique du ticket via l'API REST avec les champs pré-remplis (Titre, Description, Urgence).
* **Gestion des compléments** : Si la demande est trop vague, le système détecte le manque d'information et envoie automatiquement un email à l'utilisateur pour demander des précisions.

## Architecture Technique

[cite_start]L'infrastructure repose sur une architecture conteneurisée utilisant Docker et l'orchestrateur n8n[cite: 23].

| Composant | Rôle |
| :--- | :--- |
| **n8n** | Orchestrateur central pilotant le flux de données entre le mail, l'IA et GLPI. |
| **Docker Mailserver** | Serveur de messagerie local (Postfix/Dovecot) simulant la réception (`supportV2@decathlon.internal`). |
| **Mistral AI (API)** | LLM utilisé pour la compréhension du langage naturel et le formatage JSON. |
| **GLPI** | Solution ITSM finale accessible via API REST. |

## Prérequis et Configuration des APIs

Avant de lancer le workflow, il est nécessaire de configurer les accès aux services externes.

### 1. Configuration GLPI

Le workflow interagit avec GLPI via son API REST. Vous devez récupérer deux jetons d'authentification :

1.  **Activation de l'API** :
    * Dans GLPI, aller dans **Configuration > Générale > API**.
    * Activer l'API Rest.
    * Activer "Connexion avec identifiants externes".

2.  **App-Token (Jeton d'application)** :
    * Dans la section **Client API**, ajouter un client autorisé.
    * Générer et copier l'**App-Token**.

3.  **User-Token (Jeton utilisateur)** :
    * Aller dans le profil de l'utilisateur technique (ex: glpi_bot).
    * Dans l'onglet **Clés API**, générer le **User-Token** (jeton personnel).

### 2. Configuration Mistral AI

1.  Créer un compte sur la plateforme Mistral AI.
2.  Générer une clé API (API Key) pour permettre l'interrogation du modèle.

## Installation et Déploiement

### 1. Démarrage du Serveur Mail

Le fichier `docker-compose.yml` permet de monter le serveur de messagerie local.

```bash
docker-compose up -d

Le serveur expose les ports suivants pour la communication SMTP et IMAP:

SMTP : 25, 587

IMAP : 143, 993

### 2. Importation du Workflow n8n
Ouvrir l'interface n8n.

Créer un nouveau workflow.

Importer le contenu du fichier n8n_workflow.json.

Configuration du Workflow n8n
Une fois le workflow importé, vous devez mettre à jour les nœuds avec vos propres configurations.

Mise à jour des identifiants (Credentials)
Dans n8n, configurer les comptes suivants:

IMAP account : Identifiants du compte mail de réception (ex: supportV2@decathlon.internal).

SMTP account : Identifiants pour l'envoi des réponses automatiques.

Mistral Cloud account : Votre clé API Mistral.

Mise à jour des Nœuds HTTP Request (GLPI)
Le workflow contient deux nœuds nécessitant vos URLs et Tokens GLPI.

Nœud : HTTP Request (Initialisation Session)

URL : Remplacer http://192.168.1.202 par l'adresse IP ou le domaine de votre GLPI.

Header Authorization : Remplacer user_token CHANGEME par votre User-Token généré précédemment.

Header App-Token : Remplacer la valeur par votre App-Token.

Nœud : HTTP Request1 (Création Ticket)

URL : Mettre à jour l'IP vers votre GLPI.

Header App-Token : Remplacer la valeur CHANGEME par votre App-Token.

Mapping des Catégories (Script Javascript)
Le nœud Code in JavaScript fait correspondre l'analyse de l'IA avec les IDs de votre base de données GLPI. Vous devez adapter les IDs (entiers) selon votre installation.

Chercher la section suivante dans le code du nœud et modifier les valeurs numériques :

JavaScript

//⚠️ Mettre ici les VRAIS IDs GLPI (table itilcategories)
const CATEGORY_ID = {
  "serveur de fichier partage": 2, // Remplacer par l'ID GLPI correspondant
  "poste de travail": 1,
  "connexion internet": 3,
  "divers": 4,
};
