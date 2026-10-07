# Alerte Suivi Glycémique — Prédiction du dépistage HbA1c

Ce projet implémente un module d'**Alerte Suivi Glycémique** visant à prédire le résultat du test d'hémoglobine glyquée (**HbA1c**) dès l'admission des patients diabétiques hospitalisés, à partir de données cliniques et démographiques (jeu de données *Diabetes 130-US Hospitals*).

---

## 🎯 Contexte & Objectif

L'**hémoglobine glyquée (HbA1c)** reflète l'équilibre glycémique moyen des 2 à 3 derniers mois. C'est l'examen de référence pour évaluer le contrôle du diabète et prévenir les complications micro/macrovasculaires.

Cependant, plus de **80 % des patients diabétiques hospitalisés** dans ce jeu de données ne bénéficient d'aucun test HbA1c durant leur séjour (opportunité manquée de dépistage).

### Le rôle du modèle :
1. **Dépistage ciblé** : Repérer les patients à risque d'absence de suivi ou de mauvais contrôle glycémique dès l'admission.
2. **Priorisation des ressources** : Concentrer les examens de laboratoire sur les cas critiques.
3. **Filet de sécurité avant sortie** : Alerte pour éviter que des patients mal équilibrés sortent sans contrôle.

---

## 📊 Données

- `diabetic_data.csv` : Données hospitalières couvrant 10 ans (1999–2008) pour des patients diabétiques dans 130 hôpitaux américains.
- `IDS_mapping.csv` : Tables de correspondance pour les identifiants d'admission, de sortie et de provenance.
- Cible : `A1Cresult` (Non mesuré / None, Normal, Anormal [>7 / >8]).

---

## 🛠️ Méthodologie & Notebook (`notebook-A1Cresult.ipynb`)

1. **Exploration des données** : Constats cliniques, déséquilibre des classes et analyse exploratoire.
2. **Définition & Simplification de la cible** :
   - Cible brute (4 classes) : `None`, `Norm`, `>7`, `>8`.
   - Cible simplifiée (3 classes) : `Aucun`, `Norm`, `Anormal` (fusion de `>7` et `>8`).
3. **Nettoyage des données (Data Cleaning)** : Traitement des valeurs manquantes (`?`), exclusion des colonnes à forte cardinalité ou majorité de valeurs manquantes (ex. `weight`, `payer_code`).
4. **Sélection et Ingénierie des caractéristiques (Feature Engineering)**.
5. **Entraînement & Évaluation** :
   - Modèle de référence : Random Forest avec gestion du déséquilibre (`class_weight='balanced'`).
   - Optimisation des hyperparamètres via `GridSearchCV`.
   - Évaluation de techniques de rééquilibrage (`SMOTE`).

### 📈 Résultats clés :
- Passage d'un **F1 macro de 0.338 à 0.459** (+36% d'amélioration globale).
- Le plus grand gain (+0.089) provient de la simplification clinique de la cible (4 → 3 classes).
- Optimisation Random Forest (`n_estimators=300`, `max_depth=16`).

---

## 🚀 Utilisation

1. Cloner le dépôt :
```bash
git clone https://github.com/MolkaJebali/diabetes-a1c-screening-prediction.git
cd diabetes-a1c-screening-prediction
```

2. Installer les dépendances :
```bash
pip install numpy pandas scikit-learn matplotlib seaborn imbalanced-learn jupyter
```

3. Lancer le notebook :
```bash
jupyter notebook notebook-A1Cresult.ipynb
```
