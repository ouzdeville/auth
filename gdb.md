## Master 1 Securite des Systeme Embarque
## Ressources complémentaires et références

### Documentation technique
- **GDB Manual officiel** : https://www.gnu.org/software/gdb/documentation/
- **ABI x86-64** : System V Application Binary Interface AMD64
- **Intel Architecture Manual** : Pour les détails sur les registres et instructions

### Outils GDB enrichis
- **GEF** : GDB Enhanced Features - Interface moderne et colorée
- **PEDA** : Python Exploit Development Assistance for GDB
- **Pwndbg** : Exploit Development and Reverse Engineering with GDB

### Concepts avancés à explorer
- **DWARF Debug Information** : Format des informations de débogage
- **Core Dumps** : Analyse post-mortem avec GDB
- **Remote Debugging** : Débogage à distance avec gdbserver
- **Reverse Debugging** : Exécution inverse avec GDB

### Commandes GDB essentielles - Référence rapide

| Commande | Description | Exemple |
|----------|-------------|---------|
| `break` / `b` | Point d'arrêt | `b main`, `b *0x401000` |
| `run` / `r` | Lancer le programme | `r arg1 arg2` |
| `continue` / `c` | Continuer l'exécution | `c` |
| `next` / `n` | Instruction suivante (pas à pas) | `n` |
| `step# TP Master - GDB et Fonctionnement de la Pile

## Objectifs pédagogiques

À la fin de ce TP, vous serez capable de :
- Comprendre l'organisation et le fonctionnement de la pile d'exécution
- Utiliser GDB pour analyser la pile et les frames
- Examiner les registres liés à la pile (ESP, EBP, RSP, RBP)
- Analyser le passage de paramètres et les variables locales
- Comprendre les mécanismes d'appel et de retour de fonction

## Prérequis

- Connaissances de base en C/C++
- Notions d'assembleur x86/x64
- Environnement Linux avec GCC et GDB installés

## Partie 1 - Préparation et codes d'exemple

### 1.1 Programme simple avec appels de fonctions

Créez le fichier `pile_simple.c` :

```c
#include <stdio.h>

int addition(int a, int b) {
    int resultat = a + b;
    printf("Dans addition: a=%d, b=%d, resultat=%d\n", a, b, resultat);
    return resultat;
}

int multiplication(int x, int y) {
    int temp = x * y;
    printf("Dans multiplication: x=%d, y=%d, temp=%d\n", x, y, temp);
    return addition(temp, 10);
}

int main() {
    int val1 = 5;
    int val2 = 3;
    
    printf("Début du programme\n");
    int resultat = multiplication(val1, val2);
    printf("Résultat final: %d\n", resultat);
    
    return 0;
}
```

### 1.2 Programme avec récursion

Créez le fichier `pile_recursive.c` :

```c
#include <stdio.h>

int factorielle(int n) {
    printf("Appel factorielle(%d)\n", n);
    
    if (n <= 1) {
        printf("Cas de base: retour 1\n");
        return 1;
    }
    
    int resultat = n * factorielle(n - 1);
    printf("Retour factorielle(%d) = %d\n", n, resultat);
    return resultat;
}

int main() {
    int n = 5;
    printf("Calcul de %d!\n", n);
    int fact = factorielle(n);
    printf("Résultat: %d! = %d\n", n, fact);
    return 0;
}
```

### 1.3 Programme avec tableaux et pointeurs

Créez le fichier `pile_avancee.c` :

```c
#include <stdio.h>
#include <string.h>

void afficher_tableau(int *tab, int taille) {
    printf("Adresse du tableau: %p\n", (void*)tab);
    for (int i = 0; i < taille; i++) {
        printf("tab[%d] = %d (adresse: %p)\n", i, tab[i], (void*)&tab[i]);
    }
}

void modifier_valeur(int *ptr) {
    printf("Adresse reçue: %p\n", (void*)ptr);
    printf("Valeur avant modification: %d\n", *ptr);
    *ptr = 42;
    printf("Valeur après modification: %d\n", *ptr);
}

int main() {
    int tableau[5] = {1, 2, 3, 4, 5};
    int variable = 10;
    
    printf("=== Analyse des adresses ===\n");
    printf("Adresse de variable: %p\n", (void*)&variable);
    printf("Adresse de tableau: %p\n", (void*)tableau);
    
    printf("\n=== Affichage tableau ===\n");
    afficher_tableau(tableau, 5);
    
    printf("\n=== Modification par pointeur ===\n");
    modifier_valeur(&variable);
    printf("Variable après modification: %d\n", variable);
    
    return 0;
}
```

## Partie 2 - Configuration de l'environnement et compilation

### 2.1 Configuration GDB optimale

Créez un fichier `.gdbinit` dans votre répertoire de travail pour optimiser votre environnement :

```bash
# Syntaxe Intel (plus lisible)
set disassembly-flavor intel

# Suivre les processus enfants lors des fork()
set follow-fork-mode child

# Fonction personnalisée pour l'affichage complet
define init_debug
    layout asm
    layout regs
    info registers
end

# Fonction pour analyser la pile
define stack_analysis
    info frame
    backtrace
    info locals
    info args
    x/10x $rsp
end

# Fonction pour afficher l'état complet
define full_state
    printf "=== ÉTAT COMPLET ===\n"
    printf "RSP: %p, RBP: %p\n", $rsp, $rbp
    printf "RIP: %p\n", $rip
    backtrace
    info locals
end

# Désactiver la pagination pour les scripts
set pagination off
```

### 2.2 Compilation avec options de débogage

Compilez les programmes avec les options de débogage :

```bash
# Compilation standard pour le débogage
gcc -g -O0 -o pile_simple pile_simple.c
gcc -g -O0 -o pile_recursive pile_recursive.c
gcc -g -O0 -o pile_avancee pile_avancee.c

# Version 32 bits pour comparer (optionnel)
gcc -g -O0 -m32 -o pile_simple_32 pile_simple.c
```

**Notes importantes** : 
- `-O0` désactive les optimisations
- `-g` inclut les informations de débogage
- `-m32` force la compilation 32 bits pour comparaison

## Partie 3 - Exploration de la pile avec GDB

### 3.1 Analyse du programme simple

Lancez GDB et chargez le premier programme :

```bash
gdb ./pile_simple
```

**Exercice 1 - Points d'arrêt et navigation**

1. Chargez le programme et utilisez votre configuration :
   ```
   (gdb) file ./pile_simple
   (gdb) init_debug
   ```

2. Placez des points d'arrêt stratégiques :
   ```
   (gdb) break main
   (gdb) break multiplication  
   (gdb) break addition
   (gdb) info breakpoints
   ```

3. Exécutez et analysez avec vos fonctions personnalisées :
   ```
   (gdb) run
   (gdb) stack_analysis
   (gdb) continue
   (gdb) full_state
   ```

4. Utilisez les layouts visuels pour une meilleure compréhension :
   ```
   (gdb) layout asm
   (gdb) layout regs
   (gdb) nexti
   ```

**Questions à analyser :**
- Quelle est l'adresse de base de la pile au démarrage ?
- Comment évolue le registre RSP (Stack Pointer) lors des appels ?
- Que contient le registre RBP (Base Pointer) et comment évolue-t-il ?
- Quelle est la différence entre `nexti` et `stepi` ?

### 3.2 Analyse détaillée des frames

**Exercice 2 - Exploration des frames**

1. Affichez la pile complète :
   ```
   (gdb) backtrace full
   ```

2. Naviguez entre les frames :
   ```
   (gdb) frame 0
   (gdb) info frame
   (gdb) frame 1
   (gdb) info frame
   ```

3. Examinez les variables locales :
   ```
   (gdb) info locals
   (gdb) info args
   ```

**Questions à analyser :**
- Quelle est la taille de chaque frame ?
- Où sont stockées les variables locales ?
- Comment les paramètres sont-ils passés ?

### 3.3 Analyse mémoire de la pile et calculs dans GDB

**Exercice 3 - Examen de la mémoire et calculs**

1. Affichez le contenu de la pile avec différents formats :
   ```
   (gdb) x/20x $rsp        # Format hexadécimal
   (gdb) x/20d $rsp        # Format décimal
   (gdb) x/20i $rip        # Instructions à partir de RIP
   (gdb) x/s $rsp          # Chaîne de caractères si applicable
   ```

2. Analysez les adresses des variables et faites des calculs :
   ```
   (gdb) print &val1
   (gdb) print &val2
   (gdb) print &resultat
   
   # Calculs d'offset entre variables
   (gdb) print/x &val2 - &val1
   (gdb) print (&resultat - &val1) * sizeof(int)
   ```

3. Comparez les formats d'affichage :
   ```
   (gdb) print/x val1      # Hexadécimal
   (gdb) print/t val1      # Binaire
   (gdb) print/c val1      # Caractère ASCII
   (gdb) print/f val1      # Float (si applicable)
   ```

4. Examinez la structure de la frame courante :
   ```
   (gdb) x/20x $rbp-0x20   # 32 octets avant RBP
   (gdb) x/20x $rbp        # À partir de RBP
   (gdb) x/20x $rbp+0x20   # 32 octets après RBP
   ```

**Questions pratiques :**
- Calculez la taille exacte de chaque frame
- Identifiez l'adresse de retour dans chaque frame
- Trouvez l'emplacement des paramètres et variables locales

## Partie 4 - Analyse de la récursion

### 4.1 Visualisation de la pile récursive

**Exercice 4 - Pile récursive**

1. Lancez le programme récursif :
   ```bash
   gdb ./pile_recursive
   ```

2. Placez un point d'arrêt dans la fonction récursive :
   ```
   (gdb) break factorielle
   (gdb) run
   ```

3. Continuez l'exécution plusieurs fois et observez :
   ```
   (gdb) continue
   (gdb) backtrace
   (gdb) info frame
   ```

**Questions à analyser :**
- Comment la pile grandit-elle à chaque appel récursif ?
- Quelle est la différence d'adresse entre les frames ?
- Que se passe-t-il lors du retour des appels ?

### 4.2 Analyse de la croissance de la pile avec breakpoints conditionnels

**Exercice 5 - Breakpoints conditionnels et mesure automatisée**

1. Placez un breakpoint conditionnel pour analyser seulement certains appels :
   ```
   (gdb) break factorielle if n == 3
   (gdb) run
   ```

2. Créez une fonction GDB pour l'analyse automatique :
   ```
   (gdb) define analyze_recursion
   Type commands for definition of "analyze_recursion".
   End with a line saying just "end".
   >set $depth = 0
   >break factorielle
   >run
   >while $depth < 6
   >  printf "Profondeur %d - RSP: %p, RBP: %p, Paramètre n: %d\n", $depth, $rsp, $rbp, n
   >  info frame
   >  continue
   >  set $depth = $depth + 1
   >end
   >end
   ```

3. Script avancé pour mesurer la croissance :
   
   Créez le fichier `script_recursion_avance.gdb` :
   ```
   set pagination off
   set disassembly-flavor intel
   
   define track_stack_growth
       set $initial_rsp = $rsp
       set $call_count = 0
       break factorielle
       run
       
       while $call_count < 6
           printf "=== APPEL %d ===\n", $call_count
           printf "RSP: %p (offset: %d octets)\n", $rsp, $initial_rsp - $rsp
           printf "RBP: %p\n", $rbp
           printf "Paramètre n: %d\n", n
           printf "Adresse de retour: %p\n", *(void**)($rbp + 8)
           
           # Afficher le contenu de la frame
           printf "Contenu frame (RBP-16 à RBP+16):\n"
           x/8x $rbp-16
           
           set $call_count = $call_count + 1
           continue
       end
   end
   
   track_stack_growth
   quit
   ```

4. Exécutez le script :
   ```bash
   gdb -x script_recursion_avance.gdb ./pile_recursive
   ```

**Analyse comparative :**
- Comparez la croissance sur architecture 32 vs 64 bits
- Mesurez l'espace utilisé par frame
- Identifiez les patterns de stockage des variables

## Partie 5 - Analyse avancée avec pointeurs

### 5.1 Passage par référence et pile

**Exercice 6 - Pointeurs et pile**

1. Analysez le programme avancé :
   ```bash
   gdb ./pile_avancee
   ```

2. Placez des points d'arrêt :
   ```
   (gdb) break main
   (gdb) break afficher_tableau
   (gdb) break modifier_valeur
   ```

3. Analysez le passage de paramètres :
   ```
   (gdb) run
   (gdb) step
   (gdb) print &tableau
   (gdb) print &variable
   ```

**Questions à analyser :**
- Où sont stockés les tableaux sur la pile ?
- Comment les pointeurs sont-ils passés ?
- Quelle est la différence entre passage par valeur et par référence ?

### 5.2 Analyse des adresses mémoire

**Exercice 7 - Cartographie mémoire**

1. Créez un script d'analyse des adresses :
   
   Créez le fichier `analyse_adresses.gdb` :
   ```
   break main
   run
   printf "=== ANALYSE DES ADRESSES ===\n"
   printf "Stack Pointer (RSP): %p\n", $rsp
   printf "Base Pointer (RBP): %p\n", $rbp
   printf "Adresse de main: %p\n", main
   printf "Adresse de tableau: %p\n", &tableau
   printf "Adresse de variable: %p\n", &variable
   
   continue
   printf "\n=== DANS AFFICHER_TABLEAU ===\n"
   printf "RSP: %p\n", $rsp
   printf "RBP: %p\n", $rbp
   printf "Paramètre tab: %p\n", tab
   
   continue
   printf "\n=== DANS MODIFIER_VALEUR ===\n"
   printf "RSP: %p\n", $rsp
   printf "RBP: %p\n", $rbp
   printf "Paramètre ptr: %p\n", ptr
   
   quit
   ```

## Partie 6 - Exercices pratiques avancés

### Exercice 8 - Débordement de pile et limites système

Créez un programme qui provoque un débordement de pile contrôlé :

```c
#include <stdio.h>
#include <sys/resource.h>

void afficher_limite_pile() {
    struct rlimit limit;
    getrlimit(RLIMIT_STACK, &limit);
    printf("Limite de pile: %ld octets\n", limit.rlim_cur);
}

void fonction_infinie(int compteur) {
    char buffer[1024];  // Forcer l'allocation de pile
    printf("Appel %d - Adresse buffer: %p, RSP: %p\n", 
           compteur, (void*)buffer, (void*)__builtin_frame_address(0));
    
    // Remplir le buffer pour éviter l'optimisation
    buffer[0] = compteur % 256;
    
    fonction_infinie(compteur + 1);
}

int main() {
    afficher_limite_pile();
    printf("Début de la récursion infinie...\n");
    fonction_infinie(1);
    return 0;
}
```

**Script GDB pour analyser le débordement** (`overflow_analysis.gdb`) :

```
set pagination off
set disassembly-flavor intel

define analyze_overflow
    break main
    run
    continue
    
    # Breakpoint sur la fonction récursive avec condition
    break fonction_infinie if compteur % 100 == 0
    
    set $overflow_detected = 0
    while $overflow_detected == 0
        continue
        if $rsp < 0x7ffffffde000
            printf "ALERTE: Possible débordement détecté!\n"
            printf "RSP dangereux: %p\n", $rsp
            set $overflow_detected = 1
        end
        printf "Compteur: %d, RSP: %p\n", compteur, $rsp
    end
end

analyze_overflow
```

**Questions d'analyse :**
1. À quelle profondeur se produit le débordement ?
2. Quelle est la taille de pile utilisée par appel ?
3. Comment détecter ce problème avant le crash ?

### Exercice 9 - Analyse d'un buffer overflow avec techniques forensiques

Créez un programme vulnérable avec analyse détaillée :

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

void fonction_vulnerable(char *input) {
    char buffer[16];
    char *sauvegarde_rbp = __builtin_frame_address(0);
    
    printf("=== AVANT STRCPY ===\n");
    printf("Adresse buffer: %p\n", (void*)buffer);
    printf("Adresse RBP sauvegardé: %p\n", (void*)sauvegarde_rbp);
    printf("Adresse de retour: %p\n", (void*)__builtin_return_address(0));
    
    strcpy(buffer, input);
    
    printf("=== APRÈS STRCPY ===\n");
    printf("Buffer: %s\n", buffer);
    printf("RBP toujours valide: %s\n", 
           sauvegarde_rbp == __builtin_frame_address(0) ? "OUI" : "NON");
}

void fonction_secrete() {
    printf("ACCÈS NON AUTORISÉ! Fonction secrète exécutée!\n");
    exit(0);
}

int main(int argc, char *argv[]) {
    if (argc != 2) {
        printf("Usage: %s <string>\n", argv[0]);
        printf("Adresse fonction_secrete: %p\n", (void*)fonction_secrete);
        return 1;
    }
    
    printf("Début analyse buffer overflow\n");
    fonction_vulnerable(argv[1]);
    printf("Retour normal du programme\n");
    return 0;
}
```

**Script d'analyse forensique** (`buffer_overflow_forensics.gdb`) :

```
set pagination off
set disassembly-flavor intel

define forensic_analysis
    printf "=== ANALYSE FORENSIQUE BUFFER OVERFLOW ===\n"
    
    # Breakpoint avant strcpy
    break fonction_vulnerable
    run AAAA
    
    printf "État initial:\n"
    x/20x $rsp
    set $initial_rbp = $rbp
    set $initial_rsp = $rsp
    
    # Breakpoint après strcpy
    break *fonction_vulnerable+50  # Ajustez selon votre binaire
    continue
    
    printf "État après strcpy:\n"
    x/20x $initial_rsp
    
    # Vérifier l'intégrité de la pile
    if $rbp != $initial_rbp
        printf "CORRUPTION DÉTECTÉE: RBP modifié!\n"
        printf "RBP initial: %p, RBP actuel: %p\n", $initial_rbp, $rbp
    end
    
    continue
end

# Test avec différentes tailles
define test_overflow_sizes
    set $size = 10
    while $size <= 30
        printf "\n=== TEST AVEC %d CARACTÈRES ===\n", $size
        # Ici on devrait relancer avec une chaîne de taille $size
        # En pratique, vous feriez cela manuellement
        set $size = $size + 5
    end
end

forensic_analysis
```

**Tests à effectuer :**

1. **Test normal** :
   ```bash
   ./buffer_overflow "Hello"
   ```

2. **Test de débordement léger** :
   ```bash
   ./buffer_overflow "AAAAAAAAAAAAAAAAAAAAAA"
   ```

3. **Test avec GDB pour analyse** :
   ```bash
   gdb -x buffer_overflow_forensics.gdb ./buffer_overflow
   ```

**Exercice avancé - Construction d'exploit :**

Calculez l'adresse de `fonction_secrete` et construisez un payload pour l'exécuter :

```bash
# Trouver l'offset exact
python3 -c "print('A' * 24 + '\x[adresse_fonction_secrete]')"
```

## Partie 7 - Questions de synthèse

### Questions théoriques

1. **Organisation de la pile** : Expliquez l'organisation générale de la pile d'exécution. Quels sont les différents éléments stockés ?

2. **Registres de pile** : Quelle est la différence entre RSP et RBP ? Pourquoi a-t-on besoin des deux ?

3. **Passage de paramètres** : Comment les paramètres sont-ils passés aux fonctions selon l'ABI x86-64 ?

4. **Conventions d'appel** : Expliquez les conventions d'appel et leur impact sur la pile.

### Questions pratiques

1. **Optimisation** : Recompilez les programmes avec `-O2` et analysez les différences. Que remarquez-vous ?

2. **Architecture** : Comparez le comportement sur une architecture 32 bits vs 64 bits.

3. **Debugging** : Proposez une méthodologie pour déboguer un crash lié à la pile.

## Partie 8 - Techniques avancées et outils complémentaires

### 8.1 Création d'un analyseur de pile intelligent

Créez un programme d'analyse sophistiqué :

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/resource.h>

typedef struct {
    void *frame_address;
    void *return_address;
    int depth;
} stack_frame_info;

void analyser_pile_courante(int depth) {
    stack_frame_info info;
    info.frame_address = __builtin_frame_address(0);
    info.return_address = __builtin_return_address(0);
    info.depth = depth;
    
    printf("Frame %d: RBP=%p, Return=%p\n", 
           depth, info.frame_address, info.return_address);
    
    if (depth < 10) {
        analyser_pile_courante(depth + 1);
    }
}

void mesurer_utilisation_pile() {
    struct rlimit limit;
    getrlimit(RLIMIT_STACK, &limit);
    
    char stack_var;
    static char *stack_bottom = NULL;
    
    if (stack_bottom == NULL) {
        stack_bottom = &stack_var;
    }
    
    size_t used = stack_bottom - &stack_var;
    printf("Pile utilisée: %zu octets sur %ld disponibles\n", 
           used, limit.rlim_cur);
}

int main() {
    printf("=== ANALYSEUR DE PILE INTELLIGENT ===\n");
    mesurer_utilisation_pile();
    analyser_pile_courante(0);
    return 0;
}
```

### 8.2 Scripts GDB avancés

Créez un fichier `.gdbinit` avancé pour l'analyse de pile :

```bash
# .gdbinit avancé pour analyse de pile
set disassembly-flavor intel
set pagination off
set history save on
set history size 1000

# Couleurs (si terminal le supporte)
set prompt \033[1;32m(gdb) \033[0m

# Fonction pour analyse complète de la pile
define stack_forensics
    printf "\n=== ANALYSE FORENSIQUE DE LA PILE ===\n"
    printf "RSP: %p, RBP: %p\n", $rsp, $rbp
    printf "Différence RSP-RBP: %d octets\n", $rbp - $rsp
    
    # Afficher la chaîne d'appels
    printf "\n--- Chaîne d'appels ---\n"
    backtrace
    
    # Analyser chaque frame
    printf "\n--- Analyse détaillée des frames ---\n"
    set $frame_num = 0
    while $frame_num < 5
        if $frame_num == 0
            frame 0
        else
            up
        end
        
        printf "Frame %d:\n", $frame_num
        info frame
        info locals
        printf "\n"
        
        set $frame_num = $frame_num + 1
    end
    
    # Retour à la frame courante
    frame 0
end

# Fonction pour détecter les corruptions
define detect_corruption
    printf "\n=== DÉTECTION DE CORRUPTION ===\n"
    
    # Vérifier l'alignement de la pile
    if ($rsp & 0xf) != 0
        printf "ALERTE: Pile mal alignée! RSP = %p\n", $rsp
    else
        printf "Pile correctement alignée\n"
    end
    
    # Vérifier la cohérence RBP
    set $saved_rbp = *(void**)$rbp
    if $saved_rbp < $rbp
        printf "ALERTE: RBP sauvegardé incohérent! Saved=%p, Current=%p\n", $saved_rbp, $rbp
    else
        printf "RBP cohérent\n"
    end
    
    # Vérifier l'adresse de retour
    set $return_addr = *(void**)($rbp + 8)
    printf "Adresse de retour: %p\n", $return_addr
    info symbol $return_addr
end

# Fonction pour traquer les appels de fonction
define trace_calls
    printf "=== TRAÇAGE DES APPELS ===\n"
    
    # Breakpoint sur toutes les fonctions principales
    rbreak ^main$
    rbreak ^fonction_
    
    commands
        printf "Entrée dans: "
        info symbol $rip
        printf " | RSP: %p\n", $rsp
        continue
    end
end

# Fonction pour analyser les paramètres selon l'ABI x86-64
define analyze_params
    printf "\n=== ANALYSE DES PARAMÈTRES (ABI x86-64) ===\n"
    printf "RDI (1er param): %p (int: %d)\n", $rdi, $rdi
    printf "RSI (2ème param): %p (int: %d)\n", $rsi, $rsi
    printf "RDX (3ème param): %p (int: %d)\n", $rdx, $rdx
    printf "RCX (4ème param): %p (int: %d)\n", $rcx, $rcx
    printf "R8  (5ème param): %p (int: %d)\n", $r8, $r8
    printf "R9  (6ème param): %p (int: %d)\n", $r9, $r9
end

# Macro pour démarrage rapide
define quick_start
    break main
    run
    stack_forensics
    analyze_params
end

printf "GDB configuré pour l'analyse de pile avancée\n"
printf "Commandes disponibles:\n"
printf "  stack_forensics  - Analyse complète de la pile\n"
printf "  detect_corruption - Détection de corruption\n"
printf "  trace_calls      - Traçage des appels\n"
printf "  analyze_params   - Analyse des paramètres\n"
printf "  quick_start      - Démarrage rapide avec analyse\n"
```

### 8.3 Outils complémentaires recommandés

Installez et testez ces outils pour enrichir votre analyse :

1. **GEF (GDB Enhanced Features)** :
   ```bash
   # Installation
   wget -O ~/.gdbinit-gef.py -q https://gef.blah.cat/py
   echo "source ~/.gdbinit-gef.py" >> ~/.gdbinit
   ```

2. **Valgrind pour l'analyse mémoire** :
   ```bash
   # Analyse des erreurs de pile
   valgrind --tool=memcheck --track-origins=yes ./votre_programme
   
   # Analyse des fuites mémoire
   valgrind --leak-check=full ./votre_programme
   ```

3. **AddressSanitizer (ASan)** :
   ```bash
   # Compilation avec ASan
   gcc -g -fsanitize=address -o programme_asan programme.c
   
   # Exécution avec détection automatique
   ./programme_asan
   ```

### 8.4 Projet final : Analyseur forensique de pile

Créez un outil complet qui combine tous les concepts vus :

```c
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>
#include <execinfo.h>

#define MAX_STACK_FRAMES 64

typedef struct {
    void *addresses[MAX_STACK_FRAMES];
    int count;
    char **symbols;
} stack_trace_t;

void capture_stack_trace(stack_trace_t *trace) {
    trace->count = backtrace(trace->addresses, MAX_STACK_FRAMES);
    trace->symbols = backtrace_symbols(trace->addresses, trace->count);
}

void print_stack_analysis(stack_trace_t *trace) {
    printf("\n=== ANALYSE DE PILE FORENSIQUE ===\n");
    printf("Nombre de frames: %d\n", trace->count);
    
    for (int i = 0; i < trace->count; i++) {
        printf("Frame %d: %p - %s\n", i, trace->addresses[i], trace->symbols[i]);
    }
}

void signal_handler(int sig) {
    stack_trace_t trace;
    printf("\nSIGNAL REÇU: %d\n", sig);
    capture_stack_trace(&trace);
    print_stack_analysis(&trace);
    exit(1);
}

// Reste du programme...
```

**Commande de compilation complète** :
```bash
gcc -g -O0 -rdynamic -o analyseur_final analyseur_final.c
```

## Ressources complémentaires

- **Documentation GDB** : https://www.gnu.org/software/gdb/documentation/
- **ABI x86-64** : System V Application Binary Interface AMD64
- **Intel Architecture Manual** : Pour les détails sur les registres

## Évaluation

L'évaluation portera sur :
- La compréhension du fonctionnement de la pile (40%)
- La maîtrise des commandes GDB (30%)
- L'analyse des résultats expérimentaux (20%)
- La qualité du projet final (10%)

**Durée estimée** : 6-8 heures de travail
