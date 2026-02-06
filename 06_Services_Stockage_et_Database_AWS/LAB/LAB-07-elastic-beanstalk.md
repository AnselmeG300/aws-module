# LAB 07 — Déploiement d'une application Docker avec AWS Elastic Beanstalk

## 🎯 Objectifs

À la fin de ce lab, vous serez capable de :
- ✅ Comprendre le fonctionnement d'**AWS Elastic Beanstalk**
- ✅ Créer un environnement Elastic Beanstalk avec **haute disponibilité**
- ✅ Déployer une application **Docker** sur Elastic Beanstalk
- ✅ Configurer un **Load Balancer**, **Auto Scaling** et **RDS**
- ✅ Comprendre toutes les options de configuration (réseau, base de données, monitoring, scaling)
- ✅ Surveiller et gérer l'application via la console Elastic Beanstalk

---

## 📋 Prérequis

- ✅ Accès à la console AWS
- ✅ Région : **N. Virginia (us-east-1)**
- ✅ Navigateur web
- ✅ Connaissance de base de Docker (optionnel)
- ✅ Application Docker prête OU utilisation de l'application exemple AWS

---

## 📚 Qu'est-ce qu'AWS Elastic Beanstalk ?

### Définition

**AWS Elastic Beanstalk** est un service **PaaS** (Platform as a Service) qui permet de **déployer et gérer des applications web sans gérer l'infrastructure**.

### Analogie Simple

Imaginez que vous voulez ouvrir un restaurant :

| Approche | Équivalent AWS | Responsabilités |
|----------|----------------|-----------------|
| **Construire le bâtiment** | EC2 (IaaS) | Vous gérez tout : serveurs, réseau, OS, sécurité |
| **Louer un local équipé** | Elastic Beanstalk (PaaS) | AWS gère l'infrastructure, vous gérez l'application |
| **Service de restauration** | Lambda (FaaS) | AWS gère tout, vous fournissez juste le code |

### Fonctionnement d'Elastic Beanstalk

```
┌─────────────────────────────────────────────┐
│         VOUS FOURNISSEZ :                   │
│         • Code de l'application             │
│         • Plateforme (Docker, Java, etc.)   │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│      ELASTIC BEANSTALK GÈRE :               │
│      • Instances EC2                        │
│      • Load Balancer (ALB)                  │
│      • Auto Scaling                         │
│      • Surveillance (CloudWatch)            │
│      • Déploiement automatique              │
│      • Mise à jour de l'OS                  │
│      • Sécurité (Security Groups)           │
└─────────────────────────────────────────────┘
                    ↓
            APPLICATION DÉPLOYÉE !
```

### Avantages d'Elastic Beanstalk

| Avantage | Explication |
|----------|-------------|
| 🚀 **Rapidité** | Déploiement en quelques clics (5-15 minutes) |
| 🎯 **Simplicité** | Pas besoin de gérer EC2, ALB, ASG manuellement |
| 💰 **Coût** | **Service gratuit** (vous payez uniquement les ressources EC2, RDS, etc.) |
| 📈 **Auto Scaling** | Ajuste automatiquement le nombre d'instances selon la charge |
| 📊 **Monitoring** | CloudWatch intégré pour surveiller l'application |
| 🔄 **Mises à jour** | Déploiement de nouvelles versions avec zero-downtime |
| 🔧 **Personnalisable** | Accès aux ressources sous-jacentes (EC2, etc.) si besoin |

### Plateformes Supportées

Elastic Beanstalk supporte de nombreux langages et plateformes :

| Langage/Plateforme | Exemples de frameworks |
|-------------------|------------------------|
| 🐳 **Docker** | Applications conteneurisées personnalisées |
| ☕ **Java** | Spring Boot, Tomcat, Java SE |
| 🐍 **Python** | Django, Flask |
| 🟢 **Node.js** | Express |
| 🐘 **PHP** | Laravel |
| 💎 **Ruby** | Rails |
| 🪟 **.NET** | ASP.NET |
| 🦦 **Go** | Applications Go |

**Pour ce lab, nous utiliserons Docker** 🐳

---

## 🏗️ Architecture créée par Elastic Beanstalk

### Mode Haute Disponibilité (Configuration de ce lab)

```
┌────────────────────────────────────────────────────────────────┐
│                   AWS Elastic Beanstalk                        │
│                                                                │
│                  Application Load Balancer (ALB)              │
│                            (Public)                            │
│                         Port 80 HTTP                           │
│                              │                                 │
│         ┌────────────────────┴────────────────────┐            │
│         ▼                                         ▼            │
│  ┌──────────────────┐                    ┌──────────────────┐ │
│  │   EC2 Instance   │                    │   EC2 Instance   │ │
│  │   (t3.micro)     │                    │   (t3.micro)     │ │
│  │   us-east-1a     │                    │   us-east-1b     │ │
│  │                  │                    │                  │ │
│  │  • Docker        │                    │  • Docker        │ │
│  │  • Nginx         │                    │  • Nginx         │ │
│  │  • Your App      │                    │  • Your App      │ │
│  └────────┬─────────┘                    └────────┬─────────┘ │
│           │                                       │           │
│           └───────────────┬───────────────────────┘           │
│                           ▼                                   │
│                   ┌──────────────────┐                        │
│                   │   RDS MySQL      │                        │
│                   │   (db.t3.small)  │                        │
│                   │   Single-AZ      │                        │
│                   └──────────────────┘                        │
│                                                                │
│  Auto Scaling Group (Min: 2, Max: 4)                          │
│  CloudWatch Monitoring + Logs                                 │
└────────────────────────────────────────────────────────────────┘
```

### Ressources créées automatiquement

À la fin du processus, Elastic Beanstalk aura créé automatiquement :

#### Ressources réseau
- ✅ 1 Application Load Balancer (ALB) public
- ✅ 1 Target Group
- ✅ 2+ Security Groups :
  - SG pour le Load Balancer (port 80 ouvert au public)
  - SG pour les instances EC2 (port 80 depuis LB uniquement)
  - SG pour RDS (port 3306 depuis instances EC2)

#### Ressources de calcul
- ✅ 1 Launch Template (configuration instance)
- ✅ 1 Auto Scaling Group
- ✅ 2 à 4 instances EC2 (t3.micro/t3.small)
- ✅ Chaque instance contient :
  - Amazon Linux 2023
  - Docker Engine
  - Nginx (reverse proxy)
  - Elastic Beanstalk HealthD agent
  - CloudWatch Logs agent

#### Ressources de base de données
- ✅ 1 instance RDS MySQL 8.4.7 (db.t3.small)
- ✅ 1 DB Subnet Group
- ✅ 20 Go de stockage SSD

#### Ressources de stockage
- ✅ 1 bucket S3 pour logs et déploiements
- ✅ Volumes EBS pour chaque instance EC2

#### Ressources de surveillance
- ✅ CloudWatch Metrics (CPU, réseau, requêtes)
- ✅ CloudWatch Alarms pour Auto Scaling
- ✅ Elastic Beanstalk Enhanced Health Monitoring

#### IAM
- ✅ 1 Service Role (aws-elasticbeanstalk-service-role)
- ✅ 1 Instance Profile (aws-elasticbeanstalk-ec2-role)

---

## 🎯 Vue d'ensemble du processus

Le processus de création d'un environnement Elastic Beanstalk se décompose en **6 étapes** :

1. **Configurer l'environnement** — Nom, plateforme, code
2. **Configurer l'accès au service** — IAM roles, EC2 key pair
3. **Configurer la mise en réseau, la base de données et les identifications** — VPC, RDS
4. **Configurer le trafic et la mise à l'échelle des instances** — Load Balancer, Auto Scaling
5. **Configurer les mises à jour, la surveillance et la journalisation** — CloudWatch, logs
6. **Vérification** — Récapitulatif final

---

## 🚀 PARTIE 1 : Créer l'environnement Elastic Beanstalk

### Étape 1.1 : Accéder au service Elastic Beanstalk

1. **Connectez-vous à la console AWS**
2. **Région** : Vérifiez que vous êtes sur **N. Virginia (us-east-1)**
3. **Recherchez "Elastic Beanstalk"** dans la barre de recherche
4. **Cliquez sur "Elastic Beanstalk"**

💡 **Remarque** : Si c'est votre première fois, la page "Environments" sera vide.

---

### Étape 1.2 : Configurer l'environnement (Étape 1/6)

#### 📋 Vue d'ensemble de l'étape 1

Cette étape permet de définir :
- Le type d'environnement (serveur web ou travail)
- Le nom de l'application et de l'environnement
- La plateforme (Docker, Java, Python, etc.)
- Le code source (exemple ou votre application)
- Le niveau de disponibilité (instance unique ou haute disponibilité)

---

#### 🎯 Niveau d'environnement

**Deux options disponibles :**

1. **Environnement de serveur web** ✅ **← NOTRE CHOIX**
   - Pour des applications web ou API web
   - Traite les requêtes HTTP/HTTPS
   - **C'est ce que nous allons sélectionner**
   
2. **Environnement de travail**
   - Pour des tâches de longue durée
   - Traite les messages d'une file SQS
   - Idéal pour les workers en arrière-plan

**💡 Explication** : Un environnement de serveur web est conçu pour répondre aux requêtes HTTP entrantes. Il inclut automatiquement un Load Balancer pour distribuer le trafic et gérer la haute disponibilité.

---

#### 📝 Informations sur l'application

**Configuration à saisir :**

| Paramètre | Valeur à saisir | Exemple | Contraintes |
|-----------|-----------------|---------|-------------|
| **Nom de l'application** | `M2i-[VOTRE_PRENOM]-EB` | `M2i-Anselme-EB` | Max 100 caractères |
| **Nom de l'environnement** | `M2i-[VOTRE_PRENOM]-EB-env` | `M2i-Anselme-EB-env` | Utilisé dans l'URL |

**💡 À savoir** :
- Le **nom de l'application** est un conteneur logique pour vos environnements
- Le **nom de l'environnement** sera dans l'URL publique : `[nom-env].us-east-1.elasticbeanstalk.com`
- Si le domaine n'est pas disponible, ajoutez un chiffre : `M2i-Anselme-EB-env-2`

---

#### 🐳 Plateforme

**Configuration à sélectionner :**

| Option | Valeur | Explications |
|--------|--------|--------------|
| **Plateforme** | `Docker` ✅ | AWS gère un environnement pour conteneurs Docker |
| **Branche de plateforme** | `Docker running on 64bit Amazon Linux 2023` ✅ | Version du système d'exploitation hôte |
| **Version de la plateforme** | `4.9.2 (Recommended)` | Version spécifique d'Elastic Beanstalk |

**💡 Pourquoi Docker ?**
- **Portabilité** : Votre application s'exécute de la même manière partout
- **Isolation** : Chaque application dans son propre conteneur
- **Flexibilité** : Déployez n'importe quel stack technologique
- **Facilité** : EB gère Docker Engine, vous fournissez juste le Dockerfile

**💡 Amazon Linux 2023** :
- Dernière génération de l'OS AWS
- Support long-terme
- Optimisé pour AWS
- Sécurité renforcée

---

#### 📦 Code de l'application

**Deux options disponibles :**

1. **Exemple d'application** ✅ **← NOTRE CHOIX**
   - Application de démonstration fournie par AWS
   - Permet de tester sans avoir de code
   - Conteneur Docker simple qui affiche une page web
   - **Idéal pour ce lab**

2. **Charger votre code**
   - Upload d'un fichier ZIP contenant :
     - Votre `Dockerfile`
     - Le code source de l'application
     - `requirements.txt` ou équivalent
   - Ou référence à un bucket S3

**💡 Pour votre propre application** : Le ZIP doit contenir un `Dockerfile` à la racine. EB construira automatiquement l'image et lancera le conteneur.

---

#### ⚙️ Préréglages (Configuration de disponibilité)

**Options disponibles :**

| Préréglage | Description | Coût | Cas d'usage |
|------------|-------------|------|-------------|
| **Instance unique (gratuit)** | 1 instance EC2 seule | ~$7/mois | Développement, tests |
| **Instance unique avec Spot** | 1 instance Spot (économique) | ~$2/mois | Dev non-critique |
| **Haute disponibilité** ✅ | ALB + 2-4 instances multi-AZ | ~$60/mois | **Production** |
| **Haute disponibilité avec Spot** | ALB + instances Spot | ~$30/mois | Production tolérante |
| **Configuration personnalisée** | Libre | Variable | Besoins spécifiques |

**✅ NOTRE CHOIX : Haute disponibilité**

**Qu'est-ce que ça inclut ?**
- ✅ **Application Load Balancer** (ALB) public
- ✅ **Auto Scaling Group** avec min 2, max 4 instances
- ✅ **Répartition multi-AZ** (us-east-1a, us-east-1b)
- ✅ **Health checks** automatiques
- ✅ **Zero-downtime deployments** (rolling updates)
- ✅ **Scaling automatique** selon la charge

**💰 Estimation coût mensuel** : ~60-70€ (EC2 + ALB + RDS)

**💡 Production-ready** : Cette configuration assure :
- **Disponibilité** : Si une AZ tombe, l'autre continue
- **Performance** : Le Load Balancer distribue le trafic
- **Scalabilité** : Ajout automatique d'instances si nécessaire
- **Résilience** : Instances remplacées automatiquement si unhealthy

---

#### ✅ Actions à réaliser

1. **Niveau d'environnement** : ✅ Cochez **"Environnement de serveur web"**
2. **Nom de l'application** : Saisissez `M2i-[VOTRE_PRENOM]-EB`
3. **Nom de l'environnement** : Saisissez `M2i-[VOTRE_PRENOM]-EB-env`
4. **Plateforme** : Sélectionnez **"Docker"**
5. **Branche de plateforme** : Sélectionnez **"Docker running on 64bit Amazon Linux 2023"**
6. **Version** : Laissez la version recommandée
7. **Code** : ✅ Cochez **"Exemple d'application"**
8. **Préréglages** : ✅ Sélectionnez **"Haute disponibilité"**
9. **Cliquez sur "Suivant"**

---

### Étape 1.3 : Configurer l'accès au service (Étape 2/6)

#### 📋 Vue d'ensemble de l'étape 2

Cette étape configure les rôles IAM nécessaires pour :
- Permettre à Elastic Beanstalk de gérer vos ressources AWS
- Permettre aux instances EC2 d'accéder aux services AWS
- (Optionnel) Permettre la connexion SSH aux instances

---

#### 🔐 Accès au service

Configuration des rôles IAM utilisés par Elastic Beanstalk.

**1) Fonction du service (Service Role)**

| Paramètre | Valeur | Explication |
|-----------|--------|-------------|
| **Rôle IAM** | `aws-elasticbeanstalk-service-role` ✅ | Rôle assumé par Elastic Beanstalk |

**💡 Qu'est-ce que c'est ?**
- Rôle assumé par le **service** Elastic Beanstalk lui-même
- Permet à EB de créer et gérer les ressources AWS pour vous

**Politiques IAM incluses** :
- ✅ **AWSElasticBeanstalkEnhancedHealth** : Monitoring avancé
- ✅ **AWSElasticBeanstalkManagedUpdatesCustomerRolePolicy** : Mises à jour gérées

**Ce que ce rôle permet à EB de faire** :
- Créer et gérer des instances EC2
- Créer et configurer le Load Balancer
- Créer et gérer l'Auto Scaling Group
- Créer des Security Groups
- Publier des métriques dans CloudWatch
- Gérer les logs
- Créer des ressources RDS

**🆕 Si c'est votre première fois** :
- Cliquez sur **"Créer et utiliser un nouveau rôle de service"**
- AWS créera automatiquement le rôle avec les bonnes permissions

---

**2) Profil d'instance EC2 (EC2 Instance Profile)**

| Paramètre | Valeur | Explication |
|-----------|--------|-------------|
| **Profil IAM** | `aws-elasticbeanstalk-ec2-role` ✅ | Rôle attaché aux instances EC2 |

**💡 Qu'est-ce que c'est ?**
- Rôle assumé par les **instances EC2** elles-mêmes
- Permet aux instances d'effectuer des opérations AWS

**Politiques IAM typiques** :
- ✅ **AWSElasticBeanstalkWebTier** : Permissions de base pour instances web
- ✅ **AWSElasticBeanstalkWorkerTier** : Pour workers (optionnel)
- ✅ **AWSElasticBeanstalkMulticontainerDocker** : Pour Docker (optionnel)

**Ce que ce rôle permet aux instances de faire** :
- **S3** : Télécharger le code de l'application depuis le bucket EB
- **S3** : Uploader les logs vers S3
- **CloudWatch Logs** : Publier les logs applicatifs
- **CloudWatch Metrics** : Publier des métriques custom
- **X-Ray** : Envoyer des traces (si activé)
- **Secrets Manager / Parameter Store** : Lire des secrets
- **DynamoDB** : Accéder aux tables (si votre app en a besoin)

**Variables d'environnement automatiques** :
```bash
AWS_DEFAULT_REGION=us-east-1
AWS_REGION=us-east-1
```

**🆕 Si c'est votre première fois** :
- Cliquez sur **"Créer un nouveau profil d'instance"**
- AWS créera automatiquement le rôle

---

**3) Paire de clés EC2 (Facultatif)**

| Paramètre | Valeur | Explication |
|-----------|--------|-------------|
| **Paire de clés** | `toto` OU `Aucune` | Pour connexion SSH (optionnel) |

**💡 Qu'est-ce que c'est ?**
- Paire de clés SSH pour se connecter aux instances EC2
- **Optionnel** : pas nécessaire pour un environnement managé par EB

**Deux options** :

1. **Sélectionner une paire existante** (ex: `toto`) ✅ **← NOTRE CHOIX (EXEMPLE)**
   - Permet la connexion SSH aux instances
   - Utile pour le débogage avancé
   - Format : `ssh -i toto.pem ec2-user@[IP-INSTANCE]`

2. **Continuer sans paire de clés EC2** ✅ **← RECOMMANDÉ POUR PROD**
   - Pas de connexion SSH possible
   - **Meilleure sécurité** (principe du moindre privilège)
   - Utilisez **AWS Systems Manager Session Manager** pour accès sécurisé si besoin

**⚠️ Sécurité** :
- En production, évitez d'utiliser des paires de clés SSH
- Utilisez plutôt **AWS Systems Manager** pour accéder aux instances
- Les logs sont disponibles via CloudWatch, pas besoin de SSH

---

#### ✅ Actions à réaliser

1. **Fonction du service** :
   - ✅ Sélectionnez **`aws-elasticbeanstalk-service-role`**
   - OU cliquez sur **"Créer un rôle"** (première fois)

2. **Profil d'instance EC2** :
   - ✅ Sélectionnez **`aws-elasticbeanstalk-ec2-role`**
   - OU cliquez sur **"Créer un rôle"** (première fois)

3. **Paire de clés EC2** :
   - ✅ Sélectionnez une paire existante (`toto` dans notre exemple)
   - OU ✅ Sélectionnez **"Continuer sans paire de clés EC2"** (recommandé)

4. **Cliquez sur "Suivant"**

---

### Étape 1.4 : Configurer réseau et base de données (Étape 3/6)

#### Configuration réseau

![Configuration réseau subnets 1](./captures/05-network-subnets-1.png)

1. **VPC** :
   - ✅ Sélectionnez le VPC par défaut : `vpc-08104c570e4699f01 (172.31.0.0/16)`

2. **Adresse IP publique** :
   - ❌ **Décochez** cette option
   - Les instances n'auront pas d'IP publique directe
   - Accès via Load Balancer uniquement (meilleure sécurité)

3. **Sous-réseaux de l'instance** :

![Sélection subnets](./captures/06-network-subnets-2.png)

   Sélectionnez **au moins 2 sous-réseaux dans des zones différentes** :
   - ✅ `us-east-1b` - subnet-06adf27d3ddd35a80 (172.31.32.0/20)
   - ✅ `us-east-1a` - subnet-08c27a0475299605c (172.31.16.0/20)
   - ℹ️ Multi-AZ pour haute disponibilité

#### Configuration base de données

![Configuration base de données](./captures/07-database-config.png)

4. **Activer la base de données** :
   - ✅ **Cochez** cette option
   - Elastic Beanstalk créera une instance RDS MySQL

5. **Sous-réseaux de la base de données** :
   - ✅ Sélectionnez les **mêmes sous-réseaux** que pour les instances
   - `us-east-1b` et `us-east-1a`

![Paramètres base de données](./captures/08-database-params.png)

6. **Paramètres de la base de données** :
   - **Moteur** : `mysql`
   - **Version du moteur** : `8.4.7` (ou dernière version)
   - **Type d'instance** : `db.t3.small`
   - **Stockage** : `20 Go`
   - **Nom d'utilisateur** : `adminadmin` (ou votre choix)
   - **Mot de passe** : Choisissez un mot de passe sécurisé
   - **Disponibilité** : `Faible (une seule AZ)` (économique)
   - **Politique de suppression** : ✅ **Supprimer**

7. **Cliquez sur "Suivant"**

📖 **Plus de détails** : Consultez la section "Étape 3" dans [Explication des configurations.md](./Explication%20des%20configurations.md#-étape-3--configuration-réseau-et-base-de-données)

---

### Étape 1.5 : Configurer trafic et mise à l'échelle (Étape 4/6)

#### Configuration instances

![Configuration instances](./captures/09-instances-config.png)

1. **Volume racine** : Laissez **(Conteneur par défaut)**

2. **Surveillance Amazon CloudWatch** :
   - **Intervalle** : `5 minute` (monitoring basique gratuit)

3. **IMDSv1** :
   - ✅ **Cochez "Désactiver"**
   - IMDSv2 uniquement (sécurité renforcée)

#### Configuration capacité (Auto Scaling)

![Capacité Auto Scaling](./captures/10-capacity-autoscaling.png)

4. **Type d'environnement** :
   - ✅ **Charge équilibrée**

5. **Nombre d'instances** :
   - **Minimum** : `2 instances`
   - **Maximum** : `4 instances`

6. **Composition de la flotte** :
   - ✅ **Instances à la demande**

7. **Architecture** :
   - ✅ **x86_64**

![Types d'instances](./captures/11-instance-types.png)

8. **Types d'instances** :
   - Ajoutez : `t3.micro`
   - Ajoutez : `t3.small`

![AMI et zones](./captures/12-ami-zones.png)

9. **ID d'AMI** : Laissez la valeur par défaut (`ami-0bc2065c25676f731`)

10. **Zones de disponibilité** : `Any` (automatique)

#### Déclencheurs de mise à l'échelle

![Scaling triggers 1](./captures/13-scaling-triggers-1.png)

11. **Métrique** : `NetworkOut`
12. **Statistique** : `Average`
13. **Unité** : `Bytes`
14. **Période** : `5 Min`
15. **Durée de la faille** : `5 Min`
16. **Seuil supérieur** : `6000000` (6 Mo)
17. **Incrément d'augmentation** : `1` Instances EC2

![Scaling triggers 2](./captures/14-scaling-triggers-2.png)

18. **Seuil inférieur** : `2000000` (2 Mo)
19. **Incrément de réduction** : `-1` Instances EC2

#### Configuration Load Balancer

![Load Balancer ALB](./captures/15-load-balancer-alb.png)

20. **Visibilité** : ✅ **Public**
21. **Double pile (IPv4 et IPv6)** : ❌ Laissez décoché
22. **Type d'équilibreur de charge** : ✅ **Application Load Balancer**
23. **Mode** : ✅ **Dédié**

![Listeners et règles](./captures/16-lb-listeners-rules.png)

24. **Écouteurs** : Port `80`, Protocole `HTTP`, Processus `default` ✅ Activé
25. **Processus** : `default`, Port `80`, HTTP
26. **Règles** : Aucune règle supplémentaire
27. **Accès aux fichiers journaux** : ❌ Laissez décoché

28. **Cliquez sur "Suivant"**

📖 **Plus de détails** : Consultez la section "Étape 4" dans [Explication des configurations.md](./Explication%20des%20configurations.md#-étape-4--configuration-trafic-et-mise-à-léchelle)

---

### Étape 1.6 : Configurer surveillance et mises à jour (Étape 5/6)

#### Surveillance

![Monitoring et logs](./captures/17-monitoring-logs.png)

1. **Création de rapports d'état** :
   - **Système** : ✅ **Amélioré** (monitoring détaillé)

2. **Métriques personnalisées CloudWatch** :
   - Laissez vide (optionnel)

3. **Personnalisation règles surveillance** :
   - ❌ Laissez décoché (configuration par défaut)

#### Mises à jour gérées

![Mises à jour gérées](./captures/18-managed-updates.png)

4. **Mises à jour gérées** :
   - ✅ **Activer**

5. **Fenêtre de mise à jour hebdomadaire** :
   - **Jour** : `Mercredi` (ou votre choix)
   - **Heure** : `06:41 UTC` (ou votre choix)

6. **Mettre à jour le niveau** :
   - ✅ **Mineur et correctif**

7. **Remplacement de l'instance** :
   - ❌ **Désactiver** (updates in-place)

#### Notifications

8. **E-mail** : `user@example.com` (ou votre email)

#### Propagation des déploiements

![Deployment propagation](./captures/19-deployment-propagation.png)

9. **Politique de déploiement** : ✅ **Propagation (Rolling)**
10. **Type de taille de lot** : ✅ **Pourcentage**
11. **Taille du lot** : `30` %
12. **Propagation du type de mise à jour** : `Désactivé`

#### Logiciel de plateforme

![Platform software](./captures/20-platform-software.png)

13. **Serveur proxy** : ✅ **Nginx**
14. **Démon X-Ray** : ❌ Désactivé
15. **Rotation des journaux S3** : ❌ Désactivé
16. **Diffusion de journaux CloudWatch** : ❌ Désactivé

#### Propriétés de l'environnement

![Environment properties](./captures/21-environment-properties.png)

17. **Propriétés** : Laissez vide (aucune variable d'environnement supplémentaire)

18. **Cliquez sur "Suivant"**

📖 **Plus de détails** : Consultez la section "Étape 5" dans [Explication des configurations.md](./Explication%20des%20configurations.md#-étape-5--surveillance-et-mises-à-jour)

---

### Étape 1.7 : Vérification et lancement (Étape 6/6)

![Page de vérification](./captures/22-verification-summary.png)

1. **Vérifiez toutes les configurations** :
   - Nom de l'application
   - Plateforme Docker
   - Réseau et base de données
   - Auto Scaling (2-4 instances)
   - Load Balancer public
   - Mises à jour gérées

2. **Cliquez sur "Suivant"** (bouton orange)

3. **Patientez pendant le déploiement** ⏱️ **5-15 minutes**

![Environnement en cours de création](./captures/23-environment-launching.png)

**Pendant le déploiement, vous verrez** :

- **État** : `Unknown` (en cours de démarrage)
- **Événements** :
  ```
  ✅ Using elasticbeanstalk-us-east-1-xxxxx as S3 storage bucket
  ✅ createEnvironment is starting
  ⏳ Creating security groups...
  ⏳ Creating Auto Scaling group...
  ⏳ Launching EC2 instances...
  ⏳ Installing Docker...
  ⏳ Deploying application...
  ⏳ Configuring Load Balancer...
  ⏳ Health checks passing...
  ✅ Successfully launched environment
  ```

💡 **Astuce** : Vous pouvez suivre la progression dans l'onglet **"Événements"**

📖 **Plus de détails** : Consultez la section "Étape 6" et "Démarrage" dans [Explication des configurations.md](./Explication%20des%20configurations.md#-étape-6--vérification)

---

## ✅ PARTIE 2 : Vérifier et tester l'application

### Étape 2.1 : Vérifier l'état de l'environnement

1. **Une fois le déploiement terminé** :
   - L'état devrait afficher : **"Health: Ok"** avec un indicateur vert ✅
   - Durée : ~5-15 minutes

2. **Informations affichées** :
   - **Nom de l'environnement** : `M2i-Anselme-EB-env`
   - **État** : `Ok`
   - **URL** : `http://m2i-anselme-eb-env.us-east-1.elasticbeanstalk.com`
   - **Plateforme** : Docker running on Amazon Linux 2023

---

### Étape 2.2 : Accéder à l'application

1. **Cliquez sur l'URL de l'environnement** (en haut de la page)

2. **Vous devriez voir l'application exemple**
   - Page de démonstration Docker
   - Message de confirmation

3. **🎉 Félicitations !** Votre environnement Elastic Beanstalk est opérationnel !

---

### Étape 2.3 : Explorer la console Elastic Beanstalk

#### Onglets disponibles

1. **Événements** :
   - Historique de tous les événements de l'environnement
   - Déploiements, scaling, erreurs

2. **État** :
   - Vue détaillée du health status
   - État de chaque instance EC2

3. **Journaux** :
   - Télécharger les logs des instances
   - Dernières 100 lignes ou logs complets

4. **Surveillance** :
   - Graphiques CloudWatch (CPU, réseau, requêtes)
   - Métriques en temps réel

5. **Alarmes** :
   - Configurer des alarmes CloudWatch

6. **Mises à jour gérées** :
   - Planification des updates automatiques

7. **Configuration** :
   - Modifier les paramètres de l'environnement

---

## 📊 PARTIE 3 : Monitoring et logs

### Étape 3.1 : Consulter les métriques

1. **Cliquez sur l'onglet "Surveillance"**

2. **Vous verrez plusieurs graphiques** :
   - **Environment health** : Santé globale
   - **Instances** : Nombre d'instances actives
   - **Requests** : Nombre de requêtes HTTP
   - **Latency** : Temps de réponse
   - **CPU utilization** : Utilisation CPU
   - **Network** : Trafic réseau

3. **Générez du trafic** :
   - Ouvrez l'URL de votre application plusieurs fois
   - Rafraîchissez la page 10-20 fois
   - Attendez 2-5 minutes
   - Revenez sur "Surveillance"
   - ✅ Vous devriez voir les graphiques avec des données

---

### Étape 3.2 : Consulter les logs

1. **Cliquez sur l'onglet "Journaux"**

2. **Cliquez sur "Demander des journaux"** → **"Dernières 100 lignes"**

3. **Attendez quelques secondes** (~10-30 secondes)

4. **Cliquez sur "Télécharger"** une fois disponible

5. **Ouvrez le fichier ZIP téléchargé**

6. **Vous trouverez plusieurs fichiers de logs** :
   ```
   logs/
   ├── eb-engine.log          → Logs du moteur Elastic Beanstalk
   ├── eb-docker/             → Logs Docker
   ├── nginx/access.log       → Logs des requêtes HTTP
   └── nginx/error.log        → Logs d'erreurs Nginx
   ```

---

## 🐳 PARTIE 4 : Déployer votre propre application Docker

### Étape 4.1 : Préparer l'application

Créez un dossier `docker-app/` avec les fichiers suivants :

**1) `app.py` (Flask)**

```python
from flask import Flask
import os

app = Flask(__name__)

@app.route('/')
def hello():
    return f"""
    <h1>Bonjour depuis Elastic Beanstalk!</h1>
    <p>Application Flask dans un conteneur Docker</p>
    <p>Environnement: {os.environ.get('ENVIRONMENT', 'production')}</p>
    """

@app.route('/health')
def health():
    return {'status': 'healthy'}, 200

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=80)
```

**2) `requirements.txt`**

```
Flask==3.0.0
```

**3) `Dockerfile`**

```Dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 80

CMD ["python", "app.py"]
```

**4) `.dockerignore`** (optionnel)

```
__pycache__
*.pyc
.git
.env
```

---

### Étape 4.2 : Tester localement (optionnel)

```bash
# Construire l'image
docker build -t my-flask-app .

# Lancer le conteneur
docker run -p 8080:80 my-flask-app

# Tester dans le navigateur
# Ouvrir http://localhost:8080
```

---

### Étape 4.3 : Déployer sur Elastic Beanstalk

**Méthode 1 : Via la console**

1. **Créez un fichier ZIP** :
   ```bash
   zip -r docker-app.zip app.py requirements.txt Dockerfile .dockerignore
   ```

2. **Dans la console Elastic Beanstalk** :
   - Cliquez sur votre environnement
   - Cliquez sur **"Charger et déployer"**
   - Sélectionnez le fichier `docker-app.zip`
   - Cliquez sur **"Déployer"**

3. **Attendez le déploiement** (~5 minutes)

4. **Testez l'application** via l'URL

**Méthode 2 : Via EB CLI** (recommandé)

```bash
# Installer EB CLI
pip install awsebcli

# Initialiser EB dans le dossier
cd docker-app/
eb init -p docker my-app --region us-east-1

# Déployer sur l'environnement existant
eb use M2i-Anselme-EB-env
eb deploy

# Ouvrir l'application
eb open
```

---

## 🔄 PARTIE 5 : Mettre à jour l'application

### Modifier le code

1. **Modifiez `app.py`** :

```python
@app.route('/')
def hello():
    return """
    <h1>🎉 Application mise à jour!</h1>
    <p>Version 2.0</p>
    """
```

2. **Déployez la mise à jour** :

**Via console** : Créez un nouveau ZIP et uploadez

**Via EB CLI** :
```bash
eb deploy
```

3. **Le déploiement se fait en rolling** (batch de 30%) :
   ```
   Instance 1 → Mise à jour → OK
   Instance 2 → Mise à jour → OK
   (etc.)
   ```

4. **Pas de downtime !** 🚀

---

## 🧹 PARTIE 6 : Nettoyage des ressources

### Supprimer l'environnement

**Via console** :
1. Cliquez sur l'environnement
2. **Actions** → **Supprimer l'environnement**
3. Confirmez la suppression
4. Attendez ~5 minutes

**Via EB CLI** :
```bash
eb terminate M2i-Anselme-EB-env
```

**Ressources supprimées automatiquement** :
- ✅ Load Balancer
- ✅ Auto Scaling Group
- ✅ Instances EC2
- ✅ Instance RDS (base de données) ⚠️
- ✅ Security Groups

⚠️ **Attention** : La base de données sera **supprimée définitivement** !

---

## 💰 Estimation des coûts

| Ressource | Quantité | Coût/heure | Coût/mois |
|-----------|----------|------------|-----------|
| EC2 t3.micro | 2 | $0.0104 | ~$15 |
| Application Load Balancer | 1 | - | ~$20 |
| RDS db.t3.small | 1 | $0.034 | ~$25 |
| Volumes EBS | 4 × 8 Go | - | ~$3 |
| **Total** | | | **~$63/mois** |

💡 **Elastic Beanstalk est gratuit** - vous ne payez que les ressources (EC2, RDS, ALB)

---

## 🎯 Points clés à retenir

✅ **Elastic Beanstalk** = Déploiement PaaS simplifié  
✅ **Haute disponibilité** = Multi-AZ + Load Balancer + Auto Scaling  
✅ **Rolling deployments** = Mises à jour sans downtime  
✅ **Monitoring intégré** = CloudWatch + Enhanced Health  
✅ **Docker support** = Déployez n'importe quelle application conteneurisée  

---

## 📚 Ressources additionnelles

- 📄 [Explication des configurations.md](./Explication%20des%20configurations.md)
- 📸 [Captures d'écran](./captures/)
- 📖 [Documentation AWS Elastic Beanstalk](https://docs.aws.amazon.com/elasticbeanstalk/)
- 🐳 [Guide Docker sur EB](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_docker.html)

---

**🎉 Félicitations !** Vous maîtrisez maintenant AWS Elastic Beanstalk avec Docker !


### Étape 3.3 : Vérifier la Configuration

1. **Menu gauche → "Configuration"**

2. **Explorez les sections** :

   **Software** :
   - Platform : Java 17 (Corretto)
   - Container : Tomcat 10
   - Proxy server : Nginx

   **Instances** :
   - Instance type : t3.micro (1 vCPU, 1 GB RAM)
   - AMI : Amazon Linux 2023
   - Root volume : 10 GB (gp3)

   **Capacity** :
   - Environment type : Single instance
   - Auto Scaling : Désactivé (mode single instance)

   **Load balancer** :
   - Non applicable (single instance n'utilise pas de Load Balancer)

   **Security** :
   - Service role : aws-elasticbeanstalk-service-role
   - EC2 instance profile : aws-elasticbeanstalk-ec2-role

   **Monitoring** :
   - Health reporting : Enhanced
   - System : Basic

   **Notifications** :
   - Email : (optionnel)

---

## 🔄 PARTIE 4 : Déployer une Nouvelle Version (Optionnel)

**Cette section est optionnelle mais très instructive.**

### Étape 4.1 : Créer une Application Java Simple

**Option 1 : Télécharger l'application exemple AWS**

1. **Téléchargez l'application** :
   - [Sample Java Application](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/samples/java-tomcat-v3.zip)

2. **Décompressez le ZIP**

3. **Modifiez le fichier `index.jsp`** :
   ```jsp
   <h1>Bonjour de la part de [VOTRE_PRENOM] !</h1>
   <p>Déployé avec AWS Elastic Beanstalk</p>
   ```

4. **Recompressez en ZIP** : `my-java-app-v2.zip`

**Option 2 : Utiliser une application toute prête**

- Nous allons simplement redéployer l'application exemple pour voir le processus

---

### Étape 4.2 : Déployer la Nouvelle Version

1. **Elastic Beanstalk → Votre environnement**

2. **Cliquez sur "Upload and deploy"** (bouton en haut à droite)

3. **Cliquez sur "Choose file"**
   - Sélectionnez `my-java-app-v2.zip` (ou utilisez l'application exemple AWS)

4. **Version label** : `v2-personnalise`

5. **Cliquez sur "Deploy"**

6. **Attendez le déploiement** (~2-3 minutes)

**Vous verrez** :
```
✅ Uploading application version...
✅ Deploying new version...
⏳ Restarting application...
✅ Successfully deployed v2-personnalise
```

7. **Rafraîchissez l'URL de votre application**
   - ✅ Vous devriez voir les changements (si vous avez modifié le code)

---

## 🔍 PARTIE 5 : Comprendre les Ressources Créées

### Étape 5.1 : Vérifier les Ressources EC2

1. **EC2 → Instances**

2. **Vous verrez une instance** :
   - Nom : `M2i-Hannah-JavaApp-env` (ou similaire)
   - Type : t3.micro
   - État : Running
   - Public IP : Adresse IP publique
   - Security Group : `awseb-...`

3. **Cliquez sur l'instance → Onglet "Security"**

4. **Cliquez sur le Security Group**

5. **Onglet "Inbound rules"** :
   - ✅ Port 80 (HTTP) : `0.0.0.0/0` (accessible depuis Internet)
   - ℹ️ Elastic Beanstalk a créé ce Security Group automatiquement

---

### Étape 5.2 : Vérifier les Rôles IAM

1. **IAM → Roles**

2. **Recherchez "elasticbeanstalk"**

3. **Vous verrez 2 rôles** :

   **aws-elasticbeanstalk-service-role** :
   - Utilisé par Elastic Beanstalk pour gérer les ressources
   - Permissions : Créer EC2, CloudWatch, etc.

   **aws-elasticbeanstalk-ec2-role** :
   - Utilisé par l'instance EC2
   - Permissions : Écrire dans CloudWatch Logs, S3, etc.

---

### Étape 5.3 : Vérifier CloudWatch

1. **CloudWatch → Log groups**

2. **Recherchez votre environnement** :
   - `/aws/elasticbeanstalk/M2i-Hannah-JavaApp-env/var/log/...`

3. **Cliquez sur un log group → Log streams**

4. **Vous verrez les logs en temps réel** ✅

---

## 📖 PARTIE 6 : Concepts Clés

### Qu'est-ce que Elastic Beanstalk a créé pour nous ?

| Ressource | Description | Géré par |
|-----------|-------------|----------|
| **EC2 Instance** | Serveur virtuel qui exécute l'application | Elastic Beanstalk |
| **Security Group** | Firewall autorisant le port 80 | Elastic Beanstalk |
| **IAM Roles** | Permissions pour l'instance et Beanstalk | Elastic Beanstalk |
| **CloudWatch Logs** | Stockage des logs | Elastic Beanstalk |
| **CloudWatch Metrics** | Métriques (CPU, RAM, etc.) | Elastic Beanstalk |
| **Elastic IP** | IP publique (mode single instance) | Elastic Beanstalk |

**Total : ~6-7 ressources créées automatiquement en 5 minutes !**

---

### Comparaison : Elastic Beanstalk vs EC2 Manuel

| Tâche | EC2 Manuel | Elastic Beanstalk |
|-------|------------|-------------------|
| Créer une instance EC2 | ✋ Manuel | ✅ Automatique |
| Installer Java | ✋ SSH + apt/yum install | ✅ Automatique |
| Installer Tomcat | ✋ Télécharger + configurer | ✅ Automatique |
| Configurer Security Group | ✋ Créer manuellement | ✅ Automatique |
| Déployer l'application | ✋ SCP + copier .war | ✅ Upload ZIP |
| Configurer CloudWatch | ✋ Installer agent | ✅ Automatique |
| Mises à jour Java/Tomcat | ✋ Manuelles | ✅ Automatiques |
| Scaling | ✋ Créer ASG manuellement | ✅ Cliquer sur "Edit capacity" |
| **Temps total** | ⏱️ 1-2 heures | ⏱️ 5-10 minutes |

---

### Cas d'Usage d'Elastic Beanstalk

| Scénario | Beanstalk adapté ? | Raison |
|----------|-------------------|--------|
| **Application web Java** | ✅ Oui | Parfait pour des apps Spring, Tomcat |
| **API REST** | ✅ Oui | Déploiement simple d'APIs |
| **Site WordPress** | ⚠️ Possible | Préférer Lightsail ou EC2 |
| **Application avec base de données** | ✅ Oui | Peut se connecter à RDS |
| **Microservices** | ⚠️ Complexe | Préférer ECS ou EKS |
| **Fonction serverless** | ❌ Non | Utiliser Lambda |
| **Infrastructure custom** | ❌ Non | Utiliser EC2 |

---

## 🧹 PARTIE 7 : Nettoyage — ⚠️ TRÈS IMPORTANT

### ⚠️ IMPORTANT : Supprimez toujours vos ressources après le lab !

Elastic Beanstalk crée des ressources EC2 qui **consomment votre Free Tier**.

---

### Étape 7.1 : Supprimer l'Environnement

1. **Elastic Beanstalk → Environments**

2. **Sélectionnez votre environnement** : `M2i-[VOTRE_PRENOM]-JavaApp-env`

3. **Actions → Terminate environment**

4. **Confirmez** en tapant le nom de l'environnement

5. **Cliquez sur "Terminate"**

6. **Attendez 5-10 minutes** que l'environnement soit supprimé

**Ce qui sera supprimé automatiquement** :
- ✅ Instance EC2
- ✅ Security Group
- ✅ Elastic IP
- ✅ CloudWatch Logs (optionnel)
- ✅ Application versions (optionnel)

---

### Étape 7.2 : Supprimer l'Application

1. **Elastic Beanstalk → Applications**

2. **Sélectionnez votre application** : `M2i-[VOTRE_PRENOM]-JavaApp`

3. **Actions → Delete application**

4. **Cochez "Delete versions" et "Delete logs"**

5. **Confirmez**

---

### Étape 7.3 : Vérifier la Suppression

1. **EC2 → Instances**
   - ✅ Vérifiez qu'aucune instance Elastic Beanstalk ne tourne

2. **EC2 → Security Groups**
   - ✅ Les Security Groups `awseb-...` devraient être supprimés automatiquement

3. **CloudWatch → Log groups**
   - ✅ Les logs Elastic Beanstalk peuvent rester (pas de coût)
   - Si vous voulez les supprimer : Sélectionnez → Actions → Delete

---

## ✅ Validation du Lab

### Questions de Compréhension

1. **Qu'est-ce qu'AWS Elastic Beanstalk ?**
   - ✅ Réponse : Un service PaaS qui déploie et gère automatiquement l'infrastructure pour vos applications

2. **Quelle est la différence entre PaaS et IaaS ?**
   - ✅ PaaS (Beanstalk) : AWS gère l'infrastructure, vous gérez l'application
   - ✅ IaaS (EC2) : Vous gérez tout (OS, runtime, application)

3. **Quelles ressources AWS sont créées automatiquement par Beanstalk ?**
   - ✅ Réponse : EC2, Security Groups, IAM Roles, CloudWatch Logs/Metrics

4. **Payez-vous pour Elastic Beanstalk ?**
   - ✅ Réponse : Non, le service Beanstalk est **gratuit**. Vous payez uniquement les ressources (EC2, etc.)

5. **Quelle est la différence entre "Single instance" et "Load balanced" ?**
   - ✅ Single instance : 1 instance EC2, pas de Load Balancer (gratuit Free Tier)
   - ✅ Load balanced : Plusieurs instances + ALB + Auto Scaling (payant)

6. **Combien de temps faut-il pour déployer une application ?**
   - ✅ Réponse : ~5-10 minutes pour le premier déploiement, ~2-3 minutes pour une mise à jour

7. **Peut-on accéder à l'instance EC2 créée par Beanstalk ?**
   - ✅ Réponse : Oui, via SSH (si vous avez configuré une clé SSH) ou via AWS Systems Manager

8. **Que se passe-t-il si on supprime l'environnement ?**
   - ✅ Réponse : Toutes les ressources sont supprimées (EC2, Security Groups, etc.)

---

## 🎓 Concepts Clés à Retenir

| Concept | Explication |
|---------|-------------|
| **PaaS** | Platform as a Service — AWS gère l'infrastructure |
| **IaaS** | Infrastructure as a Service — Vous gérez tout (EC2) |
| **Environment** | Ensemble de ressources AWS qui exécutent une version de l'application |
| **Application** | Conteneur logique qui regroupe plusieurs environnements |
| **Platform** | Langage + runtime (ex: Java 17 + Tomcat) |
| **Sample application** | Application exemple fournie par AWS pour tester |
| **Deployment** | Processus de mise à jour de l'application |

---

## 📊 Comparaison des Services AWS

| Service | Type | Cas d'usage | Complexité |
|---------|------|-------------|------------|
| **EC2** | IaaS | Infrastructure personnalisée | ⚠️ Élevée |
| **Elastic Beanstalk** | PaaS | Applications web (Java, Python, etc.) | ✅ Faible |
| **Lambda** | FaaS | Fonctions événementielles | ✅ Très faible |
| **ECS** | CaaS | Conteneurs Docker | ⚠️ Moyenne |
| **EKS** | CaaS | Kubernetes | ⚠️ Très élevée |
| **Lightsail** | PaaS | Sites web simples (WordPress) | ✅ Très faible |

---

## 💰 Coûts Estimés

### Mode Single Instance (Free Tier)

| Ressource | Coût | Free Tier |
|-----------|------|-----------|
| **EC2 t3.micro** | $0.0104/heure | ✅ 750h/mois gratuit (12 mois) |
| **Elastic Beanstalk** | $0 | ✅ Gratuit |
| **Data transfer OUT** | $0.09/GB | ✅ 100 GB/mois gratuit |
| **CloudWatch** | $0 | ✅ Gratuit (métriques de base) |

**Total si vous restez dans le Free Tier : $0/mois** ✅

**Total si vous dépassez le Free Tier : ~$7.50/mois** (1 instance t3.micro 24/7)

### Mode Load Balanced (Production)

| Ressource | Coût estimé |
|-----------|-------------|
| EC2 (2× t3.small) | ~$30/mois |
| Application Load Balancer | ~$16/mois |
| **Total** | ~$46/mois |

**⚠️ Pour ce lab, utilisez uniquement "Single instance" pour rester gratuit !**

---

## 🚀 Pour Aller Plus Loin

### 1. Déployer Votre Propre Application Java

**Créez une application Spring Boot** :

```bash
# Installer Spring Boot CLI
curl -s "https://get.sdkman.io" | bash
sdk install springboot

# Créer une nouvelle application
spring init --dependencies=web myapp
cd myapp

# Ajouter un contrôleur
cat > src/main/java/com/example/demo/HelloController.java <<EOF
package com.example.demo;
import org.springframework.web.bind.annotation.*;

@RestController
public class HelloController {
    @GetMapping("/")
    public String hello() {
        return "Bonjour depuis Spring Boot sur AWS !";
    }
}
EOF

# Compiler l'application
./mvnw package

# Le fichier .jar est dans target/
# Déployez-le sur Elastic Beanstalk !
```

### 2. Activer le Load Balancer et l'Auto Scaling

1. **Configuration → Capacity → Edit**

2. **Environment type** : Changez de `Single instance` à `Load balanced`

3. **Instances** :
   - Min : 2
   - Max : 4

4. **Scaling triggers** :
   - Metric : CPU Utilization
   - Target : 70%

5. **Cliquez sur "Apply"**

**Résultat** : Elastic Beanstalk créera un ALB et un Auto Scaling Group automatiquement !

### 3. Connecter une Base de Données RDS

1. **Configuration → Database → Edit**

2. **Engine** : MySQL ou PostgreSQL

3. **Instance class** : db.t3.micro

4. **Username/Password** : Choisissez

5. **Cliquez sur "Apply"**

**Résultat** : Beanstalk créera une base de données RDS et configurera les variables d'environnement automatiquement !

### 4. Configurer un Nom de Domaine Personnalisé

1. **Route 53 → Hosted zones → Votre domaine**

2. **Create record** :
   - Record type : A
   - Alias : Yes
   - Alias target : Elastic Beanstalk environment

---

## 📚 Ressources Supplémentaires

- [AWS Elastic Beanstalk Documentation](https://docs.aws.amazon.com/elasticbeanstalk/)
- [Elastic Beanstalk Java Platform](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/java-se-platform.html)
- [Sample Applications](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/tutorials.html)
- [Best Practices](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/best-practices.html)

---

## 🐛 Troubleshooting

### Problème 1 : L'environnement reste en "Degraded"

**Symptôme** : L'état de santé est rouge ou orange

**Solutions** :
1. Consultez les logs : Logs → Request Logs → Last 100 Lines
2. Vérifiez les "Recent events" en bas de la page
3. Vérifiez que le port 80 est bien ouvert dans le Security Group

### Problème 2 : "502 Bad Gateway"

**Cause** : L'application Java ne démarre pas correctement

**Solutions** :
1. Vérifiez que le fichier .war est valide
2. Consultez `web.stdout.log` pour voir les erreurs Java
3. Vérifiez que votre application écoute sur le port 5000 (attendu par Beanstalk)

### Problème 3 : Le déploiement prend trop de temps (>15 minutes)

**Cause** : Problème réseau ou de ressources

**Solutions** :
1. Attendez encore 5-10 minutes
2. Si toujours bloqué : Terminate l'environnement et recréez-le
3. Vérifiez que votre compte AWS n'a pas de limites dépassées

### Problème 4 : "The environment name is not available"

**Cause** : Le nom est déjà utilisé (par vous ou quelqu'un d'autre)

**Solution** : Ajoutez un chiffre ou votre nom : `M2i-Hannah-JavaApp-env-2`

---

## 📝 Notes Importantes

⚠️ **Suppression des ressources** :
- **Toujours supprimer l'environnement après le lab** pour éviter les frais
- La suppression prend 5-10 minutes
- Vérifiez dans EC2 que l'instance est bien "Terminated"

💡 **Bonnes pratiques** :
- Utilisez "Single instance" pour dev/test (gratuit)
- Utilisez "Load balanced" pour production (payant mais haute disponibilité)
- Activez les "Managed updates" pour les mises à jour automatiques de sécurité
- Utilisez RDS pour les bases de données (au lieu de MySQL sur l'instance)

🎯 **Cas d'usage** :
- ✅ Applications web (Spring Boot, Django, Express)
- ✅ APIs REST
- ✅ Sites corporate
- ⚠️ Applications stateful (préférer ECS ou EKS)
- ❌ Fonctions événementielles (utiliser Lambda)

---

**Durée estimée du lab** : 45 minutes - 1h

🎉 **Félicitations !** Vous savez maintenant déployer des applications avec AWS Elastic Beanstalk !

Vous avez appris :
- ✅ Ce qu'est un service PaaS
- ✅ Comment déployer une application Java en 5 minutes
- ✅ Comment surveiller une application
- ✅ Comment mettre à jour une application
- ✅ Les ressources créées automatiquement par Beanstalk
- ✅ La différence entre PaaS et IaaS
