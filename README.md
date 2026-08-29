# BruxNav

![Java](https://img.shields.io/badge/Java-21-blue?style=flat-square)
![Maven](https://img.shields.io/badge/build-Maven-C71A36?style=flat-square)

BruxNav est une application Java en ligne de commande qui charge des données GTFS organisées par agence, construit un graphe multimodal d’arrêts et peut rechercher un itinéraire entre deux arrêts à une heure donnée. La recherche combine les segments de transport des horaires GTFS et les correspondances à pied entre arrêts proches.

> Projet académique ULB — INFO-F203.

## Aperçu

![Sortie de BruxNav](output.png)

## Sommaire

- [Fonctionnalités](#fonctionnalités)
- [Prérequis](#prérequis)
- [Installation et compilation](#installation-et-compilation)
- [Préparer les données GTFS](#préparer-les-données-gtfs)
- [Utilisation](#utilisation)
- [Architecture](#architecture)
- [Document du projet](#document-du-projet)
- [Auteur](#auteur)

## Fonctionnalités

- Chargement des arrêts, lignes, trajets et horaires depuis les fichiers CSV GTFS de chaque agence.
- Fusion des données d’agence dans un modèle global.
- Génération d’arêtes de transport entre les arrêts consécutifs de chaque trajet.
- Génération de liaisons piétonnes entre les arrêts situés dans un rayon de 1 km.
- Recherche A* dépendante de l’heure de départ, avec pénalités de changement de mode ou de ligne.
- Affichage dans le terminal des étapes à pied et en transport, avec leurs horaires.

## Prérequis

- JDK 21.
- Maven.
- Des données GTFS CSV, rangées dans un répertoire racine contenant un sous-répertoire par agence.

## Installation et compilation

```bash
git clone https://github.com/9Chrk/BruxNav.git
cd BruxNav
mvn package
```

La compilation produit le JAR exécutable `target/stibpath-1.0-SNAPSHOT.jar`.

## Préparer les données GTFS

Le programme reçoit comme premier argument le répertoire racine des données. Chaque sous-répertoire de cette racine est traité comme une agence et doit fournir les quatre fichiers suivants.

Les colonnes utilisées sont :

| Fichier | Colonnes requises |
| --- | --- |
| `stops.csv` | `stop_id`, `stop_name`, `stop_lat`, `stop_lon` |
| `routes.csv` | `route_id`, `route_short_name`, `route_long_name`, `route_type` |
| `trips.csv` | `trip_id`, `route_id` |
| `stop_times.csv` | `trip_id`, `stop_id`, `departure_time`, `stop_sequence` |

## Utilisation

Le JAR accepte un répertoire racine GTFS en premier argument. Avec ce seul argument, il charge les agences et construit le graphe multimodal.

Pour lancer une recherche, ajoutez, dans cet ordre, le nom de l’arrêt de départ, le nom de l’arrêt d’arrivée et l’heure de départ au format `HH:mm:ss`. Les noms d’arrêts sont comparés sans tenir compte de la casse et doivent être présents dans les données chargées.

Les données GTFS ne sont pas incluses dans ce dépôt ; l’exécution nécessite donc de fournir un répertoire conforme à la structure décrite ci-dessus.

## Architecture

```text
src/main/java/be/ulb/stib/
├── Main.java                 # Point d’entrée et orchestration
├── algo/AStarTD.java         # Recherche A* dépendante du temps
├── core/                     # Entités du réseau et types d’arêtes
├── data/                     # Modèles d’agence et modèle global
├── graph/                    # Graphe multimodal
├── output/                   # Mise en forme d’un itinéraire
├── parsing/                  # Chargeurs des fichiers CSV GTFS
├── spatial/                  # KD-tree et génération des liaisons piétonnes
└── tools/                    # Lecteur CSV et utilitaires de chargement
```

Le flux principal est le suivant :

```text
Données GTFS → modèles d’agence → modèle global
             → arêtes de transport + arêtes de marche → graphe multimodal
             → recherche A* → itinéraire affiché dans le terminal
```

## Document du projet

- [Projet.pdf](Projet.pdf)

## Auteur

- Cherkaoui Jawad (576517)
