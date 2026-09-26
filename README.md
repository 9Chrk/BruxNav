# BruxNav

![Java](https://shields.io)
![Maven](https://img.shields.io/badge/build-Maven-C71A36?style=flat-square)
![Données](https://shields.io)


BruxNav est un **calculateur d’itinéraires en transports publics et à pied**, développé en **Java 21**. À partir de données GTFS, il recherche un trajet selon les arrêts de départ et d’arrivée et l’heure choisie, puis affiche les étapes et les horaires dans le terminal.

Le programme rassemble plusieurs réseaux dans un graphe multimodal et utilise une recherche A* dépendante du temps. Le projet se compile avec Maven ; les données GTFS doivent être fournies séparément.

> Projet académique ULB — INFO-F203.

<a id="captures-decran"></a>

## 📸 Captures d’écran

![Image fictive](https://github.com/user-attachments/assets/6cb4cd00-c550-46c2-9ef6-e3b0935124bd)
![Sortie de BruxNav](https://github.com/user-attachments/assets/7eac17bf-69c1-45a3-ab2b-55c073e9e434)

Cette exécution illustre le chargement de plusieurs agences, la construction du graphe et l’affichage d’un itinéraire incluant marche et bus.

---

## 📖 Sommaire

- [Fonctionnalités](#fonctionnalites)
- [Prérequis](#prerequis)
- [Configuration des données GTFS](#configuration-des-donnees-gtfs)
- [Installation](#installation)
- [Lancement](#lancement)
- [Utilisation](#utilisation)
- [Données GTFS](#donnees-gtfs)
- [Architecture](#architecture)
- [Flux général](#flux-general)
- [Structure du projet](#structure-du-projet)
- [Tests](#tests)
- [Problèmes fréquents](#problemes-frequents)
- [Documentation](#documentation)

<a id="fonctionnalites"></a>

## ✨ Fonctionnalités

- **Chargement multi-agence** : parcourt chaque sous-répertoire de données et lit ses arrêts, lignes, trajets et horaires GTFS.
- **Fusion dans un réseau unique** : rassemble les modèles des agences dans `GlobalModel` afin de pouvoir établir des connexions entre leurs arrêts.
- **Segments de transport horodatés** : crée une arête entre chaque paire d’arrêts consécutifs d’un trajet, avec ses heures de départ et d’arrivée.
- **Correspondances à pied** : relie les arrêts situés à moins de 1 km et estime leur durée avec une vitesse de marche de 1,25 m/s.
- **Recherche dépendante du temps** : ne considère un segment de transport que si son départ n’a pas encore été manqué et utilise une heuristique géographique.
- **Itinéraire lisible** : affiche les segments de marche et de transport avec les arrêts, l’agence, le type de ligne, la ligne et les horaires.
- **Mesures d’exécution** : affiche les durées de chargement, de fusion, de construction du graphe et, lorsqu’une recherche est demandée, de calcul d’itinéraire.

<a id="prerequis"></a>

## 🧰 Prérequis

- Un JDK 21 : la version de compilation est définie par `maven.compiler.release` dans `pom.xml`.
- Maven pour résoudre les dépendances et construire le JAR exécutable.
- Un répertoire de données GTFS CSV conforme à la structure décrite dans la section suivante.

Les dépendances de production sont `opencsv` 5.9 pour la lecture CSV et `fastutil` 8.5.12 pour les collections. JUnit Jupiter 5.11.0 est déclaré pour les tests.

<a id="configuration-des-donnees-gtfs"></a>

## ⚙️ Configuration des données GTFS

Le premier argument du programme est le répertoire racine GTFS. Chaque sous-répertoire immédiat de cette racine est traité comme une agence ; il doit contenir les quatre fichiers suivants :

| Fichier | Colonnes lues par l’application |
| --- | --- |
| `stops.csv` | `stop_id`, `stop_name`, `stop_lat`, `stop_lon` |
| `routes.csv` | `route_id`, `route_short_name`, `route_long_name`, `route_type` |
| `trips.csv` | `trip_id`, `route_id` |
| `stop_times.csv` | `trip_id`, `stop_id`, `departure_time`, `stop_sequence` |

Les fichiers sont lus en UTF-8. Les horaires de `stop_times.csv` sont interprétés au format `HH:mm:ss`, puis convertis en secondes depuis minuit. Les lignes de ce fichier sont finalement ordonnées selon `stop_sequence` pour chaque trajet.

<a id="installation"></a>

## 📦 Installation

```bash
git clone https://github.com/9Chrk/BruxNav.git
cd BruxNav
mvn package
```

La phase `package` exécute le plugin Shade et produit le JAR exécutable `target/stibpath-1.0-SNAPSHOT.jar`, avec ses dépendances et `be.ulb.stib.Main` comme classe principale.

<a id="lancement"></a>

## ▶️ Lancement

Après compilation, lancez le JAR en lui donnant le chemin de votre répertoire racine GTFS. Avec ce seul argument, BruxNav charge les agences et construit le graphe multimodal.

L’invocation commence par `java -jar target/stibpath-1.0-SNAPSHOT.jar`, suivi du chemin réel vers ce répertoire.

Pour lancer directement la classe principale par Maven, la configuration du plugin `exec-maven-plugin` définit `be.ulb.stib.Main` comme point d’entrée.

<a id="utilisation"></a>

## 🎮 Utilisation

Pour rechercher un trajet, ajoutez après le répertoire GTFS le nom de l’arrêt de départ, celui de l’arrêt d’arrivée, puis l’heure de départ au format `HH:mm:ss` :

Après le chemin GTFS, l’ordre des arguments est : nom de l’arrêt de départ, nom de l’arrêt d’arrivée, puis heure de départ.

Les noms d’arrêt sont comparés sans tenir compte de la casse. Ils doivent correspondre à des noms présents dans les données chargées. Si aucun chemin n’est trouvé, le programme affiche `No path found.` ; si un nom d’arrêt est absent, la recherche lève une erreur indiquant que l’arrêt est introuvable.

<a id="donnees-gtfs"></a>

## 🗃️ Données GTFS

Les données ne sont pas versionnées dans le dépôt. Leur organisation par agence permet au programme de charger plusieurs réseaux depuis une même racine, puis de les fusionner avant la recherche.

`StopLoader`, `RouteLoader`, `TripLoader` et `StopTimesLoader`, dans `src/main/java/be/ulb/stib/parsing/`, alimentent chacun un `AgencyModel`. Les identifiants GTFS servent de clés pour relier les arrêts, lignes, trajets et horaires lors de cette phase.

<a id="architecture"></a>

## 🧱 Architecture

Le point d’entrée `src/main/java/be/ulb/stib/Main.java` valide le répertoire fourni, charge chaque agence, fusionne les modèles, génère les arêtes, construit le graphe puis déclenche la recherche seulement lorsque les trois arguments de recherche sont présents.

La couche `parsing/` lit les quatre fichiers GTFS à travers `tools/CsvReader.java`, qui s’appuie sur OpenCSV. Les chargeurs créent les objets de `core/` et les stockent dans `data/AgencyModel.java`. `data/GlobalModel.java` fusionne ces collections dans un modèle unique ; `data/StringPool.java` déduplique les noms d’arrêts et de lignes en les remplaçant par des indices stables.

`spatial/` produit les connexions du réseau. `TransitEdgeGenerator.java` transforme les paires d’horaires successives de chaque `Trip` en `TransitEdge`. `WalkEdgeGenerator.java` construit un `KDTree`, recherche les arrêts dans un rayon d’un kilomètre et crée les `WalkEdge` avec un coût calculé à partir des coordonnées. `graph/MultiModalGraph.java` indexe ensuite ces deux types d’arêtes sortantes par identifiant d’arrêt.

Enfin, `algo/AStarTD.java` interroge ce graphe et `output/ItineraryFormatter.java` restitue le chemin. Les données restent en mémoire pendant toute l’exécution : le projet n’emploie ni base de données ni service réseau.

<a id="flux-general"></a>

## 🧬 Flux général

```text
répertoire GTFS
  └── une agence par sous-répertoire
        └── stops / routes / trips / stop_times CSV
              ↓
        chargeurs GTFS → AgencyModel → GlobalModel
                                      ↓
                         arêtes de marche + arêtes de transport
                                      ↓
                               MultiModalGraph
                                      ↓
                         AStarTD (heure de départ, arrêt source, arrêt cible)
                                      ↓
                          ItineraryFormatter → terminal
```

Lors de la recherche, `AStarTD` maintient le meilleur temps d’arrivée connu par arrêt et une file de priorité. Pour une arête de transport, il écarte les départs déjà passés ; pour une arête piétonne, il ajoute simplement sa durée au temps courant. Sa priorité combine le temps d’arrivée, une estimation à vol d’oiseau jusqu’à la destination et des pénalités de 5 minutes lors d’un changement de mode ou de 2 minutes lors d’un changement de ligne. Les arêtes parentes permettent de reconstruire le chemin trouvé dans l’ordre du départ vers l’arrivée.

<a id="structure-du-projet"></a>

## 📂 Structure du projet

```text
BruxNav/
├── src/main/java/be/ulb/stib/
│   ├── Main.java                  # Chargement, orchestration et interface en ligne de commande
│   ├── algo/AStarTD.java          # Recherche A* dépendante du temps
│   ├── core/                      # Arrêts, lignes, trajets, horaires et arêtes du graphe
│   ├── data/                      # Modèles d’agence/global et pools de chaînes
│   ├── graph/MultiModalGraph.java # Adjacence des arêtes de marche et de transport
│   ├── output/                    # Mise en forme de l’itinéraire dans le terminal
│   ├── parsing/                   # Chargeurs des fichiers GTFS CSV
│   ├── spatial/                   # KD-tree et générateurs d’arêtes
│   └── tools/                     # Lecteur CSV UTF-8 et utilitaires de chargement
├── pom.xml                        # Dépendances, compilation et création du JAR exécutable
├── output.png                     # Capture d’une exécution du programme
├── Projet.pdf                     # Document associé au projet
└── README.md
```

<a id="tests"></a>

## 🧪 Tests

Le projet déclare JUnit Jupiter et le plugin Maven Surefire dans `pom.xml`. Aucun fichier de test n’est actuellement versionné sous `src/test/`; la commande suivante exécute donc la phase de tests Maven sans suite de tests locale :

```bash
mvn test
```

<a id="problemes-frequents"></a>

## ❗ Problèmes fréquents

### Le programme indique que le répertoire n’est pas valide

Le premier argument doit désigner un répertoire existant. Le programme s’arrête immédiatement lorsqu’il ne peut pas le lire comme répertoire.

### Une colonne GTFS est signalée comme absente

Chaque chargeur recherche les en-têtes indiqués dans la table de configuration. Vérifiez notamment l’orthographe exacte des en-têtes et la présence des quatre fichiers dans chaque répertoire d’agence.

### Un arrêt est introuvable ou aucun chemin n’est trouvé

Les deux noms doivent être présents dans les données chargées. La recherche dépend également de l’heure indiquée : un segment de transport dont l’heure de départ est déjà passée n’est pas utilisable.

### La compilation échoue sur la version de Java

Le projet est compilé avec la release 21. Vérifiez que `java --version` et `mvn --version` pointent vers un JDK 21.

<a id="documentation"></a>

## 📄 Documentation

- [Document du projet](Projet.pdf)
