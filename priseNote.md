

# PB RENCONTRÉS

Sur windows mise a jour obligatoire de power shell 2.0 vers 3.0

### Postgresql

Pour installer le serveur de base de données postgresql on utilise brew qui va installer la dernières version de postgresql:

```shell
brew install postgresql
```

Pour démarrer le serveur:

```shell
brew services start postgresql
```

Pour stoper le serveur:

```shell
brew services stop postgresql
```

Pour lister les bases de données presentes :

```shell
psql -U postgres -l
```

Par exemple :

```shell
                                List of databases
       Name       |  Owner   | Encoding | Collate | Ctype |   Access privileges
------------------+----------+----------+---------+-------+-----------------------
 apiSigFox        | postgres | UTF8     | C       | C     |
 basetest2        | postgres | UTF8     | C       | C     |
 charmettes       | postgres | UTF8     | C       | C     |
 france           | postgres | UTF8     | C       | C     |
 paca             | postgres | UTF8     | C       | C     |
 postgres         | postgres | UTF8     | C       | C     |
 promogoose       | postgres | UTF8     | C       | C     | =Tc/postgres         +
                  |          |          |         |       | postgres=CTc/postgres
 template0        | postgres | UTF8     | C       | C     | =c/postgres          +
                  |          |          |         |       | postgres=CTc/postgres
 template1        | postgres | UTF8     | C       | C     | =c/postgres          +
                  |          |          |         |       | postgres=CTc/postgres
 template_postgis | postgres | UTF8     | C       | C     |
 webmapping       | postgres | UTF8     | C       | C     |
(11 rows)
```



Pour redemarrer postgresql  apres une mise a jour:

```shell
  brew services restart postgresql
```





### osm2pgsql

**osm2pgsql** est un outil de chargement de données OpenStreetMap dans une base de données PostgreSQL / PostGIS appropriée pour des applications telles que le rendu dans une carte, le géocodage avec Nominatim ou l'analyse générale.

Pour telecharger les données OSM dans la pase de données postGIS

voir le lien https://github.com/openstreetmap/osm2pgsql 



#### Caractéristiques

- ​     Convertit les fichiers OSM en une base de données PostgreSQL
- ​     La conversion des étiquettes en colonnes est configurable dans le fichier de style
- ​     Capable de lire les fichiers .gz, .bz2, .pbf et .osm directement
- ​     Peut appliquer des différences pour maintenir la base de données à jour
- ​     Soutenir le choix de la projection en sortie
- ​     Noms de table configurables
- ​     Prise en charge du type de champ hstore pour stocker l'ensemble complet de balises dans un champ de base de données si nécessaire



Installer **osm2pgsql** avec brew 

```shell
brew install osm2pgsql
```

Pour voir toute l'aide disponible taper en ligne de commande:

```shell
osm2pgsql --help
```

La réponse dans le terminal est:

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

Pour  l'importation d'une grande quantité de données telle que la planète complète, une ligne de commande typique serait:

```shell
osm2pgsql -c -d gis --slim -C <cache size> \
  --flat-nodes <flat nodes> planet-latest.osm.pbf
```

Pour inporter la base de données complète pour la France  la commande utilisée est:

```shell
osm2pgsql -c -d webmapping -U postgres -W  -C 20000 france-latest.osm.pbf
```



### pgrouting

Installer l'outils pgRouting ave brew

```shell
brew install osm2pgrouting
```

brew installe aussi les librairies nécessaires au fonctionnement de pgrouting: 

- icu4c, 
- boost, 
- readline, 
- postgresql, 
- libpqxx, 
- sqlite, 
- geos,
-  isl, 
- gcc, 
- hdf5, 
- libdap, 
- libgeotiff, 
- libpq, 
- netcdf, 
- openblas, 
- numpy, 
- glib, 
- qt, 
- poppler, 
- zstd, 
- gdal,
-  postgis 
- pgrouting
-  icu4c

Pour rechercher l'aide de osm2pgrouting taper: 

```shell
osm2pgrouting --help
```

La réponse du système est:

```shell
Allowed options:

Help:
  --help                                Produce help message for this version.
  -v [ --version ]                      Print version string

General:
  -f [ --file ] arg                     REQUIRED: Name of the osm file.
  -c [ --conf ] arg (=/usr/share/osm2pgrouting/mapconfig.xml)
                                        Name of the configuration xml file.
  --schema arg                          Database schema to put tables.
                                          blank: defaults to default schema
                                                dictated by PostgreSQL
                                                search_path.
  --prefix arg                          Prefix added at the beginning of the
                                        table names.
  --suffix arg                          Suffix added at the end of the table
                                        names.
  --addnodes                            Import the osm_nodes, osm_ways &
                                        osm_relations tables.
  --attributes                          Include attributes information.
  --tags                                Include tag information.
  --chunk arg (=20000)                  Exporting chunk size.
  --clean                               Drop previously created tables.
  --no-index                            Do not create indexes (Use when indexes
                                        are already created)

Database options:
  -d [ --dbname ] arg                   Name of your database (Required).
  -U [ --username ] arg                 Name of the user, which have write
                                        access to the database.
  -h [ --host ] arg (=localhost)        Host of your postgresql database.
  -p [ --port ] arg (=5432)             db_port of your database.
  -W [ --password ] arg                 Password for database access.
```







## Le Routing

Les services de *routing* (dits de **navigation** ou de **calcul d'itinéraires**) aident les personnes pour se rendre d'un lieu à un autre. Les données  d'OpenStreetMap comprennent des informations pour la planification d'itinéraire suivant différents modes comprenant l'automobile, la marche à pied, le vélo ou le cheval. Il y a différentes types [hors ligne](https://wiki.openstreetmap.org/wiki/Routing/OfflineRouters) et [en ligne sur le web](https://wiki.openstreetmap.org/wiki/Routing/OnlineRouters) de services de navigation basés sur les données d'OpenStreetMap. 



### Les algorithmes 

De nombreux algorithmes de recherche de chemin dans un graphe existent, et leur utilisationne se limite pas à du calcul d'itinéraire routier. Une application fréquente de ces algorithmes est parexemple le routage dans un réseau informatique, pour sélectionner les meilleurs chemins permettantd'acheminer des données. Plusieurs algorithmes courants de recherche du plus court chemin dans ungraphe sont présentés ci-après.

#### A* (A star)

L'algorithme A* est un algorithme de recherche de chemin dans un [graphe](https://fr.wikipedia.org/wiki/Graphe_(théorie_des_graphes)) entre un [nœud](https://fr.wikipedia.org/wiki/Nœud_(mathématiques)) initial et un nœud final. Il utilise une évaluation [heuristique](https://fr.wikipedia.org/wiki/Heuristique_(mathématiques)) sur chaque nœud pour estimer le meilleur chemin y passant, et visite  ensuite les nœuds par ordre de cette évaluation heuristique. C'est un algorithme simple, ne nécessitant pas de prétraitement, et ne consommant que peu de mémoire.

### Dijkstra

L'algorithme de Dijkstra permet de résoudre un [problème algorithmique](https://fr.wikipedia.org/wiki/Problème_algorithmique) : le [problème du plus court chemin](https://fr.wikipedia.org/wiki/Problème_du_plus_court_chemin). Ce problème a plusieurs variantes. La plus simple est la suivante : étant donné un [graphe non-orienté](https://fr.wikipedia.org/wiki/Graphe_non-orienté), dont les arêtes sont munies de poids, et deux sommets de ce graphe, trouver un [chemin](https://fr.wikipedia.org/wiki/Chaîne_(théorie_des_graphes))
entre les deux sommets dans le graphe, de poids minimum. L'algorithme 
de Dijkstra permet de résoudre un problème plus général : le graphe peut
être [orienté](https://fr.wikipedia.org/wiki/Graphe_orienté),
et l'on peut désigner un unique sommet, et demander d'avoir la liste 
des plus courts chemins pour tous les autres nœuds du graphe.     

### Open Source Routing Machine



voir :

-  http://project-osrm.org/
- https://github.com/Project-OSRM/
- https://wiki.openstreetmap.org/wiki/FR:Open_Source_Routing_Machine



Open Source Routing Machine (OSRM) calcule les plus courts trajets d'un graphe routier. Il a été conçu pour fonctionner avec les données  cartographiques du projet OpenStreetMap.

Contrairement à la plupart des serveurs de routage, OSRM n'utilise pas l'algorithme [A*](http://en.wikipedia.org/wiki/A*_search_algorithm)  pour calculer le plus court chemin, mais [ The contraction hierarchies ](http://en.wikipedia.org/wiki/Contraction_hierarchies).  Il en résulte des temps de requêtes très rapides, généralement  inférieure à 1 ms pour les ensembles de données comme l'Europe, faisant  OSRM un bon candidat pour les applications et sites sensibles routage  sur le Web. 

Premiere chose à faire pour tester OSRM il faut télécharger des données de open street map au format pdf avec la commande suivante:

```shell
wget http://download.geofabrik.de/europe/france-latest.osm.pbf
```

OSRM ayant était conteneurisée avec docker il faut initialiser un conteneur docker avec la commande suivante:

```shell
docker run -t -v "${PWD}:/data" osrm/osrm-backend osrm-extract -p /opt/car.lua /data/france-latest.osm.pbf
```

RAM consommée: 1010 MBytes

Apres l'extraction on exécute:

```shell
docker run -t -v "${PWD}:/data" osrm/osrm-backend osrm-partition /data/france-latest.osrm
```

RAM consommée: 264 MBytes

Ensuite:

```shell
docker run -t -v "${PWD}:/data" osrm/osrm-backend osrm-customize /data/france-latest.osrm
```

RAM consommée: 255 MBytes 

Il faut ensuite lancer une instace du contener docker avec le serveur:

```shell
docker run -t -i -p 5000:5000 -v "${PWD}:/data" osrm/osrm-backend osrm-routed --algorithm mld  /data/france-latest.osrm
```

Une fois le serveur en fonction on peut tester une requête GET avec PostMan:

```shell
http://localhost:5000/route/v1/driving/5.9501,43.1194;5.9655,43.113?steps=true&geometries=geojson
```

!image-20190515113221813](/Users/avelferonpapel/Library/Application Support/typora-user-images/image-20190515113221813.png)

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
                    ],
                    [
                        5.951282,
                        43.118975
                    ],
                    [
                        5.952054,
                        43.118917
                    ],
                    [
                        5.953421,
                        43.118699
                    ],
                    [
                        5.955102,
                        43.117819
                    ],
                    [
                        5.955988,
                        43.117256
                    ],
                    [
                        5.95625,
                        43.117
                    ],
                    [
                        5.956516,
                        43.116543
                    ],
                    [
                        5.957094,
                        43.11611
                    ],
                    [
                        5.957015,
                        43.115929
                    ],
                    [
                        5.957507,
                        43.115347
                    ],
                    [
                        5.958067,
                        43.114262
                    ],
                    [
                        5.957922,
                        43.113757
                    ],
                    [
                        5.958561,
                        43.113748
                    ],
                    [
                        5.959566,
                        43.113861
                    ],
                    [
                        5.961582,
                        43.113535
                    ],
                    [
                        5.96319,
                        43.113596
                    ],
                    [
                        5.964858,
                        43.113276
                    ],
                    [
                        5.96552,
                        43.113024
                    ]
                ],
                "type": "LineString"
            },
            "legs": [
                {
                    "steps": [
                        {
                            "intersections": [
                                {
                                    "out": 0,
                                    "entry": [
                                        true
                                    ],
                                    "bearings": [
                                        137
                                    ],
                                    "location": [
                                        5.950025,
                                        43.119348
                                    ]
                                }
                            ],
                            "driving_side": "right",
                            "geometry": {
                                "coordinates": [
                                    [
                                        5.950025,
                                        43.119348
                                    ],
                                    [
                                        5.95009,
                                        43.119298
                                    ],
                                    [
                                        5.950102,
                                        43.119252
                                    ]
                                ],
                                "type": "LineString"
                            },
                            "mode": "driving",
                            "duration": 3.2,
                            "maneuver": {
                                "bearing_after": 137,
                                "type": "depart",
                                "modifier": "left",
                                "bearing_before": 0,
                                "location": [
                                    5.950025,
                                    43.119348
                                ]
                            },
                            "weight": 3.2,
                            "distance": 12.9,
                            "name": "Boulevard Glassendi"
                        },
                        {
                            "intersections": [
                                {
                                    "out": 0,
                                    "location": [
                                        5.950102,
                                        43.119252
                                    ],
                                    "bearings": [
                                        75,
                                        165,
                                        225,
                                        315
                                    ],
                                    "entry": [
                                        true,
                                        true,
                                        true,
                                        false
                                    ],
                                    "in": 3
                                }
                            ],
                            "driving_side": "right",
                            "geometry": {
                                "coordinates": [
                                    [
                                        5.950102,
                                        43.119252
                                    ],
                                    [
                                        5.950346,
                                        43.119312
                                    ]
                                ],
                                "type": "LineString"
                            },
                            "mode": "driving",
                            "duration": 2,
                            "maneuver": {
                                "bearing_after": 70,
                                "type": "turn",
                                "modifier": "left",
                                "bearing_before": 136,
                                "location": [
                                    5.950102,
                                    43.119252
                                ]
                            },
                            "weight": 2,
                            "distance": 20.9,
                            "name": "Avenue Général Pruneau"
                        },
                        {
                            "intersections": [
                                {
                                    "out": 1,
                                    "location": [
                                        5.950346,
                                        43.119312
                                    ],
                                    "bearings": [
                                        75,
                                        120,
                                        255
                                    ],
                                    "entry": [
                                        true,
                                        true,
                                        false
                                    ],
                                    "in": 2
                                },
                                {
                                    "out": 0,
                                    "location": [
                                        5.951282,
                                        43.118975
                                    ],
                                    "bearings": [
                                        90,
                                        195,
                                        285
                                    ],
                                    "entry": [
                                        true,
                                        true,
                                        false
                                    ],
                                    "in": 2
                                },
                                {
                                    "out": 0,
                                    "location": [
                                        5.952054,
                                        43.118917
                                    ],
                                    "bearings": [
                                        105,
                                        195,
                                        270
                                    ],
                                    "entry": [
                                        true,
                                        true,
                                        false
                                    ],
                                    "in": 2
                                },
                                {
                                    "out": 0,
                                    "location": [
                                        5.95366,
                                        43.11861
                                    ],
                                    "bearings": [
                                        135,
                                        210,
                                        300
                                    ],
                                    "entry": [
                                        true,
                                        true,
                                        false
                                    ],
                                    "in": 2
                                },
                                {
                                    "out": 1,
                                    "location": [
                                        5.953932,
                                        43.118406
                                    ],
                                    "bearings": [
                                        30,
                                        120,
                                        315
                                    ],
                                    "entry": [
                                        true,
                                        true,
                                        false
                                    ],
                                    "in": 2
                                },
                                {
                                    "out": 1,
                                    "location": [
                                        5.954862,
                                        43.117953
                                    ],
                                    "bearings": [
                                        60,
                                        120,
                                        300
                                    ],
                                    "entry": [
                                        true,
                                        true,
                                        false
                                    ],
                                    "in": 2
                                },
                                {
                                    "out": 0,
                                    "location": [
                                        5.955102,
                                        43.117819
                                    ],
                                    "bearings": [
                                        135,
                                        210,
                                        300
                                    ],
                                    "entry": [
                                        true,
                                        true,
                                        false
                                    ],
                                    "in": 2
                                },
                                {
                                    "out": 1,
                                    "location": [
                                        5.95625,
                                        43.117
                                    ],
                                    "bearings": [
                                        60,
                                        165,
                                        330
                                    ],
                                    "entry": [
                                        true,
                                        true,
                                        false
                                    ],
                                    "in": 2
                                },
                                {
                                    "out": 1,
                                    "location": [
                                        5.956516,
                                        43.116543
                                    ],
                                    "bearings": [
                                        45,
                                        135,
                                        210,
                                        270,
                                        330
                                    ],
                                    "entry": [
                                        true,
                                        true,
                                        true,
                                        true,
                                        false
                                    ],
                                    "in": 4
                                }
                            ],
                            "driving_side": "right",
                            "geometry": {
                                "coordinates": [
                                    [
                                        5.950346,
                                        43.119312
                                    ],
                                    [
                                        5.950789,
                                        43.119116
                                    ],
                                    [
                                        5.951017,
                                        43.119029
                                    ],
                                    [
                                        5.951282,
                                        43.118975
                                    ],
                                    [
                                        5.952054,
                                        43.118917
                                    ],
                                    [
                                        5.953137,
                                        43.118758
                                    ],
                                    [
                                        5.953421,
                                        43.118699
                                    ],
                                    [
                                        5.95366,
                                        43.11861
                                    ],
                                    [
                                        5.953932,
                                        43.118406
                                    ],
                                    [
                                        5.954862,
                                        43.117953
                                    ],
                                    [
                                        5.955102,
                                        43.117819
                                    ],
                                    [
                                        5.95559,
                                        43.11753
                                    ],
                                    [
                                        5.955988,
                                        43.117256
                                    ],
                                    [
                                        5.95625,
                                        43.117
                                    ],
                                    [
                                        5.956347,
                                        43.116768
                                    ],
                                    [
                                        5.956516,
                                        43.116543
                                    ],
                                    [
                                        5.956675,
                                        43.116403
                                    ],
                                    [
                                        5.956817,
                                        43.116292
                                    ],
                                    [
                                        5.956933,
                                        43.116205
                                    ],
                                    [
                                        5.957094,
                                        43.11611
                                    ]
                                ],
                                "type": "LineString"
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
                                {
                                    "out": 1,
                                    "location": [
                                        5.957598,
                                        43.115197
                                    ],
                                    "bearings": [
                                        45,
                                        165,
                                        330
                                    ],
                                    "entry": [
                                        true,
                                        true,
                                        false
                                    ],
                                    "in": 2
                                },
                                {
                                    "out": 0,
                                    "location": [
                                        5.957703,
                                        43.114936
                                    ],
                                    "bearings": [
                                        150,
                                        255,
                                        345
                                    ],
                                    "entry": [
                                        true,
                                        true,
                                        false
                                    ],
                                    "in": 2
                                }
                            ],
                            "driving_side": "right",
                            "geometry": {
                                "coordinates": [
                                    [
                                        5.957094,
                                        43.11611
                                    ],
                                    [
                                        5.957036,
                                        43.116047
                                    ],
                                    [
                                        5.957014,
                                        43.115979
                                    ],
                                    [
                                        5.957015,
                                        43.115929
                                    ],
                                    [
                                        5.957044,
                                        43.11589
                                    ],
                                    [
                                        5.957093,
                                        43.115836
                                    ],
                                    [
                                        5.95718,
                                        43.115754
                                    ],
                                    [
                                        5.957377,
                                        43.115547
                                    ],
                                    [
                                        5.957507,
                                        43.115347
                                    ],
                                    [
                                        5.957598,
                                        43.115197
                                    ],
                                    [
                                        5.957703,
                                        43.114936
                                    ],
                                    [
                                        5.957755,
                                        43.114836
                                    ],
                                    [
                                        5.957895,
                                        43.114589
                                    ],
                                    [
                                        5.958026,
                                        43.114375
                                    ],
                                    [
                                        5.958067,
                                        43.114262
                                    ],
                                    [
                                        5.958068,
                                        43.114186
                                    ],
                                    [
                                        5.95805,
                                        43.114111
                                    ],
                                    [
                                        5.958002,
                                        43.113995
                                    ],
                                    [
                                        5.957922,
                                        43.113757
                                    ]
                                ],
                                "type": "LineString"
                            },
                            "mode": "driving",
                            "duration": 46.9,
                            "maneuver": {
                                "bearing_after": 209,
                                "type": "turn",
                                "modifier": "right",
                                "bearing_before": 127,
                                "location": [
                                    5.957094,
                                    43.11611
                                ]
                            },
                            "weight": 46.9,
                            "distance": 285,
                            "name": "Rue Mondet"
                        },
                        {
                            "intersections": [
                                {
                                    "out": 1,
                                    "location": [
                                        5.957922,
                                        43.113757
                                    ],
                                    "bearings": [
                                        15,
                                        105,
                                        285
                                    ],
                                    "entry": [
                                        false,
                                        true,
                                        true
                                    ],
                                    "in": 0
                                },
                                {
                                    "out": 0,
                                    "location": [
                                        5.958561,
                                        43.113748
                                    ],
                                    "bearings": [
                                        75,
                                        165,
                                        270
                                    ],
                                    "entry": [
                                        true,
                                        true,
                                        false
                                    ],
                                    "in": 2
                                },
                                {
                                    "out": 0,
                                    "location": [
                                        5.959401,
                                        43.113848
                                    ],
                                    "bearings": [
                                        90,
                                        165,
                                        255
                                    ],
                                    "entry": [
                                        true,
                                        true,
                                        false
                                    ],
                                    "in": 2
                                },
                                {
                                    "out": 1,
                                    "location": [
                                        5.960054,
                                        43.11381
                                    ],
                                    "bearings": [
                                        15,
                                        105,
                                        285
                                    ],
                                    "entry": [
                                        true,
                                        true,
                                        false
                                    ],
                                    "in": 2
                                },
                                {
                                    "out": 1,
                                    "location": [
                                        5.960855,
                                        43.113646
                                    ],
                                    "bearings": [
                                        15,
                                        105,
                                        285
                                    ],
                                    "entry": [
                                        true,
                                        true,
                                        false
                                    ],
                                    "in": 2
                                },
                                {
                                    "out": 0,
                                    "location": [
                                        5.961186,
                                        43.11359
                                    ],
                                    "bearings": [
                                        105,
                                        135,
                                        285
                                    ],
                                    "entry": [
                                        true,
                                        true,
                                        false
                                    ],
                                    "in": 2
                                },
                                {
                                    "out": 1,
                                    "location": [
                                        5.961582,
                                        43.113535
                                    ],
                                    "bearings": [
                                        15,
                                        90,
                                        285
                                    ],
                                    "entry": [
                                        true,
                                        true,
                                        false
                                    ],
                                    "in": 2
                                },
                                {
                                    "out": 0,
                                    "location": [
                                        5.962641,
                                        43.113609
                                    ],
                                    "bearings": [
                                        90,
                                        255,
                                        345
                                    ],
                                    "entry": [
                                        true,
                                        false,
                                        true
                                    ],
                                    "in": 1
                                },
                                {
                                    "out": 0,
                                    "location": [
                                        5.963855,
                                        43.113474
                                    ],
                                    "bearings": [
                                        105,
                                        195,
                                        285
                                    ],
                                    "entry": [
                                        true,
                                        true,
                                        false
                                    ],
                                    "in": 2
                                },
                                {
                                    "out": 0,
                                    "location": [
                                        5.964219,
                                        43.113395
                                    ],
                                    "bearings": [
                                        105,
                                        195,
                                        285
                                    ],
                                    "entry": [
                                        true,
                                        true,
                                        false
                                    ],
                                    "in": 2
                                },
                                {
                                    "out": 1,
                                    "location": [
                                        5.964593,
                                        43.113329
                                    ],
                                    "bearings": [
                                        0,
                                        105,
                                        195,
                                        285
                                    ],
                                    "entry": [
                                        true,
                                        true,
                                        true,
                                        false
                                    ],
                                    "in": 3
                                }
                            ],
                            "driving_side": "right",
                            "geometry": {
                                "coordinates": [
                                    [
                                        5.957922,
                                        43.113757
                                    ],
                                    [
                                        5.958118,
                                        43.113739
                                    ],
                                    [
                                        5.958327,
                                        43.11374
                                    ],
                                    [
                                        5.958561,
                                        43.113748
                                    ],
                                    [
                                        5.959401,
                                        43.113848
                                    ],
                                    [
                                        5.959566,
                                        43.113861
                                    ],
                                    [
                                        5.959716,
                                        43.11385
                                    ],
                                    [
                                        5.959869,
                                        43.113835
                                    ],
                                    [
                                        5.960054,
                                        43.11381
                                    ],
                                    [
                                        5.960318,
                                        43.113752
                                    ],
                                    [
                                        5.960855,
                                        43.113646
                                    ],
                                    [
                                        5.961186,
                                        43.11359
                                    ],
                                    [
                                        5.961582,
                                        43.113535
                                    ],
                                    [
                                        5.961836,
                                        43.113518
                                    ],
                                    [
                                        5.962077,
                                        43.113534
                                    ],
                                    [
                                        5.962368,
                                        43.11357
                                    ],
                                    [
                                        5.962641,
                                        43.113609
                                    ],
                                    [
                                        5.962938,
                                        43.113611
                                    ],
                                    [
                                        5.96319,
                                        43.113596
                                    ],
                                    [
                                        5.963523,
                                        43.113541
                                    ],
                                    [
                                        5.963855,
                                        43.113474
                                    ],
                                    [
                                        5.964219,
                                        43.113395
                                    ],
                                    [
                                        5.964593,
                                        43.113329
                                    ],
                                    [
                                        5.964858,
                                        43.113276
                                    ],
                                    [
                                        5.965046,
                                        43.113219
                                    ],
                                    [
                                        5.965217,
                                        43.113158
                                    ],
                                    [
                                        5.96552,
                                        43.113024
                                    ]
                                ],
                                "type": "LineString"
                            },
                            "mode": "driving",
                            "duration": 57,
                            "maneuver": {
                                "bearing_after": 97,
                                "type": "end of road",
                                "modifier": "left",
                                "bearing_before": 192,
                                "location": [
                                    5.957922,
                                    43.113757
                                ]
                            },
                            "weight": 57,
                            "distance": 634.9,
                            "name": "Avenue de la Résistance"
                        },
                        {
                            "intersections": [
                                {
                                    "in": 0,
                                    "entry": [
                                        true
                                    ],
                                    "bearings": [
                                        301
                                    ],
                                    "location": [
                                        5.96552,
                                        43.113024
                                    ]
                                }
                            ],
                            "driving_side": "right",
                            "geometry": {
                                "coordinates": [
                                    [
                                        5.96552,
                                        43.113024
                                    ],
                                    [
                                        5.96552,
                                        43.113024
                                    ]
                                ],
                                "type": "LineString"
                            },
                            "mode": "driving",
                            "duration": 0,
                            "maneuver": {
                                "bearing_after": 0,
                                "location": [
                                    5.96552,
                                    43.113024
                                ],
                                "bearing_before": 121,
                                "type": "arrive"
                            },
                            "weight": 0,
                            "distance": 0,
                            "name": "Avenue de la Résistance"
                        }
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





Avec la partie front end









# Installation de pgrouting



Installation postgresql

installation de postGIs

Via: 

```shel
sudo apt install postgresql
sudo apt install postgis
```

installation de pgrouting

```shell
wget -O pgrouting-2.6.0.tar.gz https://github.com/pgRouting/pgrouting/archive/v2.6.0.tar.gz
```

Une fois téléchargé:

```shell
ls
vagrant@vagrant:~$ ls
pgrouting-2.6.0.tar.gz
vagrant@vagrant:~$
# tar xvfz pgrouting-2.6.0.tar.gz
# cd pgrouting-2.6.0

mkdir build
cd build
cmake  ..
make
sudo make install


```

Si le CMAKE_CXX est inconnu:

Executer:

```she
sudo apt-get install -y build-essential
sudo apt-get install cmake \g++ \libboost-graph-dev \libcgal-dev
```







docker nominatim

Build:

```shell
docker build -t nominatim .
```

Initialisation:



```shell
docker run -t -v /Users/avelferonpapel/Downloads/nominatim-docker-master/3.3/data:/data  nominatim  sh /app/init.sh /data/merged.osm.pbf postgresdata 4
```









# Import d'un fichier CSV dans laravel

Il existe plusieurs manières de procéder pour importer des fichiers csv ou excel dans une base de données avec Laravel. Partir from scratch ou utiliser des librairies existantes.

Sur https://packagist.org/ site des librairies et extensions php de composer la librairie [maatwebsite/*excel*](https://packagist.org/packages/maatwebsite/excel)  arrive en tête avec  11 805 113  de téléchargement et 7 167 étoiles (la deuxième librairie est à 66 000 téléchargements). C'est donc naturellement le choix que ferons.

## Caractéristiques

- Exportez facilement des collections vers Excel.

- Exporter des jeux de données encore plus volumineux ==> mettre vos exportations en file d'attente afin que tout cela se passe en arrière-plan.
- Importez des classeurs et des feuilles de calcul dans des modèles Eloquent.
- Utilisez un tableau HTML dans une vue Lame et exportez-le vers Excel.

### Première étape:

Installer les package necessaire pour l'import et l'export de fichier excel poup csv

```shell
 composer require maatwebsite/excel 
```

## Deuxième étape:

Création du model`/imports/AdressesImport.php` :

```shell
php artisan make:import AdressesImport --model=Adresse 
```

Modifier les fichier `/config.app.php`

Partie provider:

```php
'providers' => [
    /*
     * Package Service Providers...
     */
    Maatwebsite\Excel\ExcelServiceProvider::class,
]

```

Partie aliases:

```php
'aliases' => [
    ...
    'Excel' => Maatwebsite\Excel\Facades\Excel::class,
]
```



## 3ème étape:

Création du fichier de configuration `/config/excel.php`

```shell
 php artisan vendor:publish --provider="Maatwebsite\Excel\ExcelServiceProvider"
```





# ADDOK



les données sont à télécharger à l'adresse suivante:

```shell
https://adresse.data.gouv.fr/data/geocode/
```

Il s'agit de la base de données REDIS de l'ensemble des données à leur dernière mise à jour

# Conteneurs Addok pour Docker

Ces images permettent de simplifier grandement la mise en place d'une instance [addok](https://github.com/addok/addok) avec les données de références diffusées par [Etalab](https://www.etalab.gouv.fr).

## 

## Guides d'installation

Les guides suivants ont été rédigés pour un environnement Linux ou Mac. Ils peuvent être adaptés pour Windows.

### 

### Pré-requis

- Au moins 6 Go de RAM disponible (à froid)
- 8 Go d'espace disque disponible (hors logs)
- [Docker CE 1.10+](https://docs.docker.com/engine/installation/)
- [Docker Compose 1.10+](https://docs.docker.com/compose/install/)
- `unzip` ou équivalent
- `wget` ou équivalent

### 

### Installer une instance avec les données de la Base Adresse Nationale en ODbL

Tout d'abord placez vous dans un dossier de travail, appelez-le par exemple `ban`.

#### 

#### Télécharger les données pré-indexées

```
wget https://adresse.data.gouv.fr/data/geocode/ban_odbl_addok-latest.zip
```

#### 

#### Décompresser l'archive

```
mkdir addok-data
unzip -d addok-data ban_odbl_addok-latest.zip
```

#### 

#### Télécharger le fichier Compose

```
wget https://raw.githubusercontent.com/etalab/addok-docker/master/docker-compose.yml
```

#### 

#### Démarrer l'instance

Suivant votre environnement, `sudo` peut être nécessaire pour les commandes suivantes.

```
# Attachée au terminal
docker-compose up

# ou en arrière-plan
docker-compose up -d
```

Suivant les performances de votre machine, l'instance mettra entre 30  secondes et 2 minutes à démarrer effectivement, le temps de charger les  données dans la mémoire vive.

- 90 secondes sur une VPS-SSD-3 OVH (2 vCPU, 8 Go)
- 50 secondes sur une VM EG-15 OVH (4 vCPU, 15 Go)

Par défaut l'instance écoute sur le port `7878`.

#### 

#### Tester l'instance

```
curl "http://localhost:7878/search?q=1+rue+de+la+paix+paris"
```

### 

### Paramètres avancés

| Nom du paramètre | Description                                                  |
| ---------------- | ------------------------------------------------------------ |
| `WORKERS`        | Nombre de workers addok à lancer. Valeur par défaut : `1`.   |
| `WORKER_TIMEOUT` | [Durée maximale allouée à un worker](http://docs.gunicorn.org/en/0.17.2/configure.html#timeout) pour effectuer une opération de géocodage. Valeur par défaut : `30`. |