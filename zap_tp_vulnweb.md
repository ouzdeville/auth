# TP : Analyse de Sécurité Web avec ZAP
## Test sur http://testphp.vulnweb.com/

---

## 📋 Objectifs du TP

À la fin de ce TP, vous serez capable de :
- Configurer et utiliser ZAP comme proxy d'interception
- Explorer manuellement une application web
- Utiliser les fonctionnalités principales de ZAP (Breakpoints, Fuzzer, Scanner)
- Identifier et exploiter des vulnérabilités web courantes
- Générer un rapport de sécurité

---

## 🎯 Prérequis

- ZAP installé (téléchargeable sur https://www.zaproxy.org/)
- Navigateur web (Firefox ou Chrome)
- Connexion internet
- Durée estimée : 2-3 heures

---

## 📚 Partie 1 : Configuration et Prise en Main (20 min)

### Exercice 1.1 : Premier lancement et configuration

**Étapes :**

1. Lancez ZAP
2. Choisissez le mode "Manual Explore"
3. Entrez l'URL : `http://testphp.vulnweb.com/`
4. Cochez "Enable HUD" (optionnel pour découvrir l'interface)
5. Cliquez sur "Launch Browser"

**Questions :**
- Q1 : Combien de requêtes HTTP apparaissent dans l'historique après le chargement de la page d'accueil ?
- Q2 : Quels sont les différents types de ressources chargées (JS, CSS, images, etc.) ?
- Q3 : Identifiez dans l'arborescence (panneau Sites) la structure du site

**À noter :**
- Observez les 3 panneaux principaux de l'interface ZAP
- Familiarisez-vous avec les onglets "Request" et "Response"

---

## 🔍 Partie 2 : Exploration Manuelle (30 min)

### Exercice 2.1 : Navigation et cartographie

**Mission :** Explorer le site manuellement pour découvrir ses fonctionnalités

**Actions à réaliser :**
1. Naviguez vers la section "categories" du site
2. Testez la fonction de recherche (recherchez "test")
3. Accédez à la page "login" et essayez de vous connecter (utilisez n'importe quelles valeurs)
4. Explorez les différentes pages de détail des produits

**Questions :**
- Q4 : Listez au moins 5 pages/endpoints différents découverts
- Q5 : Quels paramètres GET avez-vous identifiés dans les URLs ?
- Q6 : Identifiez les formulaires présents sur le site (login, recherche, etc.)

### Exercice 2.2 : Utilisation du Spidering

**Étapes :**
1. Dans le panneau Sites, faites un clic droit sur `http://testphp.vulnweb.com`
2. Choisissez `Attack > Spider...`
3. Dans les options avancées, décochez "Process Forms" et "POST Forms"
4. Lancez le spider

**Questions :**
- Q7 : Combien d'URLs supplémentaires le spider a-t-il découvert ?
- Q8 : Quelles nouvelles pages ont été trouvées que vous n'aviez pas explorées manuellement ?

---

## 🎣 Partie 3 : Interception et Modification (25 min)

### Exercice 3.1 : Utilisation des Breakpoints

**Objectif :** Intercepter et modifier une requête de recherche

**Étapes :**
1. Activez les breakpoints globaux (bouton "break" dans la toolbar)
2. Dans le navigateur, effectuez une recherche avec le terme "laptop"
3. La requête devrait être interceptée dans l'onglet "Break"
4. Modifiez le paramètre de recherche pour chercher "camera" à la place
5. Cliquez sur le bouton de continuation (flèche verte)

**Questions :**
- Q9 : Quelle est la méthode HTTP utilisée (GET ou POST) ?
- Q10 : Quel est le nom du paramètre contenant le terme de recherche ?
- Q11 : Les résultats affichés correspondent-ils à votre modification ?

**Défi :** Essayez d'injecter des caractères spéciaux dans la recherche (`' OR 1=1 --`) et observez le comportement

### Exercice 3.2 : Request Editor

**Étapes :**
1. Désactivez les breakpoints
2. Dans l'historique, trouvez une requête vers une page de détail produit
3. Clic droit > "Open/Resend with request editor"
4. Modifiez l'ID du produit dans l'URL
5. Envoyez la requête et observez la réponse

**Questions :**
- Q12 : Pouvez-vous accéder à d'autres produits en modifiant l'ID ?
- Q13 : Que se passe-t-il si vous utilisez un ID invalide (ex: 99999) ?

---

## 💥 Partie 4 : Fuzzing et Brute Force (30 min)

### Exercice 4.1 : Fuzzing de la recherche

**Objectif :** Tester la fonction de recherche avec des payloads d'injection SQL

**Étapes :**
1. Sélectionnez une requête de recherche dans l'historique
2. Clic droit > "Attack > Fuzz..."
3. Sélectionnez le paramètre de recherche
4. Cliquez sur "Add..."
5. Choisissez "Add Custom Fuzz Strings"
6. Ajoutez manuellement ces payloads :
   ```
   ' OR '1'='1
   ' OR 1=1--
   admin' OR '1'='1
   ' UNION SELECT NULL--
   '; DROP TABLE users--
   ```
7. Lancez le fuzzer

**Questions :**
- Q14 : Quels codes de réponse HTTP obtenez-vous ?
- Q15 : Y a-t-il des différences de taille de réponse significatives ?
- Q16 : Identifiez-vous des comportements anormaux suggérant une vulnérabilité ?

### Exercice 4.2 : Test du formulaire de login

**Objectif :** Fuzzer le champ username du login

**Étapes :**
1. Allez sur la page de login
2. Dans l'historique, trouvez la requête POST de login
3. Lancez le Fuzzer sur le paramètre username
4. Utilisez des usernames courants : admin, root, test, user, administrator

**Questions :**
- Q17 : Les réponses varient-elles selon les usernames testés ?
- Q18 : Pouvez-vous détecter si certains comptes existent ?

---

## 🔎 Partie 5 : Scanner de Vulnérabilités (40 min)

### Exercice 5.1 : Scan passif

**Le scan passif est activé par défaut**

**Questions :**
- Q19 : Allez dans l'onglet "Alerts". Combien d'alertes sont remontées ?
- Q20 : Listez 3 types de vulnérabilités identifiées par le scanner passif
- Q21 : Quelle est la sévérité (High, Medium, Low) des alertes trouvées ?

### Exercice 5.2 : Active Scan ciblé

**Objectif :** Lancer un scan actif sur une page spécifique

**Étapes :**
1. Dans l'historique, sélectionnez la requête de recherche
2. Clic droit > "Attack > Active Scan..."
3. Dans l'onglet "Custom Vectors", sélectionnez uniquement le paramètre de recherche
4. Cochez "Disable Non-custom Input Vectors"
5. Lancez le scan
6. Observez la progression dans l'onglet "Active Scan"

**⚠️ Attention :** Le scan actif peut prendre plusieurs minutes

**Questions :**
- Q22 : Combien de requêtes ont été envoyées par le scanner ?
- Q23 : Quelles nouvelles vulnérabilités ont été découvertes ?
- Q24 : Le scanner a-t-il identifié une injection SQL ? Si oui, quel payload a fonctionné ?

### Exercice 5.3 : Analyse d'une injection SQL

**Si une injection SQL est détectée :**

1. Dans l'onglet "Alerts", sélectionnez l'alerte SQL Injection
2. Examinez les détails : URL, paramètre vulnérable, payload utilisé
3. Ouvrez la requête qui a permis de détecter la vulnérabilité

**Questions :**
- Q25 : Quel est le payload exact qui a confirmé l'injection SQL ?
- Q26 : Quelle est la réponse du serveur qui prouve la vulnérabilité ?
- Q27 : Le scanner propose-t-il une exploitation UNION ? Si oui, copiez le payload

**Défi avancé :**
Utilisez le Request Editor pour exploiter manuellement l'injection SQL et extraire des informations de la base de données

---

## 🎨 Partie 6 : Utilisation du HUD (Optionnel - 15 min)

### Exercice 6.1 : Découverte du HUD

**Si vous avez activé le HUD au début :**

1. Observez l'interface qui entoure la page web
2. Utilisez les boutons latéraux pour :
   - Voir l'historique des requêtes
   - Consulter les alertes en temps réel
   - Lancer un scan depuis le navigateur

**Questions :**
- Q28 : Quels sont les avantages du HUD par rapport à l'interface classique ?
- Q29 : Trouvez-vous l'interface HUD intuitive ?

---

## 📊 Partie 7 : Reporting (20 min)

### Exercice 7.1 : Génération d'un rapport

**Étapes :**
1. Dans le menu principal, allez dans `Report > Generate HTML Report...`
2. Choisissez un emplacement pour sauvegarder le rapport
3. Générez le rapport
4. Ouvrez-le dans un navigateur

**Questions :**
- Q30 : Combien de vulnérabilités au total sont listées dans le rapport ?
- Q31 : Quelle est la vulnérabilité la plus critique identifiée ?

### Exercice 7.2 : Analyse et recommandations

**Mission :** Rédiger une synthèse des résultats

**À produire :**
- Liste des 5 vulnérabilités les plus importantes trouvées
- Pour chaque vulnérabilité :
  - Nom et description
  - Impact potentiel
  - Recommandation de correction

---

## 🎓 Partie 8 : Questions de Synthèse

### Questions finales

**Q32 :** Comparez ZAP avec ce que vous connaissez d'autres outils de test de sécurité. Quels sont ses points forts ?

**Q33 :** Dans quels contextes recommanderiez-vous l'utilisation de ZAP ?

**Q34 :** Quelles limitations avez-vous rencontrées pendant ce TP ?

**Q35 :** Si vous deviez tester votre propre application web, quelle méthodologie adopteriez-vous avec ZAP ?

---

## 🏆 Défis Avancés (Bonus)

### Défi 1 : Exploitation complète d'une SQL Injection
Utilisez l'injection SQL détectée pour :
- Lister les tables de la base de données
- Extraire les noms d'utilisateurs
- Tenter de récupérer des mots de passe

### Défi 2 : Scripting ZAP
Créez un script personnalisé pour :
- Automatiser la modification d'un en-tête HTTP
- Ou extraire automatiquement certaines informations des réponses

### Défi 3 : Test XSS
Recherchez manuellement des vulnérabilités XSS sur le site :
- Testez différents points d'injection
- Documentez vos payloads et résultats

---

## 📝 Livrables Attendus

1. **Document de réponses** : Toutes les questions numérotées (Q1 à Q35)
2. **Rapport HTML** généré par ZAP
3. **Synthèse** : 1-2 pages sur les vulnérabilités critiques et recommandations
4. **Screenshots** : Au moins 5 captures d'écran montrant :
   - Interface ZAP avec historique
   - Résultat d'un fuzzing
   - Détection d'une vulnérabilité
   - Exploitation d'une injection SQL
   - Le rapport final

---

## 🔗 Ressources Complémentaires

- Documentation ZAP : https://www.zaproxy.org/docs/
- OWASP Top 10 : https://owasp.org/www-project-top-ten/
- Guide d'injection SQL : https://portswigger.net/web-security/sql-injection

---

## ⚠️ Note Éthique

Ce TP doit être réalisé **uniquement** sur http://testphp.vulnweb.com/, qui est un site spécialement conçu pour les tests de sécurité. Tester des applications web sans autorisation est illégal.

---

**Bon courage ! 🚀**