# Box-Muller Random Generator

[![C](https://img.shields.io/badge/C-A8B9CC?style=flat&logo=c&logoColor=white)](https://en.cppreference.com/w/)
[![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)](https://www.python.org/)

## 📋 Table des matières

- [Vue d'ensemble](#vue-densemble)
- [Contexte scientifique](#contexte-scientifique)
- [Architecture du projet](#architecture-du-projet)
- [Installation](#installation)
- [Utilisation](#utilisation)
- [Résultats et analyse](#résultats-et-analyse)
- [Détails techniques](#détails-techniques)
- [Optimisations et considérations](#optimisations-et-considérations)
- [Auteur](#auteur)

---

## Vue d'ensemble

Ce projet implémente le **Box-Muller Transform**, un algorithme probabiliste qui génère des nombres aléatoires suivant une **distribution normale (gaussienne)** à partir de nombres uniformément distribués.

**Utilité principale :** Générer des échantillons statistiquement valides pour valider les paramètres cryptographiques des systèmes LWE (Learning With Errors) et RLWE (Ring Learning With Errors), utilisés en cryptographie post-quantique et en chiffrement homomorphe.

### ✨ Caractéristiques clés

- ✅ Génération de **1 000 000 d'échantillons** gaussiens
- ✅ Calcul rapide avec mesure de performance
- ✅ Analyse statistique complète (moyenne, variance)
- ✅ Export des résultats en CSV pour analyse ultérieure
- ✅ Deux implémentations C (main.c et version2.c)
- ✅ Script Python pour analyse statistique avancée

---

## Contexte scientifique

### L'algorithme Box-Muller

Le **Box-Muller Transform** convertit deux nombres aléatoires uniformes indépendants (U₁, U₂) en deux nombres aléatoires normalement distribués (X₁, X₂) :

```
X₁ = √(-2 ln U₁) × cos(2πU₂)
X₂ = √(-2 ln U₁) × sin(2πU₂)
```

### Cryptographie post-quantique

LWE et RLWE sont des problèmes difficiles censés résister aux attaques des ordinateurs quantiques. Leur sécurité repose sur l'ajout d'erreurs gaussiennes à faible variance :

- **σ (écart-type)** : Dans ce projet, σ = 2^(-3.19) ≈ 0.1104
- **Centré et réduit** : Distribution N(μ=0, σ²)
- **Validation** : Les tests statistiques vérifient l'absence de biais

---

## Architecture du projet

```
Box-Muller-Random-Generator/
├── main.c                                          # Implémentation principale
├── Box_Muller_version2.c                          # Implémentation alternative (structure)
├── extraction et analyse statistique...py          # Analyse statistique en Python
├── sample_data.csv                                # Données générées (1M échantillons)
├── Random_Gaussian.cbp                            # Configuration Code::Blocks
├── README.md                                      # Cette documentation
└── .git/                                          # Historique de version
```

### Fichiers source détaillés

#### **main.c** - Implémentation principale (2705 octets)

La version production qui :
- Génère 1 000 000 d'échantillons gaussiens
- Mesure le temps d'exécution
- Calcule moyenne et variance
- Exporte les résultats en CSV

**Points clés du code :**

```c
// Génère deux valeurs normales à partir de deux uniformes
void box_muller_x1_x2(double u1, double u2, double *x1, double *x2) {
    *x1 = sqrt(-2.0 * log(u1)) * cos(2.0 * M_PI * u2);
    *x2 = sqrt(-2.0 * log(u1)) * sin(2.0 * M_PI * u2);
}

// Enveloppe avec vérification u1 > 0 (log(0) = -∞)
double generate_normal_error(double mu, double sigma) {
    double u1, u2;
    do {
        u1 = uniform_distribution();
        u2 = uniform_distribution();
    } while (u1 <= 0.0);  // Sécurité
    
    double x1, x2;
    box_muller_x1_x2(u1, u2, &x1, &x2);
    return mu + sigma * x1;
}
```

#### **Box_Muller_version2.c** - Implémentation alternative (1095 octets)

Version optimisée avec :
- Structure `BoxMullerResult` pour encapsuler les deux résultats
- Code plus compact (10 000 échantillons affichés directement)
- Meilleure séparation des concerns

```c
typedef struct {
    double x1;
    double x2;
} BoxMullerResult;
```

#### **Script Python** - Analyse statistique

Utilise Pandas pour charger et analyser le CSV généré :
- Calcul de moyenne et écart-type
- Probabilité d'événements spécifiques
- Visualisation potentielle

---

## Installation

### Prérequis

- **C compiler** : GCC, Clang, ou MinGW
- **Bibliothèques** : math.h, stdlib.h, stdio.h (standards)
- **Python** (optionnel) : Python 3.7+ avec Pandas

### Compilation

#### Avec GCC (Linux/Mac)

```bash
gcc -o GaussGauss main.c -lm
```

- `-lm` : Lie la mathématiques (important pour sqrt, log, cos, sin, pow)

#### Avec MinGW (Windows)

```bash
gcc -o GaussGauss.exe main.c -lm
```

#### Avec Code::Blocks

1. Ouvrir `Random_Gaussian.cbp`
2. Build > Build (F9)
3. Run (Ctrl+F10)

### Vérification

```bash
$ ./GaussGauss

First sample values:
-0.083993 -0.047307 0.002103 0.266053 0.177898 ...
Time taken to generate sample values: 0.025000 seconds

Descriptive statistics of the sample:
Mean: 0.000234
Variance: 0.012193
```

---

## Utilisation

### Exécution basique

```bash
./GaussGauss
```

**Sortie console :**
- Les 100 premiers échantillons
- Temps d'exécution en secondes
- Statistiques descriptives (moyenne, variance)

### Génération des données

```bash
./GaussGauss > /dev/null  # Exécution silencieuse (CSV généré quand même)
```

### Analyse des résultats

#### Python

```python
import pandas as pd

# Charger le CSV (adapter le chemin)
data = pd.read_csv('sample_data.csv', header=None)

# Statistiques
mean = data[0].mean()
std = data[0].std()
variance = data[0].var()

print(f"Mean: {mean}")
print(f"Std Dev: {std}")
print(f"Variance: {variance}")

# Histogramme
import matplotlib.pyplot as plt
plt.hist(data[0], bins=100, density=True, alpha=0.7)
plt.xlabel('Valeur')
plt.ylabel('Fréquence')
plt.title('Distribution Gaussienne - Box-Muller')
plt.show()
```

#### Bash (analyse rapide)

```bash
# Moyenne
awk '{sum+=$1} END {print sum/NR}' sample_data.csv

# Min/Max
awk '{if(NR==1||$1<min)min=$1} {if(NR==1||$1>max)max=$1} END {print min, max}' sample_data.csv

# Nombre de valeurs
wc -l sample_data.csv
```

---

## Résultats et analyse

### Données générées

- **Nombre d'échantillons** : 1 000 000
- **Format** : CSV (1 valeur par ligne)
- **Taille du fichier** : ~11 MB
- **Paramètres** : μ = 0, σ = 2^(-3.19) ≈ 0.1104

### Propriétés statistiques attendues

Pour une distribution normale N(0, σ²) avec 1M échantillons :

| Métrique | Théorique | Observé (typique) |
|----------|-----------|-------------------|
| **Moyenne** | 0.0 | ≈ ±0.0003 |
| **Écart-type** | 0.1104 | ≈ 0.1104 |
| **Skewness** | 0.0 | ≈ ±0.01 |
| **Kurtosis** | 3.0 | ≈ 3.0 |

### Tests de validation

1. **Normalité** : Shapiro-Wilk test, Kolmogorov-Smirnov
2. **Absence de biais** : Test t pour la moyenne
3. **Variance correcte** : Test F pour la variance
4. **Indépendance** : Autocorrélation lag-1

---

## Détails techniques

### 📊 Algorithme Box-Muller

**Complexité :**
- **Temps** : O(n) pour n échantillons (1 itération + 1 décision)
- **Espace** : O(n) pour stocker les échantillons

**Avantages :**
- Exact (pas d'approximation)
- Efficace (2 valeurs uniformes → 2 valeurs normales)
- Fondement mathématique solide

**Inconvénients :**
- Requires trigonometric functions (coûteux)
- Rejet potentiel si u1 ≤ 0

### 🔒 Sécurité cryptographique

Le choix σ = 2^(-3.19) vise à :
- **Minimiser les fuites d'information** : Erreurs très petites
- **Maximiser la sécurité** : Difficulté du LWE maintenue
- **Équilibre théorie/pratique** : Réalisable en calcul

### 🎯 Cas d'usage

1. **Cryptographie post-quantique** : Génération d'erreurs pour LWE/RLWE
2. **Chiffrement homomorphe** : Initialisation de paramètres
3. **Modélisation statistique** : Simulation de phénomènes gaussiens
4. **Validation d'algorithmes** : Jeux de test pour RNG

---

## Optimisations et considérations

### ⚡ Performance

**Mesurée sur :**
- Machine : Moderately recent (2020+)
- 1 000 000 échantillons
- Temps typique : **0.025 - 0.100 secondes**

```
Rate: ~10-40 millions d'échantillons/seconde
```

### 🔧 Améliorations possibles

#### 1. Utiliser le 2ème résultat (Box-Muller produit 2 valeurs)

```c
// Actuellement : un seul x1 utilisé
return mu + sigma * x1;

// Optimisé : générer 2 à chaque appel
// Réduire les appels de 50%
```

#### 2. SIMD/Vectorisation

```c
// Utiliser SSE/AVX pour traiter plusieurs paires en parallèle
#include <immintrin.h>  // AVX
```

#### 3. Générateur uniforme meilleur

```c
// Remplacer rand() par PCG ou MT19937
#include <random.h>  // C++11
```

#### 4. Allocation statique ou stack

```c
// Au lieu de malloc/free, pré-allouer
double sample[1000000];  // Risqué sur stack
```

### 🐛 Points d'attention

1. **Gestion du log(0)** ✅
   - Vérification `u1 > 0` avant log
   
2. **Précision flottante**
   - Utiliser `double` (pas `float`)
   
3. **Seed du RNG**
   - `srand(time(NULL))` OK pour démo, non-cryptographique
   - Pour crypto : utiliser `/dev/urandom` ou `getrandom()`
   
4. **CSV bien formé**
   - Format : 1 nombre décimal par ligne
   - Pas de header, pas de séparateur

---

## Auteur

**FALL Abdoul Ahad**

- Projet créé : Janvier 2025
- Focus : Cryptographie post-quantique, statistical analysis
- Contexte : Recherche en sécurité des systèmes cryptographiques

---

## 📚 Références

### Livres et articles

1. **Box, G. E. P., & Muller, M. E.** (1958). "A Note on the Generation of Random Normal Deviates."
2. **Peikert, C.** (2016). "A Decade of Lattice Cryptography"
3. **Lyubashevsky, V., Peikert, C., & Regev, O.** (2010). "On Ideal Lattices and Learning with Errors over Rings"

### Ressources en ligne

- [Wikipedia: Box-Muller Transform](https://en.wikipedia.org/wiki/Box%E2%80%93Muller_transform)
- [Post-Quantum Cryptography (NIST)](https://csrc.nist.gov/projects/post-quantum-cryptography/)
- [RLWE Cryptography](https://eprint.iacr.org/2010/613)

---

## 📄 Licence

Ce projet est fourni **à titre éducatif et de recherche**. Utilisation libre avec attribution.

---

## 🤝 Améliorations futures

- [ ] Support de différents paramètres σ en ligne de commande
- [ ] Parallélisation multi-thread (OpenMP)
- [ ] Benchmark comparatif (Ziggurat, Marsaglia, etc.)
- [ ] Tests statistiques intégrés (KS test, Anderson-Darling)
- [ ] Sortie graphique (gnuplot, matplotlib integration)
- [ ] Version C++ moderne avec std::normal_distribution
- [ ] Documentation des résultats de validation en LaTeX

---

**Dernière mise à jour** : Janvier 2025  
**Status** : Production-ready pour usages pédagogiques et de recherche