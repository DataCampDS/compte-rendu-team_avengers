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

## 5. Modèles testés et performances

Plusieurs modèles ont été entraînés et évalués après normalisation, réduction de dimensions et rééquilibrage des classes. Les principaux modèles testés sont : Lasso, SVM, XGBoost, RandomForest, LinearSVC et un ensemble Stacking.

### Résultats synthétiques

| Modèle       | Train Bal. Acc | Train CV (moy.) | Test Bal. Acc | Test CV (moy.) |
| ------------ | -------------- | --------------- | ------------- | -------------- |
| Lasso        | 100%           | 83.1%           | 77.6%         | 82.2%          |
| SVM          | 91.35%         | 70.6%           | 59.3%         | 58%            |
| XGBoost      | 100%           | 83.2%           | 77.9%         | 78.2%          |
| RandomForest | -              | -               | -             | -              |
| LinearSVC    | -              | -               | 81%           | -              |
| Stacking     | 100%           | 86.9%           | 79.9%         | 81%            |

Le Stacking et LinearSVC offrent les meilleures performances globales.

## 6. Prochaines étapes

- Approches avancées d’augmentation de données (GANs, bruit, bootstrapping).
- Sélection plus fine des gènes (highly variable genes).
- Test de pipelines comparant différentes normalisations.
- Regroupement de classes biologiquement proches et entraînement de modèles hiérarchiques.

---

## 7. Essais : Approche en deux étapes avec modèles séparés

### 7.1 Approche en deux étapes avec modèles séparés

L'analyse exploratoire révèle que les classes NK_cells et T_cells_CD8+ sont difficiles à distinguer des autres classes mais potentiellement plus faciles à séparer entre elles. Cette observation motive une approche hiérarchique en deux étapes.

#### Architecture du modèle

**Stage 1 : Modèle à 3 classes (Merged Model)**

- Fusion des classes NK_cells et T_cells_CD8+ en une seule classe : "NK_or_T_cells_CD8+"
- Classification entre : B_cells, Monocytes, NK_or_T_cells_CD8+
- Pipeline : Normalisation → Filtrage gènes (variance ≥ 0.2) → PCA (60 composantes) → Stacking Classifier

**Stage 2 : Modèle binaire (Binary Model)**

- Classification fine entre NK_cells et T_cells_CD8+ uniquement
- Pipeline : Normalisation → Filtrage gènes (variance ≥ 0.8) → PCA (80% variance) → Stacking Classifier

#### Stratégie de prédiction

1. Tous les échantillons passent par le Stage 1
2. Les échantillons prédits comme B_cells ou Monocytes conservent cette prédiction
3. Les échantillons prédits comme "NK_or_T_cells_CD8+" sont envoyés au Stage 2 pour distinction fine

#### Configuration technique

**Stacking Classifier (identique pour les deux stages)** :

- Estimateurs de base : Random Forest (200 arbres), SVM (RBF kernel), KNN (15 voisins)
- Meta-estimateur : Régression Logistique

**Paramètres optimisables via Grid Search** :

- Seuils de variance pour filtrage des gènes (Stage 1 et 2)
- Nombre de composantes PCA (Stage 1 et 2)
- Hyperparamètres des modèles de base (n_estimators, max_depth, C, n_neighbors)

#### Avantages de l'approche

- Spécialisation de chaque modèle sur un sous-problème plus simple
- Filtrage de gènes adapté à chaque étape (différents seuils de variance)
- Réduction potentielle du surapprentissage par rapport à un modèle unique 4 classes
- Architecture modulaire permettant l'optimisation indépendante de chaque stage

---

# Résumé des méthodes et résultats (Sarah)

## Méthode 1 : Stacking SVM + Random Forest

- Prétraitement : Log-normalisation pour réduire l’étendue des données.
- Augmentation de données : duplication de lignes pour équilibrer les classes.

* Réduction de dimension : Analyse en Composantes Principales (ACP).
* Modélisation : Stacking combinant SVM et Random Forest.

Résultat :

- Balanced accuracy : 0,80
- Précision par classe : > 0,8
- Précision pour _Cancer cells_ et _NK_ : > 0,9

---

## Méthode 2 : SVM avec GridSearchCV

- Prétraitement : Log-normalisation.
- Augmentation de données : duplication pour équilibrer les classes.
- Sélection de variables : SelectKBest pour ne conserver que les variables les plus informatives.
- Modélisation : SVM avec optimisation des hyperparamètres via GridSearchCV.

Résultat :

- Balanced accuracy : 0,79
- Précision par classe : > 0,8
- Précision pour _Cancer cells_ et _NK_ : > 0,9

---

## Méthode 3 : Random Forest

- Prétraitement : Log-normalisation + normalisation de la taille (_size normalization_).
- Correction du déséquilibre des classes : via les paramètres du modèle.
- Sélection de variables : SelectKBest.
- Modélisation : Random Forest simple.

Résultat :

- Balanced accuracy : **0,73**

---

## Méthode 4 : Régression Logistique

- **Prétraitement** : Log-normalisation + normalisation de la taille.
- **Correction du déséquilibre des classes** : via les paramètres du modèle.
- **Sélection de variables** : `SelectKBest`.
- **Modélisation** : Régression logistique en deux étapes avec `GridSearchCV`.

**Résultat** :

- Balanced accuracy : **0,86**
- **Inconvénient** : temps d’entraînement long

---

## Méthode 5 : Random Forest en deux étapes

- Prétraitement : Log-normalisation + augmentation des données.
- Réduction de dimension : ACP pour extraire les composantes les plus informatives.
- Modélisation : Random Forest en deux étapes

Résultat :

- Balanced accuracy : 0,76

À l’aide des représentations graphiques (ACP, t-SNE, UMAP), on observe qu’il
existe un hyperplan capable de séparer la classe Cancer cells des autres.
En revanche, la séparation entre les autres classes est moins évidente.
L’utilisation de méthodes de regroupement géométriques ou basées sur
la distance telles que SVM ou KNN.
