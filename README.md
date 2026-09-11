# FloodGuard

Système d'alerte précoce aux inondations par IA — Bénin

Projet réalisé dans le cadre du **Deep Learning IndabaX Bénin 2026**, Défi 1 : *Résilience climatique*.

📓 Notebook : [Google Colab](https://colab.research.google.com/drive/12sUKjGUHnifNtBnWk_XF5eH9k1N9XBB_?usp=sharing)

---

## Le problème

Le Bénin connaît trois profils d'inondation selon la région : crues fluviales saisonnières dans le delta de l'Ouémé (sud), crues du fleuve Niger en septembre-octobre dans le nord (Malanville, Karimama), et inondations côtières/lagunaires aggravées par l'urbanisation sur le littoral (Cotonou, Grand-Popo). Dans les trois cas, il n'existe aujourd'hui aucune alerte locale ciblée par zone.

**Bénéficiaires** : les agriculteurs des zones inondables, qui perdent leurs récoltes faute de préavis, et les autorités communales, chargées d'organiser l'évacuation ou la mise à l'abri.

## La solution

FloodGuard calcule, pour chaque zone du pays, un score de risque d'inondation à court terme (3 à 7 jours), à partir de la pluie récente, du relief et de l'occupation du sol. L'alerte est pensée pour être envoyée par SMS en langue locale, sans nécessiter de smartphone ni de connexion internet côté utilisateur. Le modèle est calibré séparément pour le sud, le nord et le littoral plutôt que d'utiliser un seuil unique national.

## Données utilisées

| Source | Ce qu'elle apporte |
|---|---|
| [CHIRPS](https://data.chc.ucsb.edu/products/CHIRPS/) | Pluviométrie satellite quotidienne, historique et suivi |
| [HydroSHEDS](https://www.hydrosheds.org/) | Relief, pente, réseau de drainage |
| [Global Flood Database](https://developers.google.com/earth-engine/datasets/catalog/GLOBAL_FLOOD_DB_MODIS_EVENTS_V1) (via Google Earth Engine) | Historique des inondations passées — vérité terrain |
| [ESA WorldCover](https://esa-worldcover.org/) | Occupation du sol à 10 m de résolution |
| [Copernicus EMS](https://emergency.copernicus.eu/) | Rapports d'urgence officiels au Bénin |

Toutes ces sources sont ouvertes et couvrent nativement l'ensemble du territoire national.

## Approche technique

1. **Collecte** — téléchargement automatisé (CHIRPS, HydroSHEDS) et manuel (Global Flood DB, WorldCover, Copernicus EMS) des rasters.
2. **Feature engineering** — extraction de la pente, du TWI (indice d'humidité topographique), de la moyenne, du pic et de l'anomalie de précipitation.
3. **Modélisation** — ensemble de trois modèles de gradient boosting (LightGBM + XGBoost + CatBoost), combinés par blending.
4. **Validation** — cross-validation par blocs géographiques (`GroupKFold` sur grille spatiale), pour éviter la fuite de données entre pixels voisins.
5. **Explicabilité** — SHAP (`TreeExplainer`) pour identifier les variables les plus déterminantes, par zone.

Une approche par ensemble de modèles de gradient boosting a été préférée à un modèle de deep learning : les données sont tabulaires (pas d'images brutes à traiter directement) et le volume de données labellisées disponible dans le temps du hackathon ne justifiait pas la complexité supplémentaire.

### Dépendances

- Python : `lightgbm`, `xgboost`, `catboost`, `shap`, `rasterio`, `scikit-learn`, `pandas`, `numpy`, `geopandas`
- Aucun modèle pré-entraîné propriétaire ni API payante — uniquement des bibliothèques open source et des données ouvertes.

## Résultats

**Ce qui fonctionne** : le pipeline complet — téléchargement des données, extraction des features à partir des rasters, entraînement de l'ensemble avec validation spatiale, génération des graphiques SHAP et de la carte de risque — tourne de bout en bout.

**Ce qui reste à faire** : le modèle est pour l'instant entraîné et validé sur un jeu de données synthétique généré pour tester l'architecture, pas encore sur les vraies features extraites des rasters réels. Aucune métrique de fiabilité (AUC-ROC) n'est donc encore significative sur données réelles, et aucun retour terrain (agriculteurs, autorités locales) n'a encore été recueilli.

## Structure du dépôt

```
.
├── data_defi1/              # Données téléchargées et traitées (générées, non versionnées)
│   ├── chirps_v3/
│   ├── hydrosheds/
│   ├── copernicus_ems/
│   ├── worldcover/
│   └── processed/
├── floodguard.py            # Pipeline : téléchargement → features → entraînement → visualisation
└── README.md
```

## Limites connues

- Pipeline validé sur données synthétiques, pas encore sur les vraies extractions rasters.
- Peu de vérité terrain disponible pour calibrer chaque zone séparément.
- Aucun retour d'agriculteurs ou d'autorités locales recueilli à ce stade.

## Prochaines étapes

- Brancher les vraies features (rasters réels au lieu des données factices).
- Valider les indicateurs et le canal d'alerte avec 2-3 contacts locaux par zone.
- Tester la diffusion SMS en langue locale sur un cas pilote.
