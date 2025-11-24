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

---

## 2. Normalisation des données

Plusieurs normalisations courantes en scRNA-seq ont été étudiées et testées :

- Normalisation simple.
- Log-normalisation.
- CPM (Counts Per Million).
- Library Size Normalization.

Objectif : réduire les effets du séquençage et rendre les cellules comparables.

---

## 3. Réduction de dimension

- Application d’une ACP (PCA).
- Conservation de 90 % de la variance expliquée, menant à 717 composantes.
- Visualisations supplémentaires via t-SNE et UMAP.

---

## 4. Gestion du déséquilibre des classes

- Déséquilibre marqué au niveau de NK_cells.
- Oversampling simple pour équilibrer les classes.
- Méthodes futures envisagées :
  - Bootstrapping.
  - Injection de bruit.
  - Génération de données synthétiques (GANs).

---

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

---

## 6. Prochaines étapes

- Approches avancées d’augmentation de données (GANs, bruit, bootstrapping).
- Sélection plus fine des gènes (highly variable genes).
- Test de pipelines comparant différentes normalisations.
- Regroupement de classes biologiquement proches et entraînement de modèles hiérarchiques.
