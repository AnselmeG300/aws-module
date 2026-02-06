# **Résumé – Module 5 : Storage and Databases (AWS Cloud Practitioner)**

---

## **1. Les types de stockage AWS**

AWS propose **trois grands types de stockage**, chacun répondant à des besoins différents :

### 🔹 Stockage par blocs (Block Storage)

* Les données sont découpées en **blocs de taille égale**.
* Principalement utilisé avec **Amazon EC2**.
* Adapté aux systèmes d’exploitation, bases de données et applications nécessitant de hautes performances.

**Deux solutions principales :**

* **Instance Store**

  * Stockage temporaire, attaché physiquement à l’instance.
  * Les données sont perdues lorsque l’instance est arrêtée ou terminée.
  * Utilisé pour du cache ou des données temporaires.
* **Amazon EBS (Elastic Block Store)**

  * Stockage persistant attaché à une instance EC2.
  * Les données sont conservées après l’arrêt ou le redémarrage de l’instance.
  * Possibilité de créer des **snapshots** (sauvegardes incrémentales).

---

### 🔹 Stockage par objets (Object Storage)

* Chaque objet contient : **données + métadonnées + clé unique**.
* Très scalable, idéal pour des données non structurées (images, vidéos, sauvegardes).

**Service principal : Amazon S3**

* Les objets sont stockés dans des **buckets**.
* Gestion fine des permissions d’accès.
* Différentes **classes de stockage** pour optimiser les coûts.

**Classes de stockage S3 :**

* **S3 Standard** : données fréquemment utilisées.
* **S3 Standard-IA** : données rarement utilisées mais accessibles immédiatement.
* **S3 One Zone-IA** : données peu critiques stockées dans une seule AZ.
* **S3 Intelligent-Tiering** : déplacement automatique des données selon l’usage.
* **S3 Glacier Instant Retrieval** : archivage avec accès rapide.
* **S3 Glacier Flexible Retrieval** : archivage à faible coût avec délai configurable.
* **S3 Glacier Deep Archive** : archivage long terme, coût minimal, récupération lente.

---

### 🔹 Stockage par fichiers (File Storage)

* Accès simultané aux fichiers par plusieurs utilisateurs ou instances.
* Comparable à un dossier réseau partagé.

**Service principal : Amazon EFS**

* Système de fichiers managé et élastique.
* Accessible par des milliers d’instances EC2 en même temps.
* Données répliquées automatiquement sur plusieurs zones de disponibilité.

---

## **2. Les bases de données AWS**

AWS propose des bases de données **relationnelles** et **non relationnelles**, selon la structure des données et les besoins applicatifs.

---

### 🔹 Bases de données relationnelles

* Données stockées dans des **tables (lignes et colonnes)**.
* Utilisation du **SQL** pour interroger et gérer les données.
* Adaptées aux données structurées et aux transactions complexes.

**Service principal : Amazon RDS**

* Service managé pour bases relationnelles.
* Automatisation des sauvegardes, mises à jour et réplication.
* Moteurs supportés :

  * Amazon Aurora
  * MySQL
  * PostgreSQL
  * MariaDB
  * Oracle
  * Microsoft SQL Server

**Amazon Aurora**

* Base relationnelle hautes performances.
* Compatible MySQL et PostgreSQL.
* Réplication de 6 copies des données sur 3 AZ.
* Plus performante et plus scalable que les bases traditionnelles.

---

### 🔹 Bases de données non relationnelles (NoSQL)

* Données stockées sous forme **clé-valeur** ou structures flexibles.
* Très adaptées aux applications modernes, massivement distribuées.

**Service principal : Amazon DynamoDB**

* Base clé-valeur **serverless**.
* Scalabilité automatique.
* Performances constantes.
* Capable de gérer des milliards de requêtes par jour.
* Accélération possible via **DynamoDB Accelerator (DAX)**.

---

## **3. Migration et services additionnels de bases de données**

### 🔹 Migration

* **AWS Database Migration Service (DMS)**

  * Permet de migrer des bases relationnelles ou non relationnelles vers AWS.
  * Minimise les interruptions de service.

---

### 🔹 Services additionnels

* **Amazon Redshift** : entrepôt de données pour l’analytique et le Big Data.
* **Amazon DocumentDB** : base orientée documents compatible MongoDB.
* **Amazon Neptune** : base de données graphe pour données fortement connectées.
* **Amazon QLDB** : registre immuable avec historique complet des modifications.
* **Amazon Managed Blockchain** : création et gestion de réseaux blockchain.
* **Amazon ElastiCache** : cache en mémoire (Redis, Memcached) pour réduire la latence.

---

## **4. Points clés à retenir pour l’examen**

* **EBS = stockage par blocs persistant**, **Instance Store = temporaire**.
* **S3 = stockage par objets**, avec plusieurs classes selon l’usage.
* **EFS = stockage par fichiers partagé et multi-AZ**.
* **RDS / Aurora = bases relationnelles (SQL)**.
* **DynamoDB = base NoSQL clé-valeur serverless**.
* **Redshift = analytique**, **DMS = migration**.

