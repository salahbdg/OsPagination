# Solution: Introduction à la pagination - TD 2

## Exercice 1 : Traduction d'adresse

### Question 1: Taille d'une page
Avec des adresses virtuelles sur 64 bits décomposées en :
- 52 bits pour le numéro de page virtuelle
- 12 bits pour le déplacement

**Taille d'une page = 2^12 = 4096 octets = 4 Ko**

### Question 2: Exécution de deux instructions avec PC = 0x3FF8

**État initial :**
- PC = 0x3FF8
- TPV[3].V = 1, TPV[3].pr = 6
- TPV[4].V = 1, TPV[4].pr = 4
- Mémoire physique donnée

**Décomposition de PC = 0x3FF8 :**
- Adresse virtuelle : 0x3FF8
- Page virtuelle : 0x3FF8 >> 12 = 0x3 = 3
- Déplacement : 0x3FF8 & 0xFFF = 0xFF8

**Traduction d'adresse pour PC :**
- Page virtuelle 3 → Page physique 6 (TPV[3].pr = 6)
- Adresse physique = (6 << 12) + 0xFF8 = 0x6FF8

**Instruction 1 à l'adresse 0x6FF8 :** `mov ebx, eax`
- Cette instruction copie le contenu d'EAX dans EBX
- Supposons EAX contient une valeur initiale (non spécifiée dans l'énoncé)

**Instruction 2 à l'adresse 0x7000 :** `mov eax, #0`
- PC s'incrémente à 0x4000 (page virtuelle 4)
- Page virtuelle 4 → Page physique 4 (TPV[4].pr = 4)
- Cette instruction place la valeur immédiate 0 dans EAX

**Contenu final d'EBX :** La valeur initiale d'EAX avant la première instruction.

### Question 3: État mémoire physique après deux instructions

**Configuration :**
- PC = 0x4FF8
- TPV[3,4,5,6] = {(1,0), (1,1), (1,2), (1,3)}

**Image mémoire virtuelle donnée :**
- 0x3FF8: mov ebx, eax
- 0x4000: mov eax, #0  
- 0x4008: mov ebx, #1

**Mapping pages virtuelles → physiques :**
- Page 3 → Page physique 0
- Page 4 → Page physique 1
- Page 5 → Page physique 2
- Page 6 → Page physique 3

**Après exécution de deux instructions :**
- Page physique 0 : contient les instructions de la page virtuelle 3
- Page physique 1 : contient les instructions de la page virtuelle 4
- EAX = 0, EBX = ancienne valeur d'EAX

### Question 4: Exécution après rechargement contexte

**Nouveau contexte :**
- PC = valeur sauvegardée (0x4008)
- TPV[3,4,5,6] = {(1,3), (1,2), (1,1), (1,0)}

**Nouveau mapping :**
- Page 3 → Page physique 3
- Page 4 → Page physique 2
- Page 5 → Page physique 1
- Page 6 → Page physique 0

**Exécution se déroule correctement car :**
1. Les pages virtuelles restent valides (bit V = 1)
2. Le contenu des pages physiques n'a pas changé
3. Seul le mapping virtuel → physique a été modifié
4. Le programme continue à accéder aux mêmes données logiques

---

## Exercice 2 : Performance de la pagination à un niveau

### Question 1: Format adresse virtuelle
- Adresses sur 32 bits
- Taille de page : 16 Ko = 2^14 octets
- **Format :** 18 bits (numéro de page) + 14 bits (déplacement)

### Question 2: Tailles en pages
**Tableau tab :**
- N = 256 × 1024 éléments
- Chaque élément : 4 octets (long)
- Taille totale : 256 × 1024 × 4 = 1 048 576 octets = 1 Mo
- **Nombre de pages :** 1 048 576 / 16 384 = 64 pages

**RAM disponible :**
- 512 Ko = 524 288 octets
- **Nombre de pages :** 524 288 / 16 384 = 32 pages

### Question 3: Table des pages au début
Au début, aucune page n'est chargée en mémoire.
**Contenu TPV :** Toutes les entrées ont V=0 (invalide)

### Question 4: Après un tour de boucle d'initialisation
Premier accès à tab[0] (adresse virtuelle 0) :
- Page virtuelle 0 non présente → défaut de page
- Chargement en page physique 0
- **TPV[0] :** (V=1, pr=0, U=1, M=1)

### Question 5: Après 16×1024 tours
- 16×1024 = 16 384 éléments initialisés
- 16 384 × 4 octets = 65 536 octets = 4 pages utilisées
- **TPV[0-3] :** (V=1, pr=0-3, U=1, M=1)
- **TPV[4-63] :** V=0

### Question 6: Séquence d'événements (40 premiers)

**Phase d'initialisation :**
1. Défaut page virtuelle 0 → allocation page physique 0
2. Défaut page virtuelle 1 → allocation page physique 1
3. ...
4. Défaut page virtuelle 31 → allocation page physique 31
5. Défaut page virtuelle 32 → remplacement page 0 (FIFO)
6. Défaut page virtuelle 33 → remplacement page 1 (FIFO)
7. ...
8. Défaut page virtuelle 63 → remplacement page 31 (FIFO)

**40 premiers événements :**
- Événements 1-32 : Défauts pages virtuelles 0-31 (allocations)
- Événements 33-40 : Défauts pages virtuelles 32-39 (remplacements)

### Question 7: Défauts totaux et taux

**Phase d'initialisation :**
- 64 pages à initialiser, 32 pages en RAM
- Total : 64 défauts de page

**Phase de sommation :**
- Même pattern : parcours séquentiel de 64 pages
- Total : 64 défauts de page supplémentaires

**Total général :** 128 défauts de page

**Taux de défaut :** 
- Accès totaux : 2 × 256 × 1024 = 524 288
- Taux = 128 / 524 288 ≈ 0.024% 

**Variation selon taille RAM :**
- Si RAM ≥ 64 pages : 64 défauts (initialisation uniquement)
- Si RAM < 64 pages : défauts proportionnels à (64 - taille_RAM)
- Le taux diminue quand la taille RAM augmente

# TD 3

# Solution: Gestion mémoire paginée - TD 3

## Analyse du système

### Format des adresses
- **Adresse virtuelle (64 bits):** 54 bits (n° page virtuelle) + 10 bits (déplacement)
- **Adresse physique (39 bits):** 29 bits (n° page réelle) + 10 bits (déplacement)
- **Taille de page:** 2^10 = 1024 octets = 1 Ko

### Structures de données
- **TPV (Table Pages Virtuelles):** V(1) + n°page_réelle(29) + n°page_disque(29) + prot(3) + U(1) + M(1)
- **TPR (Table Pages Réelles):** État + n°page_virt + propriétaire

## Implémentation des fonctions

### 1. Fonction `choix_page_à_vider`

```c
int choix_page_à_vider() {
    int page_choisie = -1;
    int tour = 0;
    const int MAX_TOURS = 2;
    
    // Algorithme de l'horloge (approximation LRU)
    while (tour < MAX_TOURS && page_choisie == -1) {
        for (int pr = 0; pr < NB_PAGES_PHYSIQUES; pr++) {
            // Ignorer les pages libres ou verrouillées
            if (TPR[pr].etat == libre || TPR[pr].etat == verrouillé) {
                continue;
            }
            
            // Récupérer l'entrée TPV correspondante
            int processus = TPR[pr].propriétaire;
            int pv = TPR[pr].no_pg_virt;
            
            // Vérifier le bit U (utilisé récemment)
            if (TPV[processus][pv].U == 0) {
                // Page non utilisée récemment -> candidat pour éviction
                page_choisie = pr;
                break;
            } else if (tour == 0) {
                // Premier tour : remettre le bit U à 0 (seconde chance)
                TPV[processus][pv].U = 0;
            }
        }
        tour++;
    }
    
    // Si aucune page trouvée, prendre la première disponible
    if (page_choisie == -1) {
        for (int pr = 0; pr < NB_PAGES_PHYSIQUES; pr++) {
            if (TPR[pr].etat == occupé) {
                page_choisie = pr;
                break;
            }
        }
    }
    
    return page_choisie;
}
```

### 2. Fonction `donner_page`

```c
int donner_page() {
    int page_libre;
    
    // 1. Chercher d'abord une page libre
    page_libre = cherche_page_libre();
    
    if (page_libre != -1) {
        // Page libre trouvée
        TPR[page_libre].etat = occupé;
        return page_libre;
    }
    
    // 2. Aucune page libre, il faut en libérer une
    int page_à_vider = choix_page_à_vider();
    
    if (page_à_vider == -1) {
        // Erreur : aucune page disponible
        return -1;
    }
    
    // 3. Sauvegarder la page évincée si elle a été modifiée
    int processus_victime = TPR[page_à_vider].propriétaire;
    int pv_victime = TPR[page_à_vider].no_pg_virt;
    
    if (TPV[processus_victime][pv_victime].M == 1) {
        // Page modifiée -> écriture sur disque
        int page_disque = TPV[processus_victime][pv_victime].no_page_disque;
        écrire_pr_sur_pd(page_à_vider, page_disque);
    }
    
    // 4. Invalider l'entrée dans la TPV du processus victime
    TPV[processus_victime][pv_victime].V = 0;
    TPV[processus_victime][pv_victime].U = 0;
    TPV[processus_victime][pv_victime].M = 0;
    
    // 5. Marquer la page comme occupée par le nouveau processus
    TPR[page_à_vider].etat = occupé;
    
    return page_à_vider;
}
```

### 3. Fonction `trait_défaut`

```c
void trait_défaut(int pv) {
    // 1. Obtenir une page physique libre
    int page_réelle = donner_page();
    
    if (page_réelle == -1) {
        // Erreur critique : pas de mémoire disponible
        // Terminer le processus ou déclencher une exception
        erreur_mémoire_insuffisante();
        return;
    }
    
    // 2. Charger la page depuis le disque
    int page_disque = TPV[élu][pv].no_page_disque;
    lire_pd_dans_pr(page_disque, page_réelle);
    
    // 3. Mettre à jour la TPV du processus courant
    TPV[élu][pv].V = 1;                    // Page valide
    TPV[élu][pv].no_page_réelle = page_réelle;
    TPV[élu][pv].U = 1;                    // Récemment utilisée
    TPV[élu][pv].M = 0;                    // Pas encore modifiée
    
    // 4. Mettre à jour la TPR
    TPR[page_réelle].etat = occupé;
    TPR[page_réelle].no_pg_virt = pv;
    TPR[page_réelle].propriétaire = élu;
}
```

## Algorithme de remplacement

L'algorithme implémenté est une **approximation du LRU** utilisant l'**algorithme de l'horloge** :

### Principe :
1. **Premier tour :** Parcourir toutes les pages, si U=0 → sélectionner, sinon U=1 → remettre U à 0
2. **Deuxième tour :** Sélectionner la première page avec U=0 rencontrée
3. **Fallback :** Si aucune page trouvée, prendre la première page disponible

### Avantages :
- Approximation efficace du LRU
- Coût en temps linéaire
- Utilise le bit U maintenu par le matériel
- Donne une "seconde chance" aux pages récemment utilisées

## Gestion des cas particuliers

### Pages verrouillées :
- Jamais sélectionnées pour éviction
- Typiquement pages du noyau ou tampons d'E/S

### Gestion des erreurs :
- Vérification de la disponibilité des pages
- Gestion des échecs d'allocation
- Protection contre les accès invalides

### Optimisations :
- Priorité lecture sur écriture disque
- Éviction différée des pages propres
- Batch des écritures disque

Cette implémentation respecte les contraintes du système et fournit un mécanisme de pagination à la demande robuste avec gestion LRU approximée.


# TD 4

# Solution: Tables des pages hiérarchiques - TD 4

## Analyse du système

### Format d'adresse virtuelle (32 bits)
- **Livre :** 10 bits (index dans table racine)
- **Page :** 10 bits (index dans table des pages)
- **Déplacement :** 12 bits (offset dans la page)

### Format d'adresse physique
- **Page physique :** 20 bits
- **Déplacement :** 12 bits

---

## Question 1: Tailles des composants

### Taille d'une page
- Déplacement sur 12 bits
- **Taille page = 2^12 = 4096 octets = 4 Ko**

### Taille d'un livre
- Un livre correspond à une table des pages
- Une table des pages contient 2^10 = 1024 entrées
- Chaque entrée couvre une page de 4 Ko
- **Taille livre = 1024 × 4 Ko = 4 Mo**

### Espace total adressable
- Adresse virtuelle sur 32 bits
- **Espace adressable = 2^32 = 4 Go**

---

## Question 2: Tailles des tables

### Taille d'une table des pages
- 2^10 = 1024 entrées
- Chaque entrée sur 32 bits = 4 octets
- **Taille table pages = 1024 × 4 = 4 Ko**

### Taille d'une table des livres (table racine)
- 2^10 = 1024 entrées
- Chaque entrée sur 32 bits = 4 octets
- **Taille table livres = 1024 × 4 = 4 Ko**

---

## Question 3: Index et contenu des tables

### Calcul des index pour les régions

#### Code: 0x400000-0x402000
- 0x400000 = 0100 0000 0000 0000 0000 0000 0000 0000
- Livre = 0x001 = 1
- Page = 0x000 = 0
- **Index table livres[1], table pages[0-1]**

#### Données: 0x800000-0x802000
- 0x800000 = 1000 0000 0000 0000 0000 0000 0000 0000
- Livre = 0x002 = 2
- Page = 0x000 = 0
- **Index table livres[2], table pages[0-1]**

#### Pile: 0x10000000-0x10002000
- 0x10000000 = 0001 0000 0000 0000 0000 0000 0000 0000
- Livre = 0x040 = 64
- Page = 0x000 = 0
- **Index table livres[64], table pages[0-1]**

### Contenu initial des tables

```
Table des livres (1024 entrées):
- Entrée [1]: V=1, pr=adresse_table_pages_code
- Entrée [2]: V=1, pr=adresse_table_pages_données  
- Entrée [64]: V=1, pr=adresse_table_pages_pile
- Autres entrées: V=0 (zones non autorisées)

Table des pages pour le code:
- Entrées [0-1]: V=1, rwx=101 (r-x), pr=pages_physiques_code
- Autres entrées: V=0

Table des pages pour les données:
- Entrées [0-1]: V=1, rwx=110 (rw-), pr=pages_physiques_données
- Autres entrées: V=0

Table des pages pour la pile:
- Entrées [0-1]: V=1, rwx=110 (rw-), pr=pages_physiques_pile
- Autres entrées: V=0
```

### Taille totale des tables
- 1 table des livres: 4 Ko
- 3 tables des pages: 3 × 4 Ko = 12 Ko
- **Total: 16 Ko**

---

## Question 4: Modifications après exécution

### Analyse du code
```c
char tab[4096];  // Tableau de 4 Ko = 1 page
long res=0;
for (int i=0; i<N; i++) tab[i] = random();
for (int i=1; i<N; i++) if (i%2==0) res = res + tab[i];
```

### Accès mémoire générés
1. **Code:** Lecture des instructions (pages code)
2. **Données:** 
   - Écriture dans tab[4096] (initialisation)
   - Lecture de tab[i] pour i pairs (sommation)
3. **Pile:** Accès aux variables locales et contexte

### Modifications des tables
```
Table des pages pour les données:
- Entrée [0]: V=1, rwx=110, pr=page_physique_données_0 (accédée et modifiée)
- Entrée [1]: V=1, rwx=110, pr=page_physique_données_1 (pour tab[4096])

Nouvelles pages potentiellement ajoutées:
- Si tab[4096] déborde sur une nouvelle page de données
- Pages supplémentaires de pile si récursion profonde
```

### État final
- **Pages code:** Accédées en lecture (exécution)
- **Pages données:** Accédées en lecture/écriture (tab[] modifié)
- **Pages pile:** Accédées selon utilisation

---

## Question 5: Algorithmes sans bits U et M matériels

### A) Algorithme de remplacement de page

Sans bits U et M matériels, nous devons utiliser des techniques logicielles :

#### **Algorithme du bit de référence logiciel**
```c
// Structure étendue pour chaque page
struct page_info {
    int ref_count;        // Compteur de références
    time_t last_access;   // Horodatage dernier accès
    int access_freq;      // Fréquence d'accès
};

int choix_page_remplacement() {
    // 1. Algorithme NRU (Not Recently Used) simulé
    // Périodiquement, réinitialiser les compteurs
    
    // 2. Recherche page avec plus faible priorité
    int page_victime = -1;
    int min_priority = INT_MAX;
    
    for (int i = 0; i < nb_pages; i++) {
        if (page_info[i].ref_count == 0) {
            // Page jamais référencée = meilleur candidat
            return i;
        }
        
        // Calcul priorité basée sur fréquence et récence
        int priority = page_info[i].access_freq * 1000 + 
                      (current_time - page_info[i].last_access);
        
        if (priority < min_priority) {
            min_priority = priority;
            page_victime = i;
        }
    }
    
    return page_victime;
}
```

#### **Techniques complémentaires:**
1. **Horodatage:** Maintenir timestamp des accès
2. **Compteurs:** Incrémenter à chaque accès
3. **Échantillonnage:** Périodiquement vérifier pages actives
4. **Working Set:** Suivre ensemble des pages utilisées

### B) Algorithme de recopie de page

#### **Stratégie de détection des modifications:**
```c
// Méthode 1: Protection en écriture + trap
void detecter_modifications() {
    // 1. Marquer toutes pages en lecture seule
    for (int i = 0; i < nb_pages; i++) {
        if (pages[i].writable) {
            set_page_readonly(i);
            pages[i].dirty_candidate = true;
        }
    }
}

void trap_ecriture(int page_num) {
    // 2. Sur trap d'écriture, marquer comme modifiée
    pages[page_num].modified = true;
    set_page_writable(page_num);
}

// Méthode 2: Recopie périodique + comparaison
void detection_par_checksum() {
    for (int i = 0; i < nb_pages; i++) {
        uint32_t checksum = calcul_checksum(page_addr[i]);
        if (checksum != pages[i].old_checksum) {
            pages[i].modified = true;
            pages[i].old_checksum = checksum;
        }
    }
}
```

#### **Stratégies de recopie:**
1. **Recopie immédiate:** Dès détection modification
2. **Recopie différée:** Batch périodique des pages modifiées
3. **Recopie intelligente:** Priorité selon fréquence modification
4. **Copy-on-Write:** Dupliquer seulement si modification

### Optimisations recommandées:
- **Lazy evaluation:** Reporter calculs coûteux
- **Batch processing:** Grouper opérations disque
- **Prédiction:** Anticiper besoins futurs
- **Cache intelligent:** Garder pages fréquemment utilisées

Cette approche logicielle compense l'absence de support matériel tout en maintenant de bonnes performances.

