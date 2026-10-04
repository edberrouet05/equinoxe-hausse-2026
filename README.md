# Collection Équinoxe : hausse de loyer 2026

Défi CodeML 2026 – JADCO « Clés en main ».

## Résultat

**Définition retenue :** croissance annualisée du **loyer effectif** (`sRentEffective`, après toutes les concessions), **à unité constante** (`hUnit`, équivalent à `sPropCode + sUnitCode`). On prend la médiane par segment (province × renouvellement/relocation), puis on pondère les segments.

| Mesure 2026 | Valeur |
|---|---|
| Hausse **effective**, centrale (concessions : tendance pleine, φ = 1) | **≈ 0,6 %** |
| Intervalle empirique (central ± erreur moyenne du backtest, 0,88 pt) | ≈ [−0,3 % ; 1,5 %] |
| Scénario haut : les concessions se stabilisent au niveau de 2025 | ≈ 3,8 % |
| Hausse **nette des seuls rabais de loyer** (sensibilité, hors stationnement et casier) | ≈ 2,4 % |
| Hausse **contractuelle** (plafond, pour fixer les loyers) | ≈ 3,8 % |

**Backtest 2023-2025** (erreur absolue moyenne, en points) :
- Modèle effectif : **0,88**.
- Modèles de référence : 1,40 (moyenne sur 3 ans) et 1,55 (naïf).
- Volet contractuel : 0,68.

Les valeurs exactes sont produites par le notebook et enregistrées dans `model_params.json`.

## Fichiers

| Fichier | Rôle |
|---|---|
| `equinoxe_hausse_2026.ipynb` | **Livrable principal.** Analyse complète, `estimate_2026()`, `backtest()`. |
| `donnees_externes.csv` | Données **publiques** (TAL, Ontario, StatCan, SCHL), avec source et URL par ligne. |
| `model_params.json` | Paramètres du modèle par composantes, **générés par le notebook** à l'exécution (non inclus dans ce dépôt). Aucun modèle ML n'est entraîné. |
| `starter.ipynb` | Notebook de départ fourni par les organisateurs (non inclus dans ce dépôt). |

Les fichiers `equinoxe_*.csv` (extrait CRM) restent intacts. **Ils ne doivent pas être inclus dans les livrables ni publiés.**

**Ce dépôt ne contient aucune donnée CRM ni aucune sortie de notebook** (graphiques, tableaux). Pour les reproduire, exécuter le notebook avec les 4 fichiers `equinoxe_*.csv` fournis aux participants.

## Exécuter le notebook

1. Placer `equinoxe_hausse_2026.ipynb` et `donnees_externes.csv` dans le même dossier que les 4 fichiers `equinoxe_*.csv`.
2. Installer les dépendances :

```bash
pip install pandas numpy matplotlib jupyter
```

3. Lancer Jupyter, ouvrir le notebook, puis *Run All*. L'exécution complète prend environ 40 s, dont la majeure partie pour le bootstrap.

```bash
jupyter notebook equinoxe_hausse_2026.ipynb
```

**Versions utilisées :** Python 3.14.2, pandas 3.0.2, NumPy 2.4.1, Matplotlib 3.10.9. Aucune autre librairie n'est nécessaire (pas de scikit-learn, statsmodels ni XGBoost).

## Méthode en bref

0. **Point de départ.** Les sections 1 à 6 de `starter.ipynb` sont reprises telles quelles. Les sorties qui affichaient des lignes CRM brutes sont remplacées par des agrégats.
1. **Colonnes vérifiées puis renommées.**
   - `sAvailable` est égal à `sLeaseFrom`.
   - `sConcession` est un indicateur 0/1.
   - `sTermSeq` suit l'unité, pas le locataire.
   - `PromoPay` est un rabais **unique** d'environ 0,8 mois de loyer.
2. **Reconstruction de `sRentEffective`.** Chaque concession est rattachée au bail le plus proche de la même unité. On retrouve `sRentEffective` = `sRent` − **toutes** les concessions étalées sur le bail, à 1 $ près pour 91 % des baux.
3. **Trois mesures de prix.**
   - Contractuelle : le plafond.
   - Nette des rabais de loyer : la sensibilité.
   - Effective : la mesure retenue, ce qui est encaissé.

   L'effet du bail précédent (rabais avant et après) est aussi mesuré.
4. **Effet de mix chiffré.** On utilise un indice de Laspeyres à composition fixe (immeuble × chambres). En 2023, environ 9 points du « +10,8 % » viennent du mix. En 2025, l'effet s'inverse : environ −2,8 points.
5. **Appariement.** Les paires sont construites sur `hUnit`, avec un écart compris entre 0,5 et 2,5 ans, puis annualisées. Une assertion vérifie l'équivalence avec `sPropCode + sUnitCode`, et `sLink` est aussi contrôlé.
6. **Renouvellements vs relocations.**
   - Au Québec, les renouvellements sont ancrés sur le TAL. Les immeubles section F (calculée à partir de la constante d'âge) se comportent comme les autres.
   - **The Met** (première occupation après le 15 novembre 2018) est **exempté** de la ligne directrice ontarienne. Ses renouvellements suivent l'IPC loyer de l'Ontario.
7. **Données externes.** Chaque série a une colonne `usage` :
   - `ancre_modele` : séries connues avant l'année prévue, utilisées dans le modèle et le backtest.
   - `contexte` : SCHL, pour l'interprétation seulement.
8. **Prévision.**
   - Hausse contractuelle : ancrage réglementaire + écart de relocation.
   - Passage au loyer effectif : rabais du bail précédent et rabais prévu pour le nouveau bail.
   - φ est choisi par backtest et plafonné *a priori* à 1. Une courbe de sensibilité au-delà de 1 est incluse.
   - Le réalisé utilise **la même agrégation** que la prévision.

## Sources publiques

- Tribunal administratif du logement : calcul de l'augmentation des loyers ; taux 2026 de 3,1 % (nouvelle méthode, moyenne IPC 3 ans). <https://www.tal.gouv.qc.ca>
  - Radio-Canada : <https://ici.radio-canada.ca/nouvelle/2221781/loyers-tal-logement-augmentation-locataires>
  - Le Devoir : <https://www.ledevoir.com/economie/949236/loyers-devraient-augmenter-moins-3-1-2026-selon-tal>
- Ontario : ligne directrice 2026 de 2,1 % et exemption post-2018. <https://www.ontario.ca/page/rent-increase-guideline>
- Statistique Canada : tableau 18-10-0005-01, IPC moyenne annuelle (loyer, ensemble), Québec et Ontario. <https://www150.statcan.gc.ca/t1/tbl1/fr/tv.action?pid=1810000501>
- SCHL : Rapport sur le marché locatif, automne 2025. <https://www.cmhc-schl.gc.ca/professionals/housing-markets-data-and-research/market-reports/rental-market-reports-major-centres>
- Gardner, E. S. & McKenzie, E. (1985), *Forecasting trends in time series*, Management Science : tendance amortie.

## Outils d'IA utilisés

- **Claude (Anthropic), via Claude Code :** exploration des données, recherche des sources publiques, rédaction du code et du texte du notebook. Toutes les sorties ont été exécutées et vérifiées.
