# AI53 MLOps – Projet Phase 1 – Rapport de lab

## Milestone 0 – Audit du starter

| # | Fichier ou réglage | Problème | Correction |
|---|--------------------|----------|------------|
| 1 | `reproducibility.py` – `set_reproducibility()` | Seule la seed de Python est fixée, pas celles de NumPy et de PyTorch. Deux entraînements avec les mêmes réglages peuvent donc donner des résultats différents. | Fixer aussi la seed de NumPy et de PyTorch. |
| 2 | `reproducibility.py` – `make_generator()` | La seed n'est pas utilisée pour mélanger les images, donc l'ordre des images change d'un entraînement à l'autre. | Créer le générateur avec la seed et compléter `seed_worker()`. |
| 3 | `validate_data.py` | Les données ne sont pas vérifiées : on pourrait entraîner le modèle sur des images manquantes ou abîmées sans s'en rendre compte. | Coder les vérifications et arrêter le pipeline en cas d'erreur. |
| 4 | Pas de `dvc.yaml` | Il n'y a pas de pipeline : rien n'oblige à vérifier les données avant de les préparer, et rien ne relance la préparation quand on change un réglage. | Créer un pipeline DVC qui indique pour chaque étape ce qu'elle lit, ce qu'elle utilise comme réglages et ce qu'elle produit. |
| 5 | `provenance.py` | On ne garde aucune trace de ce qui a servi à entraîner un modèle : commit, versions des librairies, machine utilisée. | Enregistrer ces informations au début de chaque Run. |
| 6 | `train.py` | Rien n'est enregistré dans MLflow, et `model.pt` est remplacé à chaque entraînement : on perd les anciens modèles. | Enregistrer les réglages, les résultats et le modèle de chaque Run dans MLflow. |
| 7 | `pyproject.toml` | Les versions des librairies ne sont pas fixées précisément : deux installations peuvent donner des versions différentes. | Installer avec `uv sync --locked` pour utiliser les versions exactes de `uv.lock`. |
| 8 | `params.yaml` – `device: "auto"` | Le code choisit tout seul le CPU ou le GPU selon la machine, et ce choix n'est pas enregistré dans MLflow. | Enregistrer le device utilisé dans MLflow, ou imposer le CPU. |
| 9 | `params.yaml` – `selection.run_id: ""` | Aucun modèle n'est encore choisi. Si on testait plusieurs modèles pour garder le meilleur, le score de test ne serait plus fiable. | Choisir le modèle avec le jeu de validation, puis le tester une seule fois. |
| 10 | `data/raw/` | Les images ne sont pas versionnées : on ne sait pas quelle version des données a été utilisée. | Versionner les images avec DVC. |

### Engineering checkpoint 1

**Pourquoi un fichier `model.pt` qui fonctionne ne suffit-il pas à prouver qu'une expérience est reproductible ?**

Le fichier `model.pt` contient seulement les poids du modèle entraîné. Il ne dit pas avec quel code, quelles données, quelles versions des librairies, quels réglages ni quelle seed il a été obtenu. Sans ces informations, on ne peut pas refaire l'entraînement et vérifier qu'on retrouve le même résultat.