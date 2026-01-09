# LAB 07 — Déploiement d'Odoo avec AWS Elastic Beanstalk + RDS Aurora

## 🎯 Objectifs

À la fin de ce lab, vous serez capable de :
- ✅ Créer une base de données **Amazon RDS Aurora PostgreSQL**
- ✅ Comprendre le fonctionnement d'**AWS Elastic Beanstalk**
- ✅ Déployer **Odoo** (ERP open source) avec Elastic Beanstalk
- ✅ Connecter Odoo à une base de données RDS (persistance des données)
- ✅ Accéder à l'application Odoo déployée
- ✅ Comprendre l'architecture complète d'une application web en production

---

## 📋 Prérequis

- ✅ Accès à la console AWS
- ✅ Région : **Virginia (us-east-1)**
- ✅ Connaissances de base en Docker
- ✅ Navigateur web

---

## 📚 Qu'est-ce qu'AWS Elastic Beanstalk ?

### Définition

**AWS Elastic Beanstalk** est un service **PaaS** (Platform as a Service) qui permet de déployer et gérer des applications **sans gérer l'infrastructure**.

### Fonctionnement

```
┌─────────────────────────────────────────────┐
│         Vous fournissez :                   │
│         • Code de l'application             │
│         • Configuration (Dockerfile)        │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│      Elastic Beanstalk gère :               │
│      • Instances EC2                        │
│      • Load Balancer (ELB)                  │
│      • Auto Scaling                         │
│      • Surveillance (CloudWatch)            │
│      • Déploiement automatique              │
└─────────────────────────────────────────────┘
```

### Avantages

| Avantage | Description |
|----------|-------------|
| **Simplicité** | Pas besoin de gérer l'infrastructure (EC2, ALB, ASG) |
| **Rapidité** | Déploiement en quelques clics |
| **Évolutivité** | Auto Scaling intégré |
| **Monitoring** | CloudWatch intégré |
| **Gratuit** | Vous payez uniquement les ressources utilisées (EC2, etc.) |

### Plateformes Supportées

- 🐍 **Python**
- ☕ **Java**
- 🟢 **Node.js**
- 🐘 **PHP**
- 💎 **Ruby**
- 🐳 **Docker** (ce que nous utiliserons)
- 🪟 **.NET**
- 🦦 **Go**

---

## 🏗️ Architecture Cible

```
┌────────────────────────────────────────────────────────────────┐
│                    AWS Elastic Beanstalk                       │
│                                                                │
│  ┌──────────────┐  ┌──────────────┐                           │
│  │   EC2 #1     │  │   EC2 #2     │                           │
│  │   [Docker]   │  │   [Docker]   │                           │
│  │   [Odoo 17]  │  │   [Odoo 17]  │                           │
│  └──────────────┘  └──────────────┘                           │
│         │                  │                                   │
│         └──────────┬───────┘                                   │
│                    │                                           │
│                    ▼                                           │
│         ┌─────────────────────────┐                            │
│         │  Amazon RDS Aurora      │                            │
│         │  PostgreSQL 15          │                            │
│         │  (Données persistantes) │                            │
│         └─────────────────────────┘                            │
└────────────────────────────────────────────────────────────────┘
                    │
                    ▼
               [Internet]
```

**Avantages de cette architecture** :
- ✅ **Données persistantes** : RDS Aurora sauvegarde automatiquement les données
- ✅ **Haute disponibilité** : RDS peut être configuré en Multi-AZ
- ✅ **Sauvegardes automatiques** : Snapshots quotidiens
- ✅ **Scaling facile** : Odoo et la base de données scalent indépendamment

---

## � À Propos d'Odoo

**Odoo** est un ERP (Enterprise Resource Planning) open source qui permet de gérer :
- 📊 CRM (Customer Relationship Management)
- 📦 Gestion des ventes et achats
- 📈 Comptabilité
- 📅 Gestion de projets
- 🏭 Production/Inventaire
- 💼 Ressources humaines

**Version** : Nous utiliserons Odoo 17 (dernière version stable)
**Image Docker officielle** : `odoo:17`
**Base de données** : Amazon RDS Aurora PostgreSQL 15

---

## 🗄️ PARTIE 1 : Créer la Base de Données RDS Aurora PostgreSQL

### Étape 1.1 : Accéder au service RDS

1. **Connectez-vous à la console AWS**

2. **Recherchez "RDS"** dans la barre de recherche

3. **Cliquez sur "RDS"**

4. **Cliquez sur "Create database"** (bouton orange)

---

### Étape 1.2 : Configurer la Base de Données Aurora

1. **Engine options** :
   - ✅ Sélectionnez **"Amazon Aurora"**
   - **Edition** : `Amazon Aurora PostgreSQL-Compatible Edition`
   - **Engine version** : `Aurora PostgreSQL 15.4` (ou dernière version 15.x)

2. **Templates** :
   - ✅ Sélectionnez **"Dev/Test"** (pour le lab)

3. **Settings** :
   - **DB cluster identifier** : `m2i-odoo-db-[votre-prenom]`
     - Exemple : `m2i-odoo-db-hannah`
   
   - **Master username** : `odoo`
   
   - **Master password** : Créez un mot de passe fort
     - Exemple : `OdooMasterPass123!`
     - ⚠️ **Notez ce mot de passe quelque part !**
   
   - **Confirm password** : Répétez le mot de passe

4. **DB instance class** :
   - ✅ Sélectionnez **"Burstable classes"**
   - **Instance type** : `db.t3.medium` (ou `db.t4g.medium` si disponible)

5. **Availability & durability** :
   - ✅ Sélectionnez **"Don't create an Aurora Replica"** (pour ce lab)

6. **Connectivity** :
   - **Virtual private cloud (VPC)** : Sélectionnez le VPC par défaut
   
   - **Public access** : ✅ **Yes** (pour ce lab uniquement)
     - ⚠️ En production, utilisez **No** et accédez via VPC
   
   - **VPC security group** :
     - ✅ Sélectionnez **"Create new"**
     - **Name** : `m2i-odoo-rds-sg`
   
   - **Availability Zone** : `us-east-1a`

7. **Database authentication** :
   - ✅ Sélectionnez **"Password authentication"**

8. **Additional configuration** (cliquez pour dérouler) :
   - **Initial database name** : `odoo`
   - **Backup retention period** : `7 days`
   - **Enable encryption** : ✅ (laissez coché)

9. **Cliquez sur "Create database"** (bouton orange en bas)

10. **Attendez la création** (⏱️ 10-15 minutes)

---

### Étape 1.3 : Noter l'Endpoint de la Base de Données

1. **Une fois la base créée**, cliquez sur le cluster : `m2i-odoo-db-[votre-prenom]`

2. **Onglet "Connectivity & security"**

3. **Notez l'Endpoint** :
   - Format : `m2i-odoo-db-hannah.cluster-xxxxx.us-east-1.rds.amazonaws.com`
   - ⚠️ **Copiez cet endpoint**, vous en aurez besoin !

---

### Étape 1.4 : Configurer le Security Group pour Autoriser les Connexions

1. **Cliquez sur le Security Group** : `m2i-odoo-rds-sg` (dans l'onglet Connectivity)

2. **Onglet "Inbound rules" → "Edit inbound rules"**

3. **Cliquez sur "Add rule"** :
   - **Type** : `PostgreSQL`
   - **Protocol** : `TCP`
   - **Port** : `5432`
   - **Source** : `0.0.0.0/0` (pour ce lab uniquement)
     - ⚠️ En production, restreignez au Security Group d'Elastic Beanstalk

4. **Cliquez sur "Save rules"**

---

## 🚀 PARTIE 2 : Déployer Odoo avec Elastic Beanstalk

### Étape 2.1 : Préparer le Fichier de Configuration

1. **Sur votre ordinateur, créez un dossier** : `odoo-beanstalk`

2. **Dans ce dossier, créez un fichier `Dockerrun.aws.json`** :

```json
{
  "AWSEBDockerrunVersion": "1",
  "Image": {
    "Name": "odoo:17",
    "Update": "true"
  },
  "Ports": [
    {
      "ContainerPort": 8069,
      "HostPort": 80
    }
  ],
  "Environment": [
    {
      "Name": "HOST",
      "Value": "REMPLACEZ-PAR-VOTRE-ENDPOINT-RDS"
    },
    {
      "Name": "USER",
      "Value": "odoo"
    },
    {
      "Name": "PASSWORD",
      "Value": "REMPLACEZ-PAR-VOTRE-MOT-DE-PASSE"
    },
    {
      "Name": "PORT",
      "Value": "5432"
    }
  ],
  "Logging": "/var/log/odoo"
}
```

3. **IMPORTANT** : Remplacez dans le fichier :
   - `REMPLACEZ-PAR-VOTRE-ENDPOINT-RDS` → Votre endpoint RDS
     - Exemple : `m2i-odoo-db-hannah.cluster-xxxxx.us-east-1.rds.amazonaws.com`
   - `REMPLACEZ-PAR-VOTRE-MOT-DE-PASSE` → Votre mot de passe RDS
     - Exemple : `OdooMasterPass123!`

4. **Compressez le fichier en ZIP** :
   - Windows : Clic droit sur `Dockerrun.aws.json` → "Envoyer vers" → "Dossier compressé"
   - Nom du ZIP : `odoo-beanstalk.zip`

💡 **Important** : Le ZIP doit contenir directement `Dockerrun.aws.json`, pas un dossier.

---

### Étape 2.2 : Créer l'Application Elastic Beanstalk

1. **Recherchez "Elastic Beanstalk"** dans la barre de recherche AWS

2. **Cliquez sur "Elastic Beanstalk"**

3. **Cliquez sur "Create application"** (bouton orange)

4. **Configuration de l'application** :

   **Application name** :
   - Nom : `M2i-[VOTRE_PRENOM]-Odoo`
   - Exemple : `M2i-Hannah-Odoo`

   **Platform** :
   - **Platform** : Sélectionnez `Docker`
   - **Platform branch** : `Docker running on 64bit Amazon Linux 2023`
   - **Platform version** : Choisissez la version recommandée (dernière)

   **Application code** :
   - ✅ Sélectionnez **"Upload your code"**
   - **Version label** : `odoo-rds-v1`
   - **Source code origin** : Cliquez sur **"Choose file"** → Sélectionnez `odoo-beanstalk.zip`

   **Presets** :
   - ✅ Sélectionnez **"Single instance (free tier eligible)"**

5. **Cliquez on "Next"**

---

### Étape 2.3 : Configurer le Service Access

1. **Service role** :
   - ✅ Sélectionnez **"Create and use new service role"**

2. **EC2 instance profile** :
   - ✅ Sélectionnez **"Create and use new instance profile"**

3. **Cliquez sur "Skip to review"**

---

### Étape 2.4 : Vérifier et Déployer

1. **Vérifiez la configuration** :
   - Application : `M2i-[VOTRE_PRENOM]-Odoo`
   - Platform : Docker
   - Environment : Single instance

2. **Cliquez sur "Submit"** (bouton orange)

3. **Attendez le déploiement** (⏱️ 5-10 minutes)

Vous verrez les étapes :
```
✅ Creating application
✅ Creating environment
✅ Launching environment
✅ Deploying application
```

---

## ✅ PARTIE 3 : Vérifier et Configurer Odoo

### Étape 3.1 : Accéder à Odoo

1. **Une fois l'environnement créé**, l'état devra être **"Health: Ok"** (vert)

2. **Cliquez sur l'URL de l'environnement** :
   - Format : `http://M2iHannahOdoo-env.xxxxx.us-east-1.elasticbeanstalk.com`

3. **Vous devriez voir la page d'accueil d'Odoo** 🎉

---

### Étape 3.2 : Configuration Initiale d'Odoo

1. **Sur la page d'accueil Odoo**, vous verrez le formulaire de création de base de données :

   - **Master Password** : Créez un mot de passe maître
     - Exemple : `OdooAdmin123!`
     - ⚠️ **Notez-le**, c'est le mot de passe principal d'Odoo
   
   - **Database Name** : `odoo-prod`
   
   - **Email** : Votre email (login administrateur)
     - Exemple : `admin@m2i.fr`
   
   - **Password** : Mot de passe de connexion
     - Exemple : `Admin123!`
   
   - **Phone number** : Optionnel
   
   - **Language** : ✅ Sélectionnez **French / Français**
   
   - **Country** : ✅ Sélectionnez **France**
   
   - **Demo data** : ✅ Cochez **"Load demonstration data"** (pour avoir des données d'exemple)

2. **Cliquez sur "Create database"**

3. **Attendez 2-3 minutes** que la base de données soit initialisée

4. **Vous êtes maintenant connecté à Odoo !** 🎊

---

### Étape 3.3 : Explorer Odoo

1. **Page d'accueil Odoo** : Vous verrez les modules disponibles :
   - 📊 **CRM** : Gestion de la relation client
   - 📦 **Sales** : Gestion des ventes
   - 📈 **Accounting** : Comptabilité
   - 📅 **Project** : Gestion de projets
   - 🏭 **Inventory** : Gestion des stocks
   - 💼 **HR** : Ressources humaines

2. **Installez un module** (exemple : CRM) :
   - Cliquez sur **"CRM"**
   - Cliquez sur **"Install"**
   - Attendez l'installation (~1 minute)

3. **Explorez le CRM** :
   - Créez un lead (prospect)
   - Créez une opportunité
   - Visualisez le pipeline de ventes

4. **Les données sont maintenant stockées dans RDS Aurora !** ✅
   - Si vous redémarrez Elastic Beanstalk, les données seront préservées
   - Les sauvegardes automatiques sont configurées

---

## 📊 PARTIE 4 : Monitoring et Gestion

### Étape 4.1 : Consulter les Logs Elastic Beanstalk

1. **Elastic Beanstalk → Votre environnement**

2. **Menu gauche → "Logs"**

3. **Cliquez sur "Request Logs" → "Last 100 Lines"**

4. **Téléchargez et consultez les logs** pour voir le démarrage d'Odoo

---

### Étape 4.2 : Vérifier la Base de Données RDS

1. **RDS → Databases → Cliquez sur votre cluster**

2. **Onglet "Monitoring"** :
   - CPU Utilization
   - Database Connections
   - Read/Write IOPS

3. **Les données Odoo sont stockées ici de manière persistante** ✅

---

## 🧹 PARTIE 5 : Nettoyage — ⚠️ TRÈS IMPORTANT

### Ordre de Suppression (Respectez cet ordre !) :

### Étape 5.1 : Supprimer l'Environnement Elastic Beanstalk

1. **Elastic Beanstalk → Environments**

2. **Sélectionnez votre environnement** : `M2i-[VOTRE_PRENOM]-Odoo-env`

3. **Actions → Terminate environment**

4. **Confirmez** en tapant le nom de l'environnement

5. **Attendez 5-10 minutes** que l'environnement soit supprimé

---

### Étape 5.2 : Supprimer l'Application Elastic Beanstalk

1. **Elastic Beanstalk → Applications**

2. **Sélectionnez votre application** : `M2i-[VOTRE_PRENOM]-Odoo`

3. **Actions → Delete application**

4. **Confirmez**

---

### Étape 5.3 : Supprimer la Base de Données RDS Aurora

1. **RDS → Databases**

2. **Sélectionnez votre cluster** : `m2i-odoo-db-[votre-prenom]`

3. **Actions → Delete**

4. **Décochez** "Create final snapshot" (pour ce lab)

5. **Cochez** "I acknowledge..."

6. **Tapez** `delete me` pour confirmer

7. **Cliquez sur "Delete"**

8. **Attendez 10-15 minutes** que la base soit supprimée

---

### Étape 5.4 : Supprimer le Security Group RDS

1. **VPC → Security Groups**

2. **Recherchez** : `m2i-odoo-rds-sg`

3. **Sélectionnez-le → Actions → Delete security groups**

---

## ✅ Validation du Lab

### Questions de Compréhension

1. **Pourquoi utiliser RDS Aurora au lieu de PostgreSQL dans un conteneur ?**
   - ✅ Données persistantes (survit aux redéploiements)
   - ✅ Sauvegardes automatiques
   - ✅ Haute disponibilité (Multi-AZ)
   - ✅ Scaling indépendant

2. **Qu'est-ce qu'Elastic Beanstalk gère automatiquement ?**
   - Instances EC2
   - Déploiement de l'application Docker
   - Monitoring CloudWatch
   - Auto Scaling (si configuré)

3. **Où sont stockées les données Odoo ?**
   - Dans Amazon RDS Aurora PostgreSQL
   - Endpoint : `m2i-odoo-db-xxx.cluster-xxx.rds.amazonaws.com`

4. **Que se passe-t-il si on redémarre l'environnement Elastic Beanstalk ?**
   - ✅ Les données sont **préservées** (stockées dans RDS)
   - L'application redémarre avec les mêmes données

---

## 🎓 Concepts Clés Retenus

| Concept | Explication |
|---------|-------------|
| **Elastic Beanstalk** | PaaS pour déployer des applications sans gérer l'infrastructure |
| **RDS Aurora** | Base de données PostgreSQL managée avec haute disponibilité |
| **Dockerrun.aws.json** | Fichier de configuration pour déployer un conteneur Docker |
| **Variables d'environnement** | HOST, USER, PASSWORD pour connecter Odoo à RDS |
| **Persistance des données** | Stockage en dehors des conteneurs (RDS) |

---

## 📊 Comparaison : Conteneur PostgreSQL vs RDS Aurora

| Aspect | PostgreSQL en Conteneur | RDS Aurora PostgreSQL |
|--------|-------------------------|----------------------|
| **Persistance** | ❌ Perdue si conteneur détruit | ✅ Toujours persistante |
| **Sauvegardes** | ❌ Manuelles | ✅ Automatiques (quotidiennes) |
| **Haute disponibilité** | ❌ Non | ✅ Multi-AZ disponible |
| **Scaling** | ❌ Difficile | ✅ Facile (vertical) |
| **Coût** | 💰 Inclus dans EC2 | 💰 $0.10/heure (db.t3.medium) |
| **Cas d'usage** | Tests/développement | Production |

---

## 🚀 Pour Aller Plus Loin

### Améliorations Production

1. **Haute Disponibilité RDS** :
   - Activer Multi-AZ pour RDS Aurora
   - Lecteur Aurora (Read Replica) pour meilleure performance

2. **Elastic Beanstalk en Load Balanced** :
   - Changer de "Single instance" à "Load balanced"
   - Auto Scaling activé (min: 2, max: 4)

3. **HTTPS** :
   - Configurer un certificat SSL/TLS (AWS Certificate Manager)
   - Activer HTTPS sur l'ALB

4. **Nom de domaine** :
   - Enregistrer un domaine (Route 53)
   - Créer un alias vers Elastic Beanstalk

5. **Stockage des fichiers** :
   - Utiliser Amazon S3 pour les attachments Odoo
   - Éviter de stocker les fichiers sur l'instance EC2

---

## 📚 Ressources Supplémentaires

- [AWS Elastic Beanstalk Documentation](https://docs.aws.amazon.com/elasticbeanstalk/)
- [Amazon RDS Aurora PostgreSQL](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/)
- [Odoo Documentation Officielle](https://www.odoo.com/documentation/17.0/)
- [Best Practices Elastic Beanstalk](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/best-practices.html)

---

**Durée estimée** : 2h - 2h30

🎉 **Félicitations !** Vous avez déployé une application complète (Odoo) avec Elastic Beanstalk et RDS Aurora, avec persistance des données en production !
   - Choisissez `odoo-beanstalk-v2.zip`
   - **Version label** : `odoo-v2`
   - Cliquez sur **"Deploy"**

4. **Attendez le déploiement** (~2-3 minutes)

---

### Étape 2.9 : Tester l'Application Odoo

1. **Cliquez sur l'URL de l'environnement** :
   - Exemple : `http://m2i-hannah-odoo.us-east-1.elasticbeanstalk.com`

2. **⚠️ Vous allez voir une erreur** : `Database not found`

**Pourquoi ?**
- L'image Odoo officielle nécessite une base de données PostgreSQL
- En mode simple, sans PostgreSQL, Odoo ne peut pas démarrer

**Solution pour ce lab** :
Nous allons créer un fichier `.ebextensions` qui installe PostgreSQL sur l'instance.

---

## 🛠️ PARTIE 3 : Configuration Avancée avec PostgreSQL

### Étape 3.1 : Comprendre .ebextensions

**`.ebextensions`** est un dossier spécial dans Elastic Beanstalk qui permet de :
- Installer des packages
- Exécuter des commandes
- Configurer l'environnement

**Structure** :
```
odoo-beanstalk/
├── Dockerrun.aws.json
└── .ebextensions/
    └── 01-postgres.config
```

---

### Étape 3.2 : Créer le Fichier de Configuration PostgreSQL

**⚠️ Limitation** : Installer PostgreSQL via `.ebextensions` est complexe et non recommandé en production.

**Pour ce lab, nous allons utiliser une approche simplifiée** : déployer Odoo avec une configuration qui utilise SQLite (base de données locale) au lieu de PostgreSQL.

**Malheureusement, Odoo ne supporte pas SQLite.**

---

### Étape 3.3 : Solution Alternative — Utiliser Docker Compose

**Problème** : Elastic Beanstalk en mode Docker simple ne supporte qu'un seul conteneur.

**Solution** : Utiliser **Docker Compose** avec Elastic Beanstalk.

**Nouvelle configuration : `docker-compose.yml`**

```yaml
version: '3.8'

services:
  db:
    image: postgres:15
    environment:
      - POSTGRES_DB=postgres
      - POSTGRES_USER=odoo
      - POSTGRES_PASSWORD=odoo
    volumes:
      - odoo-db-data:/var/lib/postgresql/data

  odoo:
    image: odoo:17
    depends_on:
      - db
    ports:
      - "80:8069"
    environment:
      - HOST=db
      - USER=odoo
      - PASSWORD=odoo
    volumes:
      - odoo-web-data:/var/lib/odoo

volumes:
  odoo-db-data:
  odoo-web-data:
```

**Nouvelle approche** : Utiliser **Multi-container Docker** avec Elastic Beanstalk.

---

### Étape 3.4 : Créer Dockerrun.aws.json v2 (Multi-container)

**Fichier : `Dockerrun.aws.json` (Version 2)**

```json
{
  "AWSEBDockerrunVersion": 2,
  "containerDefinitions": [
    {
      "name": "postgres",
      "image": "postgres:15",
      "essential": true,
      "memory": 512,
      "environment": [
        {
          "name": "POSTGRES_DB",
          "value": "postgres"
        },
        {
          "name": "POSTGRES_USER",
          "value": "odoo"
        },
        {
          "name": "POSTGRES_PASSWORD",
          "value": "odoo"
        }
      ],
      "mountPoints": [
        {
          "sourceVolume": "postgres-data",
          "containerPath": "/var/lib/postgresql/data"
        }
      ]
    },
    {
      "name": "odoo",
      "image": "odoo:17",
      "essential": true,
      "memory": 1024,
      "portMappings": [
        {
          "hostPort": 80,
          "containerPort": 8069
        }
      ],
      "links": [
        "postgres:db"
      ],
      "environment": [
        {
          "name": "HOST",
          "value": "db"
        },
        {
          "name": "USER",
          "value": "odoo"
        },
        {
          "name": "PASSWORD",
          "value": "odoo"
        }
      ],
      "mountPoints": [
        {
          "sourceVolume": "odoo-data",
          "containerPath": "/var/lib/odoo"
        }
      ]
    }
  ],
  "volumes": [
    {
      "name": "postgres-data",
      "host": {
        "sourcePath": "/var/app/postgres-data"
      }
    },
    {
      "name": "odoo-data",
      "host": {
        "sourcePath": "/var/app/odoo-data"
      }
    }
  ]
}
```

---

### Étape 3.5 : Déployer la Version Multi-container

1. **Sur votre ordinateur** :
   - Supprimez l'ancien fichier `Dockerrun.aws.json`
   - Créez un nouveau `Dockerrun.aws.json` avec le contenu ci-dessus (Version 2)

2. **Créez un nouveau ZIP** : `odoo-beanstalk-v3.zip`

3. **⚠️ IMPORTANT** : Le mode multi-container nécessite un **Load Balancer**, donc nous devons recréer l'environnement.

4. **Supprimez l'environnement actuel** :
   - Elastic Beanstalk → Votre environnement
   - Actions → Terminate environment
   - Confirmez

5. **Créez un nouvel environnement** :
   - Cliquez sur "Create a new environment"
   - **Environment tier** : Web server environment
   - **Application name** : `M2i-[VOTRE_PRENOM]-Odoo`
   - **Environment name** : `M2i-[VOTRE_PRENOM]-Odoo-env`
   - **Platform** : Docker
   - **Platform branch** : **Multi-container Docker running on 64bit Amazon Linux 2**
   - **Application code** : Upload `odoo-beanstalk-v3.zip`
   - **Presets** : **Single instance** (pour ce lab)
   - Cliquez sur "Create environment"

6. **Attendez 10-15 minutes** pour la création

---

### Étape 3.6 : Vérifier le Déploiement

1. **Une fois l'environnement créé**, cliquez sur l'URL

2. **Vous devriez voir la page d'accueil d'Odoo** 🎉

3. **Configuration initiale d'Odoo** :
   - **Master Password** : Créez un mot de passe maître (exemple : `admin123`)
   - **Database Name** : `odoo-db`
   - **Email** : Votre email
   - **Password** : Mot de passe de connexion (exemple : `odoo123`)
   - **Language** : French
   - **Country** : France
   - ✅ Cochez "Load demonstration data" (pour avoir des données d'exemple)
   - Cliquez sur **"Create database"**

4. **Attendez 2-3 minutes** que la base de données soit créée

5. **Vous êtes maintenant dans Odoo !** 🎊

**Explorez les modules** :
- CRM
- Sales
- Accounting
- Inventory
- etc.

---

## 📊 PARTIE 3 : Monitoring et Gestion

### Étape 3.1 : Consulter les Logs

1. **Elastic Beanstalk → Votre environnement**

2. **Menu gauche → "Logs"**

3. **Cliquez sur "Request Logs" → "Last 100 Lines"**

4. **Attendez quelques secondes**, puis cliquez sur "Download"

5. **Ouvrez le fichier** pour voir les logs des conteneurs

---

### Étape 4.2 : Consulter les Métriques CloudWatch

1. **Menu gauche → "Monitoring"**

2. **Vous verrez des graphiques** :
   - **Environment Health** : Santé de l'environnement
   - **Requests** : Nombre de requêtes
   - **Latency** : Temps de réponse
   - **CPU Utilization** : Utilisation CPU
   - **Network In/Out** : Trafic réseau

---

### Étape 3.3 : Consulter la Configuration

1. **Menu gauche → "Configuration"**

2. **Vous verrez les sections** :
   - **Software** : Configuration du runtime (Docker)
   - **Instances** : Type d'instance EC2 (t2.micro ou t3.micro)
   - **Capacity** : Auto Scaling (désactivé en mode single instance)
   - **Load balancer** : Non applicable (single instance)
   - **Security** : Rôles IAM

---

## 🧪 PARTIE 4 : Tester l'Auto Scaling (Optionnel)

**⚠️ Cette section est optionnelle et augmentera légèrement les coûts AWS.**

### Étape 4.1 : Activer le Load Balancer

1. **Elastic Beanstalk → Configuration**

2. **Capacity → Edit**

3. **Environment type** : Changez de `Single instance` à `Load balanced`

4. **Auto Scaling group** :
   - **Min instances** : 1
   - **Max instances** : 3

5. **Cliquez sur "Apply"**

6. **Attendez 5-10 minutes** que l'environnement soit modifié

**Résultat** :
- Un **Application Load Balancer** sera créé
- L'**Auto Scaling Group** sera activé
- Si la charge augmente, de nouvelles instances seront créées automatiquement

---

## 🧹 PARTIE 5 : Nettoyage — ⚠️ TRÈS IMPORTANT

### Étape 5.1 : Supprimer l'Environnement Elastic Beanstalk

1. **Elastic Beanstalk → Environments**

2. **Sélectionnez votre environnement** : `M2i-[VOTRE_PRENOM]-Odoo-env`

3. **Actions → Terminate environment**

4. **Confirmez** en tapant le nom de l'environnement

5. **Attendez 5-10 minutes** que l'environnement soit supprimé

**Ce qui sera automatiquement supprimé** :
- ✅ Instances EC2
- ✅ Security Groups
- ✅ Load Balancer (si activé)
- ✅ Auto Scaling Group
- ✅ CloudWatch Alarms

---

### Étape 5.2 : Supprimer l'Application Elastic Beanstalk

1. **Elastic Beanstalk → Applications**

2. **Sélectionnez votre application** : `M2i-[VOTRE_PRENOM]-Odoo`

3. **Actions → Delete application**

4. **Confirmez**

---

### Étape 5.3 : Vérifier les Ressources EC2

1. **EC2 → Instances**
   - Vérifiez qu'aucune instance Elastic Beanstalk ne tourne

2. **EC2 → Security Groups**
   - Supprimez les Security Groups Elastic Beanstalk (commencent par `awseb-`)

3. **EC2 → Load Balancers**
   - Vérifiez qu'aucun Load Balancer Elastic Beanstalk n'existe

---

## ✅ Validation du Lab

### Questions de Compréhension

1. **Qu'est-ce qu'AWS Elastic Beanstalk ?**
   - Réponse : Un service PaaS qui déploie et gère automatiquement l'infrastructure pour vos applications

2. **Quelles ressources AWS sont créées automatiquement par Beanstalk ?**
   - Réponse : EC2, Security Groups, Load Balancer (si activé), Auto Scaling Group, CloudWatch

3. **Quelle est la différence entre "Single instance" et "Load balanced" ?**
   - Single instance : Une seule instance EC2, pas de Load Balancer
   - Load balanced : Plusieurs instances avec Load Balancer et Auto Scaling

4. **Pourquoi avons-nous utilisé Docker multi-container pour Odoo ?**
   - Réponse : Odoo nécessite PostgreSQL, donc nous avons besoin de 2 conteneurs (Odoo + PostgreSQL)

5. **Quel est l'avantage principal d'Elastic Beanstalk par rapport à EC2 seul ?**
   - Réponse : Gestion automatique de l'infrastructure (déploiement, scaling, monitoring)

---

## 🎓 Concepts Clés Retenus

| Concept | Explication |
|---------|-------------|
| **Elastic Beanstalk** | Service PaaS pour déployer des applications sans gérer l'infrastructure |
| **PaaS** | Platform as a Service (vs IaaS comme EC2) |
| **Dockerrun.aws.json** | Fichier de configuration pour déployer des conteneurs Docker |
| **Multi-container Docker** | Déployer plusieurs conteneurs (ex: Odoo + PostgreSQL) |
| **Single instance** | Mode simple avec une seule instance EC2 |
| **Load balanced** | Mode avec Load Balancer et Auto Scaling |
| **Auto Scaling** | Ajustement automatique du nombre d'instances selon la charge |
| **.ebextensions** | Dossier pour configurer l'environnement Beanstalk |

---

## 📊 Comparaison : Elastic Beanstalk vs EC2 vs ECS

| Aspect | EC2 (IaaS) | Elastic Beanstalk (PaaS) | ECS (Container Orchestration) |
|--------|------------|--------------------------|-------------------------------|
| **Gestion infra** | ❌ Manuelle | ✅ Automatique | ⚠️ Semi-automatique |
| **Complexité** | ⚠️ Élevée | ✅ Faible | ⚠️ Moyenne |
| **Contrôle** | ✅ Total | ⚠️ Limité | ✅ Élevé |
| **Scaling** | ❌ Manuel | ✅ Auto | ✅ Auto |
| **Monitoring** | ❌ À configurer | ✅ Intégré | ✅ Intégré |
| **Déploiement** | ❌ Manuel | ✅ Simple (upload ZIP) | ⚠️ Task Definitions |
| **Coût** | 💰 Direct | 💰 Direct (pas de frais Beanstalk) | 💰 Direct |
| **Cas d'usage** | Infrastructure custom | Applications web simples | Microservices |

---

## 🚀 Pour Aller Plus Loin

### Améliorations Possibles (Production)

1. **Base de données séparée** :
   - Utiliser **Amazon RDS** (PostgreSQL) au lieu de PostgreSQL dans un conteneur
   - Avantages : Sauvegardes automatiques, haute disponibilité, meilleure performance

2. **Stockage persistant** :
   - Utiliser **Amazon EFS** pour stocker les fichiers Odoo
   - Les données survivent aux redéploiements

3. **HTTPS** :
   - Configurer un certificat SSL/TLS
   - Utiliser AWS Certificate Manager (ACM)

4. **Nom de domaine personnalisé** :
   - Enregistrer un domaine (Route 53)
   - Pointer vers l'environnement Beanstalk

5. **CI/CD** :
   - Intégrer avec AWS CodePipeline
   - Déploiement automatique depuis GitHub

---

## 📚 Ressources Supplémentaires

- [AWS Elastic Beanstalk Documentation](https://docs.aws.amazon.com/elasticbeanstalk/)
- [Elastic Beanstalk avec Docker](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_docker.html)
- [Odoo Documentation](https://www.odoo.com/documentation/17.0/)
- [Dockerrun.aws.json Reference](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_docker_v2config.html)

---

## 💡 Cas d'Usage Réels d'Elastic Beanstalk

| Entreprise | Cas d'Usage |
|------------|-------------|
| **Startups** | Déploiement rapide d'applications web sans équipe DevOps |
| **Entreprises** | Applications internes (CRM, ERP, portails) |
| **E-commerce** | Sites web avec scaling automatique pendant les pics |
| **SaaS** | Applications multi-tenants avec déploiement simplifié |

---

**Durée estimée** : 1h30 - 2h

🎉 **Félicitations !** Vous savez maintenant déployer des applications avec AWS Elastic Beanstalk !

---

## 🐛 Troubleshooting

### Problème 1 : L'environnement reste en "Degraded"

**Cause** : Les conteneurs ne démarrent pas correctement

**Solution** :
1. Consultez les logs : Logs → Request Logs → Last 100 Lines
2. Vérifiez que le fichier `Dockerrun.aws.json` est correct
3. Vérifiez que les images Docker existent (odoo:17, postgres:15)

### Problème 2 : "502 Bad Gateway"

**Cause** : L'application Odoo n'est pas accessible sur le port 80

**Solution** :
1. Vérifiez que le port mapping est correct : `"hostPort": 80, "containerPort": 8069`
2. Vérifiez que le Security Group autorise le trafic sur le port 80

### Problème 3 : Odoo affiche "Database not found"

**Cause** : PostgreSQL n'est pas démarré ou pas accessible

**Solution** :
1. Vérifiez que le conteneur PostgreSQL est en mode multi-container
2. Vérifiez les variables d'environnement (HOST=db, USER=odoo, PASSWORD=odoo)
3. Consultez les logs du conteneur PostgreSQL

### Problème 4 : L'environnement prend trop de temps à se créer

**Cause** : Elastic Beanstalk télécharge les images Docker

**Solution** :
- Soyez patient (10-15 minutes pour la première fois)
- Les déploiements suivants seront plus rapides (images en cache)

---

## 📝 Notes Importantes

⚠️ **Coûts AWS** :
- Elastic Beanstalk lui-même est **gratuit**
- Vous payez uniquement les ressources utilisées :
  - EC2 instance (t2.micro = gratuit pendant 12 mois sous Free Tier)
  - Load Balancer (si activé) : ~$16/mois
  - Trafic réseau sortant

⚠️ **Limitations du Free Tier** :
- 750 heures/mois d'instances t2.micro (1 instance = OK, 2 instances = 1500h = payant)
- Supprimez toujours les ressources après le lab !

💡 **Bonne pratique** :
- Utilisez **"Single instance"** pour les labs et tests
- Utilisez **"Load balanced"** pour les environnements de production
