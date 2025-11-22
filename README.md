🛡️ Real-Time Fraud Detection & SIEM Pipeline

Ce projet est une solution complète de détection de menaces et de fraude en temps réel. Il utilise une architecture Big Data basée sur Apache Spark, Kafka et Elasticsearch (Stack ELK) pour ingérer, analyser et visualiser des attaques de cybersécurité et des fraudes bancaires instantanément.

📑 Table des Matières

Architecture du Projet

Description des Fichiers

Prérequis

Installation et Démarrage

Utilisation et Simulation

🏗 Architecture du Projet

Le pipeline suit le pattern "Kappa Architecture" pour le traitement de flux continu :

Ingestion (Sources) :

Logs systèmes (Syslog) via syslog-ng.

Transactions financières et logs Web simulés via des scripts PowerShell.

Message Broker (Tampon) :

Apache Kafka : Centralise tous les flux de données (topics syslogs et fraud_alerts).

Zookeeper : Gestionnaire du cluster Kafka.

Processing (Cerveau) :

Apache Spark Structured Streaming : Analyse les flux en temps réel. Il détecte les modèles d'attaques (SSH Brute Force, SQL Injection, XSS) et enrichit les données (GeoIP).

Stockage & Indexation :

Elasticsearch : Base de données NoSQL optimisée pour la recherche et l'analytique.

Visualisation :

Kibana : Interface graphique pour le monitoring (Tableaux de bord, Cartes mondiales).

📂 Description des Fichiers

Voici le rôle technique de chaque fichier présent dans ce dépôt :

1. Infrastructure & Configuration

docker-compose.yml : Le fichier maître d'orchestration. Il définit et lance tous les conteneurs (Spark Master/Worker, Kafka, Zookeeper, Elasticsearch, Kibana, Syslog-ng) et configure le réseau isolé fraud-net.

syslog-ng.conf : Configuration du collecteur de logs. Il écoute sur le port 514 (UDP) et redirige automatiquement tous les logs reçus vers le topic Kafka syslogs.

start-detection.bat : Script d'automatisation pour Windows. Il :

Copie le script Python dans le conteneur Spark.

Configure les dépendances Java/Scala (Ivy).

Soumet le job (spark-submit) au cluster Spark avec les connecteurs Kafka et Elasticsearch nécessaires.

2. Logique de Traitement (Back-end)

spark_fraud_detection.py : Le cœur du système. Ce script PySpark :

Lit deux topics Kafka simultanément.

Applique des Regex pour identifier les attaques textuelles (SSH, Web).

Parse les données JSON pour les fraudes bancaires (Carding).

Unifie les données dans un format standardisé.

Écrit les résultats dans l'index Elasticsearch security_events en activant le pipeline GeoIP.

3. Simulation d'Attaques (Red Team Tools)

generate-attack.ps1 : Simule une attaque SSH Brute Force. Il génère des logs d'échec d'authentification et les injecte dans Kafka.

generate-web-attacks.ps1 : Simule des attaques Web (SQL Injection, XSS, Path Traversal, Scanners). Il utilise un encodage Base64 pour injecter des payloads complexes sans casser le shell.

generate-carding-attack.ps1 : Simule une fraude bancaire mondiale (Carding). Il génère des transactions JSON avec des montants et des coordonnées géographiques dispersées pour tester la détection de fraude financière.

4. Visualisation

dashboard.ndjson : Le fichier d'export de Kibana. Il contient la configuration complète du tableau de bord, des visualisations (Pie charts, Histogrammes) et de la carte mondiale.

⚙ Prérequis

Docker Desktop installé et en cours d'exécution (avec au moins 4Go de RAM alloués).

Git pour cloner le projet.

PowerShell (Windows) pour lancer les scripts de simulation.

🚀 Installation et Démarrage

Suivez ces étapes pour déployer le projet depuis zéro.

1. Cloner le dépôt

git clone [https://github.com/BALLEGI/realtime-fraud-detection1](https://github.com/BALLEGI/realtime-fraud-detection1)
cd realtime-fraud-detection1


2. Démarrer l'infrastructure

Lancez les conteneurs en arrière-plan :

docker-compose up -d


Attendez environ 60 secondes que tous les services (notamment Kafka et Elastic) soient prêts.

3. Configuration Initiale (Une seule fois)

A. Créer les Topics Kafka
Ouvrez un terminal et exécutez :

docker exec kafka kafka-topics --create --topic syslogs --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
docker exec kafka kafka-topics --create --topic fraud_alerts --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1


B. Configurer Elasticsearch (Pipeline GeoIP & Template)
Ouvrez Kibana (http://localhost:5601), allez dans Dev Tools et exécutez ces deux commandes (bouton Play) :

Commande 1 : Créer le pipeline de géolocalisation

PUT /_ingest/pipeline/geoip-enrichment
{
  "description": "GeoIP enrichment for SIEM",
  "processors": [
    { "geoip": { "field": "source_ip", "target_field": "geoip", "ignore_failure": true } }
  ]
}


Commande 2 : Définir le Mapping parfait

PUT _index_template/security_template
{
  "index_patterns": ["security_events*"],
  "template": {
    "mappings": {
      "properties": {
        "@timestamp": { "type": "date" },
        "geoip": { "properties": { "location": { "type": "geo_point" } } },
        "source_ip": { "type": "ip" },
        "attack_type": { "type": "keyword" },
        "transaction": { "properties": { "amount": { "type": "double" } } }
      }
    }
  }
}


C. Importer le Dashboard

Allez dans Kibana > Stack Management > Saved Objects.

Cliquez sur Import et sélectionnez le fichier dashboard.ndjson inclus dans ce dépôt.

4. Lancer le Moteur de Détection

Double-cliquez sur le fichier :
start-detection.bat
Une fenêtre console va s'ouvrir. Laissez-la ouverte, c'est votre moteur Spark qui tourne.

🎮 Utilisation et Simulation

Une fois le système lancé, ouvrez 3 fenêtres PowerShell distinctes pour simuler une cyber-guerre en temps réel.

1. Attaque Web (SQLi / XSS)

.\generate-web-attacks.ps1


2. Fraude Bancaire (Carding)

.\generate-carding-attack.ps1


3. Attaque Système (SSH)

.\generate-attack.ps1
