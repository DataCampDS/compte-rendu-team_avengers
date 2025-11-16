# suivi-datacamp

# Compte rendu du projet DataCamp

## Analyse exploratoire

- Observation des gènes hautement représentés.
- Observation des gènes les plus variables.
- Suppression des gènes hautement représentés.
- Visualisation des résultats via ACP pour vérifier une meilleure représentation.

## Normalisation

- Recherche sur différentes méthodes de normalisation.
- Méthodes identifiées : log-normalisation et library size normalisation.

## Réduction de dimension

- Détermination du nombre optimal de composantes pour l’ACP.
- Test d'autres méthodes de réduction de dimensions (tNSE, UMAP).

## Déséquilibre des classes

- Observation d’un déséquilibre important entre les classes.
- augmentation de données de façon random dans la classe sous représentée.

## Choix des modèles

- Tests préliminaires avec un Random Forest naïf, avec quelques ajustements de paramètres.
- Avec moins de 100k observations et en se basant sur la carte des méthodes de scikit-learn, sélection de LinearSVC comme modèle principal. Ajustements.
