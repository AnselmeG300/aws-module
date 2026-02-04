# 📊 RÉSUMÉ — Services de Calcul AWS

## 📚 Vue d'ensemble

Ce module couvre les **services de calcul (Compute)** d'AWS, c'est-à-dire tous les services qui permettent d'exécuter du code, des applications et des charges de travail dans le cloud.

**Durée de lecture** : 15-20 minutes  
**Niveau** : Débutant à Intermédiaire

---

## 📋 Table des matières

1. [Amazon EC2 (Elastic Compute Cloud)](#1-amazon-ec2-elastic-compute-cloud)
2. [Types d'instances EC2](#2-types-dinstances-ec2)
3. [Options de tarification EC2](#3-options-de-tarification-ec2)
4. [EC2 Auto Scaling](#4-ec2-auto-scaling)
5. [Elastic Load Balancing (ELB)](#5-elastic-load-balancing-elb)
6. [Services de messagerie](#6-services-de-messagerie)
7. [Services serverless](#7-services-serverless)
8. [Orchestration de conteneurs](#8-orchestration-de-conteneurs)
9. [Tableau de synthèse](#9-tableau-de-synthèse)
10. [Ressources officielles](#10-ressources-officielles)

---

## 1. Amazon EC2 (Elastic Compute Cloud)

### Qu'est-ce qu'EC2 ?

**EC2** est le service de **serveurs virtuels** (instances) d'AWS. C'est l'équivalent cloud d'un serveur physique dans un datacenter.

### Bénéfices clés d'EC2

| Bénéfice | Description | Exemple |
|----------|-------------|---------|
| **Élasticité** | Augmenter ou réduire la capacité en quelques minutes | Ajouter 10 serveurs pendant les soldes, puis les retirer |
| **Contrôle total** | Accès root complet au système d'exploitation | Installer n'importe quel logiciel, personnaliser la config |
| **Flexibilité** | Choix de l'OS, du type d'instance, du stockage | Windows, Linux, AMD, Intel, SSD, HDD |
| **Sécurité** | Isolation des instances, groupes de sécurité, VPC | Chaque instance dans son propre réseau virtuel |
| **Fiabilité** | SLA de 99,99%, réplication multi-AZ | Haute disponibilité garantie |
| **Pay-as-you-go** | Payer uniquement ce que vous utilisez | 1 heure d'utilisation = 1 heure facturée |

### Cas d'usage

- 🌐 Hébergement d'applications web
- 🗄️ Serveurs de bases de données
- 🎮 Serveurs de jeux
- 📊 Traitement de données (Big Data)
- 🧪 Environnements de développement/test

**📖 Documentation officielle** : [Amazon EC2](https://docs.aws.amazon.com/ec2/)

---

## 2. Types d'instances EC2

AWS propose **5 familles d'instances** optimisées pour différents cas d'usage.

### 2.1 General Purpose (Usage général)

**Familles** : T2, T3, T4g, M5, M6i, M7g

**Caractéristiques** :
- Équilibre entre calcul, mémoire et réseau
- Idéal pour débuter sur AWS
- "Burstable" (T2/T3) = Accumule des crédits CPU au repos

**Cas d'usage** :
- ✅ Applications web (WordPress, Drupal)
- ✅ Serveurs d'applications (Java, Node.js)
- ✅ Environnements de dev/test
- ✅ Petites bases de données

**Exemple** : `t3.medium` → 2 vCPU, 4 Go RAM, $0.0416/heure

### 2.2 Compute Optimized (Calcul optimisé)

**Familles** : C5, C6i, C7g

**Caractéristiques** :
- Ratio CPU/Mémoire élevé
- Processeurs les plus rapides d'AWS
- Haute performance pour le calcul

**Cas d'usage** :
- ✅ Serveurs web haute performance
- ✅ Calcul scientifique
- ✅ Encodage vidéo (batch)
- ✅ Modélisation 3D
- ✅ Serveurs de jeux (game servers)

**Exemple** : `c6i.large` → 2 vCPU, 4 Go RAM, processeur 3.5 GHz

### 2.3 Memory Optimized (Mémoire optimisée)

**Familles** : R5, R6i, X2, z1d

**Caractéristiques** :
- Grandes quantités de RAM
- Ratio Mémoire/CPU très élevé
- Idéal pour les données en mémoire

**Cas d'usage** :
- ✅ Bases de données relationnelles (MySQL, PostgreSQL)
- ✅ Bases de données NoSQL (MongoDB, Cassandra)
- ✅ Caches distribués (Redis, Memcached)
- ✅ Analyse de données en temps réel
- ✅ SAP HANA

**Exemple** : `r6i.xlarge` → 4 vCPU, 32 Go RAM

### 2.4 Accelerated Computing (Calcul accéléré)

**Familles** : P3, P4, G4, G5, Inf1

**Caractéristiques** :
- GPUs NVIDIA ou AWS Inferentia
- Calcul parallèle massif
- Spécialisé pour l'IA/ML et le graphisme

**Cas d'usage** :
- ✅ Machine Learning (entraînement de modèles)
- ✅ Deep Learning (réseaux de neurones)
- ✅ Rendu graphique 3D
- ✅ Traitement vidéo (transcodage)
- ✅ Simulation scientifique

**Exemple** : `g5.xlarge` → 4 vCPU, 16 Go RAM, 1x NVIDIA A10G GPU

### 2.5 Storage Optimized (Stockage optimisé)

**Familles** : I3, I4i, D2, H1

**Caractéristiques** :
- Disques SSD NVMe ultra-rapides
- IOPS très élevés (Input/Output Operations Per Second)
- Bande passante réseau optimisée

**Cas d'usage** :
- ✅ Bases de données NoSQL (Cassandra, MongoDB)
- ✅ Data warehouses (Redshift)
- ✅ Systèmes de fichiers distribués (HDFS)
- ✅ Traitement de logs en temps réel
- ✅ Bases de données OLTP

**Exemple** : `i4i.xlarge` → 4 vCPU, 32 Go RAM, 937 Go SSD NVMe

### 📊 Tableau comparatif des types d'instances

| Type | vCPU | RAM | Stockage | Prix/h (approx.) | Cas d'usage principal |
|------|------|-----|----------|------------------|----------------------|
| **t3.micro** (General) | 1 | 1 Go | EBS | $0.0104 | Tests, dev |
| **t3.medium** (General) | 2 | 4 Go | EBS | $0.0416 | Apps web |
| **c6i.large** (Compute) | 2 | 4 Go | EBS | $0.085 | Calcul intensif |
| **r6i.large** (Memory) | 2 | 16 Go | EBS | $0.126 | Bases de données |
| **g5.xlarge** (Accelerated) | 4 | 16 Go | EBS + GPU | $1.006 | ML/IA |
| **i4i.large** (Storage) | 2 | 16 Go | 468 Go NVMe | $0.227 | NoSQL, OLTP |

**📖 Documentation officielle** : [Types d'instances EC2](https://aws.amazon.com/ec2/instance-types/)

---

## 3. Options de tarification EC2

AWS propose **6 modèles de tarification** pour EC2, chacun adapté à un cas d'usage.

### 3.1 On-Demand (À la demande)

**Principe** : Payer à l'heure (ou à la seconde pour Linux/Windows)

**Avantages** :
- ✅ Aucun engagement
- ✅ Flexibilité totale
- ✅ Idéal pour débuter

**Inconvénients** :
- ❌ Prix le plus élevé
- ❌ Pas de remise

**Cas d'usage** :
- Applications avec pics imprévisibles
- Tests/développement
- Applications à court terme

**Prix** : $0.0416/h pour t3.medium (us-east-1)

### 3.2 Spot Instances (Instances au rabais)

**Principe** : Acheter de la capacité EC2 inutilisée à **jusqu'à 90% de réduction**

**Avantages** :
- ✅ Économies massives (50-90%)
- ✅ Parfait pour les charges de travail flexibles

**Inconvénients** :
- ❌ AWS peut récupérer l'instance avec 2 minutes de préavis
- ❌ Pas de garantie de disponibilité

**Cas d'usage** :
- Traitement batch
- Analyse de données (Big Data)
- Calcul scientifique
- Rendu d'images/vidéos
- Tests de charge

**Prix** : $0.0125/h pour t3.medium (70% de réduction vs On-Demand)

### 3.3 Reserved Instances (Instances réservées)

**Principe** : S'engager sur **1 an ou 3 ans** pour obtenir **jusqu'à 75% de réduction**

**Types** :
- **Standard Reserved** : -75%, pas de flexibilité
- **Convertible Reserved** : -54%, peut changer de type d'instance

**Avantages** :
- ✅ Économies importantes
- ✅ Capacité garantie

**Inconvénients** :
- ❌ Engagement à long terme
- ❌ Pénalités si non utilisé

**Cas d'usage** :
- Applications en production stable
- Bases de données permanentes
- Serveurs d'entreprise

**Prix** : $0.0277/h pour t3.medium (1 an, paiement initial)

### 3.4 Savings Plans (Plans d'épargne de calcul)

**Principe** : S'engager sur un **montant horaire** ($X/heure) pour **1 an ou 3 ans**

**Avantages** :
- ✅ Économies jusqu'à 72%
- ✅ Plus flexible que Reserved Instances
- ✅ S'applique à EC2, Lambda, Fargate

**Inconvénients** :
- ❌ Engagement à long terme

**Cas d'usage** :
- Charges de travail stables avec mix EC2/Lambda
- Besoin de flexibilité entre types d'instances

**Prix** : Économies de 40-72% vs On-Demand

### 3.5 Dedicated Instances (Instances dédiées)

**Principe** : Instances exécutées sur du **matériel physique dédié à votre compte**

**Avantages** :
- ✅ Isolation physique
- ✅ Conformité réglementaire

**Inconvénients** :
- ❌ Plus cher que les instances normales
- ❌ Frais supplémentaires ($2/h par région)

**Cas d'usage** :
- Exigences de conformité (HIPAA, PCI-DSS)
- Licences logicielles liées au socket/core

**Prix** : On-Demand + $2/h de frais dédiés

### 3.6 Dedicated Hosts (Hôtes dédiés)

**Principe** : **Serveur physique complet** réservé pour votre usage exclusif

**Avantages** :
- ✅ Visibilité sur les sockets, cores, host ID
- ✅ Contrôle du placement des instances
- ✅ Idéal pour licences BYOL (Bring Your Own License)

**Inconvénients** :
- ❌ Option la plus chère
- ❌ Gestion plus complexe

**Cas d'usage** :
- Licences Windows Server/SQL Server liées au serveur
- Conformité stricte (certifications gouvernementales)
- Contrôle total du matériel

**Prix** : $X/h pour le serveur entier (ex: $2.52/h pour un host m5.large)

### 📊 Tableau comparatif des options de tarification

| Option | Engagement | Économies | Flexibilité | Cas d'usage |
|--------|------------|-----------|-------------|-------------|
| **On-Demand** | Aucun | 0% | ⭐⭐⭐⭐⭐ | Tests, dev, pics imprévisibles |
| **Spot** | Aucun | 50-90% | ⭐⭐ | Batch, Big Data, interruptions OK |
| **Reserved** | 1-3 ans | 40-75% | ⭐ | Production stable |
| **Savings Plans** | 1-3 ans | 40-72% | ⭐⭐⭐ | Mix EC2/Lambda/Fargate |
| **Dedicated Instances** | Variable | Variable | ⭐⭐⭐ | Conformité réglementaire |
| **Dedicated Hosts** | 1-3 ans | Variable | ⭐ | Licences BYOL, contrôle matériel |

**📖 Documentation officielle** : [Tarification EC2](https://aws.amazon.com/ec2/pricing/)

---

## 4. EC2 Auto Scaling

### Qu'est-ce qu'EC2 Auto Scaling ?

**EC2 Auto Scaling** ajuste **automatiquement** le nombre d'instances EC2 en fonction de la demande.

### Fonctionnement

```
Charge faible → Retirer des instances → Réduire les coûts
Charge élevée → Ajouter des instances → Maintenir les performances
```

### Composants clés

1. **Launch Template (Modèle de lancement)**
   - Définit : AMI, type d'instance, security groups, user data
   - Sert de "modèle" pour créer de nouvelles instances

2. **Auto Scaling Group (Groupe Auto Scaling)**
   - Groupe logique d'instances
   - Définit : Min, Max, Desired capacity
   - Exemple : Min=2, Max=10, Desired=4

3. **Scaling Policies (Politiques de mise à l'échelle)**
   - **Target Tracking** : Maintenir une métrique cible (ex: CPU à 50%)
   - **Step Scaling** : Ajouter/retirer X instances si métrique > seuil
   - **Scheduled Scaling** : Planifier à l'avance (ex: +5 instances à 9h)

### Bénéfices

| Bénéfice | Description |
|----------|-------------|
| **Haute disponibilité** | Remplace automatiquement les instances défaillantes |
| **Optimisation des coûts** | Réduit le nombre d'instances en période creuse |
| **Performance** | Ajoute des instances en période de forte charge |
| **Elasticité** | S'adapte aux variations de trafic en temps réel |

### Exemple concret

**Scénario** : Site e-commerce

- **Heures creuses (2h-8h)** : 2 instances (charge faible)
- **Heures normales (8h-18h)** : 4 instances (charge moyenne)
- **Heures de pointe (18h-22h)** : 10 instances (charge élevée)
- **Black Friday** : 50 instances (pic exceptionnel)

**📖 Documentation officielle** : [EC2 Auto Scaling](https://docs.aws.amazon.com/autoscaling/ec2/userguide/)

---

## 5. Elastic Load Balancing (ELB)

### Qu'est-ce qu'un Load Balancer ?

Un **Load Balancer** distribue automatiquement le trafic entrant entre plusieurs instances EC2.

### Types de Load Balancers AWS

#### 5.1 Application Load Balancer (ALB)

**Niveau** : Couche 7 (HTTP/HTTPS)

**Caractéristiques** :
- Routage basé sur le contenu (URL, headers)
- Support WebSocket et HTTP/2
- Intégration avec AWS WAF
- Redirection HTTP → HTTPS

**Cas d'usage** :
- ✅ Applications web modernes
- ✅ Microservices
- ✅ API REST
- ✅ Applications conteneurisées (ECS, EKS)

**Exemple** : Router `/api/*` vers instances API, `/web/*` vers instances web

#### 5.2 Network Load Balancer (NLB)

**Niveau** : Couche 4 (TCP/UDP/TLS)

**Caractéristiques** :
- Ultra haute performance (millions de requêtes/sec)
- Latence ultra-faible (~100 microsecondes)
- IP statique par zone de disponibilité
- Supporte TCP, UDP, TLS

**Cas d'usage** :
- ✅ Applications nécessitant des performances extrêmes
- ✅ Jeux en ligne (gaming)
- ✅ IoT (Internet of Things)
- ✅ Applications financières (trading)

#### 5.3 Gateway Load Balancer (GWLB)

**Niveau** : Couche 3 (IP)

**Caractéristiques** :
- Déploiement d'appliances virtuelles tierces
- Transparent pour l'application

**Cas d'usage** :
- ✅ Firewalls tiers (Palo Alto, Fortinet)
- ✅ Systèmes de détection d'intrusion (IDS/IPS)
- ✅ Inspection du trafic réseau

#### 5.4 Classic Load Balancer (CLB)

**Statut** : ⚠️ Génération précédente (non recommandé pour nouveaux déploiements)

**Niveau** : Couche 4 et 7

**Cas d'usage** :
- Applications legacy (anciennes applications EC2-Classic)

### 📊 Tableau comparatif des Load Balancers

| Type | Couche OSI | Performance | Routage intelligent | Cas d'usage |
|------|------------|-------------|---------------------|-------------|
| **ALB** | 7 (HTTP/HTTPS) | Haute | ✅ Oui (URL, headers) | Apps web, microservices |
| **NLB** | 4 (TCP/UDP) | Ultra-haute | ❌ Non | Gaming, IoT, finance |
| **GWLB** | 3 (IP) | Haute | ❌ Non | Firewalls, IDS/IPS |
| **CLB** | 4 + 7 | Moyenne | ❌ Non | ⚠️ Legacy uniquement |

### Bénéfices des Load Balancers

- ✅ **Haute disponibilité** : Redirige le trafic si une instance tombe
- ✅ **Scalabilité** : Distribue la charge sur plusieurs instances
- ✅ **Health checks** : Détecte automatiquement les instances défaillantes
- ✅ **SSL/TLS termination** : Décharge le déchiffrement SSL des instances
- ✅ **Sticky sessions** : Maintient l'utilisateur sur la même instance

**📖 Documentation officielle** : [Elastic Load Balancing](https://docs.aws.amazon.com/elasticloadbalancing/)

---

## 6. Services de messagerie

### 6.1 Amazon SNS (Simple Notification Service)

#### Qu'est-ce que SNS ?

**SNS** est un service de **messagerie pub/sub** (publication/abonnement) qui envoie des notifications à plusieurs destinataires.

#### Fonctionnement

```
Publisher (Émetteur) → Topic SNS → Subscribers (Abonnés)
                                  ├→ Email
                                  ├→ SMS
                                  ├→ HTTP/HTTPS
                                  ├→ Lambda
                                  ├→ SQS
                                  └→ Mobile Push
```

#### Cas d'usage

- ✅ Notifications d'alertes (CloudWatch Alarms)
- ✅ Envoi d'emails en masse
- ✅ Notifications push mobiles
- ✅ Déclenchement de workflows
- ✅ Fan-out (1 message → N destinataires)

#### Exemple concret

**Scénario** : Alerte de fraude bancaire

1. Système détecte une transaction suspecte
2. Publie un message dans le topic SNS "Fraud-Alerts"
3. SNS envoie simultanément :
   - Email à l'équipe de sécurité
   - SMS au client
   - Notification push dans l'app mobile
   - Message vers une fonction Lambda (blocage de la carte)

**📖 Documentation officielle** : [Amazon SNS](https://docs.aws.amazon.com/sns/)

### 6.2 Amazon SQS (Simple Queue Service)

#### Qu'est-ce que SQS ?

**SQS** est un service de **file d'attente de messages** qui permet le **découplage** entre producteurs et consommateurs.

#### Fonctionnement

```
Producer → SQS Queue → Consumer
(Émetteur)           (Récepteur)
```

**Principe** : Les messages sont stockés dans une file jusqu'à ce qu'un consommateur les traite.

#### Types de files

1. **Standard Queue** (File standard)
   - Débit illimité
   - Ordre des messages non garanti
   - Livraison au moins une fois (peut y avoir des doublons)

2. **FIFO Queue** (First-In-First-Out)
   - Ordre des messages garanti
   - Livraison exactement une fois (pas de doublons)
   - Débit limité : 300 msg/s (3000 avec batching)

#### Cas d'usage

- ✅ Découplage de microservices
- ✅ Traitement asynchrone (envoi d'emails, génération de PDFs)
- ✅ Buffer pour absorber les pics de trafic
- ✅ Garantir le traitement de chaque message
- ✅ Intégration avec Lambda (event-driven)

#### Exemple concret

**Scénario** : Traitement de commandes e-commerce

1. Site web reçoit une commande → Message dans SQS
2. Worker 1 récupère le message → Traite la commande
3. Worker 2 récupère un autre message → Traite une autre commande
4. Pendant le Black Friday : 10 workers traitent 10 commandes en parallèle

**Avantage** : Si le worker plante, le message reste dans la queue et sera retraité.

### 📊 Comparaison SNS vs SQS

| Aspect | SNS | SQS |
|--------|-----|-----|
| **Modèle** | Pub/Sub (1 → N) | Queue (1 → 1) |
| **Durée de vie** | Message éphémère (push immédiat) | Message persistant (jusqu'à 14 jours) |
| **Consommation** | Push (SNS envoie aux abonnés) | Pull (consommateur récupère) |
| **Cas d'usage** | Notifications, alertes, fan-out | Traitement asynchrone, découplage |
| **Exemple** | Alertes CloudWatch → Email + SMS | Commande web → Worker traite en arrière-plan |

**📖 Documentation officielle** : [Amazon SQS](https://docs.aws.amazon.com/sqs/)

---

## 7. Services serverless

### Qu'est-ce que le serverless ?

**Serverless** = Vous exécutez du code **sans gérer de serveurs**. AWS gère automatiquement :
- Provisionnement des serveurs
- Scaling
- Haute disponibilité
- Patchs système
- Monitoring

**Vous ne payez que pour** :
- Le temps d'exécution (millisecondes)
- Les ressources consommées

### 7.1 AWS Lambda

#### Qu'est-ce que Lambda ?

**Lambda** est un service de **calcul serverless** qui exécute du code en réponse à des événements.

#### Fonctionnement

```
Event (Déclencheur) → Lambda Function → Résultat
```

**Déclencheurs possibles** :
- API Gateway (requête HTTP)
- S3 (upload de fichier)
- DynamoDB (insertion/modification)
- CloudWatch Events (cron)
- SNS/SQS (message)
- Alexa (commande vocale)

#### Langages supportés

- Python
- Node.js (JavaScript)
- Java
- C# (.NET)
- Go
- Ruby
- Custom runtime (n'importe quel langage via container)

#### Limites

- **Temps d'exécution max** : 15 minutes
- **Mémoire** : 128 Mo - 10 Go
- **Stockage temporaire** : 512 Mo - 10 Go (/tmp)
- **Taille du code** : 50 Mo (zippé), 250 Mo (décompressé)

#### Tarification

**Gratuit** :
- 1 million de requêtes/mois
- 400 000 Go-secondes/mois

**Payant** :
- $0.20 par million de requêtes
- $0.0000166667 par Go-seconde

**Exemple** :
- Fonction : 512 Mo RAM, s'exécute 1 seconde
- 1 million d'exécutions = $8.33

#### Cas d'usage

- ✅ APIs REST (avec API Gateway)
- ✅ Traitement de fichiers (images, vidéos, PDFs)
- ✅ Traitement de streams (IoT, logs)
- ✅ Tâches planifiées (cron jobs)
- ✅ Backends mobiles
- ✅ Chatbots (Slack, Teams)

#### Exemple concret

**Scénario** : Génération de miniatures d'images

1. Utilisateur upload une image dans S3
2. S3 déclenche une fonction Lambda
3. Lambda télécharge l'image
4. Lambda génère une miniature (resize)
5. Lambda sauvegarde la miniature dans S3

**Temps d'exécution** : 2 secondes  
**Coût** : $0.000033 par exécution

**📖 Documentation officielle** : [AWS Lambda](https://docs.aws.amazon.com/lambda/)

### 7.2 AWS Fargate

#### Qu'est-ce que Fargate ?

**Fargate** est un moteur de calcul **serverless pour conteneurs**. Vous exécutez des conteneurs Docker **sans gérer de serveurs EC2**.

#### Fonctionnement

**Sans Fargate** :
```
Vous → Gérez EC2 → Installez Docker → Exécutez conteneurs
```

**Avec Fargate** :
```
Vous → Définissez conteneur → AWS gère tout le reste
```

#### Différence avec Lambda

| Aspect | Lambda | Fargate |
|--------|--------|---------|
| **Format** | Fonction (code) | Conteneur Docker |
| **Durée max** | 15 minutes | Illimitée |
| **Cas d'usage** | Tâches courtes, événementielles | Applications long-running, microservices |
| **Stockage** | /tmp (512 Mo - 10 Go) | Volumes EFS/EBS |

#### Cas d'usage

- ✅ Microservices conteneurisés
- ✅ Applications batch long-running
- ✅ APIs REST conteneurisées
- ✅ Migration d'applications conteneurisées existantes
- ✅ CI/CD (build, test, deploy)

#### Tarification

**Payant par** :
- vCPU/heure : $0.04048/vCPU/heure
- GB RAM/heure : $0.004445/GB/heure

**Exemple** :
- Conteneur : 1 vCPU, 2 Go RAM
- Tourne 24h/24 pendant 1 mois = $35.71

**📖 Documentation officielle** : [AWS Fargate](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/AWS_Fargate.html)

---

## 8. Orchestration de conteneurs

### 8.1 Amazon ECS (Elastic Container Service)

#### Qu'est-ce qu'ECS ?

**ECS** est le service d'**orchestration de conteneurs** natif d'AWS. Il permet de **déployer, gérer et scaler** des applications conteneurisées.

#### Composants clés

1. **Task Definition** (Définition de tâche)
   - Décrit le conteneur : image Docker, CPU, RAM, ports, variables d'environnement
   - Équivalent d'un `docker-compose.yml`

2. **Task** (Tâche)
   - Instance en cours d'exécution d'une Task Definition
   - Peut contenir 1 ou plusieurs conteneurs

3. **Service**
   - Maintient un nombre défini de tasks en cours d'exécution
   - Intégration avec Load Balancer
   - Auto Scaling

4. **Cluster**
   - Groupe logique de tasks/services
   - Peut s'exécuter sur EC2 ou Fargate

#### Modes de lancement

| Mode | Gestion serveurs | Cas d'usage |
|------|------------------|-------------|
| **EC2 Launch Type** | Vous gérez les instances EC2 | Contrôle total, optimisation des coûts |
| **Fargate Launch Type** | AWS gère tout | Simplicité, serverless |

#### Cas d'usage

- ✅ Microservices
- ✅ Applications batch
- ✅ Migration de workloads conteneurisés vers AWS
- ✅ CI/CD avec conteneurs

**📖 Documentation officielle** : [Amazon ECS](https://docs.aws.amazon.com/ecs/)

### 8.2 Amazon EKS (Elastic Kubernetes Service)

#### Qu'est-ce qu'EKS ?

**EKS** est le service **Kubernetes managé** d'AWS. Kubernetes est le standard open-source pour l'orchestration de conteneurs.

#### Kubernetes vs ECS

| Aspect | ECS | EKS |
|--------|-----|-----|
| **Origine** | AWS propriétaire | Open-source (CNCF) |
| **Portabilité** | AWS uniquement | Multi-cloud, on-premise |
| **Complexité** | Simple | Complexe |
| **Écosystème** | AWS-centric | Énorme (Helm, Istio, etc.) |
| **Cas d'usage** | Apps AWS natives | Apps nécessitant Kubernetes |

#### Avantages d'EKS

- ✅ **Standard industrie** : Compétences Kubernetes transférables
- ✅ **Portabilité** : Facile de migrer vers/depuis on-premise
- ✅ **Écosystème riche** : Helm charts, Operators, Service Mesh
- ✅ **Conformité** : Certifié CNCF

#### Modes de compute

1. **EC2** : Vous gérez les worker nodes
2. **Fargate** : AWS gère les worker nodes (serverless)
3. **Hybrid** : Mix des deux

#### Cas d'usage

- ✅ Applications nécessitant Kubernetes (workloads existants)
- ✅ Stratégie multi-cloud
- ✅ Écosystème Kubernetes (Helm, Operators)
- ✅ Environnements hybrides (cloud + on-premise)

**📖 Documentation officielle** : [Amazon EKS](https://docs.aws.amazon.com/eks/)

### 📊 Comparaison ECS vs EKS

| Aspect | ECS | EKS |
|--------|-----|-----|
| **Facilité** | ⭐⭐⭐⭐⭐ Simple | ⭐⭐ Complexe |
| **Portabilité** | AWS uniquement | Multi-cloud |
| **Intégration AWS** | Native (ALB, CloudWatch, IAM) | Via add-ons |
| **Coût** | Gratuit (payez EC2/Fargate) | $0.10/h par cluster |
| **Cas d'usage** | Apps AWS natives | Apps Kubernetes existantes |
| **Compétences** | Spécifiques AWS | Transférables (Kubernetes universel) |

**📖 Documentation officielle** : [Orchestration de conteneurs](https://aws.amazon.com/containers/)

---

## 9. Tableau de synthèse

### Services de calcul AWS - Vue d'ensemble

| Service | Type | Gestion serveurs | Durée max | Cas d'usage principal |
|---------|------|------------------|-----------|----------------------|
| **EC2** | Machines virtuelles | ❌ Vous gérez | Illimitée | Applications générales, contrôle total |
| **EC2 Auto Scaling** | Automatisation | ❌ Vous gérez | Illimitée | Scalabilité automatique EC2 |
| **ELB (ALB/NLB)** | Load Balancing | ✅ AWS gère | Illimitée | Distribution de trafic |
| **Lambda** | Serverless (code) | ✅ AWS gère | 15 min | Tâches courtes, événementielles |
| **Fargate** | Serverless (conteneurs) | ✅ AWS gère | Illimitée | Conteneurs sans serveurs |
| **ECS** | Orchestration | Variable | Illimitée | Microservices AWS-natifs |
| **EKS** | Orchestration (K8s) | Variable | Illimitée | Workloads Kubernetes |
| **SNS** | Messagerie (pub/sub) | ✅ AWS gère | Instantané | Notifications, fan-out |
| **SQS** | Messagerie (queue) | ✅ AWS gère | 14 jours | Traitement asynchrone, découplage |

### Arbre de décision : Quel service choisir ?

```
┌─ Besoin de contrôle total du serveur ? ─── OUI ──→ EC2
│
├─ Application conteneurisée ?
│  ├─ Oui, avec Kubernetes ────────────────────────→ EKS
│  ├─ Oui, sans Kubernetes ────────────────────────→ ECS
│  └─ Oui, sans gérer de serveurs ─────────────────→ Fargate
│
├─ Tâche courte (<15 min) déclenchée par événement ? → Lambda
│
├─ Besoin de distribuer le trafic ? ──────────────→ ELB (ALB/NLB)
│
├─ Besoin de scalabilité automatique ? ───────────→ Auto Scaling
│
└─ Besoin de messagerie ?
   ├─ Notifications (1→N) ────────────────────────→ SNS
   └─ Traitement asynchrone (queue) ──────────────→ SQS
```

---

## 10. Ressources officielles

### 📖 Documentation AWS

1. **Amazon EC2**
   - [Documentation EC2](https://docs.aws.amazon.com/ec2/)
   - [Types d'instances](https://aws.amazon.com/ec2/instance-types/)
   - [Tarification EC2](https://aws.amazon.com/ec2/pricing/)
   - [Best Practices EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-best-practices.html)

2. **EC2 Auto Scaling**
   - [Documentation Auto Scaling](https://docs.aws.amazon.com/autoscaling/ec2/userguide/)
   - [Politiques de scaling](https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-scale-based-on-demand.html)

3. **Elastic Load Balancing**
   - [Documentation ELB](https://docs.aws.amazon.com/elasticloadbalancing/)
   - [Application Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/)
   - [Network Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/)

4. **AWS Lambda**
   - [Documentation Lambda](https://docs.aws.amazon.com/lambda/)
   - [Best Practices Lambda](https://docs.aws.amazon.com/lambda/latest/dg/best-practices.html)
   - [Lambda Pricing](https://aws.amazon.com/lambda/pricing/)

5. **Amazon SNS**
   - [Documentation SNS](https://docs.aws.amazon.com/sns/)
   - [Guide utilisateur SNS](https://docs.aws.amazon.com/sns/latest/dg/welcome.html)

6. **Amazon SQS**
   - [Documentation SQS](https://docs.aws.amazon.com/sqs/)
   - [SQS FIFO Queues](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/FIFO-queues.html)

7. **AWS Fargate**
   - [Documentation Fargate](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/AWS_Fargate.html)
   - [Fargate Pricing](https://aws.amazon.com/fargate/pricing/)

8. **Amazon ECS**
   - [Documentation ECS](https://docs.aws.amazon.com/ecs/)
   - [Best Practices ECS](https://docs.aws.amazon.com/AmazonECS/latest/bestpracticesguide/)

9. **Amazon EKS**
   - [Documentation EKS](https://docs.aws.amazon.com/eks/)
   - [Best Practices EKS](https://aws.github.io/aws-eks-best-practices/)

### 📚 Guides et whitepapers

- [AWS Well-Architected Framework - Performance Efficiency Pillar](https://docs.aws.amazon.com/wellarchitected/latest/performance-efficiency-pillar/welcome.html)
- [AWS Compute Services Overview](https://aws.amazon.com/products/compute/)
- [Optimizing your AWS Infrastructure for Sustainability](https://docs.aws.amazon.com/wellarchitected/latest/sustainability-pillar/welcome.html)

### 🎓 Formations AWS

- [AWS Training - Compute](https://aws.amazon.com/training/learn-about/compute/)
- [AWS Skill Builder - Compute Learning Plan](https://explore.skillbuilder.aws/learn/learning_plan/view/82/compute-learning-plan)
- [AWS Certified Solutions Architect - Associate](https://aws.amazon.com/certification/certified-solutions-architect-associate/)

### 🧮 Outils

- [AWS Pricing Calculator](https://calculator.aws/) - Estimer les coûts
- [EC2 Instance Comparison](https://instances.vantage.sh/) - Comparer les types d'instances
- [AWS Cost Explorer](https://aws.amazon.com/aws-cost-management/aws-cost-explorer/) - Analyser les dépenses

---

## ✅ Points clés à retenir

### EC2

- ✅ EC2 = Serveurs virtuels à la demande
- ✅ 5 familles : General Purpose, Compute, Memory, Accelerated, Storage
- ✅ 6 options de tarification : On-Demand, Spot, Reserved, Savings Plans, Dedicated
- ✅ Spot Instances = jusqu'à 90% d'économies (mais interruptible)

### Auto Scaling & Load Balancing

- ✅ Auto Scaling = Ajuste automatiquement le nombre d'instances
- ✅ ELB distribue le trafic entre plusieurs instances
- ✅ ALB = Layer 7 (HTTP/HTTPS), idéal pour apps web
- ✅ NLB = Layer 4 (TCP/UDP), ultra-performant

### Messagerie

- ✅ SNS = Pub/Sub, notifications (1 → N)
- ✅ SQS = Queue, traitement asynchrone (1 → 1)
- ✅ SNS + SQS = Pattern fan-out (1 message → plusieurs queues)

### Serverless

- ✅ Lambda = Code sans serveur, max 15 min
- ✅ Fargate = Conteneurs sans serveur, durée illimitée
- ✅ Serverless = Pas de gestion de serveurs, paiement à l'usage

### Orchestration de conteneurs

- ✅ ECS = Orchestration AWS native, simple
- ✅ EKS = Kubernetes managé, portable
- ✅ Les deux supportent EC2 et Fargate

---

**🎯 Prochaine étape** : Passez au module sur les **Services de Stockage AWS** (S3, EBS, EFS) !

---

**Durée de lecture** : 15-20 minutes  
**Dernière mise à jour** : Janvier 2026  
**Module** : 04 - Services de Calcul AWS
