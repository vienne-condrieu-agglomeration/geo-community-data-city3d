# geo-community-data-city3d

Fichiers de la **maquette 3D** du module [geo-lidar-hd](https://github.com/vienne-condrieu-agglomeration/geo-module-lidar-hd)
publiés sur GitHub Pages : **bâtiments LoD2** et **canopée** de chaque dalle LiDAR HD de l'IGN, lus par le panneau Maquette dans
le navigateur, sans serveur.

> Projet communautaire indépendant, **non affilié à Business Geografic**. GEO est un produit de Business Geografic.

## Adresse des données

`https://vienne-condrieu-agglomeration.github.io/geo-community-data-city3d/` : c'est la valeur à saisir dans le réglage
**URL des données 3D de la ville** (`city3dUrl`) du module, sans `/` final.

| Fichier | Contenu |
|---|---|
| `index.json` | version du format (`format`), source, licence, et pour chaque dalle : version de l'intégrateur, outils, édition du nuage de l'IGN, taille et sha256 de ses fichiers |
| `buildings/<dalle>.bld` | bâtiments LoD2 de la dalle, reconstruits par [roofer](https://github.com/3DBAG/roofer) (format `BLD1`) |
| `vegetation/<dalle>.veg` | canopée de la dalle, issue des points de végétation du LiDAR HD (format `VEG1`) |

Une dalle se nomme `XXXX-YYYY` : coin nord-ouest en km, Lambert-93 (0846-6494 = centre de Vienne). Une dalle absente de
l'index n'a pas de fichier : le module la laisse comme sans le réglage. Le format des fichiers est décrit dans le
`GUIDE-DEV.md` du module ; le code qui les produit est
[geo-community-integrator-city3d](https://github.com/vienne-condrieu-agglomeration/geo-community-integrator-city3d).

## Fonctionnement

Le workflow [`donnees.yml`](.github/workflows/donnees.yml), lancé **à la main** (*Actions > Données city3d > Run workflow*) :

1. établit la liste des dalles : l'entrée `dalles` (`XXXX-YYYY` séparées par des espaces, ou `around XXXX-YYYY` pour une dalle et
   ses 8 voisines), à défaut [`tiles.txt`](tiles.txt) ;
2. reconstruit chaque dalle dans un job à part (3 en parallèle) avec la version de l'intégrateur fixée dans le workflow
   (`INTEGRATOR_VERSION`), sauf si elle est déjà à jour dans l'index publié (même édition du nuage, mêmes outils, même version
   de l'intégrateur ; `force` pour passer outre) ;
3. rassemble les dalles générées et celles déjà publiées (un déploiement Pages remplace tout le site) ;
4. déploie le résultat sur GitHub Pages, **sans commit** : l'historique du dépôt ne grossit pas.

**Aucun jeton n'est nécessaire** : le `GITHUB_TOKEN` automatique suffit. Réglage fait une fois : *Settings > Pages > Source :
GitHub Actions*.

## Périmètre : les communes de l'agglomération

[`tiles.txt`](tiles.txt) liste les **497 dalles** de 1 km qui touchent les 30 communes de Vienne Condrieu Agglomération (EPCI
200077014, contours BD TOPO de l'IGN) ; elle se recalcule avec `tiles_from_communes.py` du dépôt de l'intégrateur. Une matrice GitHub
Actions accepte 256 dalles au plus, donc la liste se publie en plusieurs lancements : [`lots/lot1.txt`](lots/lot1.txt) (les dalles
les plus bâties : Vienne et les bourgs) et [`lots/lot2.txt`](lots/lot2.txt) (le reste, rural), chacun à coller dans l'entrée `dalles`
du workflow. Les 23 dalles publiées en premier (essai autour de Vienne et dalles rurales) n'y figurent pas : un lancement conserve de toute
façon les dalles déjà publiées.

Points d'attention :

- Une dalle demande environ 290 Mo de nuage et l'image PDAL (environ 4 Go) : si un job manque de disque ou de temps, ou si le tirage
  d'une image Docker est limité, relancer ou réduire le nombre de dalles.
- Pour utiliser une nouvelle version de l'intégrateur, modifier `INTEGRATOR_VERSION` après avoir lu son `CHANGELOG.md` ; elle
  régénère toutes les dalles demandées.
- Un site GitHub Pages ne dépasse pas 1 Go : environ 2 Mo par dalle urbaine et 1 Mo par dalle rurale (mesuré le 2026-11-18 sur 23 dalles : 1,4 Mo en moyenne, soit environ 600 Mo estimés pour les 497 dalles), à surveiller en étendant le périmètre.

## Licences

- **Données** : dérivées du LiDAR HD et de la BD TOPO® de l'IGN, publiées sous
  [Licence Ouverte 2.0](https://www.etalab.gouv.fr/licence-ouverte-open-licence/). Mention à reprendre :
  « Source : IGN, LiDAR HD et BD TOPO®, traitement Vienne Condrieu Agglomération ».
- **Workflow et documentation de ce dépôt** : [GPL-3.0-or-later](LICENSE) — © 2026 Vienne Condrieu Agglomération.

Les contributions au code de conversion se font sur
[geo-community-integrator-city3d](https://github.com/vienne-condrieu-agglomeration/geo-community-integrator-city3d) (signature DCO).
