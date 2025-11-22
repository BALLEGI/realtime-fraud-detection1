# 🛡️ Real-Time Fraud Detection & SIEM Pipeline

Ce projet implémente une **architecture Big Data complète** pour la détection de fraudes et de cyber-menaces en temps réel.  
Il s'appuie sur **Apache Spark (Structured Streaming)**, **Kafka** et la **stack ELK (Elasticsearch, Kibana)** pour ingérer, analyser et visualiser les attaques instantanément.

---

## ✅ Fonctionnalités

Le système est capable de détecter :

- 💳 **Fraudes Bancaires (Carding)** : Analyse des montants, vélocité et géolocalisation.
- 🔓 **Brute Force SSH** : Détection de tentatives d'intrusion répétées.
- 🌐 **Attaques Web** : Identification d'injections SQL, XSS et Path Traversal.

---

## 🏗️ Architecture du Projet

Le pipeline suit une **Kappa Architecture** :

- **Sources de Données** : Scripts de simulation (PowerShell) et logs système (Syslog-ng).
- **Message Broker** : Apache Kafka centralise les flux dans des topics dédiés (`syslogs`, `fraud_alerts`).
- **Moteur de Traitement** : Apache Spark lit les flux Kafka, applique des règles de détection (Regex, Parsing JSON), enrichit les données et agrège les métriques.
- **Stockage** : Elasticsearch indexe les alertes enrichies.
- **Visualisation** : Kibana offre un tableau de bord SIEM avec cartographie mondiale.

---

## 📂 Structure des Fichiers

### 🛠️ Infrastructure
- `docker-compose.yml` : Déploie Spark, Kafka, Zookeeper, Elasticsearch, Kibana, Syslog-ng.
- `syslog-ng.conf` : Configure Syslog pour rediriger les logs vers Kafka.

### 🧠 Cœur du Système
- `spark_fraud_detection.py` : Script PySpark principal pour :
  - Lire les flux Kafka.
  - Détecter les attaques (SSH, Web) via Regex.
  - Parser les transactions JSON pour la fraude bancaire.
  - Enrichir avec GeoIP et écrire dans Elasticsearch.
- `start-detection.bat` : Automatisation du déploiement sous Windows.

### ⚡ Simulation d'Attaques
- `generate-attack.ps1` : Simule brute force SSH.
- `generate-web-attacks.ps1` : Génère des attaques Web (SQLi, XSS).
- `generate-carding-attack.ps1` : Simule des fraudes bancaires mondiales.

### 📊 Interface
- `dashboard.ndjson` : Export Kibana du tableau de bord **Unified Security Center**.

---

## 🚀 Installation et Mise en Place

### Prérequis
- Docker Desktop
- Git
- Windows PowerShell

### Étapes
1. **Cloner le projet**
   ```bash
   git clone https://github.com/BALLEGI/realtime-fraud-detection1
   cd realtime-fraud-detection1
   ```

2. **Démarrer l'infrastructure**
   ```bash
   docker-compose up -d
   ```
   ⏳ Attendre ~60s pour l'initialisation complète.

3. **Configurer Elasticsearch**
   - Accédez à Kibana : [http://localhost:5601](http://localhost:5601)
   - Allez dans **Dev Tools** et exécutez :
     - **Pipeline GeoIP**
     ```json
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
     ```
     - **Template d'Index**
     ```json
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
     ```

4. **Créer les Topics Kafka (Optionnel)**
   ```powershell
   docker exec kafka kafka-topics --create --topic syslogs --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
   docker exec kafka kafka-topics --create --topic fraud_alerts --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
   ```

5. **Importer le Dashboard**
   - Kibana → Stack Management → Saved Objects → Import → `dashboard.ndjson`.

---

## 🎮 Utilisation

1. **Lancer le moteur de détection**
   ```powershell
   start-detection.bat
   ```
   ✅ Attendez le message : *Pipeline Unifié Actif. Écriture vers 'security_events'...*

2. **Simuler des attaques**
   - Fenêtre 1 : Fraude bancaire
     ```powershell
     .\generate-carding-attack.ps1
     ```
   - Fenêtre 2 : Attaques Web
     ```powershell
     .\generate-web-attacks.ps1
     ```
   - Fenêtre 3 : Brute Force SSH
     ```powershell
     .\generate-attack.ps1
     ```

3. **Observer en temps réel**
   - Kibana → Dashboard **Unified Security Center**.
   - Période : *Today* ou *Last 1 hour*.

---

## 👤 Auteur
Projet réalisé par **[BALLEGI]**  
📌 Dépôt : [https://github.com/BALLEGI/realtime-fraud-detection1](https://github.com/BALLEGI/realtime-fraud-detection1)
