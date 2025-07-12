# 🕵️ Mission CyberSec : Infiltration et Sécurisation
**TP OpenSSL - Sécurité Réseaux et Systèmes M1 TDSI/LACGAA**

## 📋 Briefing de Mission

### 🎯 Contexte
Vous êtes des **consultants en cybersécurité** travaillant pour l'agence **CyberGuard Sénégal**. Votre équipe de 2 experts a été missionnée pour sécuriser les communications d'une entreprise technologique sénégalaise qui a récemment subi des tentatives d'intrusion. 

**Entreprise cliente :** TechSN Solutions  
**Secteur :** Fintech (paiements mobiles)  
**Menace :** Interception de communications sensibles  
**Objectif :** Mettre en place un système de communication sécurisé

### 🚨 Situation d'urgence
Des hackers ont tenté d'intercepter les communications entre les serveurs de TechSN. Vous devez :
1. Évaluer la sécurité actuelle des systèmes
2. Implémenter des solutions cryptographiques robustes
3. Former les équipes aux bonnes pratiques
4. Tester la résistance du système aux attaques

---

## 🛠️ Présentation de l'Arme Ultime : OpenSSL

### 🌟 OpenSSL : L'Arsenal de la Cybersécurité

**OpenSSL** n'est pas juste un outil, c'est **l'arsenal complet** du spécialiste en cybersécurité ! Développé depuis 1998, il est devenu l'épine dorsale de la sécurité Internet.

### 🚀 Pourquoi OpenSSL domine le terrain ?

#### **📊 Impact dans le monde réel**
OpenSSL est utilisé massivement dans l'industrie :
- Serveurs web (Apache, Nginx)
- Applications mobiles et navigateurs
- Systèmes de paiement en ligne
- Infrastructure cloud et datacenter

#### **⚡ Arsenal cryptographique complet**
```bash
# Découvrez votre arsenal ultime
openssl version -a    # Révèle les capacités de votre arme
```

**Armes disponibles dans OpenSSL :**
- 🔐 **Chiffrement moderne** (AES, RSA, courbes elliptiques...)
- 🔏 **Fonctions de hachage** (SHA-256, SHA-3, BLAKE2...)
- 📜 **Gestion PKI complète** (certificats X.509, CRL, OCSP...)
- 🌐 **Protocoles sécurisés** (TLS/SSL, S/MIME, CMS...)
- 📊 **Tests et diagnostics** réseau avancés

#### **🏆 Avantages stratégiques**

| Aspect | Description | Impact |
|--------|-------------|--------|
| **🆓 Open Source** | Code libre et auditable | Confiance et transparence |
| **🌍 Multiplateforme** | Linux, Windows, macOS | Déploiement universel |
| **⚡ Performance** | Optimisé et rapide | Efficacité opérationnelle |
| **🔒 Sécurité** | Standards éprouvés | Fiabilité militaire |
| **📚 Maturité** | 25+ ans de développement | Stabilité légendaire |

### 🎖️ OpenSSL face à la concurrence

| Outil | Spécialité | Limitations |
|-------|------------|-------------|
| **GnuPG** | Chiffrement PGP | Spécialisé emails |
| **Windows CryptoAPI** | Intégration Windows | Écosystème fermé |
| **Java Crypto** | Applications Java | Performance limitée |
| **LibreSSL** | Fork sécurisé | Écosystème plus restreint |
| **🏆 OpenSSL** | **Arsenal complet** | **Domination totale** |

### 🚀 Architecture de votre laboratoire cryptographique

```bash
# Explorez votre arsenal
openssl list -commands    # Toutes vos armes disponibles
openssl list -ciphers     # Algorithmes de chiffrement
openssl list -digests     # Fonctions de hachage
```

#### **🧱 Structure de l'arsenal**
1. **🔧 Bibliothèque libcrypto** : Munitions cryptographiques de base
2. **🌐 Bibliothèque libssl** : Protocoles de communication sécurisée
3. **⌨️ Interface CLI** : Votre poste de commande

### 🎯 Pourquoi maîtriser cette arme ultime ?

#### **📈 Demande explosive sur le marché**
La cybersécurité est un secteur en forte croissance, et OpenSSL est omniprésent dans :
- L'administration système et réseau
- Le développement sécurisé
- L'audit et les tests de pénétration
- L'architecture de sécurité

#### **🌐 Technologies d'avenir**
OpenSSL s'adapte aux défis futurs :
- Internet des objets (IoT) et 5G
- Cloud computing et edge computing
- Blockchain et cryptomonnaies
- Intelligence artificielle

#### **🔮 Évolution spectaculaire**
OpenSSL évolue constamment :
- **Cryptographie post-quantique** (résistance aux ordinateurs quantiques)
- **TLS 1.3** : Protocoles de nouvelle génération
- **Automatisation** des certificats

### 🎪 Démonstration de puissance

#### **🚀 Test de performance spectaculaire**
```bash
# Benchmark : Testez la puissance de votre machine
openssl speed aes-256-cbc
openssl speed rsa2048
```

#### **🎭 Simulation défensive**
```bash
# Test de génération d'entropie cryptographique
time openssl rand -hex 32
```

### 🎓 Votre formation d'élite

Au cours de cette mission légendaire, vous allez maîtriser :

1. **🎯 Phase Reconnaissance** : Démystifier l'encodage vs chiffrement
2. **🛡️ Phase Défensive** : Déployer le chiffrement symétrique AES
3. **🗝️ Phase Asymétrique** : Manipuler RSA et les protocoles hybrides
4. **✍️ Phase Authentification** : Forger des signatures numériques
5. **🏛️ Phase Infrastructure** : Construire une PKI complète

### 💪 Challenge spectaculaire : Premier contact

```bash
# Votre première mission avec l'arsenal ultime
openssl version -a
```

**🎖️ Objectif :** Devenir un expert OpenSSL redoutable !

---

## 🎯 Phase 0 : Maîtrise de l'Arsenal OpenSSL

### 🚀 Mission Préliminaire : Reconnaissance de terrain

**🎭 Scénario :** Avant toute mission, un agent d'élite doit connaître parfaitement ses armes. OpenSSL est votre arsenal principal - vous devez le dominer !

#### **🔍 Mission 0.1 - Inventaire de l'arsenal**
```bash
# Découvrir votre version d'OpenSSL
openssl version -a

# Analyser les capacités disponibles
openssl help

# Compter vos armes cryptographiques
openssl help 2>&1 | grep "^[a-z]" | wc -l
```

#### **💥 Mission 0.2 - Test de puissance explosive**
```bash
# Tester la génération d'entropie
openssl rand -hex 16

# Explorer votre arsenal de chiffrement
openssl list -cipher-algorithms

# Découvrir vos fonctions de hachage
openssl list -digest-algorithms
```

#### **📚 Mission 0.3 - Manuel de l'agent d'élite**
```bash
# Consulter la documentation secrète
man openssl

# Aide spécialisée pour vos armes favorites
openssl enc -help
openssl genrsa -help
```

#### **🧠 Questions techniques de niveau expert**

**🔬 Défi Entropie :**
```bash
# Expérience 1 : Générer différentes quantités d'entropie
time openssl rand -hex 16    # 16 octets
time openssl rand -hex 1024  # 1 KB
time openssl rand -hex 10240 # 10 KB
```

**Questions d'investigation :**
1. **Qu'est-ce que l'entropie** en cryptographie ? Pourquoi est-elle cruciale ?
2. **D'où provient l'entropie** sur votre système Linux ? (Indice : `/dev/random` vs `/dev/urandom`)
3. **Pourquoi le temps de génération** augmente-t-il avec la quantité ? Que se passe-t-il si l'entropie s'épuise ?
4. **Que signifie "cryptographiquement sûr"** pour un générateur de nombres aléatoires ?

**🔍 Défi Architecture :**
```bash
# Analyser la compilation d'OpenSSL
openssl version -a | grep -E "(built|platform|compiler)"

# Explorer les engines cryptographiques
openssl engine -t
```

**Questions d'approfondissement :**
5. **Qu'est-ce qu'un "engine"** dans OpenSSL ? Quel est l'intérêt du hardware acceleration ?
6. **Pourquoi OpenSSL est-il compilé** différemment selon les plateformes ?
7. **FIPS mode** : À quoi sert cette option ? Dans quels contextes est-elle obligatoire ?

**⚡ Défi Performance :**
```bash
# Benchmarker différents algorithmes
openssl speed aes-128-cbc aes-256-cbc
openssl speed rsa1024 rsa2048 rsa4096
openssl speed sha1 sha256 sha512
```

**Questions d'analyse :**
8. **Pourquoi AES-128 est-il plus rapide** qu'AES-256 ? La sécurité en souffre-t-elle vraiment ?
9. **Relation taille/temps RSA** : Expliquez mathématiquement pourquoi RSA-4096 est exponentiellement plus lent.
10. **SHA-1 vs SHA-256** : Lequel est le plus rapide ? Pourquoi SHA-1 est-il déconseillé malgré sa rapidité ?

**🎯 Questions de debriefing spectaculaire :**
1. Combien d'armes cryptographiques avez-vous à disposition ?
2. Quels algorithmes de chiffrement légendaires reconnaissez-vous ?
3. Votre arsenal est-il prêt pour la mission ?

**🧩 Défis de recherche avancée :**
- **Recherchez** : Qu'est-ce que l'attaque "Heartbleed" et comment a-t-elle affecté OpenSSL ?
- **Investiguez** : Pourquoi OpenSSL 1.1.1 sera-t-il "End of Life" en septembre 2023 ?
- **Explorez** : Quelle est la différence entre OpenSSL et LibreSSL ?

---

## 🔐 Phase 1 : Évaluation des Vulnérabilités

### 🎭 Scénario : Le Piège du Faux Ami
**Contexte :** Un employé de TechSN a reçu un message "secret" de son collègue, mais ce message était en fait envoyé par un attaquant utilisant un simple encodage Base64.

**🎯 Mission A1 - Démonstration de Vulnérabilité**
```bash
# Agent A (vous) : Préparer le piège
echo "URGENT: Mot de passe temporaire du serveur: Admin2024!" > secret_message.txt
openssl enc -base64 -in secret_message.txt -out intercepted_msg.txt

# Transmettre le message encodé à votre binôme
cat intercepted_msg.txt
```

**🎯 Mission A2 - Contre-espionnage**
```bash
# Agent B (binôme) : Décoder le message intercepté
openssl enc -base64 -d -in intercepted_msg.txt -out decoded_msg.txt
cat decoded_msg.txt
```

**🚨 Débrief critique :**
- Combien de temps avez-vous mis pour décoder le message ?
- Un attaquant pourrait-il faire de même ?
- **Conclusion :** Base64 = Fausse sécurité !

**🧠 Questions techniques approfondies :**

**🔬 Défi Base64 :**
11. **Qu'est-ce que Base64** exactement ? Pourquoi utilise-t-il 64 caractères ?
12. **Calculez** : Un fichier de 100 octets, quelle sera sa taille après encodage Base64 ? (Formule mathématique attendue)
13. **Padding** : À quoi servent les caractères `=` à la fin des chaînes Base64 ?
14. **Sécurité par obscurité** : Donnez 3 exemples réels où Base64 est utilisé à tort comme "sécurité".

**⚔️ Défi Cryptanalyse :**
```bash
# Expérience : Encoder différents types de contenu
echo "password123" | openssl enc -base64
echo "motdepasse123" | openssl enc -base64
echo "123password" | openssl enc -base64
```

**Questions d'investigation :**
15. **Patterns Base64** : Peut-on deviner le contenu original en analysant l'encodage ?
16. **Dictionnaire d'attaque** : Comment un attaquant pourrait-il automatiser le décodage de milliers de chaînes Base64 ?
17. **Détection automatique** : Écrivez un one-liner bash pour détecter si une chaîne est en Base64.

---

## 🛡️ Phase 2 : Sécurisation des Communications

### 🎭 Scénario : Opération "Coffre-Fort Numérique"
**Contexte :** TechSN doit transmettre des données financières sensibles. Vous devez établir un canal sécurisé entre deux agents sur le terrain.

**🎯 Mission B1 - Établissement du Canal Sécurisé**
```bash
# Découvrir les armes cryptographiques disponibles
openssl enc -list | grep -i aes

# Créer un document ultra-confidentiel
echo "CONFIDENTIEL DÉFENSE
Transaction ID: TSN-2024-789456
Montant: 50,000,000 FCFA
Code d'autorisation: ALPHA-BRAVO-CHARLIE-DELTA
Destinataire: Banque Centrale du Sénégal" > document_classifie.txt
```

**🎯 Mission B2 - Chiffrement de Niveau Militaire**
```bash
# Agent A : Chiffrer avec AES-256 (niveau militaire)
openssl enc -aes-256-cbc -salt -in document_classifie.txt -out document_chiffre.enc -k "OperationCoffreFort2024!"

# Simuler la transmission (copier le fichier)
cp document_chiffre.enc transmission_secure.enc
```

**🎯 Mission B3 - Test de Résistance**
```bash
# Agent B : Tentative de déchiffrement avec mauvaise clé
openssl enc -d -aes-256-cbc -in transmission_secure.enc -out tentative_piratage.txt -k "MotDePasseFacile123"

# Puis déchiffrement avec la vraie clé (communiquée oralement)
openssl enc -d -aes-256-cbc -in transmission_secure.enc -out document_decode.txt -k "OperationCoffreFort2024!"
```

**🎯 Analyse forensique :**
```bash
# Comparer les tailles des fichiers
ls -la document_classifie.txt document_chiffre.enc
```

**Questions de debriefing :**
- Pourquoi le fichier chiffré est-il plus volumineux ?
- Que contient le "sel" cryptographique ?

**🧠 Questions techniques de haut niveau :**

**🔬 Défi Chiffrement Symétrique :**
18. **AES vs DES** : Pourquoi DES est-il considéré comme obsolète ? Quelle est la taille de clé minimale recommandée aujourd'hui ?
19. **Modes de chiffrement** : Quelle est la différence entre CBC, GCM, et CTR ? Lequel offre l'authentification ?
20. **Salage cryptographique** : À quoi sert exactement le `-salt` ? Que se passe-t-il si on l'omet ?
21. **PBKDF2** : Pourquoi utiliser `-pbkdf2` plutôt qu'un simple hachage du mot de passe ?

**⚡ Défi Sécurité :**
```bash
# Expérience : Chiffrer le même contenu plusieurs fois
echo "message secret" | openssl enc -aes-256-cbc -salt -k "password" | openssl enc -base64
echo "message secret" | openssl enc -aes-256-cbc -salt -k "password" | openssl enc -base64
```

**Questions d'investigation :**
22. **Pourquoi les résultats sont-ils différents** alors que le contenu et le mot de passe sont identiques ?
23. **Attaque par dictionnaire** : Comment le salage protège-t-il contre les rainbow tables ?
24. **IV (Initialization Vector)** : Quel est son rôle ? Que se passerait-il avec un IV fixe ?

**🛡️ Défi Performance vs Sécurité :**
```bash
# Comparer différents algorithmes
time openssl enc -des-cbc -salt -in gros_fichier.txt -out des.enc -k "password"
time openssl enc -aes-128-cbc -salt -in gros_fichier.txt -out aes128.enc -k "password"  
time openssl enc -aes-256-cbc -salt -in gros_fichier.txt -out aes256.enc -k "password"
```

**Questions d'analyse :**
25. **Compromis sécurité/performance** : Dans quels cas préférer AES-128 à AES-256 ?
26. **Attaque par force brute** : Combien de temps faudrait-il pour casser AES-256 avec la technologie actuelle ?
27. **Cryptographie post-quantique** : Pourquoi AES résistera-t-il aux ordinateurs quantiques ?

---

## 🗝️ Phase 3 : Opération "Clés Asymétriques"

### 🎭 Scénario : Réseau d'Espionnage International
**Contexte :** Vos deux agents doivent établir un canal de communication sécurisé sans avoir pu se rencontrer au préalable. Ils opèrent depuis Dakar et Saint-Louis.

**🎯 Mission C1 - Génération des Identités Cryptographiques**
```bash
# Agent A (Dakar) : Créer son identité secrète
openssl genrsa -out agent_dakar.pem 2048

# Examiner votre identité (TOP SECRET)
openssl rsa -in agent_dakar.pem -text -noout | head -20
```

**🎯 Mission C2 - Identité Renforcée**
```bash
# Agent B (Saint-Louis) : Créer une identité ultra-protégée
openssl genrsa -des3 -out agent_saint_louis.pem 2048
# Mot de passe suggéré : "SaintLouis2024Secure!"

# Vérifier la protection
cat agent_saint_louis.pem
```

**🎯 Mission C3 - Échange d'Identités Publiques**
```bash
# Chaque agent extrait sa carte d'identité publique
openssl rsa -in agent_dakar.pem -pubout -out identite_publique_dakar.pem
openssl rsa -in agent_saint_louis.pem -pubout -out identite_publique_saint_louis.pem

# Afficher les identités publiques
echo "=== IDENTITÉ PUBLIQUE AGENT DAKAR ==="
cat identite_publique_dakar.pem
echo "=== IDENTITÉ PUBLIQUE AGENT SAINT-LOUIS ==="
cat identite_publique_saint_louis.pem
```

**📧 Transmission sécurisée :** Échangez vos identités publiques par email (simulation d'un canal compromis mais acceptable pour les clés publiques).

**🎯 Mission C4 - Test de Limitation RSA**
```bash
# Créer un petit message secret
echo "RDV demain 14h Plateau" > message_court.txt

# Chiffrer avec RSA (ça marche)
openssl rsautl -encrypt -pubin -inkey identite_publique_binome.pem -in message_court.txt -out message_rsa.enc

# Tenter avec un rapport complet
dd if=/dev/zero of=rapport_mission.txt bs=1024 count=1
echo "RAPPORT MISSION CONFIDENTIEL..." >> rapport_mission.txt

# Tentative de chiffrement (va échouer)
openssl rsautl -encrypt -pubin -inkey identite_publique_binome.pem -in rapport_mission.txt -out rapport_rsa.enc
```

**🚨 Problème identifié :** RSA ne peut pas chiffrer de gros volumes !

**🎯 Mission C5 - Protocole Hybride "Diplomate"**
```bash
# Agent A : Opération combinée
# 1. Créer un dossier sensible
echo "PLAN SÉCURITÉ FINTECH SÉNÉGAL 2024
- Implémentation blockchain gouvernementale
- Partenariat avec banques centrales UEMOA
- Budget alloué: 2 milliards FCFA
- Échéance: Décembre 2024" > plan_strategic.txt

# 2. Chiffrer avec AES (rapidité)
openssl enc -aes-256-cbc -salt -in plan_strategic.txt -out plan_chiffre.enc -k "CleTemporaire2024"

# 3. Chiffrer la clé AES avec RSA (sécurité)
echo "CleTemporaire2024" > cle_aes.txt
openssl rsautl -encrypt -pubin -inkey identite_publique_binome.pem -in cle_aes.txt -out cle_chiffree.enc

# 4. Transmission des deux fichiers
echo "Transmission sécurisée prête: plan_chiffre.enc + cle_chiffree.enc"
```

**🎯 Mission C6 - Déchiffrement par l'Agent B**
```bash
# Agent B : Récupération des données
# 1. Déchiffrer la clé AES
openssl rsautl -decrypt -inkey agent_saint_louis.pem -in cle_chiffree.enc -out cle_recuperee.txt

# 2. Utiliser la clé pour déchiffrer le document
CLE_AES=$(cat cle_recuperee.txt)
openssl enc -d -aes-256-cbc -in plan_chiffre.enc -out plan_decode.txt -k "$CLE_AES"

# 3. Vérifier la mission
cat plan_decode.txt
```

---

## ✍️ Phase 4 : Opération "Sceau Royal"

### 🎭 Scénario : Authentification des Ordres de Mission
**Contexte :** Des faux ordres de mission circulent. Vous devez mettre en place un système d'authentification inviolable.

**🎯 Mission D1 - Théorie du Sceau**
**Question stratégique :** Pour authentifier un ordre de mission, utilisez-vous votre clé publique ou privée ? Pourquoi ?

**🎯 Mission D2 - Création du Sceau Numérique**
```bash
# Agent A : Créer un ordre de mission officiel
echo "ORDRE DE MISSION N°2024-CYBER-001
Agent: $(whoami)
Mission: Sécurisation infrastructure TechSN
Autorisation: Direction Générale CyberGuard
Date limite: 31/12/2024
Classification: SECRET DÉFENSE" > ordre_mission.txt

# Apposer le sceau numérique
openssl dgst -sha256 -sign agent_dakar.pem -out sceau_numerique.sig ordre_mission.txt

# Préparer la transmission
echo "Ordre de mission signé et scellé prêt pour transmission"
```

**🎯 Mission D3 - Opération "Faux Documents"**
```bash
# Créer trois ordres de mission
echo "ORDRE MISSION A - Mission reconnaissance" > ordre_a.txt
echo "ORDRE MISSION B - Mission infiltration" > ordre_b.txt  
echo "ORDRE MISSION C - Mission extraction" > ordre_c.txt

# Signer tous les ordres
openssl dgst -sha256 -sign agent_dakar.pem -out ordre_a.sig ordre_a.txt
openssl dgst -sha256 -sign agent_dakar.pem -out ordre_b.sig ordre_b.txt
openssl dgst -sha256 -sign agent_dakar.pem -out ordre_c.sig ordre_c.txt

# SABOTAGE : Modifier secrètement un ou deux ordres
echo "ORDRE MISSION A - Mission reconnaissance ANNULÉE" > ordre_a.txt
echo "ORDRE MISSION B - Mission infiltration REPORTÉE" > ordre_b.txt
```

**🕵️ Mission D4 - Contre-espionnage**
```bash
# Agent B : Vérifier l'authenticité des ordres reçus
openssl dgst -sha256 -verify identite_publique_dakar.pem -signature ordre_a.sig ordre_a.txt
openssl dgst -sha256 -verify identite_publique_dakar.pem -signature ordre_b.sig ordre_b.txt
openssl dgst -sha256 -verify identite_publique_dakar.pem -signature ordre_c.sig ordre_c.txt
```

**🎯 Rapport d'enquête :** Quels ordres ont été falsifiés ?

**🧠 Questions techniques avancées :**

**🔬 Défi Signature Numérique :**
41. **Mathématiques RSA** : Expliquez la relation entre signature et déchiffrement. Pourquoi utilise-t-on la clé privée ?
42. **Hash-then-sign** : Pourquoi signer le hash plutôt que le document directement ?
43. **Collision SHA-256** : Que se passerait-il si on trouvait deux documents avec le même hash ?
44. **Non-répudiation** : Comment une signature prouve-t-elle légalement l'identité du signataire ?

**⚔️ Défi Attaques sur les Signatures :**
```bash
# Expérience : Tenter une attaque par substitution
cp ordre_c.txt ordre_malveillant.txt
cp ordre_a.sig ordre_malveillant.sig

# Vérifier
openssl dgst -sha256 -verify identite_publique_dakar.pem -signature ordre_malveillant.sig ordre_malveillant.txt
```

**Questions d'investigation :**
45. **Attaque par substitution** : Pourquoi cette attaque échoue-t-elle ?
46. **Réutilisation de signature** : Un attaquant peut-il réutiliser une signature valide sur un autre document ?
47. **Timestamps** : Comment prouver qu'un document a été signé à un moment donné ?

**🛡️ Défi Intégrité :**
```bash
# Analyser l'impact des modifications
echo "ORDRE MISSION A - Mission reconnaissance CRITIQUE" > ordre_a_modifie.txt

# Comparer les hashs
openssl dgst -sha256 ordre_a.txt
openssl dgst -sha256 ordre_a_modifie.txt
```

**Questions d'analyse :**
48. **Effet avalanche** : Pourquoi une petite modification change-t-elle complètement le hash ?
49. **Détection de modification** : Quelle est la probabilité qu'une modification passe inaperçue ?
50. **MAC vs Signature** : Quand utiliser HMAC plutôt qu'une signature RSA ?

---

## 🏛️ Phase 5 : Opération "Autorité Suprême"

### 🎭 Scénario : Création d'une Autorité de Certification Nationale
**Contexte :** Le gouvernement sénégalais vous confie la création d'une AC nationale pour certifier les identités numériques des entreprises fintech.

**🎯 Mission E1 - Demande d'Accréditation**
```bash
# Agent A : Créer une demande d'accréditation pour TechSN
openssl req -new -key agent_dakar.pem -out demande_techsn.csr

# Informations à renseigner :
# Country Name (2 letter code) [AU]: SN
# State or Province Name: Dakar
# Locality Name: Dakar
# Organization Name: TechSN Solutions
# Organizational Unit Name: Département Sécurité
# Common Name: techsn.sn
# Email Address: security@techsn.sn

# Examiner la demande
openssl req -in demande_techsn.csr -text -noout
```

**🎯 Mission E2 - Création de l'Autorité Nationale**
```bash
# Agent B : Devenir l'Autorité de Certification du Sénégal
openssl genrsa -out autorite_senegal.key 4096

# Créer le certificat racine du Sénégal
openssl req -new -x509 -key autorite_senegal.key -out certificat_senegal.crt -days 3650

# Informations pour l'AC nationale :
# Country Name: SN
# State: Dakar
# City: Dakar
# Organization: République du Sénégal
# Organizational Unit: Autorité de Certification Nationale
# Common Name: AC-Sénégal
# Email: contact@ac-senegal.gov.sn

# Examiner le certificat de l'État
openssl x509 -in certificat_senegal.crt -text -noout
```

**🎯 Mission E3 - Certification Officielle**
```bash
# L'AC Sénégal certifie TechSN
openssl x509 -req -in demande_techsn.csr -CA certificat_senegal.crt -CAkey autorite_senegal.key -CAcreateserial -out certificat_techsn.crt -days 365

# Vérifier la chaîne de confiance
openssl verify -CAfile certificat_senegal.crt certificat_techsn.crt
```

**🎯 Mission E4 - Test d'Intégrité Nationale**
```bash
# Simuler une tentative de falsification
cp certificat_techsn.crt certificat_falsifie.crt

# Modifier le certificat falsifié (changer quelques caractères)
sed -i 's/A/B/g' certificat_falsifie.crt

# Tenter la vérification
openssl verify -CAfile certificat_senegal.crt certificat_falsifie.crt
```

**🚨 Résultat attendu :** Échec de vérification = Sécurité confirmée !

**🧠 Questions techniques ultimes :**

**🔬 Défi Infrastructure PKI :**
51. **Chaîne de confiance** : Expliquez le concept de "root of trust". Que se passe-t-il si l'AC racine est compromise ?
52. **Certificate Revocation** : Comment révoquer un certificat compromis ? CRL vs OCSP ?
53. **Cross-certification** : Comment deux ACs indépendantes peuvent-elles se faire mutuellement confiance ?
54. **Pinning de certificat** : Pourquoi les navigateurs "épinglent-ils" certains certificats ?

**⚔️ Défi Attaques PKI :**
```bash
# Analyser la structure X.509
openssl x509 -in certificat_techsn.crt -text -noout | grep -A5 "Subject:"
openssl x509 -in certificat_techsn.crt -text -noout | grep -A5 "Issuer:"

# Vérifier les extensions
openssl x509 -in certificat_techsn.crt -text -noout | grep -A10 "X509v3"
```

**Questions d'investigation :**
55. **Substitution d'AC** : Un attaquant pourrait-il créer une fausse AC "République du Sénégal" ?
56. **Subject Alternative Names** : À quoi servent les SAN ? Pourquoi sont-ils critiques pour HTTPS ?
57. **Key Usage vs Extended Key Usage** : Quelle est la différence ? Pourquoi ces restrictions ?
58. **Certificate Transparency** : Comment les logs CT protègent-ils contre les certificats malveillants ?

**🛡️ Défi Sécurité Avancée :**
```bash
# Analyser la robustesse cryptographique
openssl x509 -in certificat_techsn.crt -noout -fingerprint -sha256
openssl x509 -in certificat_senegal.crt -noout -fingerprint -sha256

# Vérifier les algorithmes utilisés
openssl x509 -in certificat_techsn.crt -text -noout | grep "Signature Algorithm"
```

**Questions d'analyse :**
59. **Fingerprint** : À quoi sert l'empreinte d'un certificat ? Comment la vérifier hors-bande ?
60. **SHA-1 deprecated** : Pourquoi les certificats SHA-1 sont-ils interdits ? Quand la migration a-t-elle eu lieu ?
61. **Quantum resistance** : Quels algorithmes de signature résisteront aux ordinateurs quantiques ?

**🎯 Défis de recherche experts :**
- **Recherchez** : Qu'est-ce que l'attaque "Logjam" et comment affecte-t-elle DHE ?
- **Investiguez** : Comment fonctionne Certificate Transparency ? Trouvez le log CT de Google.
- **Explorez** : Qu'est-ce que DANE (DNS-based Authentication of Named Entities) ?
- **Analysez** : Pourquoi Let's Encrypt a révolutionné les certificats SSL ?

### 🏆 Questions bonus pour les experts légendaires

**62. Zero-Trust Architecture :** Comment adapter PKI dans un modèle Zero-Trust ?
**63. Hardware Security Modules :** Pourquoi utiliser un HSM pour stocker les clés d'AC ?
**64. Post-Quantum PKI :** Quels défis pose la migration vers la cryptographie post-quantique ?
**65. Blockchain et PKI :** Comment la blockchain pourrait-elle remplacer les ACs traditionnelles ?

---

## 📊 Debriefing Final : Évaluation de Mission

### 🎯 Objectifs Atteints
Cochez les compétences maîtrisées :

- [ ] **Analyse de vulnérabilités** - Démontré la faiblesse du Base64
- [ ] **Chiffrement symétrique** - Sécurisé des communications sensibles
- [ ] **Cryptographie asymétrique** - Établi des canaux sécurisés sans rencontre préalable
- [ ] **Protocoles hybrides** - Contourné les limitations RSA
- [ ] **Signature numérique** - Authentifié des documents officiels
- [ ] **Infrastructure PKI** - Créé une chaîne de confiance nationale

### 🏆 Niveaux de Sécurité Atteints

**🥉 Niveau Bronze - Agent Junior**
- Utilise correctement les commandes OpenSSL
- Comprend les différences entre chiffrement et encodage

**🥈 Niveau Argent - Agent Expérimenté**
- Implémente des protocoles hybrides
- Détecte les tentatives de falsification

**🥇 Niveau Or - Expert en Cybersécurité**
- Conçoit une infrastructure PKI complète
- Analyse les failles de sécurité et propose des solutions

### 🚨 Incidents de Sécurité Identifiés
1. **Faille Base64** - Encodage ≠ Chiffrement
2. **Limitation RSA** - Taille maximale des données
3. **Gestion des clés** - Protection des clés privées
4. **Chaîne de confiance** - Importance de la vérification

### 📝 Rapport de Mission

**Questions de débriefing pour votre rapport :**

1. **Analyse tactique :** Pourquoi le chiffrement hybride est-il plus efficace que l'asymétrique seul ?

2. **Évaluation des risques :** Un attaquant qui intercepte votre clé publique peut-il compromettre vos communications ?

3. **Stratégie de défense :** Comment détectez-vous qu'un document signé a été modifié ?

4. **Infrastructure critique :** Pourquoi avons-nous besoin d'une autorité de certification ?

5. **Scénario d'attaque :** Que se passerait-il si la clé privée de l'AC était compromise ?

### 🎖️ Certification de Mission
```
CERTIFICAT DE MISSION ACCOMPLIE
Agent: ___________________
Binôme: ___________________
Date: ___________________
Niveau atteint: ___________________
Signature de l'instructeur: ___________________
```

---

## 📚 Arsenal Cryptographique - Références

### 🔧 Commandes Essentielles Maîtrisées
```bash
# Chiffrement moderne
openssl enc -aes-256-cbc -salt -pbkdf2

# Génération RSA sécurisée
openssl genrsa -des3 -out private.pem 4096

# Signature avec SHA-256
openssl dgst -sha256 -sign private.pem

# Certification X.509
openssl x509 -req -days 365 -sha256
```

### 🌐 Ressources pour Agents d'Élite
- **Documentation OpenSSL** : Manuel de l'agent secret
- **RFC 8446 (TLS 1.3)** : Protocoles de nouvelle génération
- **NIST Guidelines** : Standards de sécurité gouvernementaux

---

**🔒 FIN DE MISSION - CLASSIFIED**

*"Un système n'est sécurisé que si ses utilisateurs comprennent les enjeux et maîtrisent les outils."*

**Contact Emergency :** cybersec.instructor@tdsi.sn
