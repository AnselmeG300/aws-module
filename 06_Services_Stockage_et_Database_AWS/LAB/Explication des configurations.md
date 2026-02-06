# Explication des Configurations Elastic Beanstalk

Ce document explique en détail toutes les configurations visibles dans les captures d'écran du processus de création d'un environnement Elastic Beanstalk avec Docker.

---

## 🎯 Vue d'ensemble du processus

Le processus de création d'un environnement Elastic Beanstalk se décompose en **6 étapes** :

1. **Configurer l'environnement** - Nom, plateforme, code
2. **Configurer l'accès au service** - IAM roles, EC2 key pair
3. **Configurer la mise en réseau, la base de données et les identifications** - VPC, RDS
4. **Configurer le trafic et la mise à l'échelle des instances** - Load Balancer, Auto Scaling
5. **Configurer les mises à jour, la surveillance et la journalisation** - CloudWatch, logs
6. **Vérification** - Récapitulatif final

---

## 📋 Étape 1 : Configurer l'environnement

### Niveau d'environnement

**Deux options disponibles :**

1. **Environnement de serveur web** ✅ (sélectionné)
   - Pour des applications web ou API web
   - Traite les requêtes HTTP/HTTPS
   
2. **Environnement de travail**
   - Pour des tâches de longue durée
   - Traite les messages d'une file SQS

### Informations sur l'application

- **Nom de l'application** : `M2i-Anselme-EB`
  - Identificateur unique de votre application
  - Maximum 100 caractères

- **Nom de l'environnement** : `M2i-Anselme-EB-env`
  - Nom de l'environnement d'exécution
  - Sera utilisé dans l'URL publique

### Plateforme

- **Plateforme** : `Docker` ✅
  - AWS gère un environnement pour conteneurs Docker
  
- **Branche de plateforme** : `Docker running on 64bit Amazon Linux 2023`
  - Version du système d'exploitation hôte
  - Amazon Linux 2023 est la dernière génération
  
- **Version de la plateforme** : `4.9.2 (Recommended)`
  - Version spécifique de la plateforme Elastic Beanstalk

### Code de l'application

**Deux options :**

1. **Exemple d'application** ✅ (sélectionné)
   - Application de démonstration fournie par AWS
   - Permet de tester sans avoir de code

2. **Charger votre code**
   - Upload d'un fichier ZIP ou référence S3
   - Contient votre Dockerfile et code source

### Préréglages

- **Haute disponibilité** ✅ (sélectionné)
  - Déploiement multi-AZ avec Load Balancer
  - Auto Scaling activé (min 2 instances)
  - **Recommandé pour la production**

**Autres options disponibles :**
- Instance unique (gratuit, développement)
- Instance unique avec Spot (économique)
- Haute disponibilité avec Spot
- Configuration personnalisée

---

## 🔐 Étape 2 : Configurer l'accès au service

### Accès au service

Configuration des rôles IAM utilisés par Elastic Beanstalk.

#### Fonction du service

- **Rôle IAM** : `aws-elasticbeanstalk-service-role`
  - Rôle assumé par Elastic Beanstalk en tant que service
  - Permet à EB de créer et gérer les ressources AWS
  - **Politiques requises** :
    - Gestion EC2 (instances, security groups)
    - Gestion Auto Scaling
    - Gestion Elastic Load Balancer
    - Accès CloudWatch pour logs et métriques

#### Profil d'instance EC2

- **Profil IAM** : `aws-elasticbeanstalk-ec2-role`
  - Rôle attaché aux instances EC2
  - Permet aux instances d'effectuer des opérations AWS
  - **Politiques typiques** :
    - Lecture/écriture dans S3 (pour logs et déploiements)
    - CloudWatch Logs
    - X-Ray (si activé)
    - Secrets Manager / Parameter Store

#### Paire de clés EC2 (facultatif)

- **Sélection** : `toto`
  - Permet la connexion SSH aux instances EC2
  - **Optionnel** : pas nécessaire pour un environnement managé
  - **Usage** : débogage avancé uniquement

---

## 🌐 Étape 3 : Configuration réseau et base de données

### Paramètres de l'instance

#### VPC (Virtual Private Cloud)

- **VPC sélectionné** : `vpc-08104c570e4699f01 (172.31.0.0/16)`
  - Réseau virtuel isolé dans AWS
  - CIDR 172.31.0.0/16 = 65,536 adresses IP
  - **VPC par défaut** de la région us-east-1

#### Adresse IP publique

- **Activée** : ❌ Non cochée
  - Les instances n'auront **pas d'IP publique directe**
  - **Accès via Load Balancer uniquement**
  - Meilleure sécurité (instances en sous-réseaux privés)

#### Sous-réseaux de l'instance

**Sous-réseaux sélectionnés** (pour les instances EC2) :

| Zone | Sous-réseau | CIDR |
|------|-------------|------|
| us-east-1b | subnet-06adf27d3ddd35a80 | 172.31.32.0/20 |
| us-east-1a | subnet-08c27a0475299605c | 172.31.16.0/20 |

- **Multi-AZ** : Instances réparties dans 2 zones de disponibilité
- **Haute disponibilité** : Si une AZ tombe, l'autre continue

### Base de données

#### Activer la base de données

- **Activée** : ✅ Oui
  - Elastic Beanstalk crée et gère une instance RDS
  - Intégration automatique (variables d'environnement)

#### Restauration d'un instantané

- **Aucun** : Nouvelle base de données vierge

#### Sous-réseaux de la base de données

**Sous-réseaux sélectionnés** (pour RDS) :

| Zone | Sous-réseau | CIDR |
|------|-------------|------|
| us-east-1b | subnet-06adf27d3ddd35a80 | 172.31.32.0/20 |
| us-east-1a | subnet-08c27a0475299605c | 172.31.16.0/20 |

- **Multi-AZ RDS** : Base de données répliquée automatiquement

#### Paramètres de la base de données

| Paramètre | Valeur | Explication |
|-----------|--------|-------------|
| **Moteur** | `mysql` | Système de gestion de base de données |
| **Version** | `8.4.7` | Version MySQL (dernière stable) |
| **Type d'instance** | `db.t3.small` | 2 vCPU, 2 Go RAM |
| **Stockage** | `20 Go` | Espace disque SSD (gp3) |
| **Nom d'utilisateur** | `adminadmin` | Compte administrateur DB |
| **Mot de passe** | `••••••••••` | Mot de passe sécurisé |
| **Disponibilité** | `Faible (une seule AZ)` | Mode single-AZ (économique) |
| **Politique de suppression** | `Supprimer` ✅ | DB supprimée si env supprimé |

**Variables d'environnement créées automatiquement** :
```
RDS_HOSTNAME=xxx.rds.amazonaws.com
RDS_PORT=3306
RDS_DB_NAME=ebdb
RDS_USERNAME=adminadmin
RDS_PASSWORD=••••••••••
```

---

## ⚖️ Étape 4 : Configuration trafic et mise à l'échelle

### Instances

#### Volume racine (périphérique de démarrage)

| Paramètre | Valeur | Explication |
|-----------|--------|-------------|
| **Type de volume** | `(Conteneur par défaut)` | Volume EBS standard |
| **Taille** | - Go | Défini par l'AMI |
| **IOPS** | 100 | Opérations I/O par seconde |
| **Débit** | 125 MiB/s | Bande passante disque |

#### Surveillance Amazon CloudWatch

- **Intervalle** : `5 minute`
  - Fréquence de collecte des métriques
  - **Monitoring basique** (gratuit)
  - Monitoring détaillé (1 min) = payant

#### Service de métadonnées d'instance (IMDS)

- **IMDSv1** : ✅ Désactivé
  - **IMDSv2 uniquement** (plus sécurisé)
  - Protection contre SSRF attacks

### Capacité

#### Groupe Auto Scaling

**Type d'environnement** : `Charge équilibrée` ✅
- Plusieurs instances derrière un Load Balancer
- Ajustement automatique selon la charge

**Nombre d'instances** :
- **Minimum** : `2 instances`
  - Nombre minimum d'instances toujours actives
  - **Haute disponibilité** garantie
  
- **Maximum** : `4 instances`
  - Limite supérieure lors des pics de charge
  - **Contrôle des coûts**

#### Composition de la flotte

- **Instances à la demande** ✅ (sélectionné)
  - Instances classiques, toujours disponibles
  - Prix fixe, prévisible
  
- **Combiner les options d'achat** (non sélectionné)
  - Mélange On-Demand + Spot instances
  - Économies possibles mais moins stable

#### Architecture

- **x86_64** ✅ (sélectionné)
  - Architecture processeur Intel/AMD standard
  - Compatible avec la plupart des outils
  
- **arm64** (non sélectionné)
  - Architecture AWS Graviton
  - Meilleur rapport performance/prix

#### Types d'instances

1. **t3.micro**
   - 2 vCPU, 1 Go RAM
   - Performance baseline faible
   - **Développement/test**

2. **t3.small**
   - 2 vCPU, 2 Go RAM
   - Performance baseline modérée
   - **Production légère**

**Recommandation** : au moins 2 types d'instances pour flexibilité

#### ID d'AMI

- `ami-0bc2065c25676f731`
  - Amazon Machine Image (image système)
  - Préconfigurée pour Elastic Beanstalk
  - Contient Docker, agents EB, CloudWatch

#### Zones de disponibilité

- **Any** : Auto Scaling choisit automatiquement
  - Répartition optimale entre AZ disponibles
  - Équilibrage de charge géographique

#### Placement

- **Choisir des zones de disponibilité (AZ)** : grisé
  - Géré automatiquement par Auto Scaling

### Déclencheurs de mise à l'échelle

Configuration des règles d'Auto Scaling basées sur métriques CloudWatch.

#### Métrique surveillée

- **NetworkOut** : Trafic réseau sortant
  - Mesure la quantité de données envoyées
  - Indicateur de charge applicative

**Autres métriques possibles** :
- CPUUtilization (utilisation CPU)
- RequestCount (nombre de requêtes)
- NetworkIn (trafic entrant)

#### Statistique

- **Average** : Moyenne des valeurs
  - Lissage des pics temporaires
  - Évite les scaling trop fréquents

#### Unité

- **Bytes** : Octets de données réseau

#### Période

- **5 Min** : Fenêtre d'évaluation
  - CloudWatch agrège les données sur 5 minutes
  - Balance réactivité et stabilité

#### Durée de la faille

- **5 Min** : Temps avant déclenchement
  - Seuil doit être dépassé pendant 5 minutes consécutives
  - **Évite les faux positifs**

#### Seuil supérieur

- **6000000 Bytes** (6 Mo)
  - Si trafic sortant > 6 Mo pendant 5 min
  - **Déclenche un scale-up**

#### Incrément d'augmentation

- **1 Instances EC2**
  - Ajoute 1 instance à la fois
  - Scaling progressif et contrôlé

#### Seuil inférieur

- **2000000 Bytes** (2 Mo)
  - Si trafic sortant < 2 Mo pendant 5 min
  - **Déclenche un scale-down**

#### Incrément de réduction

- **-1 Instances EC2**
  - Retire 1 instance à la fois
  - Économies de coûts graduelle

**Exemple de fonctionnement** :
```
Trafic > 6 Mo pendant 5 min → +1 instance (2→3)
Trafic > 6 Mo encore 5 min → +1 instance (3→4, max atteint)
Trafic < 2 Mo pendant 5 min → -1 instance (4→3)
```

### Paramètres réseau de l'équilibreur de charge

#### Visibilité

- **Public** ✅
  - Load Balancer accessible depuis Internet
  - Obtient une IP publique et un DNS public
  
**Alternative** :
- **Private** : Load Balancer interne au VPC seulement

#### Double pile (IPv4 et IPv6)

- **Désactivée** ❌
  - IPv4 uniquement
  - IPv6 nécessite un VPC avec configuration IPv6

#### Sous-réseaux de l'équilibreur de charge

**Pas de sous-réseaux affichés dans la capture**
- Elastic Beanstalk utilisera les mêmes que les instances
- ou des sous-réseaux publics si disponibles

### Type d'équilibreur de charge

- **Application Load Balancer (ALB)** ✅
  - **Layer 7 (HTTP/HTTPS)**
  - Routage par URL, headers
  - WebSockets, HTTP/2
  - **Idéal pour applications web**

**Alternatives** :
- **Classic Load Balancer** : Legacy, HTTP/HTTPS/TCP
- **Network Load Balancer** : Layer 4 (TCP/UDP), ultra performance

#### Mode Load Balancer

- **Dédié** ✅
  - Un Load Balancer créé exclusivement pour cet environnement
  
- **Partagé** (option grisée)
  - Partager un LB entre plusieurs environnements
  - Économie de coûts

### Écouteurs

**Configuration actuelle** :

| Port | Protocole | Certificat SSL | Processus | Actif |
|------|-----------|----------------|-----------|-------|
| 80 | HTTP | — | default | ✅ |

- **Port 80** : Port HTTP standard
- **Protocole HTTP** : Trafic non chiffré
- **Processus default** : Target group par défaut
- **Actif** : Écouteur opérationnel

**Pour ajouter HTTPS** :
- Ajouter un écouteur sur port 443
- Attacher un certificat SSL/TLS (ACM)

### Processus

**Configuration actuelle** :

| Nom | Port | Protocole | Code HTTP | Chemin surveillance | Permanence |
|-----|------|-----------|-----------|---------------------|------------|
| default | 80 | HTTP | - | / | Disabled |

- **Nom** : `default` (processus par défaut)
- **Port** : `80` (port cible sur les instances)
- **Protocole** : `HTTP`
- **Chemin surveillance** : `/` (health check path)
- **Permanence (Sticky sessions)** : Désactivée

**Health checks** :
- Le Load Balancer envoie des requêtes GET / toutes les X secondes
- Si l'instance répond 200 OK → healthy
- Si timeout ou erreur → unhealthy → instance remplacée

### Règles

**Aucune règle d'écouteur supplémentaire configurée**

Les **règles** permettent de :
- Rediriger le trafic selon l'URL path
- Router vers différents target groups
- Bloquer ou autoriser selon IP/headers

**Exemple de règle avancée** :
```
Si path = /api/* → Target group "backend"
Si path = /admin/* → Target group "admin-servers"
Sinon → Target group "default"
```

### Accès aux fichiers journaux

**Stocker les journaux** : ❌ Non activé
- Les logs du Load Balancer ne sont **pas stockés dans S3**
- **Pour activer** : cocher la case
- **Frais S3 applicables**

**Si activé** :
- Logs accessibles dans un bucket S3
- Format : IP client, timestamp, requête, réponse, latence
- Utile pour analytics et débogage

---

## 📊 Étape 5 : Surveillance et mises à jour

### Surveillance

#### Création de rapports d'état

**Système de surveillance** : `Amélioré` ✅

- **Basique** :
  - Métriques CloudWatch standards (5 min)
  - Gratuit
  
- **Amélioré** ✅ :
  - Métriques au niveau de l'instance (1 min)
  - Health détaillé par instance
  - Surveillance en temps réel
  - **Coût supplémentaire**

#### Métriques personnalisées CloudWatch

- **Instance** : Non configuré
- **Environnement** : Non configuré

**Usage** :
- Publier des métriques custom depuis l'application
- Exemple : nombre d'utilisateurs connectés, temps de traitement

#### Personnalisation des règles de surveillance de l'état

- **Ignorer l'application 4xx** : ❌ Non activé
  - Les erreurs 4xx (400, 404) affectent le health status
  
- **Ignorer l'équilibreur de charge 4xx** : ❌ Non activé
  - Les 4xx du LB sont comptabilisés

**Si activé** :
- Les erreurs 4xx (erreurs client) n'impactent pas le health
- Seules les erreurs 5xx (serveur) dégradent le statut

#### Diffusion en continu d'événements d'état vers CloudWatch

**Non visible dans les captures mais configurable** :
- Stream des événements EB vers CloudWatch Logs
- Conservation jusqu'à 10 ans
- Utile pour audit et traçabilité

### Mises à jour gérées de la plateforme

#### Mises à jour gérées

- **Activer** : ✅ Activé
  - Elastic Beanstalk applique automatiquement les mises à jour de plateforme
  - **Patches de sécurité** et **nouvelles versions**

#### Fenêtre de mise à jour hebdomadaire

- **Jour** : `Mercredi`
- **Heure** : `06:41 UTC`
  - Moment où EB planifie les updates
  - **Choisir une période de faible trafic**

#### Mettre à jour le niveau

- **Mineur et correctif** ✅
  - Mises à jour mineures (bug fixes, patches)
  - **Pas de breaking changes**
  
**Autres options** :
- **Patch uniquement** : sécurité seulement
- **Toutes versions** : mises à jour majeures (risque)

#### Remplacement de l'instance

- **Désactivé** ❌
  - Les mises à jour se font **in-place** (sans remplacer l'instance)
  - Plus rapide mais moins sûr

**Si activé** :
- EB lance de nouvelles instances avec la nouvelle version
- Puis termine les anciennes (rolling update)
- **Zero-downtime** mais plus long

### Notifications par e-mail

#### E-mail

- **user@example.com** (exemple placeholder)
  - Adresse email pour recevoir les notifications
  - **Événements importants** :
    - Déploiements réussis/échoués
    - Changements de health status
    - Mises à jour de plateforme
    - Erreurs critiques

### Propagation des mises à jour et déploiements

#### Déploiements d'applications

**Politique de déploiement** : `Propagation` ✅

Options disponibles :

1. **Tout à la fois (All at once)**
   - Déploie sur toutes les instances simultanément
   - **Downtime** de quelques secondes
   - Rapide mais risqué

2. **Propagation (Rolling)** ✅
   - Déploie par lot (batch) d'instances
   - **Pas de downtime**
   - Plus lent mais sécurisé

3. **Propagation avec batch supplémentaire**
   - Crée des instances temporaires pour éviter réduction capacité
   - **Capacité constante**

4. **Immutable**
   - Crée un nouveau groupe d'instances
   - Bascule le trafic si succès
   - **Rollback facile**

#### Type de taille de lot

- **Pourcentage** ✅ (sélectionné)
  - Définit la taille du batch en %
  
- **Fixe** (non sélectionné)
  - Nombre fixe d'instances par batch

#### Taille du lot de déploiement

- **30%** d'instances à la fois
  - Avec 4 instances : 30% × 4 = 1.2 ≈ **1 instance par batch**
  - Déploiement en 4 vagues
  - **Équilibre sécurité/vitesse**

**Exemple avec 4 instances** :
```
Batch 1: Instance A mise à jour → OK → 
Batch 2: Instance B mise à jour → OK →
Batch 3: Instance C mise à jour → OK →
Batch 4: Instance D mise à jour → OK ✅
```

#### Répartition du trafic

- Non renseigné dans capture
- % de trafic vers la nouvelle version
- Utile pour tests A/B ou canary deployments

#### Durée d'évaluation de la répartition du trafic

- Non renseigné
- Temps d'observation avant basculement complet

#### Mises à jour de configuration

**Propagation du type de mise à jour** : `Désactivé`
- Les changements de configuration **ne déclenchent pas de rolling update**
- Appliqués immédiatement sur toutes les instances

**Si activé (Rolling)** :
- Les changements de config (variables env, etc.) se propagent par batch
- Évite les redémarrages simultanés

#### Taille de lot

- Instances à mettre à jour par batch pour les config changes

### Logiciel de plateforme

#### Options de conteneur

**Serveur proxy** : `Nginx` ✅
- Nginx agit comme reverse proxy devant Docker
- **Avantages** :
  - Gestion SSL/TLS
  - Compression gzip
  - Cache statique
  - Logs d'accès

**Alternative** :
- **Aucun** : trafic direct vers le conteneur

#### Amazon X-Ray

**Démon X-Ray** : ❌ Désactivé
- Service de tracing distribué AWS
- **Si activé** :
  - Trace les requêtes à travers les services
  - Analyse de performance
  - Détection de goulots d'étranglement
  - **Coût supplémentaire**

#### Stockage des journaux S3

**Rotation des journaux** : ❌ Désactivée
- Les logs ne sont **pas archivés dans S3**

**Si activé** :
- Elastic Beanstalk envoie les logs vers S3 toutes les X heures
- Conservation long terme
- Analyse avec Athena ou autres outils

#### Diffusion des journaux d'instances vers CloudWatch

**Diffusion de journaux** : ❌ Désactivée
- Les logs des instances **ne sont pas streamés vers CloudWatch Logs**

**Si activé** :
- Logs en temps réel dans CloudWatch
- Recherche et filtrage avancés
- Alarmes basées sur patterns
- Intégration avec Lambda

### Propriétés de l'environnement

**Aucune propriété configurée**

Les **propriétés** sont des **variables d'environnement** injectées dans l'application.

**Exemples d'usage** :
```bash
API_KEY=abc123xyz
ENVIRONMENT=production
DEBUG_MODE=false
EXTERNAL_SERVICE_URL=https://api.example.com
```

Ces variables sont accessibles dans le conteneur Docker :
```python
import os
api_key = os.environ.get('API_KEY')
```

---

## ✅ Étape 6 : Vérification

### Page récapitulative

La page de vérification affiche un **résumé de toutes les configurations** avant création :

#### Étape 1 : Configurer l'environnement

| Paramètre | Valeur |
|-----------|--------|
| Niveau d'environnement | Environnement de serveur web |
| Nom de l'application | M2i-Anselme-EB |
| Nom de l'environnement | M2i-Anselme-EB-env |
| Plateforme | Docker running on 64bit Amazon Linux 2023/4.9.2 |
| Code de l'application | Exemple d'application |

#### Étape 2 : Configurer l'accès au service

| Paramètre | Valeur |
|-----------|--------|
| Fonction du service | arn:aws:iam::925037323203:role/service-role/aws-elasticbeanstalk-service-role |
| Paire de clés EC2 | toto |
| Profil d'instance EC2 | aws-elasticbeanstalk-ec2-role |

#### Étape 3 : Configurer réseau, base de données et identifications

**Réseau** :
- VPC : vpc-08104c570e4699f01
- Sous-réseaux instances : us-east-1b, us-east-1a
- Sous-réseaux DB : us-east-1b, us-east-1a

**Base de données** :
- Moteur : MySQL 8.4.7
- Instance : db.t3.small
- Stockage : 20 Go
- Utilisateur : adminadmin

#### Étape 4 : Configurer trafic et mise à l'échelle

**Instances** :
- Volume racine : Conteneur par défaut
- CloudWatch : 5 minutes
- IMDSv1 : Désactivé

**Capacité** :
- Type : Charge équilibrée
- Instances : 2 (min) à 4 (max)
- Types : t3.micro, t3.small

**Load Balancer** :
- Type : Application Load Balancer (dédié)
- Visibilité : Public
- Écouteur : Port 80 HTTP

#### Étape 5 : Surveillance et mises à jour

**Surveillance** :
- Système : Amélioré

**Mises à jour** :
- Gérées : Activées
- Fenêtre : Mercredi 06:41 UTC
- Niveau : Mineur et correctif

**Déploiement** :
- Politique : Propagation
- Taille lot : 30%

**Plateforme** :
- Proxy : Nginx
- X-Ray : Désactivé
- Logs S3 : Désactivés

---

## 🚀 Démarrage de l'environnement

### Page de lancement

Une fois validé, Elastic Beanstalk démarre la création de l'environnement.

**Informations affichées** :

- **État** : `Unknown` (en cours de démarrage)
- **ID de l'environnement** : `e-kty27yo2gq`
- **Nom de l'application** : `M2i-Anselme-EB`
- **Nom de l'environnement** : `M2i-Anselme-EB-env`
- **Plateforme** : `Docker running on 64bit Amazon Linux 2023/4.9.2`
- **Version en cours d'exécution** : `–` (pas encore déployée)
- **État de la plateforme** : `Supported` ✅

### Événements

**Événements visibles** :

1. **"Using elasticbeanstalk-us-east-1-925037323203 as Amazon S3 storage bucket for environment data"**
   - EB crée un bucket S3 pour stocker les artefacts
   - Format : `elasticbeanstalk-{region}-{account-id}`

2. **"createEnvironment is starting"**
   - Démarrage du processus de création

**Événements à venir (non visibles mais attendus)** :

```
✅ Creating security groups
✅ Creating Auto Scaling group
✅ Creating Launch Template
✅ Launching EC2 instances
✅ Installing Docker
✅ Deploying sample application
✅ Configuring Load Balancer
✅ Registering targets
✅ Health checks passing
✅ Successfully launched environment
```

**Durée estimée** : 5-15 minutes

---

## 📊 Récapitulatif des ressources créées

À la fin du processus, Elastic Beanstalk aura créé automatiquement :

### Ressources réseau

- ✅ 1 Application Load Balancer (ALB)
- ✅ 1 Target Group
- ✅ 2+ Security Groups :
  - SG pour le Load Balancer (port 80 ouvert au public)
  - SG pour les instances EC2 (port 80 depuis LB uniquement)
  - SG pour RDS (port 3306 depuis instances EC2)

### Ressources de calcul

- ✅ 1 Launch Template (configuration instance)
- ✅ 1 Auto Scaling Group
- ✅ 2 à 4 instances EC2 (t3.micro/t3.small)
- ✅ Chaque instance contient :
  - Amazon Linux 2023
  - Docker Engine
  - Nginx (reverse proxy)
  - Elastic Beanstalk HealthD agent
  - CloudWatch Logs agent

### Ressources de base de données

- ✅ 1 instance RDS MySQL 8.4.7 (db.t3.small)
- ✅ 1 DB Subnet Group
- ✅ 20 Go de stockage SSD

### Ressources de stockage

- ✅ 1 bucket S3 pour logs et déploiements
- ✅ Volumes EBS pour chaque instance EC2

### Ressources de surveillance

- ✅ CloudWatch Metrics (CPU, réseau, requêtes)
- ✅ CloudWatch Alarms pour Auto Scaling
- ✅ Elastic Beanstalk Enhanced Health Monitoring

### IAM

- ✅ 1 Service Role (aws-elasticbeanstalk-service-role)
- ✅ 1 Instance Profile (aws-elasticbeanstalk-ec2-role)

---

## 💰 Estimation des coûts

**Ressources facturées** (région us-east-1, estimation mensuelle) :

| Ressource | Quantité | Coût unitaire | Coût mensuel |
|-----------|----------|---------------|--------------|
| EC2 t3.micro (2 min) | 2 × 730h | $0.0104/h | ~$15 |
| EC2 t3.small (2 max) | 2 × 100h | $0.0208/h | ~$4 |
| Application Load Balancer | 1 | $16/mois + trafic | ~$20 |
| RDS db.t3.small (single-AZ) | 1 × 730h | $0.034/h | ~$25 |
| RDS stockage SSD | 20 Go | $0.115/Go | ~$2.30 |
| Volumes EBS (gp3) | 4 × 8 Go | $0.08/Go | ~$2.56 |
| **Total estimé** | | | **~$69/mois** |

**Services gratuits** :
- Elastic Beanstalk lui-même (service gratuit)
- Transfert de données (dans la limite du Free Tier)

---

## 🔍 Points d'attention et bonnes pratiques

### Sécurité

✅ **Bonnes pratiques appliquées** :
- IMDSv2 obligatoire (protection SSRF)
- Instances sans IP publique
- Accès via Load Balancer uniquement
- Security Groups restrictifs

⚠️ **Améliorations possibles** :
- [ ] Activer HTTPS (certificat SSL/TLS)
- [ ] Activer RDS Multi-AZ (haute disponibilité DB)
- [ ] Chiffrer les volumes EBS
- [ ] Activer RDS encryption at rest
- [ ] Utiliser Secrets Manager pour les credentials DB

### Performance

✅ **Bien configuré** :
- Auto Scaling avec min 2, max 4
- Répartition multi-AZ
- Enhanced monitoring

⚠️ **Optimisations possibles** :
- [ ] Utiliser t3.small comme instance minimum
- [ ] Configurer CloudFront (CDN) devant l'ALB
- [ ] Activer le cache Nginx pour assets statiques

### Coûts

⚠️ **Optimisations possibles** :
- [ ] Utiliser Spot Instances (économie jusqu'à 90%)
- [ ] RDS Reserved Instance (économie 30-60%)
- [ ] Auto Scaling scheduled actions (scale-down la nuit)
- [ ] Désactiver la surveillance améliorée (économie mineure)

### Disponibilité

✅ **Haute disponibilité** :
- Multi-AZ pour instances EC2
- Load Balancer avec health checks
- Auto Scaling automatique

⚠️ **Production-ready** :
- [ ] RDS Multi-AZ (failover automatique DB)
- [ ] Min 2 instances → considérer min 3 pour éviter sous-capacité
- [ ] Configurer des alarmes CloudWatch

---

## 🎓 Concepts clés à retenir

### 1. Elastic Beanstalk = PaaS

- Vous fournissez : **code + configuration**
- AWS gère : **infrastructure + déploiement + scaling + monitoring**

### 2. Haute disponibilité

- **Multi-AZ** : répartition dans plusieurs zones
- **Load Balancer** : distribution du trafic
- **Auto Scaling** : ajustement automatique de capacité

### 3. Rolling deployments

- Déploiement progressif par batch
- **Zero-downtime** maintenu
- Rollback possible en cas d'échec

### 4. Monitoring intégré

- CloudWatch Metrics automatiques
- Enhanced Health Monitoring
- Logs centralisés (optionnel S3/CloudWatch)

### 5. Coût = ressources sous-jacentes

- Elastic Beanstalk est **gratuit**
- Vous payez EC2, RDS, ALB, etc.
- **Optimisation possible** avec Reserved Instances et Spot

---

## 📚 Ressources additionnelles

- [Documentation AWS Elastic Beanstalk](https://docs.aws.amazon.com/elasticbeanstalk/)
- [Guide de déploiement Docker sur EB](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_docker.html)
- [Auto Scaling Best Practices](https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-scaling-best-practices.html)
- [Application Load Balancer Documentation](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/)

---

**Document créé le** : 5 février 2026  
**Auteur** : Lab AWS M2i  
**Version** : 1.0
