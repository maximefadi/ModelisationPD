# Modélisation Python de la PD Corporate IRB — Credit Risk Modelling

## 1. Présentation du projet

Ce projet porte sur la conception d'une chaîne complète de **modélisation de la Probability of Default (PD)** pour un portefeuille Corporate, dans le contexte de la gestion du risque de crédit bancaire.

L'objectif est de développer un modèle statistique permettant d'estimer la probabilité qu'une entreprise fasse défaut au cours des 12 prochains mois, de transformer cette probabilité en une notation interne (rating), puis de construire une PD finale calibrée à long terme (*Through-The-Cycle – TTC*).

Le projet couvre l'ensemble du cycle de vie d'un modèle de risque de crédit :

* Construction de la population Corporate.
* Définition du défaut (*Definition of Default – DoD*).
* Préparation des échantillons de développement et de validation.
* Analyse statistique et sélection des variables.
* Construction d'un modèle de régression logistique.
* Évaluation de la performance et de la stabilité.
* Calibration TTC.
* Construction d'une Master Scale.
* Calcul de la Margin of Conservatism (MoC).
* Backtesting et monitoring.
* Scoring d'un portefeuille Corporate.

Le projet est développé sous Python dans un notebook Jupyter reproductible.

---

## 2. Objectifs du projet

Les principaux objectifs sont les suivants :

1. Construire une population d'entreprises Corporate observées sur plusieurs années.
2. Définir une variable de défaut à horizon 12 mois.
3. Identifier les facteurs financiers, économiques et qualitatifs associés au risque de défaut.
4. Développer un modèle de régression logistique explicatif.
5. Mesurer la capacité discriminante du modèle.
6. Vérifier sa robustesse sur une période temporelle indépendante.
7. Calibrer les probabilités de défaut sur une approche TTC.
8. Construire une échelle de notation interne à 13 grades.
9. Appliquer les ajustements de prudence et les contrôles nécessaires.
10. Mettre en place des indicateurs de backtesting et de stabilité.
11. Développer un moteur de scoring utilisable sur un portefeuille Corporate.

---

## 3. Architecture du projet

Le notebook principal est :

`modelisationPD_corporate.ipynb`

Il contient un pipeline organisé autour des étapes suivantes :

| Étape | Description                              |
| ----- | ---------------------------------------- |
| 1     | Environnement et fonctions générales     |
| 2     | Definition of Default                    |
| 3     | Construction de la population Corporate  |
| 4     | Construction du panel annuel             |
| 5     | Variables macroéconomiques               |
| 6     | Variables financières et qualitatives    |
| 7     | Gestion des données manquantes           |
| 8     | Simulation du risque latent et du défaut |
| 9     | Génération des événements observables    |
| 10    | Suppression des observations post-défaut |
| 11    | Séparation Development / OOT             |
| 12    | Winsorisation                            |
| 13    | Binning monotone                         |
| 14    | Transformation Weight of Evidence        |
| 15    | Sélection par Information Value          |
| 16    | Transformation WoE                       |
| 17    | Filtre de corrélation                    |
| 18    | Filtre VIF                               |
| 19    | Régression logistique                    |
| 20    | Évaluation de la performance             |
| 21    | Calibration Development                  |
| 22    | Validation Out-of-Time                   |
| 23    | Analyse par année et taille              |
| 24    | Refit sur l'historique complet           |
| 25    | Calibration TTC et Long-Run Default Rate |
| 26    | Centrality Shift                         |
| 27    | Construction de la Master Scale          |
| 28    | Attribution des grades                   |
| 29    | Long-Run Average (LRA) par grade         |
| 30    | Lissage des PD                           |
| 31    | Margin of Conservatism                   |
| 32    | PD finale par grade et PD floor          |
| 33    | Overrides                                |
| 34    | Backtesting binomial                     |
| 35    | Test de Jeffreys                         |
| 36    | Backtesting annuel                       |
| 37    | Test de Vasicek simplifié                |
| 38    | Population Stability Index               |
| 39    | PSI des variables explicatives           |
| 40    | Scoring du portefeuille 2023             |
| 41    | Distribution des PD finales              |
| 42    | Calibration finale par grade             |
| 43    | Contrôle de monotonie                    |
| 44    | Contrôles qualité finaux                 |
| 45    | Synthèse du modèle                       |
| 46    | Inventaire des variables                 |
| 47    | Fonction de scoring de production        |
| 48    | Tests du moteur de scoring               |
| 49    | Exports des résultats                    |
| 50    | Rapport final                            |

---

## 4. Données et périmètre

### 4.1 Population étudiée

Le projet utilise une population Corporate simulée de 3 500 entreprises (*obligors*).

Les entreprises sont caractérisées par différentes informations :

**Caractéristiques structurelles :**

* Secteur d'activité.
* Taille de l'entreprise.
* Rating structurel.
* Levier financier structurel.
* Marge structurelle.
* Volatilité financière.

**Indicateurs financiers :**

* Leverage.
* Debt / EBITDA.
* EBITDA Margin.
* Return on Assets (ROA).
* Liquidity Ratio.
* Cash Ratio.
* Utilization.

**Indicateurs qualitatifs :**

* Management Quality.
* Governance Score.

**Variables macroéconomiques :**

* GDP Growth.
* Macro Stress.

La période historique étudiée couvre 2015 à 2023.

### 4.2 Construction du panel

Les données sont organisées sous la forme d'un panel annuel.

Chaque ligne représente une observation entreprise-année :

`(obligor_id, year)`

Le panel permet de suivre l'évolution des caractéristiques financières et du risque de défaut dans le temps.

### 4.3 Definition of Default

La définition simplifiée du défaut utilisée dans le notebook repose sur trois critères :

* Retard de paiement supérieur à 90 jours.
* Ratio arriérés / exposition supérieur à 1 %.
* Présence d'un indicateur Unlikeliness To Pay (UTP).

La variable de défaut est reconstruite à partir des événements observables, plutôt qu'en utilisant directement la probabilité latente simulée.

**Important :** cette définition est pédagogique et ne constitue pas une définition réglementaire universelle du défaut.

---

## 5. Préparation des données

### 5.1 Séparation Development / Out-of-Time

Le découpage temporel retenu est le suivant :

| Échantillon | Période   | Utilisation                          |
| ----------- | --------- | ------------------------------------ |
| Development | 2015–2020 | Construction et estimation du modèle |
| Out-of-Time | 2021–2023 | Validation temporelle indépendante   |

L'échantillon OOT n'intervient pas dans la construction des bins, le calcul des WoE, la sélection des variables ou l'estimation initiale des coefficients.

Cette séparation permet d'évaluer la capacité du modèle à généraliser à des périodes différentes de celles utilisées pour son développement.

### 5.2 Traitement des valeurs extrêmes

Les variables financières sont susceptibles de présenter des valeurs extrêmes.

Une winsorisation aux percentiles 1 % et 99 % est appliquée sur l'échantillon de développement.

Les mêmes paramètres sont ensuite réutilisés sur l'échantillon OOT afin d'éviter toute fuite d'information (*data leakage*).

### 5.3 Gestion des valeurs manquantes

Les valeurs manquantes sont traitées comme une modalité distincte lors de la transformation WoE.

Cette approche permet de conserver une information potentiellement pertinente sur la qualité et la disponibilité des données financières Corporate.

---

## 6. Construction du modèle statistique

### 6.1 Binning monotone

Les variables explicatives sont discrétisées en classes afin de rechercher une relation monotone entre chaque variable et le taux de défaut.

L'objectif est de garantir une meilleure interprétabilité et une relation cohérente entre les caractéristiques financières et le risque de crédit.

### 6.2 Weight of Evidence (WoE)

La transformation WoE permet de convertir les classes en valeurs numériques.

La convention retenue est :

WoE :

`WoE_i = ln(%Bad_i / %Good_i)`

Le calcul utilise un léger lissage pour éviter les divisions par zéro.

### 6.3 Information Value (IV)

L'Information Value mesure le pouvoir discriminant individuel des variables.

La formule utilisée est :

`IV = Σ (%Bad_i - %Good_i) × WoE_i`

Le notebook applique un seuil indicatif de :

`IV > 0.02`

Ce filtre constitue une première présélection et ne remplace pas l'analyse économique et statistique des variables.

### 6.4 Sélection des variables

La sélection repose sur plusieurs filtres successifs :

* Information Value.
* Corrélation entre variables.
* Variance Inflation Factor (VIF).

Les paramètres principaux sont :

| Paramètre            | Seuil |
| -------------------- | ----: |
| IV minimum           |  0.02 |
| Corrélation maximale |  0.75 |
| VIF maximal          |     5 |
| P-value maximale     |  0.05 |

Lorsque deux variables présentent une corrélation excessive, celle ayant l'IV le plus élevé est privilégiée.

Le VIF permet ensuite de limiter les problèmes de multicolinéarité.

### 6.5 Régression logistique

Le modèle final repose sur une régression logistique :

`logit(PD) = β0 + β1 × WoE1 + ... + βn × WoEn`

La probabilité de défaut est obtenue grâce à la fonction logistique :

`PD = 1 / (1 + exp(-Score))`

Cette approche permet d'obtenir une estimation individuelle de la probabilité de défaut pour chaque entreprise.

---

## 7. Évaluation de la performance

Plusieurs indicateurs sont utilisés pour évaluer la qualité du modèle.

| Indicateur            | Objectif                                        |
| --------------------- | ----------------------------------------------- |
| AUC                   | Mesurer la capacité de discrimination           |
| Gini                  | Mesurer le pouvoir discriminant                 |
| KS                    | Mesurer la séparation entre bons et mauvais     |
| Brier Score           | Évaluer la qualité probabiliste des prédictions |
| Observed Default Rate | Mesurer le taux de défaut observé               |
| Mean PD               | Mesurer la PD moyenne prédite                   |

Le coefficient de Gini est calculé comme suit :

`Gini = 2 × AUC - 1`

La calibration est également examinée par décile de risque, en comparant les taux de défaut observés aux PD moyennes prédites.

Les analyses sont réalisées sur Development et OOT.

Des analyses complémentaires sont effectuées par année et par taille d'entreprise.

---

## 8. Calibration Through-The-Cycle (TTC)

L'objectif de cette étape est de passer d'une estimation du risque sensible à la conjoncture (PIT) à une calibration de long terme (TTC).

### 8.1 Long-Run Default Rate

Le Long-Run Default Rate (LRDR) est calculé à partir des défauts et des observations historiques :

`LRDR = Total Defaults / Total Observations`

La définition de l'unité d'exposition et de la pondération doit être documentée pour toute application réelle.

### 8.2 Centrality Shift

Un ajustement est effectué sur l'échelle logit afin de rapprocher la PD moyenne du modèle du LRDR cible.

Le paramètre de déplacement δ est déterminé de manière à satisfaire :

`Mean[σ(logit(PD) + δ)] = LRDR`

Cette approche permet de conserver la structure de classement du modèle tout en ajustant son niveau global de risque.

---

## 9. Construction de la Master Scale

Le projet construit une échelle de notation Corporate composée de 13 grades.

Chaque entreprise est affectée à un grade en fonction de sa PD.

La règle d'affectation consiste à sélectionner le premier grade dont la borne supérieure de PD est supérieure ou égale à la PD de l'entreprise.

La Master Scale utilisée est illustrative et destinée à démontrer le processus de construction d'une échelle interne.

### 9.1 Long-Run Average Default Rate

Pour chaque grade, le taux de défaut annuel est calculé :

`DR(g,t) = Defaults(g,t) / Observations(g,t)`

Le taux de défaut moyen de long terme est ensuite calculé par grade.

### 9.2 Lissage des PD

Une régression log-linéaire est utilisée :

`ln(PD_g) = a + b × g`

Si la relation estimée ne respecte pas la monotonie attendue, une régression isotone croissante (*Isotonic Regression*) est utilisée comme solution de secours.

L'objectif est de garantir une progression cohérente du risque entre les grades.

---

## 10. Margin of Conservatism et PD Floor

### 10.1 Margin of Conservatism

Le notebook illustre une décomposition du MoC en trois composantes :

* MoC A : incertitude statistique.
* MoC B : incertitude liée à la définition du défaut.
* MoC C : incertitude liée au modèle.

La formule utilisée est :

`MoC = MoC_A + MoC_B + MoC_C`

### 10.2 PD après MoC

La PD ajustée est calculée comme suit :

`PD_MoC = PD_Smooth × (1 + MoC)`

### 10.3 PD Floor

Un plancher de PD est ensuite appliqué :

`PD_Final = max(PD_MoC, PD_Floor)`

Ces traitements sont simplifiés et illustratifs. Ils ne constituent pas une méthodologie réglementaire universelle.

---

## 11. Overrides

Le projet intègre une étape d'override après l'attribution du rating statistique.

Les overrides permettent de prendre en compte certains éléments qualitatifs ou événements spécifiques non suffisamment capturés par le modèle statistique.

Ils doivent respecter plusieurs principes :

* Justification économique.
* Documentation.
* Traçabilité.
* Encadrement.
* Suivi indépendant.

Le notebook simule un nombre limité d'overrides afin d'illustrer leur intégration dans le processus de notation.

---

## 12. Backtesting et validation réglementaire illustrative

Plusieurs méthodes de backtesting sont implémentées.

### 12.1 Test binomial

Le test binomial permet de comparer le taux de défaut observé à la PD attendue.

L'hypothèse alternative porte sur une sous-estimation potentielle du risque :

`H0 : p = PD_expected`

`H1 : p > PD_expected`

### 12.2 Test de Jeffreys

Une approche bayésienne est utilisée avec une distribution a posteriori :

`p | Data ~ Beta(d + 0.5, n - d + 0.5)`

La probabilité que le taux de défaut réel dépasse la PD de référence est ensuite examinée.

### 12.3 Backtesting annuel

Le ratio de suivi est calculé :

`Ratio_t = DR_t / PD_t`

Un ratio durablement supérieur à 1 peut signaler une sous-estimation du risque.

### 12.4 Test de Vasicek simplifié

Une approche inspirée du modèle de Vasicek est également utilisée afin d'illustrer la prise en compte de la corrélation entre les défauts.

Le paramètre de corrélation utilisé est illustratif.

---

## 13. Analyse de stabilité : Population Stability Index

Le Population Stability Index (PSI) permet de mesurer l'évolution de la distribution des populations entre deux périodes.

La formule est :

`PSI = Σ (Actual_i - Expected_i) × ln(Actual_i / Expected_i)`

Les seuils indicatifs utilisés sont :

| PSI               | Interprétation       |
| ----------------- | -------------------- |
| PSI < 0.10        | Faible changement    |
| 0.10 ≤ PSI < 0.25 | Surveillance         |
| PSI ≥ 0.25        | Changement important |

Le PSI est calculé sur les variables explicatives afin de détecter d'éventuelles dérives de population entre le développement et la période de référence.

---

## 14. Scoring du portefeuille Corporate 2023

Une fois le modèle construit et calibré, le portefeuille 2023 est scoré.

Pour chaque entreprise, le moteur produit :

* La PD issue du modèle.
* Le grade statistique.
* Le grade final.
* La PD finale après calibration et ajustements.

Le processus de scoring suit la chaîne :

`Données brutes → Bins → WoE → Score → PD → Grade → PD finale`

Une fonction dédiée permet de reproduire le scoring et de vérifier la cohérence entre les résultats du pipeline et ceux du moteur de scoring.

---

## 15. Livrables et exports

Le notebook génère les fichiers CSV suivants :

| Fichier                               | Description                                 |
| ------------------------------------- | ------------------------------------------- |
| `corporate_master_scale.csv`          | Master Scale Corporate calibrée             |
| `corporate_model_variables.csv`       | Inventaire des variables du modèle          |
| `corporate_backtesting_grade.csv`     | Résultats du backtesting par grade          |
| `corporate_backtesting_year.csv`      | Résultats du backtesting annuel             |
| `corporate_psi.csv`                   | Indicateurs de stabilité PSI                |
| `corporate_scored_portfolio_2023.csv` | Portefeuille Corporate scoré 2023           |
| `corporate_model_summary.csv`         | Synthèse des principaux résultats du modèle |

Ces fichiers sont générés dans le répertoire courant lors de l'exécution du notebook.

---

## 16. Environnement technique

Le projet utilise :

* Python >= 3.10
* Jupyter Notebook
* NumPy
* Pandas
* SciPy
* Statsmodels
* Scikit-learn

### Installation des dépendances

```bash
pip install numpy pandas scipy statsmodels scikit-learn jupyter
```

### Lancement du notebook

Cloner le dépôt :

```bash
git clone https://github.com/USERNAME/REPOSITORY.git
```

Accéder au répertoire :

```bash
cd REPOSITORY
```

Installer les dépendances :

```bash
pip install numpy pandas scipy statsmodels scikit-learn jupyter
```

Lancer Jupyter :

```bash
jupyter notebook
```

Ouvrir ensuite :

`modelisationPD_corporate.ipynb`

Exécuter les cellules dans l'ordre, de haut en bas.

---

## 17. Reproductibilité

Une graine aléatoire fixe est utilisée :

`SEED = 42`

Cette configuration permet de reproduire les simulations et les principaux résultats du notebook dans un environnement logiciel compatible.

La période étudiée est :

`2015–2023`

La population simulée contient :

`3 500 obligors Corporate`

---

## 18. Limites et précautions d'utilisation

**Ce projet est un cas pratique de modélisation du risque de crédit et ne constitue pas un modèle IRB directement utilisable en production bancaire.**

Plusieurs éléments sont simulés ou simplifiés :

* Les données financières et qualitatives.
* Les événements de défaut.
* Les variables macroéconomiques.
* La définition du défaut.
* La calibration TTC.
* Les paramètres de la Master Scale.
* Les composantes du MoC.
* Le PD Floor.
* Les overrides.
* Le paramètre de corrélation utilisé pour Vasicek.

Dans un contexte bancaire réel, il serait nécessaire de :

1. Remplacer les données simulées par des données historiques fiables.
2. Documenter et valider la Definition of Default.
3. Vérifier la représentativité et la qualité des données.
4. Justifier les méthodes de sélection et de calibration.
5. Documenter les hypothèses de long terme.
6. Définir une méthodologie de MoC conforme aux exigences applicables.
7. Mettre en place une gouvernance des overrides.
8. Effectuer une validation indépendante du modèle.
9. Documenter les limites et les éventuelles restrictions d'utilisation.
10. Vérifier la conformité aux exigences réglementaires applicables à l'établissement.

---

## 19. Conclusion

Ce projet illustre une démarche complète de développement d'un modèle de Probability of Default Corporate, depuis la construction des données jusqu'à la production d'une PD finale et d'un grade de notation interne.

Il permet de mettre en pratique plusieurs techniques fondamentales du Credit Risk Modelling :

* Data Preparation.
* Default Modelling.
* Weight of Evidence.
* Information Value.
* Logistic Regression.
* Model Validation.
* TTC Calibration.
* Master Scale Construction.
* Margin of Conservatism.
* Backtesting.
* Stability Monitoring.
* Production Scoring.

L'ensemble constitue une base de travail pédagogique pour comprendre les différentes étapes de construction, de calibration, de validation et de suivi d'un modèle de PD Corporate.

---

**Technologies :** Python · Pandas · NumPy · SciPy · Statsmodels · Scikit-learn · Jupyter Notebook

**Domaine :** Banking · Credit Risk Modelling · Corporate PD · Risk Analytics · IFRS 9 / IRB concepts
