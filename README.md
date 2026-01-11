<div align="center">
  <img src="https://readme-typing-svg.herokuapp.com?font=Orbitron&weight=900&size=48&duration=3000&pause=1000&color=667EEA&background=FFFFFF00&center=true&vCenter=true&multiline=true&repeat=false&width=600&height=100&lines=TEAM+AVENGERS" alt="Team Avengers" />
</div>

---


# suivi-datacamp

# Compte rendu TEAM_AVENGERS

## 1. Exploration des données

### 1.1 Répartition des classes

| Classe       | Effectif | Proportion |
| ------------ | -------- | ---------- |
| T_cells_CD8+ | 237      | 0.342      |
| T_cells_CD4+ | 85       | 0.336      |
| Cancer_cells | 336      | 0.237      |
| NK_cells     | 342      | 0.085      |

Une sous-représentation marquée de la classe NK_cells a été observée.

### 1.2 Structure des données

- 13 551 gènes au total.
- Matrice extrêmement creuse (sparse).
- Analyse des gènes les plus exprimés et les plus variables.
- Suppression de certains gènes trop représentés pour réduire les biais.
- Visualisation via ACP pour explorer la structure globale.

### 1.3 Statistiques descriptives

- Analyse des moyennes/écarts-types par classe.
- Comparaison avant et après normalisation.

## 2. Normalisation des données

Plusieurs normalisations courantes en scRNA-seq ont été étudiées et testées :

- Normalisation simple.
- Log-normalisation.
- CPM (Counts Per Million).
- Library Size Normalization.

Objectif : réduire les effets du séquençage et rendre les cellules comparables.

## 3. Réduction de dimension

- Application d’une ACP (PCA).
- Conservation de 90 % de la variance expliquée, menant à 717 composantes.
- Visualisations supplémentaires via t-SNE et UMAP.

## 4. Gestion du déséquilibre des classes

- Déséquilibre marqué au niveau de NK_cells.
- Oversampling simple pour équilibrer les classes.
- Méthodes futures envisagées :
  - Bootstrapping.
  - Injection de bruit.
  - Génération de données synthétiques (GANs).

## 5. Séléction du modèle et approche d'apprentissage 

### 5.1 Approche 1 :

Plusieurs modèles ont été entraînés et évalués après normalisation, réduction de dimensions et rééquilibrage des classes. L'objectif est de choisir le modèle le plus performant en terme de **balanced accuracy**.

Les principaux modèles testés sont : 
Logistic regression (Lasso), SVM, XGBoost, RandomForest, LinearSVC, LightGBM et un ensemble Stacking.
#### 5.1.a Classidication par Régression Logistique avec régularisation L1 (Lasso)
- Prétraitement : Log-normalisation pour réduire l’étendue des données.
* Réduction de dimension : Analyse en Composantes Principales (ACP) avec une variance expliquée de 90%.
* Modélisation : Régression Logistique avec régularisation de Lasso pour la séléction des variables discriminantes.

**Résultat** : 

- Balanced accuracy_test après Cross_Vald : **0,82**
- Pouvoir de prédiction des classes minoritaires : 
 115 cellules de Cancer_cells parmi 118
 22 cellules de NK_cells parmi 43

#### 5.1.b Classification par XGBoost
- Prétraitement : Log-normalisation pour réduire l’étendue des données.
* Réduction de dimension : Analyse en Composantes Principales (ACP) avec une variance expliquée de 90%.
* Modélisation : eXtreme Gradient boosting basé sur l'entraînement de plusieurs arbes tout en courigeant les erreurs des précédantes.

**Résultat** : 

- Balanced accuracy_test après Cross_Vald: **0,782**
- Pouvoir de prédiction des classes minoritaires : 
 114 cellules de Cancer_cells parmi 118
 19 cellules de NK_cells parmi 43

#### 5.1.c Stacking SVM + Random Forest + Lasso 

- Prétraitement : Log-normalisation pour réduire l’étendue des données.
- Augmentation de données : duplication de lignes pour équilibrer les classes.

* Réduction de dimension : Analyse en Composantes Principales (ACP).
* Modélisation : Stacking combinant SVM et Random Forest en premier niveau puis la regression logistique en deuxième niveau.

**Résultat** :

- Balanced accuracy_test après Cross_Vald : **0,81**
- Pouvoir de prédiction des classes minoritaires : 
 116 cellules de Cancer_cells parmi 118
 19 cellules de NK_cells parmi 43

#### 5.1.d LightGBM : Light Gradient Boosting

- Prétraitement : Log-normalisation pour réduire l’étendue des données.
- Pour LGBM, il performe mieux sans ACP puisqu'il choisit lui même les variables utiles et ignorent les autres.  

* Modélisation : LightGBM est entraîné en apprennant lentement (lr=0.05) pour une meilleure généralisation, en faisant des sous échantillonnage des observations et de variables (subsample, colsample_bytree) pour un anti-overffiting et en ajustant les poids des classes.

**Résultat** : 

- Balanced accuracy_test après Cross_Vald : **0,85**
- Pouvoir de prédiction des classes minoritaires : 
 116 cellules de Cancer_cells parmi 118
 24 cellules de NK_cells parmi 43


#### 5.1.e Random Forest

- Prétraitement : Log-normalisation + normalisation de la taille (_size normalization_)
- Correction du déséquilibre des classes : via les paramètres du modèle
- Sélection de variables : SelectKBest
- Modélisation : Random Forest simple

**Résultat** :

- Balanced accuracy : **0,73**


#### 5.1.f Random Forest en deux étapes

- Prétraitement : Log-normalisation + augmentation des données
- Réduction de dimension : ACP pour extraire les composantes les plus informatives
- Modélisation : Random Forest en deux étapes

**Résultat** :

- Balanced accuracy : **0,76**

#### 5.1.g Régression logistique (cancer cells vs autre) + lgbMClassifier (NK vs T) + SVC (T CD+4 vs T CD+8)

- Prétraitement : Log-normalisation + Normalisation de la taille des librairies (size normalization)
- Sélection de variables : highly variable gene
- Modélisation : Classification en plusieurs étapes

**Résultat** :

- Balanced accuracy : **0,89**

 #### 5.1.h Régression logistique (cancer cells vs autre) + lgbMClassifier (NK vs T) + lgbMClassifier (T CD+4 vs T CD+8)

- Prétraitement : Log-normalisation + Normalisation de la taille des librairies (size normalization)
- Sélection de variables : highly variable gene
- Modélisation : Classification en plusieurs étapes

**Résultat** :

- Balanced accuracy : **0,88**


À l’aide des représentations graphiques (ACP, t-SNE, UMAP), on observe qu’il existe un hyperplan capable de séparer la classe Cancer cells des autres.
En revanche, la séparation entre les autres classes est moins évidente.
L’utilisation de méthodes de regroupement géométriques ou basées sur
la distance telles que SVM ou KNN apparaît pertinent.

### 5.2 Approche 2 : Approche en deux étapes avec modèles séparés
### 5.2.a Approche en deux étapes avec modèles séparés

L'analyse exploratoire révèle que les classes NK_cells et T_cells_CD8+ sont difficiles à distinguer des autres classes mais potentiellement plus faciles à séparer entre elles. Cette observation motive une approche hiérarchique en deux étapes.

#### 5.2.b Architecture du modèle

**Stage 1 : Modèle à 3 classes (Merged Model)**

- Fusion des classes NK_cells et T_cells_CD8+ en une seule classe : "NK_or_T_cells_CD8+"
- Classification entre : B_cells, Monocytes, NK_or_T_cells_CD8+
- Pipeline : Normalisation → Filtrage gènes (variance ≥ 0.2) → PCA (60 composantes) → Stacking Classifier

**Stage 2 : Modèle binaire (Binary Model)**

- Classification fine entre NK_cells et T_cells_CD8+ uniquement
- Pipeline : Normalisation → Filtrage gènes (variance ≥ 0.8) → PCA (80% variance) → Stacking Classifier

#### 5.2.c Stratégie de prédiction

1. Tous les échantillons passent par le Stage 1
2. Les échantillons prédits comme B_cells ou Monocytes conservent cette prédiction
3. Les échantillons prédits comme "NK_or_T_cells_CD8+" sont envoyés au Stage 2 pour distinction fine

#### 5.2.d Configuration technique

**Stacking Classifier (identique pour les deux stages)** :

- Estimateurs de base : Random Forest (200 arbres), SVM (RBF kernel), KNN (15 voisins)
- Meta-estimateur : Régression Logistique

**Paramètres optimisables via Grid Search** :

- Seuils de variance pour filtrage des gènes (Stage 1 et 2)
- Nombre de composantes PCA (Stage 1 et 2)
- Hyperparamètres des modèles de base (n_estimators, max_depth, C, n_neighbors)

#### 5.2.e Avantages de l'approche

- Spécialisation de chaque modèle sur un sous-problème plus simple
- Filtrage de gènes adapté à chaque étape (différents seuils de variance)
- Réduction potentielle du surapprentissage par rapport à un modèle unique 4 classes
- Architecture modulaire permettant l'optimisation indépendante de chaque stage

**Résultat** : 
- Balanced_acc : **0.88**

### Résultats synthétiques

| Modèle       | Train Bal. Acc | Train Cross_Vald (moy.) | Test Bal. Acc | Test Ccross_Vald (moy.) |
| ------------ | -------------- | ------------------------| ------------- | ------------------------|      
|LogiReg(Lasso)| 100%           | 83.1%                   | 77.6%         | 82.2%                   |
| XGBoost      | 100%           | 83.2%                   | 77.9%         | 78.2%                   |
| LightGBM     | 100%           | 88,5                    | 82,9%         | 84,6%                   |
| Stacking     | 100%           | 86,9%                   | 77,8%-        | 80,9%                   |
| RandomForest | -              | -                       | 73%           | -                       |
| RandomForest | -              | -                       | 76%           | -                       |
| Approche 2   | 100%           | 83.2%                   | 77.9%         | 78.2%                   |




