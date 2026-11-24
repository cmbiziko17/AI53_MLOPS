# AI53 MLOps – Projet Phase 1 – Rapport de lab

## Milestone 0 – Audit du starter

| # | Fichier ou réglage | Problème | Correction |
|---|--------------------|----------|------------|
| 1 | `reproducibility.py` – `set_reproducibility()` | Seul le hasard de Python est fixé, pas NumPy ni PyTorch. Deux entraînements identiques donnent des résultats différents. | Fixer aussi la seed de NumPy et de PyTorch. |
| 2 | `reproducibility.py` – `make_generator()` | La seed n'est pas utilisée pour mélanger les images. | Donner la seed au générateur et compléter `seed_worker()`. |
| 3 | `validate_data.py` | Les données ne sont pas vérifiées du tout. | Coder les vérifications et arrêter si une erreur est trouvée. |
| 4 | Pas de `dvc.yaml` | Rien n'oblige à valider les données avant de les préparer, ni à refaire la préparation si on change un réglage. | Créer un pipeline DVC avec les dépendances entre étapes. |
| 5 | `provenance.py` | On ne sait pas avec quel code, quelles versions ni quelle machine un modèle a été entraîné. | Enregistrer ces informations au début de chaque Run. |
| 6 | `train.py` | Rien n'est enregistré dans MLflow, et `model.pt` est écrasé à chaque entraînement. | Enregistrer les paramètres, les métriques et le modèle dans MLflow. |
| 7 | `pyproject.toml` | Les versions des librairies ne sont pas exactes. | Utiliser `uv.lock` avec `uv sync --locked`. |
| 8 | `params.yaml` – `device: "auto"` | Le matériel (CPU ou GPU) change selon la machine et n'est pas enregistré dans MLflow. | Noter le device dans MLflow ou forcer `cpu`. |
| 9 | `params.yaml` – `selection.run_id: ""` | On pourrait tester plusieurs modèles et garder le meilleur, ce qui fausse le score de test. | Choisir le modèle avec la validation, puis le tester une seule fois. |
| 10 | `data/raw/` | Les images ne sont pas versionnées. | Les suivre avec DVC. |

### Engineering checkpoint 1

**Pourquoi un fichier `model.pt` qui fonctionne ne suffit-il pas à prouver qu'une expérience est reproductible ?**

Le fichier `model.pt` contient seulement le modèle entraîné. Il ne dit pas avec quel code, quelles données, quelles versions des librairies, quels réglages ni quelle seed il a été obtenu. Sans ces informations, on ne peut pas refaire l'entraînement et vérifier qu'on obtient le même résultat.