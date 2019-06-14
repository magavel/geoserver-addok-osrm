# Introduction

## La société:



En 2016 la société Terra Nova rejoint Sysoco et est spécialisée dans les solutions de collecte d'informations à distance dans des domaines liés à la Propreté Urbaine, au Transport et à la Viabilité Hivernale.

Les solutions logicielles SYGETRACK de SYSOCO sont utilisées par des clients importants comme Montpellier Méditerranée Métropole, la Métropole Aix-Marseille, Nice Côte d'Azur Métropole ou le Syndicat Mixte du Bois de l'Aumône.  La Police Municipale de la Ville de Nice figure également dans la liste de références de Terra Nova ainsi que le Conseil Départemental du Loir-et-Cher. Dans la région Rhône Alpes, des clients comme les Villes de Saint-Etienne,  d'Echirolles et Chambéry Métropole utilisent quotidiennement des solutions déployées par Terra nova. Des entreprises privées comme,  SILIM & BRONZO Environnement, Derichebourg PolyUrbaine ou le Groupe Nicollin ont choisi les solutions développées et déployées par la société.

Depuis mars 2019 Sysoco est intégré au sein de Vinci énergies.

Sysoco mobility est la société au sein de Vinci qui assure la mise en oeuvre et le développement des solutions SYGETRACK. Elle est composée d'un directeur, de quatre ingénieurs développeurs, deux secrétaires qui assurent le SAV et les contrats et deux responsables commerciaux.





## Le logiciel

Les solutions mise en œuvre par SYGETRACK permettent aux opérateurs de programmer  les tournées de ramassage des ordures ménagères. Chaque camion benne à ordures ménagères ou BOM est équipé de multiples capteurs; puces GPS, capteurs de levée de bac, lecteurs de puces RFID permettant de remonter vers les serveurs toutes les informations de poids, d'identification, de chaque mouvement de bac. De plus chaque information est géolocalisée. 

SYGETRACK permets pour l'opérateur des remontée de statistiques sur l'ensemble des tournée, d'effectuer des calcul de carbone consommée pour l'établissement des taxes carbone, etc… Ces taxes carbone sont ensuite payer par l'organisme. Cela montre l'importance d'optimiser les tournées.

Comme chaque informations est géolocalisée par une latitude et une longitude il est nécessaire d'effectuer un « reverse-geocoding » de ces positions pour retrouver une adresse avec numéro de rue, nom de rue et ville. Il existe des services payants ou gratuits qui permettent ce reverse-geocoding mais avec des limites. 

Le gouvernement à mis en place une API  permettant de faire du geocoding et reverse-geocoding à l'adresse suivante: 

​		`https://adresse.data.gouv.fr/api`.

Cette API est gratuite mais avec les limites suivantes:

- L'API unitaire est disponible à hauteur de 10 appels par seconde et par adresse IP.
- Le géocodage de masse (CSV) est disponible à hauteur d'un appel simultané par adresse IP.  

SYGETRACK à besoin de retrouver plusieurs centaines d'adresses par jour. Cette solution n'est donc pas possible. Il existe aussi des sociétés privés qui vendent des services de geocoding et de geocoding inverse. C'est la solution actuelle retenue par SYGETRACK. Mais elle à un coût.

Pour l'organisation des tournées de ramassage SYGETRACK fait appel à une société qui calcule la meilleurs tournée entre plusieurs points, c'est la société `benomad`:` https://benomad.com`.



## Présentation de la mission

La mission qui m'est confiée comporte deux parties. 

La première partie concerne les serveurs de tuiles cartographiques. Il s'agit de faire un état de l'art des technologies existantes puis de mettre en place ce serveur.

La deuxième partie concerne les API de **géocodage** et **routage**

Le géocodage consiste à affecter des [coordonnées géographiques](https://fr.wikipedia.org/wiki/Coordonnées_géographiques) ([longitude](https://fr.wikipedia.org/wiki/Longitude)/[latitude](https://fr.wikipedia.org/wiki/Latitude)) à une adresse postale. Son pendant est le géocodage inversé. Le géocodage inversé (ou en [anglais](https://fr.wikipedia.org/wiki/Anglais) : *reverse geocoding*) consiste à effectuer l'opération inverse du [géocodage](https://fr.wikipedia.org/wiki/Géocodage), c'est-à-dire d'attribuer une adresse à des [coordonnées géographiques](https://fr.wikipedia.org/wiki/Coordonnées_géographiques). L'adresse ainsi retrouvée peut être utilisée dans des applications de [géolocalisation](https://fr.wikipedia.org/wiki/Géolocalisation) capables d'indiquer l'adresse où se trouve la personne utilisatrice.

Le **routage** est le mécanisme par lequel des chemins sont sélectionnés dans un [réseau](https://fr.wikipedia.org/wiki/Réseau_informatique) routier pour acheminer un voyageur d'un point de départ  vers un ou plusieurs point de destination. Il existe traditionnellement le routage entre deux points ou la routage sur une multitude de point pour organiser au mieux une tournée.



# Configuration du serveur

Pour créer, tester les différentes configuration il sera fait le choix d'un serveur sous  VM avec virtualBox. Cette VM sera montée sur un PC tournant sous windows 7.

Vu les besoins de mémoire la configuration minimale sera la suivante:

- disque dur de 75 Go.
- 4 coeurs virtuels
- Un minimum de 35 Go de RAM
- Si la RAM est inférieur mettre en place une swap de 35 Go

Les prérequis logiciels sont:

- docker

- Docker-compose

- git

- wget

- Unzip

- postgresql

- Postgis

  

#  Serveur cartographique

 Actuellement SYSOCO utilise un serveur de tuiles cartographique ayant été mis en place il y a une dizaine d'année sous Flash/Adobe en actionscript 2. Ce serveur n'a jamais été mis à jour et se retrouve bloqué par les politiques des navigateurs internet refusant la technologie flash.

SYSOCO utilise aussi un serveur de tuiles cartographique sur internet mise en place par OpenStreetMap et soumis à des mise à jour non maitrisée par SYSOCO.

SYSOCO désire maitriser les mises à jour du serveur en mettant en place son propre serveur de tuiles cartographiques.

Après recherche des différentes technologies existantes pour mettre en place un serveur de tuiles cartographiques il s'avère que deux serveurs dominent le marché open source des serveurs cartographiques: Mapserver et Geoserver.

Mapserver ne possède pas d'interface graphique et l'intégralité de la configuration se fait avec des fichiers.map.

Geoserver se configure grâce à une interface graphique ou l'on peut mettre en place les différentes services avec leurs sources de données. Geoserver est écrit en java et nécessite l'installation d'un serveur TOMCAT.

La difficulté sur ces deux serveurs est de mettre en place des fichiers de configuration de styles permettant la visualisation des informations cartographiques aux couleurs habituelles pour les clients, à savoir OpenStreetMap.

Ces deux serveurs sont connectés à un serveur de base de données.

En terme de gestion de la données physique, nous utiliserons un serveur de base de données Postgresql avec PostGis. Le plugin Postgis est doté de toutes les fonctionnalités d'un système d'information géographique (SIG). Postgresql+Postgis est basé sur une architecture client-serveur accessible depuis le web grâce à pgAdmin. 

Pour télécharger les données issue de OpenStreetMap il est nécessaire d'utiliser un outils supplémentaire: **osm2pgsql**

## Installation de PostGresql

### Méthode recommandée

 Installez simplement le paquet postgresql, par ligne de commande : 

```shell
sudo apt install postgresql
```

 PostgreSQL est un serveur qui permet de se connecter à différentes bases de données. Par défaut, seul l'utilisateur *postgres* peut se connecter. 

 Toutes les opérations d'administration se font, au départ, avec l'utilisateur *postgres*.  À la fin de l'installation, celui-ci ne possède pas de mot de passe :  c'est un utilisateur bloqué et le mieux est qu'il le reste. La première  chose à faire sera de créer un nouvel utilisateur, mais pour ce faire,  il faut se connecter au moins une fois en tant qu'utilisateur *postgres*. Pour devenir *postgres* et faire les opérations d'administration qui suivent, utilisez sudo : 

```shell
$ sudo -i -u postgres 
Password: 
```

 **exit** permettra, à la fin de cette session d'administration dans PostgreSQL, de reprendre la main en tant qu'utilisateur du système. 

Désormais, l'invite de commande doit mentionner que vous êtes actif en tant que *postgres*. Pour lancer l'invite de commande SQL de PostgreSQL, tapez simplement : 

```shell
psql
```



## Installation de PostGis



```shell
    sudo apt-get install postgis
```



Pour créer une base de données nommée webmapping avec l'extension PostGis il faut:

```shell
sudo -u postgres createdb -O postres webmapping
sudo -u postgres psql -c "CREATE EXTENSION PostGIS; CREATE EXTENSION PostGis_topology;" webmapping
```

Cette extension permettra de mettre en place les différentes tables

## Installation d'Osm2pgsql

**Osm2pgsql** est un outil de chargement de données **OpenStreetMap** dans une base de données **PostgreSQL** / **PostGIS** appropriée pour des applications telles que le rendu dans une carte.

Pour télécharger les données OSM dans la base de données postGIS voir le lien https://github.com/openstreetmap/osm2pgsql.

### Caractéristiques

- ​     Convertit les fichiers OSM en une base de données PostgreSQL
- ​     La conversion des étiquettes en colonnes est configurable dans le fichier de style
- ​     Capable de lire les fichiers .gz, .bz2, .pbf et .osm directement
- ​     Peut appliquer des différences pour maintenir la base de données à jour
- ​     Soutenir le choix de la projection en sortie
- ​     Noms de table configurables
- ​     Prise en charge du type de champ hstore pour stocker l'ensemble complet de balises dans un champ de base de données si nécessaire

### Installation

Prerequis:

```shell
sudo apt-get install make cmake g++ libboost-dev libboost-system-dev \
  libboost-filesystem-dev libexpat1-dev zlib1g-dev \
  libbz2-dev libpq-dev libproj-dev lua5.2 liblua5.2-dev
```



Pour installer de manière simple osm2pgsql on peut passer par **apt-get**:

```shell
apt-get install osm2pgsql
```



Pour installer osm2pgsql sous Ubuntu en compilant les sources il faut télécharger sur github la dernière version :

```shell
git clone git://github.com/openstreetmap/osm2pgsql.git
```

Une fois les dépendances installées il faut générer le MakeFile: 

```shell
mkdir build && cd build
cmake ..
```

Une fois le makeFile créé, exécuter:

```shell
make
```

Les fichiers compilés peuvent être installés avec:

```shell
sudo make install
```

Pour vérifier que osm2pgsql est installé il faut exécuter:

```shell
osm2pgsql --help
```



La réponse devra être:

```shell
osm2pgsql version 0.96.0 (64 bit id space)

Usage:
	osm2pgsql [options] planet.osm
	osm2pgsql [options] planet.osm.{pbf,gz,bz2}
	osm2pgsql [options] file1.osm file2.osm file3.osm

This will import the data from the OSM file(s) into a PostgreSQL database
suitable for use by the Mapnik renderer.

    Common options:
       -a|--append      Add the OSM file into the database without removing
                        existing data.
       -c|--create      Remove existing data from the database. This is the
                        default if --append is not specified.
       -l|--latlong     Store data in degrees of latitude & longitude.
       -m|--merc        Store data in proper spherical mercator (default).
       -E|--proj num    Use projection EPSG:num.
       -s|--slim        Store temporary data in the database. This greatly
                        reduces the RAM usage but is much slower. This switch is
                        required if you want to update with --append later.
       -S|--style       Location of the style file. Defaults to
                        /usr/local/Cellar/osm2pgsql/0.96.0_1/share/osm2pgsql/default.style.
       -C|--cache       Use up to this many MB for caching nodes (default: 800)
       -F|--flat-nodes  Specifies the flat file to use to persistently store node
                        information in slim mode instead of in PostgreSQL.
                        This file is a single > 40Gb large file. Only recommended
                        for full planet imports. Default is disabled.

    Database options:
       -d|--database    The name of the PostgreSQL database to connect to.
       -U|--username    PostgreSQL user name (specify passsword in PGPASS
                        environment variable or use -W).
       -W|--password    Force password prompt.
       -H|--host        Database server host name or socket location.
       -P|--port        Database server port.

A typical command to import a full planet is
    osm2pgsql -c -d gis --slim -C <cache size> -k \
      --flat-nodes <flat nodes> planet-latest.osm.pbf
where
    <cache size> is 20000 on machines with 24GB or more RAM
      or about 75% of memory in MB on machines with less
    <flat nodes> is a location where a 19GB file can be saved.

A typical command to update a database imported with the above command is
    osmosis --rri workingDirectory=<osmosis dir> --simc --wxc - \
      | osm2pgsql -a -d gis --slim -k --flat-nodes <flat nodes> -r xml -
where
    <flat nodes> is the same location as above.
    <osmosis dir> is the location osmosis replication was initialized to.

Run osm2pgsql --help --verbose (-h -v) for a full list of options.
```



Un appel  pour charger les données dans la base de données gis pour le rendu serait

```shell
osm2pgsql --create --database gis data.osm.pbf
```

Cela chargera les données de data.osm.pbf dans les tables 

- planet_osm_point, 

- planet_osm_line, 

- planet_osm_roads et 

- planet_osm_polygon.

  

Pour importer l'extraction complète de OpenStreetMap pour la France  la commande utilisée est:

```shell
wget http://download.geofabrik.de/europe/france-latest.osm.pbf
// une fois le téléchargement terminé on peut importer le fichier dans la base de données.

osm2pgsql -c -d webmapping -U postgres -W  -C 20000 france-latest.osm.pbf
```

Où

- -d => webmapping est le nom de la base de données
- -c donne l'instruction create
- -W force la demande de mot de passe

- -U postrgres est le nom d'utilisateur
- _C = <cache size>  vaut 75% de la mémoire en MiB, avec un maximum de 30000. 20000 semble correct.



## Installation de geoserver



# Les API

## Introduction

Il existe un certain nombre de site proposant des API de routing ou geocoding sur internet. Les services gratuits sont bien souvent assortis de limite ne permettant pas d'utiliser ces services de manière industrielle. Quant aux services sans limite leur coût est bien souvent très important.

Toutes les routes des API sont testées avec le logiciel POSTMAN.

## Le routing

### Les algorithmes utilisés

De nombreux algorithmes de recherche de chemin dans un graphe existent, et leur utilisation se limite pas à du calcul d'itinéraire routier. Une application fréquente de ces algorithmes est par exemple le routage dans un réseau informatique, pour sélectionner les meilleurs chemins permettant d'acheminer des données. Plusieurs algorithmes courants de recherche du plus court chemin dans un graphe sont présentés ci-après.



#### A* (A star)

L'algorithme A* est un algorithme de recherche de chemin dans un [graphe](https://fr.wikipedia.org/wiki/Graphe_(théorie_des_graphes)) entre un [nœud](https://fr.wikipedia.org/wiki/Nœud_(mathématiques)) initial et un nœud final. Il utilise une évaluation [heuristique](https://fr.wikipedia.org/wiki/Heuristique_(mathématiques)) sur chaque nœud pour estimer le meilleur chemin y passant, et visite  ensuite les nœuds par ordre de cette évaluation heuristique. C'est un algorithme simple, ne nécessitant pas de prétraitement, et ne consommant que peu de mémoire.



#### Dijkstra

L'algorithme de Dijkstra permet de résoudre un [problème algorithmique](https://fr.wikipedia.org/wiki/Problème_algorithmique) : le [problème du plus court chemin](https://fr.wikipedia.org/wiki/Problème_du_plus_court_chemin). Ce problème a plusieurs variantes. La plus simple est la suivante : étant donné un [graphe non-orienté](https://fr.wikipedia.org/wiki/Graphe_non-orienté), dont les arêtes sont munies de poids, et deux sommets de ce graphe, trouver un [chemin](https://fr.wikipedia.org/wiki/Chaîne_(théorie_des_graphes))

entre les deux sommets dans le graphe, de poids minimum. L'algorithme 

de Dijkstra permet de résoudre un problème plus général : le graphe peut

être [orienté](https://fr.wikipedia.org/wiki/Graphe_orienté),

et l'on peut désigner un unique sommet, et demander d'avoir la liste 

des plus courts chemins pour tous les autres nœuds du graphe.    



### Les outils

Pour le routing une bibliothèque est plébiscité par l'ensemble de la communauté des utilisateurs des services géographiques. Il s'agit d'OSRM. 

L'**Open Source Routing Machine** ou **OSRM** est l'implémentation [C++](https://fr.wikipedia.org/wiki/C%2B%2B) d'un moteur de recherche d'itinéraire haute performance afin d'obtenir les [plus courts chemins](https://fr.wikipedia.org/wiki/Problèmes_de_cheminement) dans un [réseau routier](https://fr.wikipedia.org/wiki/Réseau_routier).  OSRM est en développement actif. Sur Github le dernier commit date de 13 jours. OSRM est actuellement maintenu par [Dennis Luxen](http://algo2.iti.kit.edu/english/luxen.php) docteur en recherche informatique au sein du KIT le Karlruhe Institute of Technology.

Ce moteur de routage est prévu de fonctionner avec les données de OpenStreetMap.

Les services suivants sont disponible via une API en HTTP:

- Nearest - Enregistre les coordonnées sur le réseau de rues et renvoie les correspondances les plus proches. 
- Route - Trouve l'itinéraire le plus rapide entre les coordonnées 
- Table - Calcule la durée ou les distances de l'itinéraire le plus rapide entre toutes les paires de coordonnées fournies 
- Trip - Résout le problème du voyageur de commerce en utilisant une heuristique gloutonne 
- Tiles - Génère des tuiles vectorielles Mapbox avec des métadonnées de routage internes 



Contrairement à la plupart des serveurs de routage, OSRM n'utilise pas l'algorithme  [A*](http://en.wikipedia.org/wiki/A*_search_algorithm)  pour calculer le plus court chemin, mais mais le théorème de la contraction des hiérarchies.  Il en résulte des temps de requêtes très rapides, généralement  inférieure à 1 ms pour les ensembles de données comme l'Europe, faisant  OSRM un bon candidat pour les applications et sites sensibles routage  sur le Web. 

### Installation de OSRM

Première chose à faire pour tester OSRM il faut télécharger des données de open street map au format pbf. 

Ces données sont librement téléchargeable sur le site de 

[`http://download.geofabrik.de/`](http://download.geofabrik.de/)

On utilise la commande suivante dans le répertoire de travail:

```shell
wget http://download.geofabrik.de/europe/france-latest.osm.pbf
```



OSRM ayant était conteneurisée avec docker il faut initialiser un conteneur docker avec la commande suivante:

```shell
docker run -t -v "${PWD}:/data" osrm/osrm-backend osrm-extract -p /opt/car.lua /data/france-latest.osm.pbf
```

Cette commande va extraire du fichier natif de OSM les fichiers contenant les notes, way et relations permettant d'effectuer le calcul de routing.

Cette opération nécessite beaucoup de mémoire.

A titre d'exemple pour le fichier paca-latest.osm.pbf de 260 Mo le serveur consomme en RAM: *1 Go*

Apres l'extraction on exécute:

```shell
docker run -t -v "${PWD}:/data" osrm/osrm-backend osrm-partition /data/france-latest.osrm
```

Ensuite:

```shell
docker run -t -v "${PWD}:/data" osrm/osrm-backend osrm-customize /data/france-latest.osrm
```



Il faut ensuite lancer une instance du conteneur docker avec le serveur:

```shell
docker run -t -i -p 5000:5000 -v "${PWD}:/data" osrm/osrm-backend osrm-routed --algorithm mld /data/france-latest.osrm
```



Une fois le serveur en fonction on peut tester une requête GET avec PostMan:

```shell
http://localhost:5000/route/v1/driving/5.9501,43.1194;5.9655,43.113?steps=true&geometries=geojson
```



La réponse du serveur sera du type:

La réponse en JSON sera:

```json
{
    "code": "Ok",
    "routes": [
        {
            "geometry": {
                "coordinates": [
                    [
                        5.950025,
                        43.119348
                    ],
                    [
                        5.950102,
                        43.119252
                    ],
                    [
                        5.950346,
                        43.119312
                    ...
                      ...
                      ...
                      ]
               
           
                            },
                            "mode": "driving",
                            "duration": 63,
                            "maneuver": {
                                "bearing_after": 120,
                                "type": "turn",
                                "modifier": "right",
                                "bearing_before": 70,
                                "location": [
                                    5.950346,
                                    43.119312
                                ]
                            },
                            "weight": 63,
                            "distance": 684.6,
                            "name": "Avenue François Nardi"
                        },
                        {
                            "intersections": [
                                {
                                    "out": 1,
                                    "location": [
                                        5.957094,
                                        43.11611
                                    ],
                                    "bearings": [
                                        90,
                                        210,
                                        315
                                    ],
                                    "entry": [
                                        true,
                                        true,
                                        false
                                    ],
                                    "in": 2
                                },
                                
                    ],
                    "distance": 1638.3,
                    "duration": 172.1,
                    "summary": "Avenue François Nardi, Avenue de la Résistance",
                    "weight": 172.1
                }
            ],
            "distance": 1638.3,
            "duration": 172.1,
            "weight_name": "routability",
            "weight": 172.1
        }
    ],
    "waypoints": [
        {
            "hint": "a5AAgMWRAIBPAAAABwAAAAsAAAAFAAAAgcmuQmFz9UC910ZBN32mQE8AAAAHAAAACwAAAAUAAAD4AgAAScpaAPTykQKUyloAKPORAgEAjwj51xz2",
            "distance": 8.403752,
            "name": "Boulevard Glassendi",
            "location": [
                5.950025,
                43.119348
            ]
        },
        {
            "hint": "5JAAgDuQAIAZAAAAEAAAADEAAACHAAAAIGzmQdp5hkGQj1lChrwWQxkAAAAQAAAAMQAAAIcAAAD4AgAA0AZbAEDakQK8BlsAKNqRAgMAnwf51xz2",
            "distance": 3.123755,
            "name": "Avenue de la Résistance",
            "location": [
                5.96552,
                43.113024
            ]
        }
    ]
}
```



### Mise en oeuvre de OSRM



Toutes les requêtes HTTP OSRM utilisent une structure commune.



```shell
GET/{service}/{version}/{profile}/{coordinates}[.{format}]?option=value&option=value
```





#### Requête:



| Paramètres  | Description                                                  |
| ----------- | ------------------------------------------------------------ |
| Service     | L' une des valeurs suivantes: [route](http://project-osrm.org/docs/v5.22.0/api/#route-service) , [nearest](http://project-osrm.org/docs/v5.22.0/api/#nearest-service) , [table](http://project-osrm.org/docs/v5.22.0/api/#table-service) , [match](http://project-osrm.org/docs/v5.22.0/api/#match-service) , [trip](http://project-osrm.org/docs/v5.22.0/api/#trip-service) , [tile](http://project-osrm.org/docs/v5.22.0/api/#tile-service) |
| version     | Version du protocole implémenté par le service. v1 pour toutes les installations OSRM 5.x |
| profile     | Le mode de transport, est déterminé statiquement par le profil Lua utilisé pour préparer les données à l'aide de osrm-extract . Généralement car , bike ou foot si vous utilisez l'un des profils fournis. |
| coordinates | Chaîne de format {longitude},{latitude};{longitude},{latitude}[;{longitude},{latitude} ...] ou polyline({polyline}) or polyline6({polyline6}) . |
| format      | Seul json est pris en charge pour le moment. Ce paramètre est optionnel et vaut par défaut json . |





#### Réponses:

Voici les réponses possibles:



| **Type**       | **La description**                                           |
| -------------- | ------------------------------------------------------------ |
| Ok             | La demande pourrait être traitée comme prévu.                |
| InvalidUrl     | La chaîne d'URL n'est pas valide.                            |
| InvalidService | Le nom du service est invalide.                              |
| InvalidVersion | La version est introuvable.                                  |
| InvalidOptions | Les options ne sont pas valides.                             |
| InvalidQuery   | La chaîne de requête est incorrectement synchronisée.        |
| InvalidValue   | Les paramètres de requête analysés avec succès ne sont pas valides. |
| NoSegment      | L'une des coordonnées d'entrée fournies n'a pas pu s'aligner sur le segment de rue. |
| TooBig         | La taille de la demande enfreint l'une des restrictions de taille de la demande spécifique au service. |



Les Services







Les différents services de l'API



Nearest:

A partir d'une coordonnées en latitude et longitude OSRM recherche et renvoie les n correspondances les plus proche. N correspond au paramètre number.







| **Option** | **Values**                | **Description**                         |
| ---------- | ------------------------- | --------------------------------------- |
| number     | integer >= 1 (default 1 ) | Retourne le nombre de segments demandés |



Exemple :



\# Querying nearest three snapped locations of `13.388860,52.517037` with a bearing between `20° - 340°`.

curl 'http://router.project-osrm.org/nearest/v1/driving/13.388860,52.517037?number=3&bearings=0,20'



## Le geocoding

Le gouvernement français mets à la disposition des développeurs un répertoire github (https://github.com/etalab) dans lequel on trouvera l'ensemble des produits et API développé par ETALAB.

**La mission Etalab** fait  partie de la Direction interministérielle du numérique et du système  d'information et de communication de l'Etat (DINSIC), dont les missions  et l'organisation sont fixées par les [décrets du 20 Novembre 2017. ](http://www.modernisation.gouv.fr/documentation/decrets/une-nouvelle-organisation-pour-la-transformation-publique-et-numerique-de-letat-decrets-du-20-novembre-2017)

**Etalab** est une [mission](https://fr.wikipedia.org/wiki/Missions_et_programmes) placée au sein de la [Direction interministérielle du numérique et du système d'information et de communication de l'État](https://fr.wikipedia.org/wiki/Direction_interministérielle_du_numérique_et_du_système_d'information_et_de_communication_de_l'État) (DINSIC).    

Etalab  améliore le service public :   

- en menant une politique de la donnée (ouverture et  partage des  données publiques ou "open data", exploitation des données et  intelligence artificielle...). Etalab met notamment en œuvre les  missions de l'Administrateur général des données (AGD), fixées par le  décret n° 2014-1050 du 16 septembre 2014[1](https://fr.wikipedia.org/wiki/Etalab#cite_note-1).
- en promouvant  une action publique plus transparente et collaborative grâce au numérique, pour tendre vers un [gouvernement ouvert](https://fr.wikipedia.org/wiki/Gouvernement_ouvert).

Etalab développe et maintient le portail des données ouvertes du gouvernement français [data.gouv.fr](https://fr.wikipedia.org/wiki/Data.gouv.fr). 

Etalab contribue à la promotion des  sciences des données (« datasciences ») et de l'intelligence  artificielle dans la sphère publique, en menant [des  projets, en facilitant l'expérimentation et le partage de bonnes  pratiques, en animant des communautés de datascientists et porteurs de  projet d'IA.](https://www.etalab.gouv.fr/datasciences-et-intelligence-artificielle)



Pour palier aux limites imposées sur leur API ETALAB propose en téléchargement libre le code source de leur moteur de coding à l'adresse suivante: 

`https://github.com/etalab/addok`

Addok fonctionne avec Redis en back-end.

- il importe et indexe les données (et peut aussi être importé depuis une base de données Nominatim) en ligne de commande
- il sert une API basée sur du GeoJSON (en utilisant Werkzeug et Gunicorn)
- il fait du géocodage inversé ("reverse geocoding")
- il fait du géocodage par batch (via du CSV pour le moment)
- il fournit une ligne de commande de debug pour inspecter l'index
- il est configurable sur de nombreux détails
- il est open source.

### Pré-requis

- Au moins 6 Go de RAM disponible (à froid)
- 8 Go d'espace disque disponible (hors logs)Installer une instance avec les données de la Base Adresse Nationale en ODbL

### Installation

Tout d'abord il faut se placer dans un dossier de travail par exemple `ban`.

```shell
mkdir ban && cd ban
```

Télécharger les données pré-indexées

```shell
wget https://adresse.data.gouv.fr/data/geocode/ban_odbl_addok-latest.zip
```

Décompresser l'archive

```shell
mkdir addok-data
unzip -d addok-data ban_odbl_addok-latest.zip
```

Télécharger le fichier Compose

```shell
wget https://raw.githubusercontent.com/etalab/addok-docker/master/docker-compose.yml
```

Démarrer l'instance

Suivant votre environnement, `sudo` peut être nécessaire pour les commandes suivantes.

```shell
# Attachée au terminal
docker-compose up

# ou en arrière-plan
docker-compose up -d
```

Suivant les performances de la machine, l'instance mettra entre 30  secondes et 2 minutes à démarrer effectivement, le temps de charger les  données dans la mémoire vive.

- 90 secondes sur une VPS-SSD-3 OVH (2 vCPU, 8 Go)
- 50 secondes sur une VM EG-15 OVH (4 vCPU, 15 Go)

Par défaut l'instance écoute sur le port `7878`.

#### 

#### Tester l'instance

```shell
curl "http://localhost:7878/search?q=1+rue+de+la+paix+paris"
```

### 

### Mise en oeuvre de l'API

#### /search/



Point d'entrée pour le géocodage.

Utiliser le paramètre **q** pour faire une recherche plein texte:

```shell
curl https://api-adresse.data.gouv.fr/search/?q=8+bd+du+port
```

Avec **limit** on peut contrôler le nombre d'éléments retournés:

```shell
curl https://api-adresse.data.gouv.fr/search/?q=8+bd+du+port&limit=15
```

Avec **autocomplete** on peut désactiver les traitements d'auto-complétion:

```shell
curl https://api-adresse.data.gouv.fr/search/?q=8+bd+du+port&autocomplete=0
```

Avec **lat** et **lon** on peut donner une priorité géographique:

```shell
curl https://api-adresse.data.gouv.fr/search/?q=8+bd+du+port&lat=48.789&lon=2.789
```

Les filtres **type**, **postcode** (code Postal) et **citycode** (code INSEE) permettent de restreindre la recherche:

```shell
curl https://api-adresse.data.gouv.fr/search/?q=8+bd+du+port&postcode=44380
curl https://api-adresse.data.gouv.fr/search/?q=paris&type=street
```

Le retour est un geojson *FeatureCollection* respectant la spec [GeoCodeJSON](https://github.com/yohanboniface/geocodejson-spec):

```javascript
{
  "attribution": "BAN",
  "licence": "ODbL 1.0",
  "query": "8 bd du port",
  "type": "FeatureCollection",
  "version": "draft",
  "features": [
    {
      "properties": {
        "context": "80, Somme, Picardie",
        "housenumber": "8",
        "label": "8 Boulevard du Port 80000 Amiens",
        "postcode": "80000",
        "citycode": "80021",
        "id": "ADRNIVX_0000000260875032",
        "score": 0.3351181818181818,
        "name": "8 Boulevard du Port",
        "city": "Amiens",
        "type": "housenumber"
      },
      "geometry": {
        "type": "Point",
        "coordinates": [
          2.29009,
          49.897446
        ]
      },
      "type": "Feature"
    },
    {
      "properties": {
        "context": "34, Hérault, Languedoc-Roussillon",
        "housenumber": "8",
        "label": "8 Boulevard du Port 34140 Mèze",
        "postcode": "34140",
        "citycode": "34157",
        "id": "ADRNIVX_0000000284423783",
        "score": 0.3287575757575757,
        "name": "8 Boulevard du Port",
        "city": "Mèze",
        "type": "housenumber"
      },
      "geometry": {
        "type": "Point",
        "coordinates": [
          3.605875,
          43.425232
        ]
      },
      "type": "Feature"
    }
  ]
}
```

Les attributs retournés sont :

- *id* : identifiant de l'adresse (non stable: actuellement identifiant IGN)

- *type* : type de résultat trouvé

- - *housenumber* : numéro « à la plaque »
  - *street* : position « à la voie », placé approximativement au centre de celle-ci
  - *locality* : lieu-dit
  - *municipality* : numéro « à la commune »

- *score* : valeur de 0 à 1 indiquant la pertinence du résultat

- *housenumber* : numéro avec indice de répétition éventuel (bis, ter, A, B)

- *name* : numéro éventuel et nom de voie ou lieu dit

- *postcode* : code postal

- *citycode* : code INSEE de la commune

- *city* : nom de la commune

- *context* : n° de département, nom de département et de région

- *label* : libellé complet de l'adresse





#### /reverse/

Point d'entrée pour le géocodage inverse.

Les paramètres **lat** et **lon** sont obligatoires:

```
curl https://api-adresse.data.gouv.fr/reverse/?lon=2.37&lat=48.357
```

Le paramètre **type** permet forcer le type de retour:

```
curl https://api-adresse.data.gouv.fr/reverse/?lon=2.37&lat=48.357&type=street
```

Même format de réponse que pour le point d'entrée [**/search/**](https://adresse.data.gouv.fr/api#search).

#### /search/csv/

Point d'entrée pour le géocodage de masse à partir d'un fichier CSV.

Le fichier csv, encodé en UTF-8 et limité actuellement à 8Mo, doit être passé via le paramètre **data**:

```
curl -X POST -F data=@path/to/file.csv https://api-adresse.data.gouv.fr/search/csv/
```

Par  défaut, toutes les colonnes sont concaténées pour constituer l'adresse  qui sera géocodée. On peut définir les colonnes à utiliser via de  multiples paramètres **columns**:

```
curl -X POST -F data=@path/to/file.csv -F columns=voie -F columns=ville https://api-adresse.data.gouv.fr/search/csv/
```

Il est possible de préciser le nom d'une colonne contenant le **code INSEE** ou le **code Postal** pour limiter les recherches, exemple :

```shell
curl -X POST -F data=@path/to/file.csv -F columns=voie -F columns=ville -F citycode=ma_colonne_code_insee https://api-adresse.data.gouv.fr/search/csv/
curl -X POST -F data=@path/to/file.csv -F columns=voie -F columns=ville -F postcode=colonne_code_postal https://api-adresse.data.gouv.fr/search/csv/
```

##### Exemples

```SHELL
http -f POST http://localhost:7878/search/csv/ columns='voie' columns='ville' data@path/to/file.csv
http -f POST http://localhost:7878/search/csv/ columns='rue' postcode='code postal' data@path/to/file.csv
```

### 

#### /reverse/csv/

Point d'entrée pour le géocodage inverse de masse à partir d'un fichier CSV.

Le fichier csv, encodé en UTF-8 et limité actuellement à 8Mo, doit être passé via le paramètre **data**. Il doit contenir les colonnes **latitude** (ou *lat*) et **longitude** (ou *lon* ou *lng*).

```sheel
curl -X POST -F data=@path/to/file.csv https://api-adresse.data.gouv.fr/reverse/csv/
```









# Bibliographie et liens

Editions papier

- COLLADO David  : Géomatique, Webmapping en Open Source aux éditions *ellipse*

-  LACOVELLA Stefano: GeoServer Beginner's Guide - Second Edition: Share geospatial data using Open Source standards (English Edition):  Édition Kindle       

- YOUNGBLOOD Brian: GeoServer Beginner's Guide (English Edition): Édition Kindle  

  

Sites internets

- http://project-osrm.org/
- https://adresse.data.gouv.fr/
- https://adresse.data.gouv.fr/api
- https://forum.etalab.gouv.fr/
- https://github.com/Project-OSRM/
- https://wiki.openstreetmap.org/wiki/FR:Open_Source_Routing_Machine
- https://fr.wikipedia.org/wiki/Géocodage_inversé

- https://fr.wikipedia.org/wiki/Géocodage
- https://fr.wikipedia.org/wiki/Routage
- https://github.com/openstreetmap/osm2pgsql