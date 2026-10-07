# Diabetes Care Gap Prediction
### Anticiper les lacunes de dépistage HbA1c chez les patients diabétiques hospitalisés

![Python](https://img.shields.io/badge/Python-3.12-blue)
![scikit--learn](https://img.shields.io/badge/scikit--learn-ML-orange)
![imbalanced--learn](https://img.shields.io/badge/imbalanced--learn-SMOTE-yellow)
![License](https://img.shields.io/badge/License-MIT-green)

---

## Contexte

L'**hémoglobine glyquée (HbA1c)** est l'examen de référence pour évaluer l'équilibre glycémique d'un patient diabétique sur les 2 à 3 derniers mois. Un mauvais contrôle non détecté expose le patient à des complications à long terme (rénales, cardiovasculaires, rétiniennes).

L'analyse du jeu de données révèle un constat frappant : **plus de 80 % des patients diabétiques hospitalisés ne reçoivent aucun test HbA1c pendant leur séjour** — une occasion de dépistage manquée à grande échelle, alors même que le patient est présent et suivi par l'équipe médicale.

Ce projet construit un modèle de classification capable d'identifier, dès l'admission, les patients présentant le plus grand risque de ne pas être dépistés ou de présenter un mauvais contrôle glycémique, afin de transformer un dépistage aujourd'hui largement laissé au hasard en un dépistage ciblé.

## Objectifs du projet

| | |
|---|---|
| **Besoin métier** | Identifier, dès l'admission, les patients diabétiques dont le suivi glycémique risque d'être insuffisant ou de révéler un mauvais contrôle du diabète, afin de cibler en priorité les actions de suivi médical et de prévention. |
| **Besoin Data Science** | Construire un modèle de classification supervisée qui prédit le résultat du test HbA1c (`A1Cresult`) à partir des informations cliniques et démographiques disponibles pour le patient. |
| **Cible** | `A1Cresult`, simplifiée en 3 classes : `None` (non testé), `Norm` (normal), `Anormal` (`>7` ou `>8`, mauvais contrôle) |
| **Type de tâche** | Classification multiclasse supervisée |

## Jeu de données

- **Source :** [Diabetes 130-US hospitals for years 1999–2008](https://archive.ics.uci.edu/dataset/296/diabetes+130-us+hospitals+for+years+1999-2008) (UCI Machine Learning Repository)
- **Volume :** 101 766 hospitalisations, 50 variables (démographiques, cliniques, administratives, traitements)
- **Référentiel associé :** `IDS_mapping.csv` (table de correspondance des identifiants administratifs)

## Pipeline du projet

```
1. Chargement et exploration des données
2. Analyse et simplification de la variable cible (A1Cresult, 4 → 3 classes)
3. Data Cleaning
   ├── Dédoublonnage des patients (1 épisode par patient)
   ├── Suppression des colonnes trop incomplètes (weight, payer_code...)
   ├── Imputation des valeurs manquantes
   └── Suppression des colonnes à variance quasi nulle
4. Feature Selection
5. Data Transforms
   ├── Regroupement clinique des diagnostics (ICD-9 → grandes catégories)
   ├── Encodage ordinal (age)
   ├── Encodage one-hot (variables nominales)
   └── Standardisation des variables numériques
6. Feature Engineering (total_visits_prior, nb_meds_changed)
   └── Vérification des risques de fuite de données
7. Discussion sur la réduction de dimensionnalité (PCA)
8. Modélisation de référence (Random Forest)
9. Évaluation (F1 macro, matrice de confusion, importance des variables)
10. Optimisation (GridSearchCV, SMOTE) et comparaison finale
```

## Résultats

| Approche | F1 macro (test) |
|---|---|
| Cible brute à 4 classes, modèle par défaut | 0,338 |
| Cible simplifiée à 3 classes, modèle par défaut | 0,427 |
| **Cible à 3 classes, hyperparamètres optimisés (GridSearchCV)** | **0,459** |
| Cible à 3 classes, GridSearchCV + SMOTE | 0,458 |

**Modèle retenu :** Random Forest optimisé par GridSearchCV (sans SMOTE, le gain apporté étant négligeable face à la complexité ajoutée).

Comparé à des baselines naïves (classe majoritaire seule : F1 macro = 0,225 ; tirage aléatoire selon les proportions : 0,247), le modèle final apporte un gain significatif et exploitable pour prioriser les patients à dépister.

## Points méthodologiques clés

- **Aucune fuite de données sur la cible** : contrairement à une cible qui serait reconstruite à partir de plusieurs variables croisées, `A1Cresult` est utilisée telle qu'elle existe nativement dans les données.
- **Vérification active des risques de fuite sur les features** : les variables `max_glu_serum` et `nb_meds_changed`, potentiellement corrélées à la cible, ont été conservées puis vérifiées via l'importance des variables — leur poids s'est révélé faible, écartant un risque de fuite dominante.
- **Choix de métrique justifié** : le F1 macro a été préféré à l'accuracy en raison du fort déséquilibre des classes (82 % de la classe majoritaire).
- **Comparaison chiffrée de chaque levier d'amélioration** (simplification de cible, tuning, rééchantillonnage), plutôt qu'un empilement de techniques sans mesure de leur apport individuel.

## Stack technique

- **Langage :** Python 3.12
- **Manipulation de données :** pandas, numpy
- **Visualisation :** matplotlib, seaborn
- **Machine Learning :** scikit-learn (RandomForestClassifier, GridSearchCV, OrdinalEncoder, StandardScaler, SimpleImputer)
- **Rééquilibrage des classes :** imbalanced-learn (SMOTE)

## Structure du dépôt

```
diabetes-a1c-screening-prediction/
├── data/
│   ├── diabetic_data.csv
│   └── IDS_mapping.csv
├── notebook_A1Cresult_final.ipynb
├── README.md
└── requirements.txt
```

## Installation et utilisation

```bash
# Cloner le dépôt
git clone https://github.com/<votre-username>/diabetes-a1c-screening-prediction.git
cd diabetes-a1c-screening-prediction

# Installer les dépendances
pip install -r requirements.txt

# Lancer le notebook
jupyter notebook notebook_A1Cresult_final.ipynb
```

**`requirements.txt`**
```
pandas
numpy
matplotlib
seaborn
scikit-learn
imbalanced-learn
jupyter
```

## Pistes d'amélioration

- Comparer avec d'autres familles d'algorithmes (régression logistique multinomiale, XGBoost, LightGBM)
- Enrichir les features avec des interactions (ex. âge × nombre de diagnostics)
- Valider le modèle final par validation croisée complète plutôt qu'un seul split train/test

## Auteur

**Molka Jebali**
- Esprit

---

*Projet réalisé dans le cadre d'un travail de groupe sur le jeu de données Diabetes 130-US hospitals, chaque membre traitant un objectif métier et data science distinct.*
