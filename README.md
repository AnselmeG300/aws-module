# Module Amazon Web Services — Objectifs pédagogiques et rythme (5 jours)

**Durée**: 5 jours • **Activités Type (AT)**: 1, 3 • **Compétences (CP)**: 3, 4, 9

---

## Objectifs Globaux

- Comprendre les avantages du modèle Cloud computing
- Appréhender les services fondamentaux de AWS
- Exploiter les services fondamentaux (Compute, Storage, Network)
- Mettre en œuvre les services principaux au sein d'AWS

---

## Objectifs Détaillés

### 1. Généralités
- Comprendre les principes du Cloud Computing
- Mettre en œuvre les services principaux au sein d'AWS

### 2. Les principes du Cloud
**Les modèles principaux**
- **IaaS** (Infrastructure as a Service): infrastructure virtualisée à la demande (serveurs, stockage, réseau)
- **PaaS** (Platform as a Service): plateforme de développement et déploiement sans gérer l'infrastructure
- **SaaS** (Software as a Service): applications prêtes à l'emploi accessibles via Internet

### 3. Plateforme AWS et accès
- **Infrastructure Globale**: régions, zones de disponibilité, zones locales et réseau mondial AWS
- **Comptes AWS et Organisations**: gestion multi-comptes, contrôle centralisé des politiques et de la facturation
- **Console, CLI et API**: interfaces graphiques et programmatiques pour administrer et automatiser AWS

### 4. Amazon Identity and Access Management (IAM)
- **Gestion Utilisateurs, Groupes, Roles**: identités et délégation d'accès pour personnes, applications et services
- **Définition de Stratégies**: documents JSON définissant des autorisations de manière fine (Allow/Deny, conditions)

### 5. Amazon VPC
- **Création et gestion d'un VPC**: espace réseau isolé avec sous-réseaux, tables de routage et passerelles
- **Réseaux publics, privés, NAT**: exposition contrôlée à Internet et sortie sécurisée depuis des sous-réseaux privés
- **Sécurisation avec les SGs** (Security Groups): pare-feu d'état au niveau instance pour filtrer le trafic

### 6. Amazon EC2
- **Création et gestion de VMs**: choix des AMI, types d'instances, clés SSH, EBS, démarrage/arrêt et sécurité

### 7. Amazon S3
- **Création et gestion de Buckets**: classes de stockage, versioning, lifecycle et chiffrement
- **Gérer l'accès via IAM et Bucket Policies**: gouvernance d'accès au niveau identité et ressource

### 8. Amazon Elastic Beanstalk
- **Déploiement d'applications**: plateforme managée pour provisionner, déployer et scaler sans gérer l'infrastructure

---

## Rythme Pédagogique (5 jours)

### **Jour 1 — Introduction au Cloud Computing et AWS**
**Objectifs visés**: Généralités, Principes du Cloud

**Contenu théorique**:
- Comprendre les avantages du modèle Cloud computing (agilité, coûts, élasticité, résilience)
- Différencier les modèles de service: IaaS, PaaS, SaaS
- Différencier les modèles de déploiement: Public, Privé, Hybride

**Activités pratiques**:
- Découverte des services AWS via la console
- Panorama des services fondamentaux (Compute, Storage, Network)

**Livrables**: 
- Fiche de synthèse IaaS/PaaS/SaaS
- Compréhension des avantages du cloud

**Labs associés**: LAB-01-decouverte-services-aws.md

---

### **Jour 2 — Infrastructure Globale AWS et Accès aux Services**
**Objectifs visés**: Infrastructure Globale, Comptes AWS, Méthodes d'accès

**Contenu théorique**:
- Architecture de l'infrastructure globale AWS (Régions, Zones de Disponibilité)
- Gestion des comptes AWS et Organizations
- Comparaison des coûts entre régions

**Activités pratiques**:
- Comparaison des coûts entre régions
- Prise en main des différentes méthodes d'interaction avec AWS (Console, CLI, API)
- Configuration de l'environnement de travail (AWS CLI)

**Livrables**: 
- Environnement CLI opérationnel
- Analyse comparative des régions

**Labs associés**: 
- LAB-02-comparaison-couts-regions.md
- LAB-03-interactions-AWS.md

---

### **Jour 3 — Sécurité et Gestion des Identités (IAM)**
**Objectifs visés**: Maîtriser Amazon IAM

**Contenu théorique**:
- Principes de sécurité AWS
- Gestion des utilisateurs, groupes et rôles
- Création et application de stratégies (policies) IAM
- Bonnes pratiques de sécurité (MFA, principe du moindre privilège)

**Activités pratiques**:
- Création d'utilisateurs, groupes et rôles IAM
- Rédaction et test de policies personnalisées
- Mise en place de l'authentification multi-facteurs (MFA)
- Application du principe du moindre privilège

**Livrables**: 
- Politiques IAM validées
- Environnement sécurisé avec utilisateurs et rôles configurés
- Documentation des bonnes pratiques de sécurité

**Labs associés**: LAB-04-IAM.md

---

### **Jour 4 — Services de Calcul et Stockage (EC2 & S3)**
**Objectifs visés**: Maîtriser Amazon EC2 et Amazon S3

**Contenu théorique**:
- Amazon EC2: types d'instances, AMI, EBS, cycle de vie
- Amazon S3: architecture, classes de stockage, versioning, lifecycle
- Bonnes pratiques de gestion des instances et du stockage

**Activités pratiques**:
- Création et gestion d'instances EC2 (lancement, connexion SSH, configuration)
- Déploiement d'une application sur EC2 avec Elastic Beanstalk (Odoo)
- Création et gestion de buckets S3
- Configuration des politiques d'accès S3 (IAM et Bucket Policies)
- Mise en place du versioning et des règles de cycle de vie

**Livrables**: 
- Instance EC2 fonctionnelle
- Bucket S3 sécurisé avec règles de cycle de vie
- Application Odoo déployée sur Elastic Beanstalk

**Labs associés**: 
- LAB-05-service-calcul-ec2.md
- LAB-05-01-elastic-beanstalk-odoo.md

---

### **Jour 5 — Services Réseau et Déploiement d'Applications**
**Objectifs visés**: Maîtriser Amazon VPC et Elastic Beanstalk

**Contenu théorique**:
- Amazon VPC: architecture réseau, sous-réseaux publics/privés
- Tables de routage, Internet Gateway, NAT Gateway
- Security Groups et Network ACLs
- Elastic Beanstalk: déploiement et gestion d'applications
- Bonnes pratiques transverses (sécurité, coûts, tagging)

**Activités pratiques**:
- Création d'un VPC complet (subnets publics/privés, routes, IGW/NAT)
- Configuration des Security Groups
- Déploiement d'une application Java sur Elastic Beanstalk
- Monitoring et logs
- Mise en place d'une architecture complète et sécurisée

**Livrables**: 
- Diagramme d'architecture VPC
- Application Java déployée sur Elastic Beanstalk
- Check-list de bonnes pratiques
- Documentation de l'architecture complète

**Labs associés**: 
- LAB-06-vpc-reseau-complet.md
- LAB-07-elastic-beanstalk-java.md

---

## Capacités Attendues

À l'issue de cette formation, les apprenants seront capables de:
- ✅ Comprendre les avantages du modèle cloud computing (agilité, coûts, élasticité)
- ✅ Appréhender les services fondamentaux d'AWS et leurs cas d'usage
- ✅ Exploiter les services clés (Compute, Storage, Network) de manière sécurisée
- ✅ Créer et gérer une infrastructure AWS complète (VPC, EC2, S3)
- ✅ Sécuriser les ressources avec IAM et les Security Groups
- ✅ Déployer des applications avec Elastic Beanstalk
- ✅ Appliquer les bonnes pratiques AWS en termes de sécurité, coûts et architecture

---

## Récapitulatif des Labs par Jour

| Jour | Labs |
|------|------|
| **Jour 1** | LAB-01-decouverte-services-aws.md |
| **Jour 2** | LAB-02-comparaison-couts-regions.md<br>LAB-03-interactions-AWS.md |
| **Jour 3** | LAB-04-IAM.md |
| **Jour 4** | LAB-05-service-calcul-ec2.md<br>LAB-05-01-elastic-beanstalk-odoo.md |
| **Jour 5** | LAB-06-vpc-reseau-complet.md<br>LAB-07-elastic-beanstalk-java.md |

---

## Progression Pédagogique

Cette formation suit une progression logique et incrémentale:

1. **Fondations** (Jours 1-2): Compréhension des concepts et de la plateforme
2. **Sécurité** (Jour 3): Base essentielle avant toute manipulation
3. **Services Core** (Jour 4): Compute et Storage, briques fondamentales
4. **Intégration** (Jour 5): Réseau et déploiement d'applications complètes

Chaque journée combine théorie et pratique avec des labs hands-on pour consolider les acquis.
