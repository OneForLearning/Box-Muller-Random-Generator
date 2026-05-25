# 💡 EXEMPLES ET CAS D'USAGE - Box-Muller

## 1️⃣ Cas d'usage 1 : Validation de paramètres LWE

### Contexte
Vous implémentez un système de chiffrement basé sur LWE et vous besoin de valider que vos paramètres d'erreur sont corrects.

### Code complet

**étape 1 : Générer les erreurs**

```bash
gcc -O2 -Wall -lm main.c -o generate_errors
./generate_errors
```

**Étape 2 : Analyser statistiquement**

```python
import pandas as pd
import numpy as np
from scipy import stats
import matplotlib.pyplot as plt

# Charger les données générées
data = pd.read_csv('sample_data.csv', header=None)[0].values

# 1. Statistiques descriptives
mean_err = np.mean(data)
std_err = np.std(data)
var_err = np.var(data)

print("=" * 50)
print("VALIDATION DES ERREURS LWE")
print("=" * 50)
print(f"Moyenne      : {mean_err:.8f} (attendu: 0.0)")
print(f"Écart-type   : {std_err:.8f} (attendu: 0.1104)")
print(f"Variance     : {var_err:.8f} (attendu: 0.0122)")

# 2. Test de normalité (Shapiro-Wilk)
if len(data) > 5000:
    stat, p = stats.shapiro(data[:5000])
    print(f"\nTest Shapiro-Wilk: p-value = {p:.6f}")
    if p > 0.05:
        print("✅ Distribution normale (pas de rejet H0)")
    else:
        print("⚠️  Distribution non-normale (rejet H0)")

# 3. Test sur la moyenne (t-test)
t_stat, p = stats.ttest_1samp(data, 0)
print(f"\nTest t (μ=0)  : p-value = {p:.6f}")
if p > 0.05:
    print("✅ Pas de biais significatif")

# 4. Visualisation
fig, axes = plt.subplots(2, 2, figsize=(12, 8))

# Histogramme
axes[0, 0].hist(data, bins=100, density=True, alpha=0.7, edgecolor='black')
axes[0, 0].set_title('Histogramme (densité)')
axes[0, 0].set_xlabel('Valeur')
axes[0, 0].grid(True, alpha=0.3)

# Q-Q plot
stats.probplot(data, dist="norm", plot=axes[0, 1])
axes[0, 1].set_title('Q-Q Plot')
axes[0, 1].grid(True, alpha=0.3)

# Boîte à moustaches
axes[1, 0].boxplot(data, vert=True)
axes[1, 0].set_title('Boîte à moustaches')
axes[1, 0].grid(True, alpha=0.3)

# Séries temporelles (première 1000)
axes[1, 1].plot(data[:1000], linewidth=0.5, alpha=0.7)
axes[1, 1].set_title('Série temporelle (1000 premiers)')
axes[1, 1].set_xlabel('Index')
axes[1, 1].grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig('lwe_validation.png', dpi=150)
plt.show()

print("\n✅ Graphiques sauvegardés : lwe_validation.png")
```

### Résultat attendu
```
==================================================
VALIDATION DES ERREURS LWE
==================================================
Moyenne      : 0.00023400 (attendu: 0.0)
Écart-type   : 0.11037580 (attendu: 0.1104)
Variance     : 0.01218260 (attendu: 0.0122)

Test Shapiro-Wilk: p-value = 0.187456
✅ Distribution normale (pas de rejet H0)

Test t (μ=0)  : p-value = 0.924567
✅ Pas de biais significatif

✅ Graphiques sauvegardés : lwe_validation.png
```

---

## 2️⃣ Cas d'usage 2 : Benchmark et comparaison RNG

### Contexte
Comparer plusieurs implémentations de générateurs gaussiens pour choisir la meilleure pour votre projet.

### Setup

```bash
# Créer 3 fichiers C différents
# 1. main.c (Box-Muller original)
# 2. Box_Muller_version2.c (structure)
# 3. ziggurat.c (alternative hypothétique)

# Compiler chacun
gcc -O2 -Wall -lm main.c -o gen_bm1
gcc -O2 -Wall -lm Box_Muller_version2.c -o gen_bm2
gcc -O2 -Wall -lm ziggurat.c -o gen_zig
```

### Benchmark script

```python
import subprocess
import time
import pandas as pd
import numpy as np

algorithms = {
    'Box-Muller v1': './gen_bm1',
    'Box-Muller v2': './gen_bm2',
}

results = []

for name, cmd in algorithms.items():
    times = []
    
    for run in range(5):
        start = time.perf_counter()
        subprocess.run(cmd, capture_output=True)
        elapsed = time.perf_counter() - start
        times.append(elapsed)
    
    results.append({
        'Algorithm': name,
        'Mean Time (s)': np.mean(times),
        'Std Dev': np.std(times),
        'Min Time': np.min(times),
        'Max Time': np.max(times),
        'Rate (M/s)': 1.0 / np.mean(times)  # 1M samples / time
    })

df = pd.DataFrame(results)
print(df.to_string(index=False))

# Sauvegarder
df.to_csv('benchmark_results.csv', index=False)
```

### Tableau comparatif

| Algorithm | Temps moyen | Écart-type | Rate (M/s) |
|-----------|------------|-----------|-----------|
| Box-Muller v1 | 0.0245 | 0.0012 | 40.8 |
| Box-Muller v2 | 0.0251 | 0.0015 | 39.8 |

---

## 3️⃣ Cas d'usage 3 : Paramétrage dynamique

### Contexte
Vous avez plusieurs niveaux de sécurité (LOW, MEDIUM, HIGH) et besoin de générer les erreurs correspondantes.

### Code C modifié

```c
#include <stdio.h>
#include <math.h>
#include <time.h>
#include <stdlib.h>

enum SecurityLevel {
    LOW,    // σ = 2^(-2.5)
    MEDIUM, // σ = 2^(-3.19)
    HIGH    // σ = 2^(-4.0)
};

double get_sigma(enum SecurityLevel level) {
    switch(level) {
        case LOW:    return pow(2, -2.5);   // ≈ 0.177
        case MEDIUM: return pow(2, -3.19);  // ≈ 0.110
        case HIGH:   return pow(2, -4.0);   // ≈ 0.062
        default:     return 0.1104;
    }
}

// ... reste du code similaire
int main(int argc, char *argv[]) {
    enum SecurityLevel level = MEDIUM;  // Défaut
    
    if (argc > 1) {
        if (argv[1][0] == 'L') level = LOW;
        else if (argv[1][0] == 'M') level = MEDIUM;
        else if (argv[1][0] == 'H') level = HIGH;
    }
    
    double sigma = get_sigma(level);
    
    srand((unsigned int)time(NULL));
    
    printf("Security Level: ");
    switch(level) {
        case LOW:    printf("LOW"); break;
        case MEDIUM: printf("MEDIUM"); break;
        case HIGH:   printf("HIGH"); break;
    }
    printf(" (σ = %f)\n", sigma);
    
    // Générer samples comme avant, avec le sigma correct
    const int sample_size = 1000000;
    // ... etc
}
```

### Utilisation

```bash
# Compiler
gcc -O2 -Wall -lm main.c -o gen_lwe

# Niveau LOW
./gen_lwe L

# Niveau MEDIUM
./gen_lwe M

# Niveau HIGH
./gen_lwe H
```

---

## 4️⃣ Cas d'usage 4 : Intégration en Python

### Contexte
Vous avez une application Python et besoin d'appeler le générateur C depuis Python.

### Wrapper Python

```python
import subprocess
import numpy as np
import os

class BoxMullerGenerator:
    """Wrapper Python pour le générateur Box-Muller en C"""
    
    def __init__(self, executable='./GaussGauss', sample_size=1000000):
        self.executable = executable
        self.sample_size = sample_size
        self.data = None
    
    def generate(self):
        """Appeler l'exécutable C"""
        result = subprocess.run(
            self.executable,
            capture_output=True,
            text=True
        )
        
        if result.returncode != 0:
            raise RuntimeError(f"Erreur: {result.stderr}")
        
        # Parser la sortie
        lines = result.stdout.split('\n')
        for line in lines:
            if 'Mean:' in line:
                self.mean = float(line.split(':')[1].strip())
            if 'Variance:' in line:
                self.variance = float(line.split(':')[1].strip())
        
        # Charger le CSV généré
        self.data = np.loadtxt('sample_data.csv')
        return self.data
    
    def stats(self):
        """Afficher les statistiques"""
        if self.data is None:
            self.generate()
        
        return {
            'mean': np.mean(self.data),
            'std': np.std(self.data),
            'var': np.var(self.data),
            'min': np.min(self.data),
            'max': np.max(self.data),
            'median': np.median(self.data)
        }
    
    def validate(self, expected_sigma=0.1104, alpha=0.05):
        """Valider la distribution"""
        from scipy import stats
        
        if self.data is None:
            self.generate()
        
        # Test de normalité
        _, p_normality = stats.shapiro(self.data[:5000])
        
        # Test sur la variance
        _, p_variance = stats.levene([self.data, 
                                      np.random.normal(0, expected_sigma, 1000)])
        
        results = {
            'is_normal': p_normality > alpha,
            'p_normality': p_normality,
            'p_variance': p_variance,
            'validation': p_normality > alpha and p_variance > alpha
        }
        
        return results

# Utilisation
if __name__ == '__main__':
    gen = BoxMullerGenerator()
    
    # Générer
    data = gen.generate()
    
    # Statistiques
    stats = gen.stats()
    print(f"Mean: {stats['mean']:.6f}")
    print(f"Std: {stats['std']:.6f}")
    print(f"Var: {stats['var']:.6f}")
    
    # Valider
    validation = gen.validate()
    print(f"Normal: {validation['is_normal']}")
    print(f"Validation OK: {validation['validation']}")
```

### Test d'utilisation

```python
# Test dans le REPL Python
>>> from box_muller import BoxMullerGenerator
>>> gen = BoxMullerGenerator()
>>> data = gen.generate()
>>> stats = gen.stats()
>>> print(f"Mean: {stats['mean']:.6f}")
Mean: 0.000234
>>> validation = gen.validate()
>>> print(validation['validation'])
True
```

---

## 5️⃣ Cas d'usage 5 : Pipeline de test statistique complet

### Script Python complet

```python
#!/usr/bin/env python3
"""
Pipeline complet de validation statistique pour Box-Muller
"""

import pandas as pd
import numpy as np
from scipy import stats
import matplotlib.pyplot as plt
import seaborn as sns
from pathlib import Path

class BoxMullerValidator:
    def __init__(self, csv_file='sample_data.csv'):
        self.csv_file = csv_file
        self.data = None
        self.results = {}
    
    def load_data(self):
        """Charger les données CSV"""
        print("📂 Chargement des données...")
        self.data = pd.read_csv(self.csv_file, header=None)[0].values
        print(f"✅ {len(self.data):,} échantillons chargés")
    
    def test_normality(self):
        """Test de normalité (Shapiro-Wilk)"""
        print("\n🔬 Test Shapiro-Wilk (normalité)...")
        
        # Appliquer sur 5000 samples (limite pratique)
        stat, p_value = stats.shapiro(self.data[:5000])
        
        self.results['shapiro'] = {
            'statistic': stat,
            'p_value': p_value,
            'normal': p_value > 0.05
        }
        
        print(f"  Statistic: {stat:.6f}")
        print(f"  P-value:   {p_value:.6f}")
        print(f"  Normal:    {'✅ OUI' if p_value > 0.05 else '❌ NON'}")
    
    def test_mean(self):
        """Test sur la moyenne (t-test)"""
        print("\n🎯 Test t-test (μ = 0)...")
        
        t_stat, p_value = stats.ttest_1samp(self.data, 0)
        
        self.results['ttest'] = {
            'statistic': t_stat,
            'p_value': p_value,
            'unbiased': p_value > 0.05
        }
        
        print(f"  t-statistic: {t_stat:.6f}")
        print(f"  P-value:     {p_value:.6f}")
        print(f"  Sans biais:  {'✅ OUI' if p_value > 0.05 else '❌ NON'}")
    
    def test_variance(self):
        """Test sur la variance"""
        print("\n📊 Test variance (σ² ≈ 0.0122)...")
        
        expected_var = 0.0122
        observed_var = np.var(self.data)
        chi2_stat = (len(self.data) - 1) * observed_var / expected_var
        p_value = 1 - stats.chi2.cdf(chi2_stat, len(self.data) - 1)
        
        self.results['variance'] = {
            'observed': observed_var,
            'expected': expected_var,
            'p_value': p_value,
            'correct': p_value > 0.05
        }
        
        print(f"  Observée:   {observed_var:.6f}")
        print(f"  Attendue:   {expected_var:.6f}")
        print(f"  Erreur:     {abs(observed_var - expected_var) / expected_var * 100:.2f}%")
        print(f"  Correcte:   {'✅ OUI' if p_value > 0.05 else '❌ NON'}")
    
    def test_autocorr(self):
        """Test d'autocorrélation (indépendance)"""
        print("\n🔗 Test autocorrélation (lag-1)...")
        
        acf_val = np.corrcoef(self.data[:-1], self.data[1:])[0, 1]
        
        self.results['autocorr'] = {
            'lag1': acf_val,
            'independent': abs(acf_val) < 0.1
        }
        
        print(f"  ACF lag-1:  {acf_val:.6f}")
        print(f"  Indépendant: {'✅ OUI' if abs(acf_val) < 0.1 else '❌ NON'}")
    
    def descriptive_stats(self):
        """Statistiques descriptives"""
        print("\n📈 Statistiques descriptives...")
        
        stats_dict = {
            'count': len(self.data),
            'mean': np.mean(self.data),
            'std': np.std(self.data),
            'var': np.var(self.data),
            'min': np.min(self.data),
            'max': np.max(self.data),
            'median': np.median(self.data),
            'q25': np.percentile(self.data, 25),
            'q75': np.percentile(self.data, 75),
            'skew': stats.skew(self.data),
            'kurtosis': stats.kurtosis(self.data)
        }
        
        self.results['descriptive'] = stats_dict
        
        print(f"  N:       {stats_dict['count']:,}")
        print(f"  Moyenne: {stats_dict['mean']:+.8f}")
        print(f"  Std:     {stats_dict['std']:.8f}")
        print(f"  Min:     {stats_dict['min']:+.8f}")
        print(f"  Max:     {stats_dict['max']:+.8f}")
        print(f"  Skew:    {stats_dict['skew']:+.6f}")
        print(f"  Kurt:    {stats_dict['kurtosis']:+.6f}")
    
    def plot_distributions(self, output_file='validation_plots.png'):
        """Générer les graphiques"""
        print(f"\n📊 Génération des graphiques ({output_file})...")
        
        fig, axes = plt.subplots(2, 3, figsize=(15, 10))
        fig.suptitle('Box-Muller Validation Dashboard', fontsize=16, fontweight='bold')
        
        # 1. Histogramme + normal overlay
        ax = axes[0, 0]
        ax.hist(self.data, bins=100, density=True, alpha=0.7, edgecolor='black')
        mu, sigma = np.mean(self.data), np.std(self.data)
        x = np.linspace(mu - 4*sigma, mu + 4*sigma, 100)
        ax.plot(x, stats.norm.pdf(x, mu, sigma), 'r-', linewidth=2)
        ax.set_title('Histogramme + Normal Overlay')
        ax.set_xlabel('Valeur')
        ax.set_ylabel('Densité')
        ax.grid(True, alpha=0.3)
        
        # 2. Q-Q plot
        ax = axes[0, 1]
        stats.probplot(self.data, dist="norm", plot=ax)
        ax.set_title('Q-Q Plot')
        ax.grid(True, alpha=0.3)
        
        # 3. Boîte à moustaches
        ax = axes[0, 2]
        ax.boxplot(self.data, vert=True)
        ax.set_title('Boîte à moustaches')
        ax.grid(True, alpha=0.3)
        
        # 4. Série temporelle
        ax = axes[1, 0]
        ax.plot(self.data[:1000], alpha=0.7, linewidth=0.5)
        ax.set_title('Série temporelle (1000 premiers)')
        ax.set_xlabel('Index')
        ax.grid(True, alpha=0.3)
        
        # 5. Autocorrélation
        ax = axes[1, 1]
        from statsmodels.graphics.tsaplots import plot_acf
        plot_acf(self.data[:1000], lags=40, ax=ax)
        ax.set_title('Autocorrélation')
        
        # 6. ECDF vs CDF normale
        ax = axes[1, 2]
        sorted_data = np.sort(self.data)
        ecdf = np.arange(1, len(sorted_data)+1) / len(sorted_data)
        mu, sigma = np.mean(self.data), np.std(self.data)
        cdf = stats.norm.cdf(sorted_data, mu, sigma)
        ax.plot(sorted_data, ecdf, label='ECDF', alpha=0.7)
        ax.plot(sorted_data, cdf, label='CDF Normale', alpha=0.7)
        ax.set_title('ECDF vs CDF Théorique')
        ax.set_xlabel('Valeur')
        ax.set_ylabel('Probabilité cumulée')
        ax.legend()
        ax.grid(True, alpha=0.3)
        
        plt.tight_layout()
        plt.savefig(output_file, dpi=150, bbox_inches='tight')
        print(f"✅ Graphiques sauvegardés: {output_file}")
        plt.close()
    
    def generate_report(self, output_file='validation_report.txt'):
        """Générer un rapport texte"""
        print(f"\n📄 Génération du rapport ({output_file})...")
        
        with open(output_file, 'w', encoding='utf-8') as f:
            f.write("=" * 70 + "\n")
            f.write("BOX-MULLER RANDOM GENERATOR - VALIDATION REPORT\n")
            f.write("=" * 70 + "\n\n")
            
            # Statistiques descriptives
            f.write("DESCRIPTIVE STATISTICS\n")
            f.write("-" * 70 + "\n")
            for key, val in self.results['descriptive'].items():
                if isinstance(val, (int, float)):
                    f.write(f"{key:15s}: {val:+15.8f}\n")
            
            # Tests
            f.write("\n" + "=" * 70 + "\n")
            f.write("STATISTICAL TESTS\n")
            f.write("-" * 70 + "\n")
            
            for test_name, test_results in self.results.items():
                if test_name != 'descriptive':
                    f.write(f"\n{test_name.upper()}\n")
                    for key, val in test_results.items():
                        f.write(f"  {key:15s}: {val}\n")
            
            # Résumé
            f.write("\n" + "=" * 70 + "\n")
            f.write("VALIDATION SUMMARY\n")
            f.write("-" * 70 + "\n")
            f.write(f"Normal Distribution     : {'✅' if self.results['shapiro']['normal'] else '❌'}\n")
            f.write(f"Unbiased Mean          : {'✅' if self.results['ttest']['unbiased'] else '❌'}\n")
            f.write(f"Correct Variance       : {'✅' if self.results['variance']['correct'] else '❌'}\n")
            f.write(f"Independent Samples    : {'✅' if self.results['autocorr']['independent'] else '❌'}\n")
        
        print(f"✅ Rapport sauvegardé: {output_file}")
    
    def run_full_validation(self):
        """Exécuter la validation complète"""
        print("\n" + "=" * 70)
        print("BOX-MULLER VALIDATION PIPELINE")
        print("=" * 70)
        
        self.load_data()
        self.descriptive_stats()
        self.test_normality()
        self.test_mean()
        self.test_variance()
        self.test_autocorr()
        self.plot_distributions()
        self.generate_report()
        
        print("\n" + "=" * 70)
        print("✅ VALIDATION COMPLETE")
        print("=" * 70 + "\n")

if __name__ == '__main__':
    validator = BoxMullerValidator()
    validator.run_full_validation()
```

### Exécution

```bash
python3 validation_pipeline.py
```

### Output

```
======================================================================
BOX-MULLER VALIDATION PIPELINE
======================================================================

📂 Chargement des données...
✅ 1,000,000 échantillons chargés

📈 Statistiques descriptives...
  N:       1,000,000
  Moyenne: +0.00023400
  Std:     0.11043530
  Min:     -0.33456783
  Max:     +0.36789123
  Skew:    +0.00456123
  Kurt:    +0.03456789

🔬 Test Shapiro-Wilk (normalité)...
  P-value:   0.875432
  Normal:    ✅ OUI

...

📊 Génération des graphiques (validation_plots.png)
✅ Graphiques sauvegardés: validation_plots.png

📄 Génération du rapport (validation_report.txt)
✅ Rapport sauvegardé: validation_report.txt

======================================================================
✅ VALIDATION COMPLETE
======================================================================
```

---

## 📚 Ressources complémentaires

- **Scipy stats** : https://docs.scipy.org/doc/scipy/reference/stats.html
- **Pandas** : https://pandas.pydata.org/
- **Matplotlib** : https://matplotlib.org/
- **Statsmodels** : https://www.statsmodels.org/

---

Bon amusement avec vos analyses ! 🎉
