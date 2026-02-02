# Résumé — Infrastructure Globale d'AWS

## 📍 Architecture Globale d'AWS

### Structure hiérarchique

AWS déploie son infrastructure à l'échelle mondiale selon une hiérarchie bien définie :

```
Monde (Infrastructure globale)
  ↓
Régions géographiques (Zones géographiques majeures)
  ├─ Datacenters
  │   └─ Zones de disponibilité (AZ)
  │       └─ 3 AZ minimum par région
  │
  └─ Edge Locations (Points de présence périphériques)
```

---

## 🌐 1. Régions (Regions)

**Définition**: Une région AWS est une zone géographique isolée contenant un ensemble complet d'infrastructures AWS.

**Caractéristiques** :
- Réparties à l'échelle mondiale (ex : us-east-1, eu-west-1, ap-northeast-1)
- Chaque région est **complètement indépendante** des autres
- Contiennent au minimum **3 zones de disponibilité**
- Offrent une **résilience** et une **faible latence** locale

**Exemples** :
- N. Virginia (us-east-1)
- Irlande (eu-west-1)
- Paris (eu-west-3)
- Tokyo (ap-northeast-1)

---

## 🏢 2. Zones de Disponibilité (Availability Zones — AZ)

**Définition**: Une zone de disponibilité est un **datacenter AWS physique** isolé dans une région.

**Caractéristiques** :
- **Minimum 3 AZ par région** pour garantir la redondance
- Connectées entre elles par **fibres optiques haut débit**
- Isolées l'une de l'autre pour éviter les défaillances en cascade
- **Chaque AZ = 1 datacenter complet** avec alimentation, refroidissement, réseau indépendants

**Avantage** :
- **Haute disponibilité** : si une AZ tombe, les autres continuent
- **Faible latence** : communication inter-AZ quasi instantanée

**Exemple** :
- Région Paris (eu-west-3) contient 3 AZ :
  - eu-west-3a
  - eu-west-3b
  - eu-west-3c

---

## 🌍 3. Edge Locations (Points de présence périphériques)

**Définition**: Des points de présence situés à proximité des utilisateurs, en dehors des régions principales.

**Caractéristiques** :
- **Plus nombreuses** que les régions (300+ edge locations mondiale)
- Utilisées pour **CDN** (Content Delivery Network) et **caching**
- Réduisent la latence pour les utilisateurs éloignés

**Services utilisant les Edge Locations** :
- **CloudFront** (CDN — livraison de contenu rapide)
- **Route 53** (DNS global)
- **AWS Shield** (protection DDoS)

**Exemple** :
- Contenu statique (images, vidéos) cachés à proximité de l'utilisateur
- Chargement ultra-rapide grâce aux edge locations

---

## 🛠️ 4. Critères de choix d'une région

Avant de choisir une région, prenez en compte les éléments suivants :

### **1️⃣ Disponibilité des services**
- Tous les services AWS ne sont pas disponibles dans toutes les régions
- Vérifier que le service dont vous avez besoin existe dans la région ciblée

### **2️⃣ Proximité des utilisateurs**
- Choisir une région proche de vos utilisateurs finaux
- Réduction de la latence = meilleure performance

### **3️⃣ Conformité et gouvernance des données**
- Réglementations (RGPD, HIPAA, etc.)
- Localisation obligatoire des données (ex : données européennes en Europe)
- Restrictions gouvernementales

### **4️⃣ Coût des services**
- Les tarifs varient d'une région à l'autre
- Exemple : une instance EC2 coûte moins cher en Virginie qu'en Paris
- Arbitrage entre coût et performance

### **5️⃣ Support AWS et SLA**
- Support technique plus proche
- Conformité avec les SLA (Service Level Agreements)

---

## 🌐 5. Services Régionaux vs Services Globaux

### **Services Régionaux** 🔒
**Définition** : Services limités à une région spécifique.

**Caractéristiques** :
- Déployés dans une seule région
- Données stockées dans cette région
- Bon pour la conformité et la localisation

**Exemples** :
- **EC2** (instances de calcul)
- **RDS** (bases de données relationnelles)
- **S3** (stockage — bien que global, les buckets sont régionaux)
- **VPC** (réseau privé virtuel)
- **Lambda** (fonctions sans serveur)

### **Services Globaux** 🌍
**Définition** : Services disponibles partout, sans référence à une région spécifique.

**Caractéristiques** :
- Accessibles depuis n'importe quelle région
- Offrent une couverture mondiale
- Idéaux pour les CDN et les services frontal

**Exemples** :
- **CloudFront** (CDN — livraison de contenu)
- **Route 53** (DNS global)
- **IAM** (Identity and Access Management)
- **AWS Shield** (protection DDoS)
- **AWS WAF** (Web Application Firewall)

---

## 💻 6. Moyens d'interagir avec AWS

Pour accéder et gérer vos ressources AWS, vous disposez de **3 interfaces principales** :

### **1️⃣ AWS Management Console** 🖥️
- **Interface web graphique** (GUI)
- Accès via navigateur : https://console.aws.amazon.com
- **Idéale pour** : configuration, monitoring, apprentissage
- **Avantage** : visuelle et intuitive

### **2️⃣ AWS Command Line Interface (AWS CLI)** ⌨️
- **Interface ligne de commande** pour scripter et automatiser
- Installation locale sur votre machine
- **Commandes type** :
  ```bash
  aws ec2 describe-instances --region eu-west-3
  aws s3 ls
  aws iam list-users
  ```
- **Idéale pour** : automatisation, CI/CD, gestion batch

### **3️⃣ AWS SDK** 📚
- **Kits de développement logiciel** pour plusieurs langages
- Langages supportés : Python, Node.js, Java, Go, C++, etc.
- Intégration directe dans votre code applicatif
- **Idéale pour** : développement d'applications cloud-native

**Exemple Python (SDK Boto3)** :
```python
import boto3

ec2 = boto3.client('ec2', region_name='eu-west-3')
instances = ec2.describe_instances()
```

---

## 📊 Synthèse en tableau

| Concept | Description | Exemple |
|---------|-------------|---------|
| **Région** | Zone géographique avec datacenters complets | us-east-1, eu-west-3 |
| **Zone de disponibilité (AZ)** | 1 datacenter physique dans une région | eu-west-3a, eu-west-3b |
| **Edge Location** | Point de présence pour CDN et caching | 300+ emplacements mondiaux |
| **Service régional** | Disponible dans une région | EC2, RDS, S3 |
| **Service global** | Disponible partout | CloudFront, Route 53, IAM |
| **Interface AWS** | Moyen d'accès (Console, CLI, SDK) | Web browser, terminal, code |

---

## 🎯 Résumé clé à retenir

✅ **Infrastructure** : Régions → Datacenters (AZ) → Edge Locations  
✅ **Minimum 3 AZ par région** pour la redondance  
✅ **Choisir une région** selon : service, proximité, conformité, coût  
✅ **Services régionaux** : limités à une région (EC2, RDS)  
✅ **Services globaux** : disponibles partout (CloudFront, Route 53)  
✅ **3 façons d'accéder à AWS** : Console, CLI, SDK  

---

## 📚 Documentation officielle AWS — Pour aller plus loin

### Infrastructure globale
- [Vue d'ensemble de l'infrastructure mondiale AWS](https://aws.amazon.com/fr/about-aws/global-infrastructure/)
- [Régions et zones de disponibilité](https://aws.amazon.com/fr/about-aws/global-infrastructure/regions_availability-zones/)
- [Localisation des services par région](https://aws.amazon.com/fr/about-aws/global-infrastructure/regional-product-services/)
- [Edge Locations et CloudFront](https://aws.amazon.com/fr/cloudfront/details/)

### Services régionaux et globaux
- [FAQ — Régions et zones de disponibilité](https://aws.amazon.com/fr/faqs/which-services-region/)
- [Services globaux AWS](https://docs.aws.amazon.com/general/latest/gr/global_services.html)

### Interfaces d'accès
- [AWS Management Console](https://console.aws.amazon.com/)
- [AWS CLI — Guide utilisateur](https://docs.aws.amazon.com/cli/latest/userguide/)
- [AWS SDK pour Python (Boto3)](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html)
- [AWS SDK pour Node.js](https://docs.aws.amazon.com/sdk-for-javascript/)

### Tarification
- [AWS Pricing Calculator](https://calculator.aws/)
- [Modèles de tarification AWS](https://aws.amazon.com/fr/pricing/)

### Conformité et gouvernance
- [Conformité AWS par région](https://aws.amazon.com/fr/compliance/)
- [Résidences des données AWS](https://aws.amazon.com/fr/about-aws/global-infrastructure/data-residency/)
