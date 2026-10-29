# Boussole forestière — Spécifications

Outil d'aide à la compréhension pour les petits propriétaires forestiers, open source (EUPL-1.2), fondé uniquement sur des données publiques.

> L'outil est une aide à la compréhension, pas un substitut à l'avis d'un professionnel. Il doit le dire clairement.

## 1. Contexte

En France, la forêt est privée à 75 % (environ 12,3 millions d'hectares), répartie entre environ 3,3 millions de propriétaires. Près de 3 millions possèdent moins de 4 hectares et ne sont soumis à aucun plan de gestion. La plupart ne savent ni ce qu'il y a sur leur parcelle, ni comment elle va évoluer avec le changement climatique, ni quoi faire.

Des outils experts existent (BioClimSol, ClimEssences), mais ils s'adressent aux gestionnaires et aux agents forestiers. La Forêt Bouge (CNPF) propose une bourse foncière mais aucune aide à la décision. Il manque un outil simple, pensé pour le propriétaire, qui transforme les données publiques en informations compréhensibles et l'oriente vers les bons professionnels.

La valeur du projet est dans l'intégration et la pédagogie : on s'appuie sur les méthodes existantes (ClimEssences, BioClimSol, lidaRtRee de l'INRAE) plutôt que de réinventer des modèles scientifiques.

## 2. Objectifs

Permettre à un petit propriétaire de :

1. retrouver sa parcelle ;
2. comprendre ce qu'elle contient (essences, hauteur, densité, relief) ;
3. évaluer sa vulnérabilité face au climat futur ;
4. découvrir quelques options de gestion compréhensibles et les contacts utiles (CNPF, experts forestiers, coopératives, regroupement avec les voisins).

## 3. Périmètre

### V0 — prototype

Limité à une seule commune : **Arancou (Pyrénées-Atlantiques, INSEE 64031)**.

Objectifs vérifiables :

1. À partir d'une référence cadastrale d'Arancou, ou d'un clic sur la carte, l'outil affiche le contour exact de la parcelle (au sens du cadastre).
2. Il calcule et affiche pour cette parcelle : la surface, la hauteur moyenne et la hauteur dominante de la végétation (MNH LiDAR HD), et le type de peuplement (BD Forêt).
3. Les calculs sont reproductibles et couverts par des tests sur des parcelles de référence dont les valeurs attendues sont connues.
4. Chaque donnée affichée indique sa source et sa date.
5. Un texte simple explique au propriétaire ce que signifient ces chiffres.

### V1 — après validation de la V0

- Vulnérabilité climatique (projections DRIAS / TRACC).
- Options de gestion et contacts utiles.
- Densité du peuplement.
- Relief : pente et exposition (MNT LiDAR HD).
- Recherche par adresse, lieu-dit ou commune, pour centrer la carte.

## 4. Commune pilote : Arancou

Vérifications faites le 29/10/2026 sur les données de l'IGN :

| Point | Constat |
|---|---|
| Surface de la commune | ≈ 520 ha, altitude 12 à 142 m |
| Forêt (BD Forêt V2) | ≈ 117 ha de forêt et 7 ha de lande ; surtout forêts fermées de feuillus en mélange (87 ha) et de chênes décidus purs (17 ha) |
| Forêt publique (ONF) | aucune : toute la forêt est privée |
| Parcelles cadastrales | 1 097 |
| Parcelles boisées à plus de 50 % | 217 ; surface médiane 0,24 ha ; 191 de moins de 1 ha, 25 entre 1 et 4 ha, 1 de plus de 4 ha |
| MNH LiDAR HD | 14 dalles, 100 % de la commune couverte, aucun pixel nodata (pas de ZICAD) |
| Date du LiDAR | acquisition mai à août 2022, édition 26/09/2025 |
| Date de la BD Forêt | édition du département 64 du 01/04/2014 |
| Contenance / surface calculée | écart médian de 2,8 % |

## 5. Sources de données (V0)

Toutes sous **Licence Ouverte / Etalab 2.0** (réutilisation libre, citation de la source obligatoire).

| Source | Producteur | Usage | Format et accès |
|---|---|---|---|
| Cadastre | IGN (Parcellaire Express) ou Etalab | contour, IDU, section, numéro, contenance | WFS Géoplateforme ou GeoJSON par commune ; choix à trancher dans l'issue #2 |
| MNH LiDAR HD | IGN | hauteur de la végétation | GeoTIFF float32, dalles de 1 km × 1 km au pas de 50 cm, nodata −9999 ; index des dalles via la couche WFS des métadonnées LiDAR HD |
| BD Forêt V2 | IGN | type de peuplement | WFS Géoplateforme (`LANDCOVER.FORESTINVENTORY.V2:formation_vegetale`) ou Shapefile départemental |
| BD TOPO (bâtiments) | IGN | exclusion du bâti, repères sur la carte | WFS Géoplateforme |
| Photo aérienne | IGN | fond de carte | WMTS Géoplateforme, sans clé |

### Pièges connus

- Les GeoTIFF du MNH servis par le WMS-R ont une projection mal décrite : forcer EPSG:2154 à la lecture.
- Le MNH contient des hauteurs négatives (jusqu'à −3,4 m à Arancou).
- La BD Forêt du 64 (2014) est antérieure d'au moins 8 ans au LiDAR (2022) : une parcelle coupée entre-temps peut être classée « forêt fermée » avec un MNH bas.
- La BD Forêt ne cartographie rien sous 0,5 ha, alors que la parcelle boisée médiane fait 0,24 ha.
- Le cadastre est un assemblage de feuilles sans correction : le contour n'a pas la précision d'un bornage.
- Les ZICAD sont codées en nodata dans le MNH (aucune à Arancou, mais le calcul doit gérer le cas).

## 6. Règles fonctionnelles

### 6.1 Retrouver une parcelle

- Saisie de la **section + numéro** (commune fixée à Arancou), ou de l'**IDU complet** à 14 caractères.
- Saisie tolérante : casse indifférente, espaces ignorés, zéros de tête facultatifs (`B 123` = `b0123` = `640310000B0123`).
- Les préfixes de section sont ignorés en V0.
- **Sélection par clic** sur une parcelle de la carte.
- Erreurs : format invalide → message indiquant le format attendu et où trouver la référence (avis de taxe foncière, acte de propriété) ; référence absente → « parcelle introuvable dans la commune pilote ».

### 6.2 Surface

- Par défaut : la **contenance cadastrale**, libellée « surface cadastrale (celle de vos documents) ».
- Dans « voir détails » : la surface calculée à partir du contour, en EPSG:2154, avec l'écart et une phrase qui l'explique.

### 6.3 Type de peuplement

- Par défaut : le **type majoritaire** seul.
- Dans « voir détails » : la répartition précise, en pourcentages et en hectares, avec « hors forêt » compté comme une catégorie.

### 6.4 Hauteurs

Règles communes :

- les hauteurs négatives sont ramenées à 0 ;
- les emprises des bâtiments BD TOPO, avec une marge de 1 à 2 m, sont **exclues du calcul** ;
- le seuil de végétation est de **2 m** (constante nommée).

**Hauteur moyenne** : moyenne des pixels au-dessus de 2 m. Elle est affichée avec la part de la parcelle concernée (« hauteur moyenne des arbres : 15 m, sur 60 % de la parcelle »).

**Hauteur dominante** : grille de 10 m × 10 m alignée sur des multiples de 10 m en Lambert-93 (soit 100 cellules par hectare) ; hauteur maximale par cellule ; moyenne de ces maxima sur les cellules contenant de la végétation au-dessus de 2 m. Seules les cellules dont au moins 50 % de la surface est dans la parcelle sont retenues.

Sont aussi renvoyés : le taux de couverture effective du MNH (pixels valides / pixels de la parcelle) et le nombre de cellules retenues pour la hauteur dominante.

### 6.5 Carte

- Fond photo aérienne IGN.
- Contours cadastraux, parcelle sélectionnée en surbrillance.
- **Bâtiments affichés** comme repères pour que le propriétaire se situe.
- Utilisable sur mobile (largeur 375 px).

### 6.6 Traçabilité et textes

- Chaque valeur affichée porte sa source et sa date (date de la donnée, pas du calcul).
- Une phrase simple explique chaque chiffre.
- Avertissement visible sans action de l'utilisateur : aide à la compréhension, pas un avis professionnel.
- Mentions : précision du cadastre (pas un bornage), ancienneté de la BD Forêt par rapport au LiDAR, chiffres non vérifiés sur le terrain.
- Lien vers le CNPF régional.

## 7. Tests et validation

Développement en **TDD** : tests écrits en premier et validés un par un, analyse statique, CI dès le départ.

Deux niveaux de référence :

| Niveau | Ce qu'il prouve | Tolérance |
|---|---|---|
| Rasters et géométries synthétiques | le code calcule ce qui est défini | 0,01 |
| 5 parcelles réelles d'Arancou recalculées indépendamment dans QGIS | le calcul est reproductible sur de vraies données | hauteurs ±0,1 m ; surface calculée ±0,1 % ; répartition ±1 point |

Les 5 parcelles réelles couvrent : une parcelle entièrement boisée d'un seul type ; une parcelle en lisière ; une parcelle avec un bâtiment ; une parcelle de moins de 0,1 ha ; une parcelle à cheval sur deux dalles MNH. Elles sont sélectionnées automatiquement puis validées avant que les valeurs attendues soient figées.

La justesse terrain (mesures au dendromètre, inventaire d'expert) est hors V0.

## 8. Architecture

- **Pipeline Python 3.12** (geopandas, shapely, rasterio, pyproj) : téléchargement, calculs en Lambert-93, export.
- **Site 100 % statique** : le pipeline précalcule tout et génère un GeoJSON des parcelles, un GeoJSON des bâtiments (tous deux en WGS84 pour la carte) et un JSON de résultats par parcelle. La page (MapLibre ou Leaflet) lit ces fichiers ; la recherche se fait dans le navigateur.
- Pas de base de données ni d'API en V0 (YAGNI) ; elles deviendront utiles au passage à plusieurs communes.
- Qualité : pytest, ruff, mypy strict, GitHub Actions.
- Dépôt : `nmoral/boussole-forestiere` sur GitHub. Hébergement du site et des données des communes : à décider.

## 9. Règles de travail

- Aucune supposition : si une information manque (format d'une donnée, choix technique, comportement attendu), poser la question avant d'avancer.
- Chaque issue a des critères d'acceptation vérifiables.
- Pour chaque source de données : vérifier licence, format, couverture et méthode d'accès avant de l'intégrer.
- Simplicité (YAGNI) : rien qui ne serve pas directement les objectifs de la version en cours.

## 10. Décisions restant ouvertes

| Sujet | Où |
|---|---|
| Outil de gestion des dépendances (uv ou pip-tools) | issue #1 |
| Source du cadastre (Parcellaire Express IGN ou Etalab) | issue #2 |
| Marge autour des bâtiments (1 ou 2 m) | issue #7 |
| Hébergement du site | après la V0 |
