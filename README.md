# TP : Mise en place d'un système d'authentification centralisé avec Keycloak

## Objectif du TP

Vous êtes développeur(se) dans une entreprise qui souhaite centraliser la gestion de l’authentification pour ses différentes applications web internes. La direction souhaite mettre en place une solution de type SSO (Single Sign-On) grâce à **Keycloak**.

Dans le modèle traditionnel d'authentification client-serveur, une application cliente accède à une ressource protégée en utilisant directement les identifiants de l'utilisateur (nom d'utilisateur et mot de passe). Pour permettre à une application tierce d'accéder à ces ressources, l'utilisateur doit partager ses identifiants, ce qui pose plusieurs problèmes :

🔓 L'application tierce doit stocker les identifiants de l'utilisateur, souvent en clair.

⚠️ Le serveur doit gérer l'authentification par mot de passe, avec tous les risques que cela comporte.

🚪 L'application tierce obtient un accès trop large, sans limitation précise.

🔒 L'utilisateur ne peut pas révoquer l'accès à une seule application sans changer son mot de passe, ce qui impacte toutes les autres applications.

💥 Si une application tierce est compromise, toutes les données de l’utilisateur sont en danger.

Dans cd TP, **OpenID Connect (OIDC)** une couche d'identification s'appuyant sur **OAuth2** pour authentifier les utilisateurs et fournir des informations sur leur identité. Le principe est le suivant :

* L’utilisateur (le propriétaire de la ressource) autorise une application (le client) à accéder à certaines ressources sans partager ses identifiants.

* Le client obtient un jeton d’accès (access_token) émis par un serveur d’autorisation, qui précise :

  * ce à quoi il peut accéder,

  * pour combien de temps,

  * avec quelles permissions.

Par exemple, une utilisatrice peut autoriser un service d’impression à accéder à ses photos stockées sur un site sans donner son mot de passe. Elle s’authentifie sur un serveur de confiance (le serveur d’autorisation), qui délivre un jeton temporaire spécifique à cette autorisation.

**Votre mission consiste à :**

* Installer Keycloak localement
* Créer un répertoire d’authentification (realm), des utilisateurs, rôles, et applications clientes
* Comprendre les protocoles OAuth2 / OpenID Connect
* Intégrer Keycloak avec un client PHP, une API Node.js et une API Spring Boot
---
## Rôles OAuth 2.0 – Résumé

- **Resource Owner** : Possède les ressources protégées (souvent l’utilisateur).
- **Client** : Application qui demande l’accès à une ressource au nom du Resource Owner.
- **Authorization Server** : Authentifie le Resource Owner et délivre un access token au Client.
- **Resource Server** : Héberge les ressources protégées et accepte les access tokens valides.

> Le Resource Owner autorise le Client via l’Authorization Server, qui émet un access token. Le Client utilise ce token pour accéder aux données sur le Resource Server.

## 🗝️ Résumé – Types de Grant OAuth 2.0

### 🔐 Flux général OAuth 2.0 – Résumé

1. **(A) Le client demande l'autorisation**
   - Le client (application) demande l’autorisation d’accéder à une ressource.
   - Cette demande peut être faite directement à l’utilisateur (resource owner) ou, de préférence, via le **serveur d'autorisation**.

2. **(B) L’utilisateur accorde une autorisation**
   - Le client reçoit une **authorization grant** : une preuve d’autorisation (ex : code d'autorisation, jeton...).
   - Ce grant dépend du **type de flow** utilisé (`authorization_code`, `client_credentials`, etc.).

3. **(C) Le client demande un access token**
   - Le client **s'authentifie auprès du serveur d'autorisation**.
   - Il présente son authorization grant pour obtenir un **access token**.

4. **(D) Le serveur d’autorisation vérifie et délivre le token**
   - Il **authentifie le client**, vérifie la validité du grant.
   - Si tout est correct, il **renvoie un access token**.



### 🔐 Authorization Grant
Une **authorization grant** est une "preuve" (jeton intermédiaire) fournie par l'utilisateur, que le client utilise pour obtenir un **access token**. Un type de grant définit comment une application obtient un access token dans OAuth 2.0. C’est la porte d’entrée au système d’autorisation, choisie en fonction du contexte et du niveau de sécurité requis. OAuth 2.0 définit 4 types de grants :

### 1. Authorization Code (Standard flow and **grant_type=authorization_code** sur Keycloak)
- ⚙️ Utilisé dans le flow le plus sécurisé.
- 👤 L'utilisateur s'authentifie via un **serveur d'autorisation**.
- Le serveur d’autorisation redirige alors le propriétaire de la ressource (l’utilisateur) vers le client (l’application), en lui transmettant un **code temporaire** d’autorisation.
- 🧾 L’application échange ensuite ce **code temporaire** contre un **access_token** sécurisé aupres du **serveur d'autorisation**
- ✅ Les identifiants de l'utilisateur ne sont **jamais partagés** avec le client.
- 🔐 Permet d'authentifier le client et de garder le `access_token` hors du navigateur.
- 💥 un attaquant peut intercepter le code d’autorisation et l’échanger contre un token.
- Solution le coupler avec **PKCE (Proof Key for Code Exchange)**, un mécanisme de sécurité essentiel dans OAuth 2.0, surtout pour les applications mobiles ou SPA (JavaScript côté navigateur).
  - Le client génère :
    - un **code_verifier** (chaîne aléatoire secrète)
    - un **code_challenge = Hash(code_verifier), souvent SHA-256** et envoie **&code_challenge=XYZ&code_challenge_method=S256** au **serveur d'autorisation**
    - le client recoit un **code temporaire** d’autorisation.
    - le client échange ensuite ce **code temporaire** contre un **access_token** sécurisé aupres du **serveur d'autorisation** en lui revelant **code_verifier**.
  - PKCE dans Keycloak
    - Type de client : Public
    - Standard Flow Enabled : ✅
    - PKCE activé automatiquement dès qu'on envoie **code_challenge** dans la requête

### 2. Implicit (Implicit flow sur Keycloak)
- ⚙️ Flow simplifié (pour applications 100% JS dans un navigateur).
- Le client (SPA dans le navigateur) redemande directement un **access_token** via une redirection après login.
- Pas de code intermédiaire comme dans le flow Authorization Code.
- L’access_token est envoyé dans l’URL de redirection (fragment #access_token=...).
- 😬 Plus rapide, mais **moins sécurisé** : pas d'authentification du client, token exposé dans le navigateur.
- 🛑 Plus vulnérable à des attaques comme :
  - Token leakage
  - Replay attacks
  - XSS (cross-site scripting)
- ❗ Flow **déconseillé aujourd’hui**, remplacé par **PKCE**.

### 3. Resource Owner Password Credentials (ROPC), Direct access grants sur Keycloak
- 👥 L’utilisateur fournit **login + mot de passe directement au client**.
- 🔓 À utiliser uniquement si le **client est hautement fiable** (ex : OS, app système).
- 🎫 Le mot de passe est échangé contre un `access_token`
- dans Keycloak **Direct access grants**, **grant_type=password**
- ⚠️ Non recommandé dans des applications publiques ou tierces.

### 4. Client Credentials (Service accounts roles sur Keycloak)
- 🤖 Flow sans utilisateur : le **client agit en son propre nom**.
- 🔐 Utilise `client_id` + `client_secret` pour obtenir un access token.
- 📦 Idéal pour les **microservices, scripts, API internes**.
- 💡 Le client est aussi le "resource owner".

----
> ✅ À privilégier : Authorization Code + PKCE  
> ❌ À éviter sauf exception : Implicit, ROPC


## Refresh Token
### 🔁 Refresh Token (Jeton de renouvellement)

Un **refresh token** est un **jeton spécial** délivré par le serveur d'autorisation (ex : Keycloak), permettant à un client de **renouveler un access token expiré**, sans que l'utilisateur ait besoin de se reconnecter.

#### 📌 Caractéristiques :

- 💡 Délivré **en même temps que l’access token** (à l’étape (D) du flow OAuth).
- ⏳ Sert à obtenir un **nouvel access token** quand l’ancien expire.
- 🔐 **Jamais envoyé au resource server** (API) : utilisé uniquement entre **le client et le serveur d’autorisation**.
- 📦 Est **opaque** pour le client : c’est une chaîne unique, mais le client ne connaît pas son contenu.
- 🎯 Peut aussi être utilisé pour obtenir des tokens avec un **scope plus restreint**.
- 🧩 L’émission d’un refresh token est **optionnelle**, selon la politique du serveur.
- 🔄 Les refresh tokens ont souvent une **durée de vie plus longue** que les access tokens.
- ✅ Leur usage **limite la fréquence** des authentifications utilisateurs tout en maintenant la sécurité.

---

#### 🔁 Diagramme du flow OAuth avec refresh token

```text
+--------+                                           +---------------+
|        |--(A)------- Authorization Grant --------->|               |
|        |                                           |               |
|        |<-(B)----------- Access Token -------------|               |
|        |               & Refresh Token             |               |
|        |                                           |               |
|        |                            +----------+   |               |
|        |--(C)---- Access Token ---->|          |   |               |
|        |                            |          |   |               |
|        |<-(D)- Protected Resource --| Resource |   | Authorization |
| Client |                            |  Server  |   |     Server    |
|        |--(E)---- Access Token ---->|          |   |               |
|        |                            |          |   |               |
|        |<-(F)- Invalid Token Error -|          |   |               |
|        |                            +----------+   |               |
|        |                                           |               |
|        |--(G)----------- Refresh Token ----------->|               |
|        |                                           |               |
|        |<-(H)----------- Access Token -------------|               |
+--------+           & Optional Refresh Token        +---------------+

```

## 👥 Types de clients OAuth 2.0

OAuth distingue deux types de clients selon leur capacité à protéger leurs identifiants :

### 🔒 Client confidentiel
- Peut garder un `client_secret` sécurisé.
- Exécuté sur un serveur de confiance (ex : backend, API).

### 🌐 Client public
- Ne peut **pas** garder de secret.
- Exécuté sur l'appareil de l'utilisateur (ex : mobile, navigateur).

| Type de client   | Peut garder un secret ? | Exemples                 |
|------------------|--------------------------|---------------------------|
| Confidential     | ✅ Oui                   | Serveur backend, API     |
| Public           | ❌ Non                   | Mobile, JS, desktop      |


## Partie 1 – Installation de Keycloak (via Docker)

```bash
docker run -p 8080:8080 -e KEYCLOAK_ADMIN=admin -e KEYCLOAK_ADMIN_PASSWORD=admin quay.io/keycloak/keycloak:24.0.3 start-dev
```

Accéder à l’interface Keycloak : [http://localhost:8080](http://localhost:8080)

---

## Partie 2 – Configuration de Keycloak

### 2.1 Créer un Realm
Dans l’interface d’administration Keycloak, cliquer sur le menu en haut à gauche.

* Cliquer sur Realm settings, puis Add realm.

* Nommer le realm `tdsi` et cliquer sur Create.

### Qu'est-ce qu'un realm dans Keycloak ?

Un **realm** dans Keycloak est un **espace de gestion isolé** pour les utilisateurs, les applications et les règles de sécurité.

Concrètement :

- Un realm regroupe un ensemble d’utilisateurs, de rôles, de clients (applications) et de configurations d’authentification.
- Chaque realm est indépendant des autres, permettant de gérer plusieurs environnements ou groupes d’utilisateurs distincts dans la même instance Keycloak.
- Par exemple, une entreprise peut avoir un realm pour ses employés internes et un autre realm pour ses clients externes.
- Le realm est le **contexte dans lequel se déroule toute la gestion des identités** (connexion, rôles, permissions…).

### Que trouve-t-on dans un realm Keycloak ?

Un **realm** contient plusieurs éléments clés qui permettent de gérer l’authentification et l’autorisation :

- **Utilisateurs**  
  Comptes des personnes qui vont se connecter (ex : `alice`, `bob`).

- **Groupes**  
  Regroupements d’utilisateurs pour faciliter la gestion des permissions.

- **Rôles**  
  Définitions des droits et permissions attribués aux utilisateurs ou groupes (ex : `user`, `admin`).

- **Clients**  
  Applications ou services qui utilisent Keycloak pour authentifier les utilisateurs (ex : un site web, une API).

- **Identifiants des clients**  
  Clés et secrets utilisés par les clients pour s’authentifier auprès de Keycloak.

- **Protocoles**  
  Support des standards d’authentification comme OAuth2 et OpenID Connect.

- **Paramètres d’authentification**  
  Configuration des méthodes d’authentification (mot de passe, OTP, etc.).

- **Sessions**  
  Gestion des connexions actives des utilisateurs.

- **Événements**  
  Journalisation des actions liées à la sécurité (connexion, échec, etc.).

---

**En résumé :**  
Le realm est une zone où sont centralisés tous les éléments nécessaires pour gérer qui peut se connecter, comment, et avec quels droits.



### 2.2 Créer des Utilisateurs
#### 2.2.0 Required User Actions dans Keycloak

Les **Required User Actions** (actions utilisateur requises) sont des étapes supplémentaires que Keycloak peut demander à un utilisateur lors de sa connexion pour renforcer la sécurité ou compléter son profil.

##### Exemples courants d’actions requises :

- **Changer le mot de passe**  
  L'utilisateur doit modifier son mot de passe (par exemple après une première connexion ou suite à une politique de sécurité).

- **Configurer l’authentification à deux facteurs (2FA)**  
  L’utilisateur doit activer et configurer un moyen d’authentification supplémentaire (comme une application OTP).

- **Vérifier l’adresse e-mail**  
  L’utilisateur doit confirmer son adresse e-mail via un lien envoyé.

- **Configurer les informations du profil**  
  Compléter ou mettre à jour des informations personnelles obligatoires.

- **Déconnecter d'autres sessions**  
  Inviter l'utilisateur à fermer d'autres connexions actives.

##### Pourquoi utiliser les Required User Actions ?

- Pour améliorer la sécurité du système d’authentification.
- Pour s’assurer que les données utilisateurs sont complètes et à jour.
- Pour imposer des politiques de sécurité avant d’autoriser l’accès aux ressources.

---

**Résumé :**  
Les Required User Actions sont des tâches que Keycloak force un utilisateur à réaliser lors de la connexion afin d’assurer la conformité et la sécurité de l’accès.

**Créer des Utilisateurs**
- Aller dans Users > Add user.

Exemple :

- Username : `alice`

- Email : alice@example.com

- Cliquer sur `Create`

- Dans l’onglet Credentials, définir un mot de passe (`alice123`) et désactiver Temporary.

  Répéter pour bob (mot de passe : `bob123`).

### 2.3 Créer des Rôles
- Aller dans Roles > Add Role
- Créer
- -  `user`
- - `admin`

### 2.4 Associer les rôles aux utilisateurs

Pour attribuer des rôles aux utilisateurs dans Keycloak, suivez ces étapes :

1. Connectez-vous à l'interface d'administration Keycloak :  
   `http://localhost:8080/admin`

2. Sélectionnez le **realm** créé (ex. `tdsi`).

3. Dans le menu de gauche, cliquez sur **Users** (Utilisateurs).

4. Cliquez sur l'utilisateur auquel vous voulez attribuer des rôles (ex. `alice`).

5. Allez dans l'onglet **Role Mappings** (Attribution des rôles).

6. Dans la liste des rôles disponibles, sélectionnez un ou plusieurs rôles (ex. `user`, `admin`).

7. Cliquez sur **Add selected** (Ajouter la sélection).

8. Les rôles choisis apparaissent désormais dans la section **Assigned Roles** (Rôles attribués).

---

**Note :**  
Les rôles attribués définissent les permissions et droits de l'utilisateur dans les applications clientes qui utilisent ce realm.


### 2.5 Créer des Clients

####  Configuration d’un client Keycloak pour une API sécurisée

Cette documentation explique comment configurer un client dans Keycloak pour sécuriser une API en utilisant le **mot de passe utilisateur** ou **l'identité du client** (service account).

---

##### 1. Création de clients

Architecture OAuth 2.0 avec App Mobile, Backend et Keycloak

```text
[ Utilisateur ]
     ⇅
[ App mobile ] ⇄ (token) ⇄ [ Backend API ]
     ⇅                     ⇅ 
 (PKCE Flow)           [ Keycloak ]
```

| Composant   | Rôle OAuth 2.0       | Type de client | Détails                                         |
| ----------- | -------------------- | -------------- | ----------------------------------------------- |
| App mobile  | Client               | Public         | Utilise le flow Authorization Code + PKCE       |
| Backend API | Resource Server      | Confidential\* | Vérifie les access\_token JWT                   |
| Keycloak    | Authorization Server | —              | Authentifie l’utilisateur et délivre les tokens |

- L'app mobile ne stocke aucun secret (client public).

- Le backend ne fait pas d’authentification directe, il vérifie seulement les tokens JWT reçus.

- Keycloak doit être configuré avec un client public et le PKCE activé automatiquement pour renforcer la sécurité.

###### a) CLient **Backend API**
- Aller dans l’interface d’administration Keycloak.
- Naviguer dans le *realm* souhaité.
- Cliquer sur **Clients** > **Create client**.

Remplir les champs suivants :

1. **Client ID** : `api-node`
2. **Root URL** : `http://localhost:3000`
3. Cliquer sur **Save**
4. Dans la configuration :
   - **Client type** : `Confidential` Le client est une application serveur qui peut stocker un client_secret en toute sécurité.
   - **Standard Flow Enabled** : `OFF` Pas de redirection : l'API ne gère pas d'interface utilisateur.
   - **Direct Access Grants Enabled** : `ON` Permet à un utilisateur d’obtenir un token avec login + mot de passe (utile pour l’authentification directe depuis un script ou une app mobile).
   - **Service Accounts Enabled** : `ON` Permet à l'API de s’authentifier en tant que client, sans utilisateur (compte technique).
5. Dans l'onglet **Credentials**, noter le `Client Secret`

Cliquer sur **Save**.


###### Configuration du client

Une fois le client créé, configurer comme suit :

| Champ                             | Valeur         | Description                                                                 |
|----------------------------------|----------------|-----------------------------------------------------------------------------|
| **Client Type**                  | `Confidential` | Le client garde un `Client Secret` en toute sécurité (ex : backend). En Keycloak : Un client public est destiné à des applications comme des applis **mobiles** ou des *Single Page Applications (SPA)*, qui ne peuvent pas garder secret un mot de passe ou un client secret. Un client confidentiel est utilisé pour les applications back-end qui peuvent stocker un secret et donc être authentifiées de manière sécurisée. L'accès au service account est réservé aux clients confidentiels.      |
| **Standard Flow Enabled**        | `OFF` (API)         | Pas de flux d'authentification basé sur le navigateur ( `ON` site web).                     |
| **Direct Access Grants Enabled** | `OFF`           | Permet à un utilisateur de s’authentifier avec login/mot de passe sur keycloak (si `ON` L’application peut faire une requête directe à Keycloak avec le nom d’utilisateur et le mot de passe, et recevoir un jeton d’accès.).         |
| **Service Accounts Enabled**     | `ON`           | Permet à l’application(micro service) elle-même de s’authentifier sans utilisateur à Keycloak avec ses propres identifiants (`client_id`, `client_secret`).       |

---

###### Récupération du `Client Secret`

- Aller dans l’onglet **Credentials**.
- Copier le **Client Secret** généré.

Exemple (Ce backend ne fera que verifier les Access_token aupres de Keycloak) :

- Client ID     : api-node
- Client Secret : WWrZ3E7CnwkFJsZSQ2YgXH2uD9n637xa

---
###### b) Client **App mobile** (Postman)

L'application mobile est configurée comme un **client public** utilisant le **flow Authorization Code + PKCE**. On va le tester sur **Postman**

##### Étapes de création du client dans Keycloak

- Aller dans l’interface d’administration Keycloak.
- Naviguer dans le *realm* souhaité.
- Cliquer sur **Clients** > **Create client**.

Remplir les champs :

- **Client ID** : `mobile-app`  
- **Root URL** : `vide` ou l’URL de redirection de l’application mobile (AndroidManifest.xml pour Android, info.plist pour iOS).
- Valid redirect URIs:`*` 
- Cliquer sur **Save**
- Aller dans l’onglet "Avancé" du client enregistré
- Aller dans l’onglet Advanced Settings (ou "Settings", puis tout en bas)
- Repérer le champ Proof Key for Code Exchange Code Challenge Method (ou juste PKCE Code Challenge Method selon la version).
- Définir le mode de challenge sur S256 (SHA-256 pour transformer le code_verifier en code_challenge, empêchant toute interception de token.)

---

###### Configuration recommandée

| Champ                       | Valeur     | Description                                                                                         |
|----------------------------|------------|---------------------------------------------------------------------------------------------------|
| **Client Type**             | `Public`   | Pas de client secret, adapté aux applications mobiles ou SPA où le secret ne peut pas être gardé.  |
| **Standard Flow Enabled**   | `ON`       | Active le flow Authorization Code nécessaire pour PKCE.                                           |
| **Direct Access Grants Enabled** | `OFF`  | Désactivé pour éviter l’usage de Resource Owner Password Credentials Grant.                        |
| **Implicit Flow Enabled**   | `OFF`       | Désactivé, car PKCE remplace l'implicit flow pour plus de sécurité.                               |
| **Service Accounts Enabled**| `OFF`       | Non nécessaire pour une app mobile (pas d’authentification machine-to-machine).                   |
| **Valid Redirect URIs**     | * | URI de redirection personnalisée utilisée par l’application mobile pour récupérer le code. |

---

###### Notes importantes

- L'app mobile utilise le **PKCE** (Proof Key for Code Exchange) pour sécuriser le flow Authorization Code sans secret.
- Aucun **Client Secret** n’est stocké dans l’application mobile.
- Le backend valide les tokens reçus et n’a pas besoin de gérer la connexion utilisateur.

---








---

##  Partie 4 – Test avec un client 
### 4.1 Créer une API Express

```bash
npm init -y
npm install express keycloak-connect session-file-store express-session
```
creer un fichier index.js
```js
const express = require('express');
const session = require('express-session');
const Keycloak = require('keycloak-connect');

const app = express();

const memoryStore = new session.MemoryStore();

app.use(session({
  secret: 'un_secret_pour_session', // change en production
  resave: false,
  saveUninitialized: true,
  store: memoryStore
}));

const keycloak = new Keycloak({
  store: memoryStore
}, {
  realm: 'tdsi',                        // Ton realm Keycloak
  'auth-server-url': 'http://localhost:8080/auth', // URL Keycloak
  'ssl-required': 'external',
  resource: 'backend-api',              // Client ID dans Keycloak
  'public-client': false,               // client confidentiel
  credentials: {
    secret: 'WWrZ3E7CnwkFJsZSQ2YgXH2uD9n637xa'  // Client secret
  }
});

app.use(keycloak.middleware());

app.get('/', (req, res) => {
  res.send('API publique');
});

app.get('/protected', keycloak.protect(), (req, res) => {
  res.json({ message: 'Accès autorisé à la ressource protégée' });
});

const PORT = 3000;
app.listen(PORT, () => {
  console.log(`API démarrée sur http://localhost:${PORT}`);
});
```
Lancer le client avec 
```bash
node ./index.js
```
Tester l’API
Dans un navigateur ou Postman, ouvre :
- http://localhost:3000/ → route publique

- http://localhost:3000/protected → route protégée (tu dois envoyer un token valide)
---

### 4.2 Test avec PostMan
Objectif
Simuler une application mobile publique qui utilise PKCE pour obtenir un access token depuis Keycloak via Postman.

1. 🚀 Configuration de Postman
a. Ouvrir l’onglet Authorization
Type : OAuth 2.0


b. Renseigner les champs :

| Champ                     | Valeur                                                                     |
| ------------------------- | -------------------------------------------------------------------------- |
| **Token Name**            | `Keycloak-PKCE-Test` (nom libre)                                           |
| **Grant Type**            | `Authorization Code (With PKCE)`                                           |
| **Callback URL**          | `https://oauth.pstmn.io/v1/callback`                                       |
| **Auth URL**              | `http://localhost:8080/realms/<votre_realm>/protocol/openid-connect/auth`  |
| **Access Token URL**      | `http://localhost:8080/realms/<votre_realm>/protocol/openid-connect/token` |
| **Client ID**             | `mobile-app` (ou le client\_id que vous avez défini dans Keycloak)         |
| **Code Challenge Method** | `S256`                                                                     |
| **Scope**                 | `openid` ou autre (selon vos besoins)                                      |
| **Client Authentication** | `None` (car c’est un client public)                                        |


2. ✅ Récupération du Token
Cliquer sur Get New Access Token

S’authentifier via la page Keycloak qui s’ouvre

Autoriser si nécessaire

Être redirigé vers Postman avec le token

Cliquer sur Use Token

3. 🔁 Utiliser le token dans une requête API
Postman ajoute automatiquement le token dans l’en-tête :
```text
Authorization: Bearer <access_token>
```
Lancer une requête vers votre API protégée.


##  Rapport attendu

* Captures d’écran des interfaces Keycloak, clients, utilisateurs, etc.
* Commandes d’installation
* Code des clients PHP / Node.js / Spring Boot
* Token JWT et réponses d’API
* Questions de compréhension :

  * Différence entre OAuth2 et OpenID Connect ?
  * Que contient un JWT ?
  * Comment Keycloak vérifie-t-il les permissions ?

---

**Fin du TP**

