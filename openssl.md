# Mission CyberSec : Infiltration et Sécurisation
**TP OpenSSL - Sécurité Réseaux et Systèmes M1 TDSI/LACGAA**

##  Briefing de Mission

###  Contexte
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

**🔍 Analyse forensique :**
```bash
# Comparer les tailles des fichiers
ls -la document_classifie.txt document_chiffre.enc
```

**Questions de debriefing :**
- Pourquoi le fichier chiffré est-il plus volumineux ?
- Que contient le "sel" cryptographique ?

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
