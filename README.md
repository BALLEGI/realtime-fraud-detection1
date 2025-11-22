🛡️ Real-Time Fraud Detection & SIEM Pipeline

Ce projet implémente une architecture Big Data complète pour la détection de fraudes et de cyber-menaces en temps réel. Il utilise la puissance d'Apache Spark (Structured Streaming) couplé à Kafka et la stack ELK (Elasticsearch, Kibana) pour ingérer, analyser et visualiser des attaques instantanément.

Le système est capable de détecter :

💳 Fraudes Bancaires (Carding) : Analyse des montants, vélocité et géolocalisation.

🔓 Brute Force SSH : Détection de tentatives d'intrusion répétées.

🌐 Attaques Web : Identification d'injections SQL, XSS, et Path Traversal.

🏗️ Architecture du Projet

Le pipeline suit une architecture de traitement de flux ("Kappa Architecture") :

Sources de Données : Scripts de simulation (PowerShell) et Logs Système (Syslog-ng).

Message Broker : Apache Kafka centralise les flux dans des topics dédiés (syslogs, fraud_alerts).

Moteur de Traitement : Apache Spark lit les flux Kafka, applique des règles de détection (Regex, Parsing JSON), enrichit les données et agrège les métriques.

Stockage : Elasticsearch indexe les alertes de sécurité enrichies.

Visualisation : Kibana offre un tableau de bord unifié (SIEM) avec cartographie mondiale.

📂 Description des Fichiers

Voici le détail technique de chaque fichier contenu dans ce dépôt :

🛠️ Infrastructure

docker-compose.yml : Le fichier d'orchestration principal. Il déploie l'ensemble de la stack (Spark Master/Worker, Kafka, Zookeeper, Elasticsearch, Kibana, Syslog-ng) dans un réseau isolé nommé fraud-net.

syslog-ng.conf : Configuration du serveur Syslog. Il écoute sur les ports 514/601 et redirige les logs reçus directement vers le topic Kafka syslogs.

🧠 Cœur du Système

spark_fraud_detection.py : Le script PySpark principal. Il :

Lit les flux Kafka en temps réel.

Détecte les attaques via des expressions régulières (SSH, Web).

Parse les transactions JSON pour la fraude bancaire.

Prépare l'enrichissement GeoIP et écrit les résultats dans Elasticsearch.

start-detection.bat : Script d'automatisation pour Windows. Il facilite le déploiement en copiant le script Python dans le conteneur Docker, en gérant les dépendances Java (Ivy) et en lançant le spark-submit.

⚡ Simulation d'Attaques (Red Team)

generate-attack.ps1 : Simule une attaque par dictionnaire (Brute Force) sur un service SSH fictif.

generate-web-attacks.ps1 : Génère du trafic web malveillant (SQL Injection, XSS, Scanners). Utilise un encodage Base64 pour une injection fiable via Docker.

generate-carding-attack.ps1 : Simule une fraude bancaire mondiale. Génère des transactions JSON avec des montants variables et des coordonnées géographiques dispersées (Nigeria, Brésil, Russie, etc.).

📊 Interface

dashboard.ndjson : Fichier d'export Kibana contenant la configuration complète du "Unified Security Center" (Visualisations, Index Patterns, Carte).

🚀 Installation et Mise en Place

Prérequis

Docker Desktop installé et lancé.

Git installé.

Windows PowerShell (pour les scripts de simulation).

1. Cloner le projet

Ouvrez votre terminal et récupérez le code source :

git clone [https://github.com/BALLEGI/realtime-fraud-detection1](https://github.com/BALLEGI/realtime-fraud-detection1)
cd realtime-fraud-detection1


2. Démarrer l'infrastructure

Lancez les conteneurs en arrière-plan :

docker-compose up -d


⏳ Attendez environ 60 secondes que tous les services (Kafka, Elastic, Spark) soient complètement initialisés.

3. Configuration Initiale (Indispensable)

Pour que la carte géographique et les montants s'affichent, vous devez configurer Elasticsearch.

Accédez à Kibana : http://localhost:5601

Allez dans Dev Tools (icône clé à molette dans le menu de gauche).

Copiez-collez et exécutez (bouton Play) les commandes suivantes une par une :

Commande A : Créer le Pipeline de Géolocalisation

PUT /_ingest/pipeline/geoip-enrichment
{
  "description": "GeoIP enrichment for SIEM",
  "processors": [
    {
      "geoip": {
        "field": "source_ip",
        "target_field": "geoip",
        "ignore_failure": true
      }
    }
  ]
}


Commande B : Créer le Template d'Index (Mapping)

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


Commande C : Créer les Topics Kafka (Optionnel mais recommandé)
Dans un terminal PowerShell :

docker exec kafka kafka-topics --create --topic syslogs --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
docker exec kafka kafka-topics --create --topic fraud_alerts --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1


4. Importer le Tableau de Bord

Dans Kibana, allez dans Stack Management > Saved Objects.

Cliquez sur Import en haut à droite.

Sélectionnez le fichier dashboard.ndjson présent dans le dossier du projet.

Si demandé, associez la vue de données au nouvel index security_events.

🎮 Utilisation

1. Lancer le Moteur de Détection

Double-cliquez sur le fichier start-detection.bat.
Une fenêtre de commande va s'ouvrir.

✅ Attendez de voir le message : Pipeline Unifié Actif. Écriture vers l'index 'security_events'...

2. Lancer les Attaques

Ouvrez trois fenêtres PowerShell différentes et exécutez les commandes suivantes pour bombarder le système :

Fenêtre 1 : Fraude Bancaire

.\generate-carding-attack.ps1


Fenêtre 2 : Attaques Web

.\generate-web-attacks.ps1


Fenêtre 3 : Brute Force SSH

.\generate-attack.ps1


3. Observer en Temps Réel

Retournez sur Kibana et ouvrez le Dashboard "Unified Security Center".
Assurez-vous que la période de temps (en haut à droite) est réglée sur "Today" ou "Last 1 hour".

Vous verrez :

🌍 La carte s'animer avec les localisations des fraudes.

📈 Le compteur de montant de fraude augmenter.

🥧 Le graphique de répartition des types d'attaques évoluer.

👤 Auteur

Projet réalisé par [BALLEGI].
Lien du dépôt : https://github.com/BALLEGI/realtime-fraud-detection1
