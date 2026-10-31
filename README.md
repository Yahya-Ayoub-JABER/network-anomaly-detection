# Analyse et détection de comportements atypiques dans le trafic réseau

Projet individuel réalisé dans le cadre du cours **Sciences des données 2**, en cycle ingénieur à l'ECAM-EPMI.

## 1. Présentation du projet

Ce projet étudie l'utilisation de méthodes d'apprentissage automatique pour analyser le trafic réseau et identifier des comportements atypiques.

**Problématique :** Peut-on identifier différents profils de trafic réseau et repérer des comportements atypiques à l'aide de méthodes d'apprentissage non supervisé ?

L'objectif est de construire un prototype expérimental de détection d'anomalies réseau, d'évaluer ses performances et d'étudier les compromis entre détection des attaques, fausses alertes et coûts de traitement.

Ce travail constitue également une première base technique pouvant être approfondie dans le domaine des **Réseaux & Systèmes Intelligents (RSI)**.

## 2. Jeu de données

Le projet utilise le jeu de données public **UNSW-NB15**, développé par l'UNSW Canberra, contenant des caractéristiques de communications réseau normales et associées à différentes attaques.

Les expérimentations utilisent :

- `UNSW_NB15_training-set.csv` : 82 332 observations.
- `UNSW_NB15_testing-set.csv` : 175 341 observations.

Cette répartition correspond aux fichiers effectivement utilisés dans le projet et diffère du découpage de référence habituellement associé à UNSW-NB15.

**Les deux fichiers CSV sont inclus à la racine de ce dépôt GitHub**, afin de faciliter la reproduction des expérimentations et l'exécution des notebooks.

Le jeu de données UNSW-NB15 provient de l'UNSW Canberra. Sa source officielle est accessible depuis la [page officielle UNSW-NB15](https://research.unsw.edu.au/projects/unsw-nb15-dataset).

## 3. Organisation du projet

```text
network-anomaly-detection/
├── 01_cadrage_eda.ipynb
├── 02_modelisation.ipynb
├── 03_decision_roi.ipynb
├── UNSW_NB15_training-set.csv
├── UNSW_NB15_testing-set.csv
├── reports/
│   └── figures/
│       ├── comparaison_detection_attaques.png
│       ├── comparaison_fausses_alertes.png
│       └── sensibilite_roi.png
├── README.md
├── requirements.txt
└── .gitignore
```

Les deux fichiers CSV sont directement disponibles dans le dépôt, au même niveau que les notebooks. Aucun téléchargement supplémentaire des données n'est nécessaire après le clonage du dépôt.

### Rôle des notebooks

**01_cadrage_eda.ipynb** — Cadrage et analyse exploratoire : structure du jeu de données, répartition des observations, statistiques descriptives, doublons, incohérences d'étiquetage et qualité des données.

**02_modelisation.ipynb** — Préparation et modélisation : transformation des variables, entraînement des configurations de K-Means, calibration des seuils, comparaison des performances et analyse des limites.

**03_decision_roi.ipynb** — Décision et retour sur investissement : comparaison économique des modèles C et D, analyse de sensibilité, estimation du seuil de rentabilité et recommandation métier.

## 4. Méthodologie

Les expérimentations reposent sur trois configurations principales de K-Means :

| Modèle | Approche | Objectif |
|---|---|---|
| B | Clustering sur l'ensemble du trafic d'entraînement | Baseline entièrement non supervisée |
| C | Apprentissage sur les communications normales | Détection de nouveauté |
| D | Apprentissage sur les communications normales avec validation séparée | Calibration du seuil de détection |

Le score d'anomalie correspond à la distance entre une observation et son centroïde le plus proche.

Les étiquettes `label` et `attack_cat` ne sont pas utilisées comme variables explicatives. Cependant, `label` sert à sélectionner les communications normales pour entraîner les modèles C et D. Ces deux modèles ne sont donc pas entièrement non supervisés au sens strict.

### Justification du choix méthodologique

Le projet privilégie la détection de comportements atypiques plutôt que la classification supervisée des catégories d'attaques connues.

Cette approche permet d'étudier les communications qui s'écartent des profils de référence, sans supposer qu'une anomalie correspond systématiquement à une attaque.

Les trois configurations de K-Means permettent de comparer l'influence des données d'entraînement et de la calibration des seuils. Elles ne constituent pas trois algorithmes distincts.

### Validation et limite relative aux cinq plis

Les consignes du projet prévoient une validation croisée à cinq plis. Celle-ci n'a pas été réalisée sous sa forme classique dans les expérimentations présentées.

Le protocole retenu repose sur une séparation des données d'entraînement et de test, complétée, pour le modèle D, par un ensemble de validation distinct constitué de communications normales afin de calibrer le seuil d'anomalie.

Ce choix permet d'étudier une démarche de détection de nouveauté en préservant un jeu de test indépendant de l'entraînement et du réglage du seuil. Il ne constitue toutefois **pas un équivalent de la validation croisée à cinq plis**, et ne fournit pas d'estimation de la variabilité des performances sur cinq partitions.

Une évolution du projet pourrait intégrer une procédure de validation croisée adaptée à la détection de nouveauté, avec une moyenne et un écart-type des performances, ainsi qu'une comparaison de plusieurs algorithmes.

## 5. Principaux résultats

Les performances suivantes ont été obtenues sur le jeu de test utilisé dans le projet :

| Indicateur | Modèle B | Modèle C | Modèle D |
|---|---:|---:|---:|
| Attaques détectées | 889 | 39 409 | 44 393 |
| Fausses alertes | 767 | 560 | 2 862 |
| Précision | 53,68 % | 98,60 % | 93,94 % |
| Rappel | 0,74 % | 33,02 % | 37,20 % |
| ROC-AUC | 0,3161 | 0,7162 | 0,7269 |

Le modèle D obtient le meilleur rappel parmi les configurations évaluées, mais génère également davantage de fausses alertes que le modèle C.

Ces résultats ne démontrent pas une capacité à détecter toutes les attaques ni à identifier automatiquement des catégories de trafic parfaitement définies.

## 6. Analyse économique et décision métier

Une analyse de retour sur investissement incrémental compare les modèles C et D.

Le modèle D détecte **4 984 attaques supplémentaires**, mais génère **7 286 alertes supplémentaires**.

Dans un scénario illustratif reposant sur cinq minutes de traitement par alerte et un coût horaire de 40 € par analyste, le surcoût de traitement estimé est de **24 286,67 €**.

Le seuil de rentabilité correspond à une valeur hypothétique d'environ **4,87 € par attaque supplémentaire détectée**.

Ces montants sont des hypothèses appliquées au jeu de test. Ils ne représentent ni des coûts observés en entreprise ni un ROI annuel.

La recommandation est d'envisager le modèle D dans le cadre d'un **pilote avec supervision humaine**, et non comme un système autonome de cybersécurité.

## 7. Installation et exécution

### Prérequis

- Python 3
- Jupyter Notebook ou JupyterLab
- Les bibliothèques Python répertoriées dans `requirements.txt`
- Git pour cloner le dépôt

Les deux fichiers CSV nécessaires aux expérimentations sont déjà inclus dans le dépôt.

### Installation

Cloner le dépôt :

```bash
git clone https://github.com/<UTILISATEUR>/network-anomaly-detection.git
cd network-anomaly-detection
```

Remplacer `<UTILISATEUR>` par le nom du compte GitHub hébergeant le dépôt.

Créer un environnement virtuel :

```bash
python -m venv .venv
```

Sous Windows, l'activer :

```cmd
.venv\Scripts\activate
```

Installer les dépendances :

```bash
pip install -r requirements.txt
```

Lancer Jupyter Notebook depuis la racine du projet :

```bash
jupyter notebook
```

Les fichiers CSV étant déjà présents dans le dossier, aucune étape supplémentaire de téléchargement ou de déplacement des données n'est nécessaire.

Exécuter les notebooks dans cet ordre :

1. `01_cadrage_eda.ipynb`
2. `02_modelisation.ipynb`
3. `03_decision_roi.ipynb`

Pour vérifier la reproductibilité, utiliser **Restart & Run All** sur chaque notebook.

## 8. Limites et perspectives

Le projet présente plusieurs limites :

- Les expérimentations utilisent un jeu de données historique et non un trafic réseau collecté en temps réel.
- Les trois configurations principales reposent sur K-Means.
- La validation croisée à cinq plis n'a pas été réalisée.
- Le choix du nombre de clusters n'a pas fait l'objet d'une optimisation exhaustive.
- Les cinq clusters n'ont pas été caractérisés individuellement comme des profils métier.
- Les estimations économiques reposent sur des hypothèses illustratives.

Les perspectives comprennent la comparaison d'autres méthodes de détection d'anomalies, l'amélioration de l'interprétabilité, l'étude de nouveaux jeux de données et le développement d'une interface de supervision réseau.

## 9. Références

- [UNSW-NB15 — UNSW Canberra](https://research.unsw.edu.au/projects/unsw-nb15-dataset)
- [Documentation scikit-learn — K-Means](https://scikit-learn.org/stable/modules/generated/sklearn.cluster.KMeans.html)
- [Documentation scikit-learn — Novelty and Outlier Detection](https://scikit-learn.org/stable/modules/outlier_detection.html)

## 10. Statut du projet

**Prototype académique expérimental — Sciences des données 2.**

Ce dépôt constitue une base de travail destinée à être enrichie ultérieurement. Il ne représente pas un système de détection d'intrusion prêt pour un déploiement opérationnel.
