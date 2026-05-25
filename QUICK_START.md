# 🚀 GUIDE DE DÉMARRAGE RAPIDE - Box-Muller

## En 5 minutes chrono ⏱️

### 1️⃣ Compilation

```bash
# Linux/Mac
gcc -O2 -Wall -lm main.c -o GaussGauss

# Windows (MinGW)
gcc -O2 -Wall -lm main.c -o GaussGauss.exe
```

✅ **Résultat attendu :** Un exécutable `GaussGauss` (ou `.exe`)

---

### 2️⃣ Exécution

```bash
./GaussGauss
```

✅ **Résultat :**

```
First sample values:
-0.083993 -0.047307 0.002103 -0.266053 ...

Time taken to generate sample values: 0.025000 seconds

Descriptive statistics of the sample:
Mean: 0.000234
Variance: 0.012193
```

📁 **Fichier créé :** `sample_data.csv` (11 MB, 1 million de valeurs)

---

### 3️⃣ Analyse des données

#### Avec Python (recommandé)

```python
import pandas as pd
import numpy as np

# Charger les données
data = pd.read_csv('sample_data.csv', header=None)[0]

# Statistiques
print(f"Moyenne: {data.mean():.6f}")
print(f"Écart-type: {data.std():.6f}")
print(f"Variance: {data.var():.6f}")
print(f"Min: {data.min():.6f}")
print(f"Max: {data.max():.6f}")

# Histogramme
import matplotlib.pyplot as plt
plt.hist(data, bins=100, density=True, alpha=0.7, edgecolor='black')
plt.xlabel('Valeur')
plt.ylabel('Fréquence (densité)')
plt.title('Distribution Gaussienne - Box-Muller Transform')
plt.axvline(data.mean(), color='r', linestyle='--', label='Moyenne')
plt.legend()
plt.grid(True, alpha=0.3)
plt.show()
```

#### Avec Bash (rapide)

```bash
# Moyenne
awk '{sum+=$1; n++} END {print "Mean:", sum/n}' sample_data.csv

# Min/Max
awk '{
  if(NR==1) {min=$1; max=$1}
  if($1<min) min=$1
  if($1>max) max=$1
}
END {print "Min:", min, "Max:", max}' sample_data.csv

# Nombre de lignes
wc -l sample_data.csv
```

---

## 📊 Paramètres clés à connaître

### Dans `main.c`

```c
const int sample_size = 1000000;  // Nombre d'échantillons
sample[i] = generate_normal_error(0, pow(2, -3.19));
//                                 ^             ^
//                              moyenne     écart-type
```

**Changer les paramètres :**

```c
// Exemple : Distribution N(5, 2²)
sample[i] = generate_normal_error(5, 2.0);

// Recompiler
gcc -O2 -Wall -lm main.c -o GaussGauss
./GaussGauss
```

### Valeurs de σ courantes

| Paramètre | σ calculé | Usage |
|-----------|-----------|-------|
| `2^(-3.19)` | 0.1104 | Crypto post-quantique (défaut) |
| `1.0` | 1.0 | Distribution standard N(0,1) |
| `0.5` | 0.5 | Erreurs plus petites |
| `2.0` | 2.0 | Erreurs plus grandes |

---

## 🔍 Vérification rapide

### Validation manuelle

```bash
# Les valeurs doivent être petites (σ ≈ 0.1104)
head -20 sample_data.csv

# Attendu : des valeurs entre -0.5 et +0.5 (~99.7% des données)
# -0.083993
# -0.047307
#  0.002103
#  ...
```

### Tests statistiques (Python)

```python
from scipy import stats

# Charger les données
data = pd.read_csv('sample_data.csv', header=None)[0]

# Test de normalité (Shapiro-Wilk)
# H0: Les données suivent une loi normale
statistic, p_value = stats.shapiro(data[:5000])
print(f"Shapiro-Wilk p-value: {p_value:.6f}")
if p_value > 0.05:
    print("✅ La distribution est normale (α=0.05)")
else:
    print("❌ La distribution n'est pas normale")

# Test sur la moyenne (t-test)
# H0: Moyenne = 0
t_stat, p_value = stats.ttest_1samp(data, 0)
print(f"t-test p-value: {p_value:.6f}")
if p_value > 0.05:
    print("✅ Moyenne = 0 (pas de biais significatif)")
```

---

## 🛠️ Dépannage

| Problème | Solution |
|----------|----------|
| **`error: undefined reference to 'sqrt'`** | Ajouter `-lm` : `gcc -lm main.c ...` |
| **`cannot open output file: Permission denied`** | Exécuter depuis dossier writable |
| **`segmentation fault`** | Vérifier malloc/free, ou réduire à 100K samples |
| **CSV trop volumineux** | Compresser : `gzip sample_data.csv` → 0.5 MB |
| **Résultats varient** | Normal ! `srand(time(NULL))` = seed aléatoire |

---

## 💡 Astuces pratiques

### Créer des résultats reproductibles

Modifier `main.c` ligne 33 :

```c
// Avant (aléatoire)
srand((unsigned int)time(NULL));

// Après (reproductible)
srand(12345);  // Seed fixe
```

Recompiler et exécuter : mêmes résultats à chaque fois ✅

### Comparer deux implémentations

```bash
# Version 1
./GaussGauss > output1.txt

# Version 2 (Box_Muller_version2.c)
gcc -O2 -Wall -lm Box_Muller_version2.c -o GaussGauss2
./GaussGauss2 | head -10000 > output2.txt

# Différence ?
diff output1.txt output2.txt
```

### Pipeline Unix

```bash
# Générer et analyser en une seule commande
./GaussGauss | head -100 | awk '{sum+=$1} END {print "Mean:", sum/NR}'
```

---

## 📈 Changer la taille de l'échantillon

**Pour 10 millions d'échantillons :**

```c
// main.c ligne 36
const int sample_size = 10000000;  // ← changer 1000000 en 10000000
```

⚠️ **Attention :** 
- RAM : 80 MB (au lieu de 8 MB)
- Temps : ~0.25 sec (au lieu de 0.025 sec)
- Fichier CSV : 110 MB

---

## 🎯 Cas d'usage courants

### Cas 1 : Tester la qualité du générateur

```bash
# Compiler
gcc -O2 -Wall -lm main.c -o GaussGauss

# Exécuter 5 fois, vérifier reproductibilité
for i in {1..5}; do
  ./GaussGauss | grep "Mean:"
done
```

### Cas 2 : Générer de petits fichiers pour test

```c
// Modifier main.c
const int sample_size = 10000;  // 10K au lieu de 1M
// Résultat : ~80 KB CSV, exécution instantanée
```

### Cas 3 : Utiliser pour cryptographie

```c
// Changer σ selon vos besoins NIST/post-quantum
double sigma = pow(2, -2.5);  // Autre valeur
```

---

## 📚 Fichiers importants

| Fichier | Rôle | À modifier ? |
|---------|------|-------------|
| `main.c` | Code principal | ✅ Oui (sample_size, σ) |
| `Box_Muller_version2.c` | Alternative compact | ⚠️ Rarement |
| `sample_data.csv` | Données générées | ❌ Non |
| `Random_Gaussian.cbp` | Projet Code::Blocks | ⚠️ Si IDE |

---

## ✨ Prochaines étapes

1. **Compiler et exécuter** → `gcc -O2 -Wall -lm main.c -o GaussGauss && ./GaussGauss`
2. **Analyser les résultats** → Python script ci-dessus
3. **Valider statistiquement** → Tests scipy (voir section validation)
4. **Explorer variations** → Changer σ, sample_size, etc.

---

## 🎓 Pour aller plus loin

- **Parallélisation :** Ajouter OpenMP (`#pragma omp parallel for`)
- **Vectorisation :** Utiliser SSE/AVX pour SIMD
- **Meilleur RNG :** Remplacer `rand()` par PCG/xoshiro
- **Utiliser x2 :** Récupérer le 2ème résultat de Box-Muller (+50% efficacité)
- **Tests statistiques complets :** Kolmogorov-Smirnov, Anderson-Darling, etc.

---

**Bon calcul ! 🚀**

Questions ? Consultez `README_COMPLET.md` pour plus de détails.
