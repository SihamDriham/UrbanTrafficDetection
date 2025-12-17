# UrbanTrafficDetection – Big Data Pipeline
## Project Overview
Le projet vise à prévoir le volume de trafic urbain à partir de données historiques, météorologiques et temporelles en utilisant un pipeline Big Data complet basé sur Apache Spark, avec intégration dans Zeppelin et stockage des résultats dans Hive.

L’objectif est de construire un workflow scalable, distribué et reproductible, couvrant l’ingestion des données, le prétraitement, le machine learning distribué, la visualisation et le stockage des résultats.

## Pipeline Big Data
### 1. Architecture du Projet
<img width="1123" height="544" alt="Image" src="https://github.com/user-attachments/assets/d178d918-48e6-4ee6-94f7-577278a68b43" />

### 2. Installer tous les outils nécessaires

L’ensemble des outils nécessaires au pipeline Big Data ont été installé et configuré à l’aide de **Docker** et d’un fichier **`docker-compose.yaml`**, ce qui permet de déployer un environnement homogène, reproductible et facile à maintenir.

Les composants utilisés sont :

- **Apache Spark** : moteur de calcul distribué pour le traitement des données massives.
- **PySpark** : interface Python pour l’utilisation de Spark ML.
- **Apache Zeppelin** : notebook interactif pour l’exécution et la visualisation des traitements Spark.
- **HDFS (Hadoop Distributed File System)** : système de fichiers distribué pour le stockage et l’accès aux données.
- **Apache Hive** : data warehouse permettant le stockage des résultats et l’exécution de requêtes SQL sur les données distribuées.

Cette approche basée sur Docker garantit une installation rapide, portable et cohérente de l’ensemble de l’environnement Big Data, facilitant ainsi le développement et l’expérimentation du pipeline.

### 3. Choix du format de données – Parquet

Le format **Parquet** a été choisi pour le stockage des données traitées, en raison de ses avantages pour les environnements Big Data :

- **Stockage en colonnes** : permet une lecture rapide et ciblée des colonnes nécessaires.
- **Compression efficace** : réduit l’espace disque utilisé.
- **Optimisation pour Apache Spark** : améliore les performances des traitements distribués.

Le fichier final est stocké sous le format : **`traffic_volume_cleaned_encoded.parquet`**

### 4. Dataset utilisé – Metro Interstate Traffic Volume 

 Le dataset *Metro Interstate Traffic Volume* regroupe des données de trafic routier urbain, incluant le volume de circulation, les conditions météorologiques et des informations temporelles. Il est exploité pour analyser le trafic et identifier les **patterns de congestion urbaine**. 

##### Structure des données

Les données ont été collectées par le **Minnesota Department of Transportation (MnDOT)** sur la période **2012–2018**. Les principales caractéristiques du jeu de données sont :

- **holiday** : variable catégorielle indiquant la présence de jours fériés nationaux ou régionaux.
- **temp** : température moyenne exprimée en kelvin.
- **rain_1h** : quantité de pluie (en millimètres) enregistrée sur une heure.
- **snow_1h** : quantité de neige (en millimètres) enregistrée sur une heure.
- **clouds_all** : pourcentage de couverture nuageuse.
- **weather_main** : description succincte des conditions météorologiques.
- **weather_description** : description détaillée des conditions météorologiques.
- **date_time** : date et heure de la mesure (heure locale CST).
- **traffic_volume** : volume horaire du trafic routier.

### 5. Construire le pipeline complet
#### 5.1 Analyse exploratoire (EDA)

* Comprendre les colonnes disponibles.
* Identifier les valeurs manquantes.
* Vérifier les types de données (catégoriel, numérique).
* Détecter les outliers.
* Visualisation rapide via **Zeppelin** (tableaux, histogrammes).


#### 5.2 Prétraitement des données

- **Nettoyage** : suppression ou interpolation des valeurs manquantes.
- **Transformation des variables** :

  * Encodage des variables catégorielles (ex. `weather_description` → entier).
  * Normalisation ou standardisation si nécessaire.
- **Création de nouvelles features** :

  * Features temporelles cycliques : `hour_sin`, `hour_cos` pour capturer le cycle 24h.
  * Features de pointe : `is_peak_hour`, `hour_peak`, `temp_peak`.
  * Features non linéaires : `Hour_sq`, `temp_sq`, `rain_sq`.


#### 5.3 Entraîner plusieurs modèles de Machine Learning

- Les modèles distribués suivants sont entraînés sur le dataset à l’aide de **Spark ML** pour un traitement rapide sur de grands volumes de données :

  * **Decision Tree Regressor**
  * **Random Forest Regressor**
  * **Gradient-Boosted Tree (GBT) Regressor**


#### 5.4 Comparer les modèles

- Évaluation sur le jeu de test :

  * **R²** – proportion de variance expliquée
  * **RMSE** – racine de l’erreur quadratique moyenne
  * **MAE** – erreur absolue moyenne
- Comparaison des performances pour sélectionner le modèle le plus performant.


#### 5.5 Visualiser les résultats

* Avec **Zeppelin** :

  * Graphiques prédictions vs valeurs réelles
  * Histogrammes des erreurs
  * Courbes de performance

#### 5.6 Stocker les résultats dans Hive

Les performances et les prédictions sont sauvegardées dans **Hive** via **SparkSQL**, ce qui permet d’avoir un **workflow prêt pour la production**, facilement intégrable à des outils BI ou dashboards. 



## Étapes de mise en place

### 1. Lancer l’environnement Docker

Construire et démarrer les services nécessaires à l’aide de Docker Compose :

```bash
docker compose build zeppelin
docker-compose up -d
```

Cette étape permet de lancer l’ensemble des services Big Data requis : **Hadoop, Hive, Spark et Zeppelin**.


### 2. Accéder au conteneur Zeppelin

Entrer dans le conteneur Zeppelin pour effectuer les configurations nécessaires :

```bash
docker exec -it zeppelin bash
```


### 3. Supprimer les fichiers temporaires

Nettoyer les fichiers temporaires générés automatiquement (`._*`) :

```bash
find /opt/zeppelin -name "._*" -delete
```

### 4. Redémarrer le service Zeppelin

Quitter le conteneur :

```bash
exit
```

Puis redémarrer le service Zeppelin :

```bash
docker restart zeppelin
```


### 5. Créer le warehouse Hive dans HDFS

Accéder au conteneur **NameNode** :

```bash
docker exec -it namenode bash
```

Créer le répertoire Hive dans HDFS et attribuer les droits nécessaires :

```bash
hdfs dfs -mkdir -p /user/hive/warehouse
hdfs dfs -chmod -R 777 /user/hive
```

Cette étape est indispensable pour permettre à **Hive** de créer, stocker et gérer ses tables dans HDFS.


### 6. Copier le dataset vers HDFS

Créer le dossier des données brutes dans HDFS :

```bash
hdfs dfs -mkdir -p /data/raw
```

Copier le fichier CSV vers HDFS :

```bash
hdfs dfs -put /data/raw/Metro_Interstate_Traffic_Volume.csv /data/raw/
```

Les données sont désormais disponibles dans **HDFS** pour être exploitées par **Spark** et **Hive**.


### 7. Utilisation de Zeppelin

* Ouvrir un navigateur et accéder à :
  **[http://localhost:8090/](http://localhost:8090/)**
* Aller dans **Import Note**
* Glisser-déposer le fichier **`.ipynb`**
* Exécuter le notebook
* Visualiser les résultats :

  * traitement des données
  * analyses exploratoires
  * entraînement des modèles
  * visualisations et métriques


