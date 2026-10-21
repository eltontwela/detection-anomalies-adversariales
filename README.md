# Détection robuste d'anomalies face aux intrusions adverses réseaux

Mémoire de Master 2 — IA et Data Science  
Université de Kinshasa (UNIKIN), Faculté des Sciences et Technologies  
Année académique 2025–2026

**Auteure :** Elton Twela Mway

---

## Résumé

Ce travail étudie la robustesse des systèmes de détection d'intrusion (IDS) basés sur un autoencodeur face aux attaques adversariales. Le modèle est entraîné sur le trafic normal du dataset CICIDS2017 et détecte les anomalies par erreur de reconstruction (MSE). Deux types d'attaques adversariales sont évalués : FGSM (Fast Gradient Sign Method) et PGD (Projected Gradient Descent). Deux stratégies de défense sont ensuite comparées, dont une défense semi-supervisée par pénalité charnière (Défense B) qui se révèle efficace contre PGD.

---

## Structure du repository

```
├── README.md
├── figures/
│   ├── fig1_vulnerabilite_baseline.png
│   ├── fig2_comparaison_defenses.png
│   ├── fig3_rappel_par_famille.png
│   ├── fig4_generalisation.png
│   └── fig5_stabilite_15graines.png
└── notebooks/
    ├── experience.ipynb       # Notebook principal (entraînement + expériences)
    └── generer_figures.py     # Script de génération des figures
├── results/
│   ├── resultats_graines_global.csv      # Vulnérabilité du baseline (10 graines)
│   ├── resultats_defense.csv             # Comparaison des défenses (10 graines)
│   ├── resultats_defense_types.csv       # Résultats par famille d'attaque
│   ├── resultats_generalisation.csv      # Protocole leave-one-family-out
│   └── resultats_stabilite.csv           # Stabilité sur 15 graines
```

---

## Dataset

**CICIDS2017** — Canadian Institute for Cybersecurity Intrusion Detection System 2017  
Disponible sur : https://www.unb.ca/cic/datasets/ids-2017.html

Le dataset contient du trafic réseau normal et 14 types d'attaques réparties en 6 familles : DoS, DDoS, PortScan, Patator (brute force), Bot et Web Attacks.

---

## Modèle

Autoencodeur dense entraîné uniquement sur le trafic normal :

| Paramètre | Valeur |
|-----------|--------|
| Architecture | 64 → 32 → 16 → 32 → 64 |
| Activation | ReLU (encodeur), Linéaire (sortie) |
| Optimiseur | Adam |
| Fonction de perte | MSE |
| Époques | 20 |
| Batch size | 256 |
| Seuil de détection | 95e percentile (sur trafic normal) |

La détection repose sur l'erreur de reconstruction : un flux dont la MSE dépasse le seuil est classé comme anomalie.

---

## Attaques adversariales évaluées

**FGSM** (Fast Gradient Sign Method) — perturbation en un seul pas dans la direction du gradient de la perte, budget ε ∈ {0,02 ; 0,05}.

**PGD** (Projected Gradient Descent) — attaque itérative (40 itérations, α = 0,005, norme L∞), budget ε ∈ {0,02 ; 0,05 ; 0,10}.

---

## Défenses comparées

**Défense A** — perturbation non supervisée des exemples normaux pendant l'entraînement. Inefficace : la défense dégrade le rappel propre sans améliorer la robustesse.

**Défense B** — pénalité charnière semi-supervisée. Pendant l'entraînement, des exemples d'attaques camouflées (PGD sur des attaques connues) sont injectés avec une contrainte qui force leur erreur de reconstruction à rester au-dessus du seuil. Hyperparamètres : λ = 1, marge K = 3, budget d'entraînement ε = 0,05.

---

## Protocole expérimental

- **Objectif 1** : vulnérabilité du baseline — 10 graines, FGSM et PGD à 3 budgets
- **Objectif 2** : comparaison des défenses — 10 graines, PGD ε = 0,05
- **Objectif 3** : généralisation — protocole leave-one-family-out, 5 graines par configuration
- **Objectif 4** : stabilité — Défense B, 15 graines, 20 et 40 époques

---

## Principaux résultats

| Condition | Rappel propre | Rappel sous PGD (ε=0,05) |
|-----------|:-------------:|:------------------------:|
| Baseline  | ~75 %         | ~44 %                    |
| Défense A | ~64 %         | ~42 %                    |
| Défense B | ~88 %         | ~75 %                    |

La Défense B améliore substantiellement la robustesse sous PGD. Un comportement bimodal est observé : environ 25 % des graines échouent, indépendamment du nombre d'époques, ce qui suggère une sensibilité à l'initialisation.

---

## Reproduire les expériences

1. Télécharger le dataset CICIDS2017 (lien ci-dessus)
2. Ouvrir `notebooks/experience.ipynb` dans Google Colab
3. Monter Google Drive et ajuster les chemins vers les fichiers CSV du dataset
4. Exécuter les cellules dans l'ordre
5. Pour régénérer les figures : `python notebooks/generer_figures.py`

---

## Dépendances

```
tensorflow >= 2.10
scikit-learn
pandas
numpy
matplotlib
```

Installation : `pip install tensorflow scikit-learn pandas numpy matplotlib`

---

## Encadrement

Mémoire réalisé sous la direction du Professeur Ph.D Jordan Félicien Masakuna enseignant de la Faculté des Sciences et Technologies, Département Informatique, UNIKIN.
