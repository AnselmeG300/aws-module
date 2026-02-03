# 🔐 Résumé - Module Sécurité AWS

## 📚 Vue d'ensemble

Ce module couvre les **services et concepts de sécurité essentiels** dans AWS, permettant de sécuriser vos applications, données et infrastructure cloud.

---

## 1️⃣ Modèle de Responsabilité Partagée (Shared Responsibility Model)

### 🎯 Concept clé

AWS et le client **partagent les responsabilités de sécurité** :

| Responsabilité | AWS | Client |
|----------------|-----|--------|
| **Sécurité DU cloud** | ✅ Infrastructure physique, réseau, hyperviseur | ❌ |
| **Sécurité DANS le cloud** | ❌ | ✅ Données, IAM, chiffrement, OS, applications |

### 📖 Principe simple

- **AWS** = Sécurise l'infrastructure (datacenters, serveurs, réseau)
- **Client** = Sécurise ce qu'il met DANS le cloud (données, accès, configuration)

### 📌 Exemple concret

| Service | AWS responsable de | Client responsable de |
|---------|-------------------|----------------------|
| **EC2** | Hyperviseur, réseau physique | OS, patches, pare-feu, données |
| **RDS** | Infrastructure, backups automatiques | Chiffrement, IAM, accès réseau |
| **S3** | Stockage physique, réplication | Chiffrement objets, permissions bucket |

### 🔗 Documentation officielle
- [Shared Responsibility Model](https://aws.amazon.com/compliance/shared-responsibility-model/)

---

## 2️⃣ AWS IAM (Identity and Access Management)

### 🎯 Rôle

**Contrôler qui peut accéder à quoi** dans AWS de manière sécurisée et granulaire.

### 🔑 Composants principaux

#### A) **IAM User** (Utilisateur)
- Identité individuelle avec credentials (nom d'utilisateur + mot de passe ou clés d'accès)
- Exemple : `M2i_John` pour un développeur

#### B) **IAM Policy** (Politique de permissions)
- Document JSON définissant **ce qui est autorisé ou refusé**
- Exemple : Autoriser la lecture de S3 mais pas l'écriture

```json
{
  "Effect": "Allow",
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::my-bucket/*"
}
```

#### C) **IAM Group** (Groupe)
- Conteneur pour **regrouper des utilisateurs** avec les mêmes permissions
- Exemple : Groupe "DevOps" avec accès EC2 + S3

#### D) **IAM Role** (Rôle)
- Permissions **temporaires** pour des services AWS (EC2, Lambda) ou utilisateurs externes
- Exemple : Rôle permettant à une instance EC2 d'accéder à S3 sans clés en dur

#### E) **MFA (Multi-Factor Authentication)**
- **Double authentification** pour renforcer la sécurité
- Exemple : Mot de passe + code de l'application Google Authenticator

### 📌 Bonne pratique

✅ **Ne jamais utiliser le compte root** au quotidien  
✅ **Activer MFA** sur tous les comptes sensibles  
✅ **Principe du moindre privilège** : donner uniquement les permissions nécessaires  

### 🔗 Documentation officielle
- [AWS IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html)
- [IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)

---

## 3️⃣ AWS Organizations

### 🎯 Rôle

**Consolider et gérer plusieurs comptes AWS** à partir d'un compte central (organisation).

### 🏢 Cas d'usage

- Entreprise avec plusieurs départements (Dev, Prod, Finance)
- Chaque département a son propre compte AWS
- AWS Organizations permet de tout gérer depuis un seul endroit

### 🔑 Fonctionnalités principales

#### A) **Gestion centralisée des comptes**
- Créer de nouveaux comptes AWS automatiquement
- Facturation consolidée (une seule facture pour tous les comptes)

#### B) **Service Control Policies (SCP)**
- Politiques de sécurité **appliquées à tous les comptes** de l'organisation
- Exemple : Interdire la création d'instances en dehors de `us-east-1`

#### C) **Hiérarchie organisationnelle**

```
Root (Racine)
├── Organizational Unit (OU): Production
│   ├── Compte AWS: Prod-Web
│   └── Compte AWS: Prod-DB
└── Organizational Unit (OU): Development
    ├── Compte AWS: Dev-Team1
    └── Compte AWS: Dev-Team2
```

- **Root** : Racine de l'organisation (compte principal)
- **OU (Organizational Unit)** : Regroupement logique de comptes
- **Comptes membres** : Comptes AWS individuels

### 📌 Exemple concret

**Scénario** : Interdire la suppression de logs CloudTrail dans tous les comptes

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "cloudtrail:DeleteTrail",
      "Resource": "*"
    }
  ]
}
```

### 🔗 Documentation officielle
- [AWS Organizations](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_introduction.html)
- [Service Control Policies](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html)

---

## 4️⃣ AWS Artifact

### 🎯 Rôle

**Fournir des rapports de conformité et de sécurité** AWS à la demande (self-service).

### 📄 Que contient AWS Artifact ?

- Rapports de conformité : ISO, PCI-DSS, SOC, HIPAA
- Accords légaux : BAA (Business Associate Agreement), GDPR
- Audits de sécurité AWS

### 📌 Cas d'usage

**Scénario** : Votre entreprise doit prouver qu'AWS est conforme à la norme ISO 27001 pour un audit.

➡️ Téléchargez le rapport ISO 27001 depuis **AWS Artifact** et fournissez-le à l'auditeur.

### 🔗 Documentation officielle
- [AWS Artifact](https://docs.aws.amazon.com/artifact/latest/ug/what-is-aws-artifact.html)

---

## 5️⃣ AWS WAF (Web Application Firewall)

### 🎯 Rôle

**Protéger les applications web et API** contre les attaques courantes.

### 🛡️ Types d'attaques bloquées

| Attaque | Description | Exemple |
|---------|-------------|---------|
| **SQL Injection** | Injection de code SQL malveillant | `' OR 1=1 --` dans un formulaire |
| **XSS (Cross-Site Scripting)** | Injection de scripts malveillants | `<script>alert('Hacked')</script>` |
| **HTTP Flood** | Surcharge de requêtes HTTP | Milliers de requêtes/seconde |

### 🔑 Fonctionnalités

#### A) **Règles personnalisées (AWS WAF Rules)**
- Bloquer les requêtes contenant des patterns spécifiques
- Exemple : Bloquer toutes les requêtes contenant `SELECT * FROM`

#### B) **Managed Rules** (Règles gérées)
- Règles préconfigurées par AWS ou partenaires (OWASP Top 10)

### 📌 Exemple concret

**Scénario** : Protéger une API contre les SQL Injections

➡️ Créer une règle WAF qui bloque toutes les requêtes contenant `SELECT`, `DROP`, `UNION`

### 🔗 Documentation officielle
- [AWS WAF](https://docs.aws.amazon.com/waf/latest/developerguide/waf-chapter.html)
- [AWS WAF Rules](https://docs.aws.amazon.com/waf/latest/developerguide/waf-rules.html)

---

## 6️⃣ AWS Shield

### 🎯 Rôle

**Protection contre les attaques DDoS** (Distributed Denial of Service).

### 🛡️ Deux niveaux

| Niveau | Couverture | Coût | Protection |
|--------|-----------|------|-----------|
| **Shield Standard** | Automatique pour tous les clients | **Gratuit** | Protection de base contre DDoS (couche 3/4) |
| **Shield Advanced** | Protection avancée + support 24/7 | **$3,000/mois** | Protection avancée + WAF gratuit + support DRT |

### 📌 Attaques DDoS courantes

- **Volumétrique** : Surcharge de bande passante (UDP flood, ICMP flood)
- **Protocole** : Exploitation des faiblesses de protocoles (SYN flood)
- **Application** : Surcharge de requêtes HTTP

### 🔗 Documentation officielle
- [AWS Shield](https://docs.aws.amazon.com/waf/latest/developerguide/shield-chapter.html)
- [Shield Standard vs Advanced](https://aws.amazon.com/shield/features/)

---

## 7️⃣ AWS KMS (Key Management Service)

### 🎯 Rôle

**Gérer et créer des clés de chiffrement** pour protéger les données au repos et en transit.

### 🔐 Fonctionnalités principales

#### A) **Chiffrement des données**
- S3 : Chiffrer les objets stockés
- EBS : Chiffrer les volumes des instances EC2
- RDS : Chiffrer les bases de données

#### B) **Types de clés**

| Type | Gestion | Usage |
|------|---------|-------|
| **AWS Managed Keys** | Gérées par AWS | Services AWS (S3, RDS) |
| **Customer Managed Keys (CMK)** | Gérées par le client | Contrôle total (rotation, permissions) |

### 📌 Exemple concret

**Scénario** : Chiffrer tous les objets d'un bucket S3

➡️ Créer une clé KMS et activer le chiffrement par défaut sur le bucket S3

### 🔗 Documentation officielle
- [AWS KMS](https://docs.aws.amazon.com/kms/latest/developerguide/overview.html)
- [KMS Best Practices](https://docs.aws.amazon.com/kms/latest/developerguide/best-practices.html)

---

## 8️⃣ Amazon GuardDuty

### 🎯 Rôle

**Détection intelligente de menaces** en analysant les logs et activités AWS.

### 🔍 Que surveille GuardDuty ?

- **Logs CloudTrail** : Activités API suspectes
- **Logs VPC Flow** : Trafic réseau anormal
- **Logs DNS** : Requêtes DNS malveillantes

### 🚨 Alertes automatiques

| Menace détectée | Exemple | Action recommandée |
|-----------------|---------|-------------------|
| **Instance compromise** | Instance EC2 contactant un serveur C&C (Command & Control) | Isoler l'instance, analyser |
| **Accès non autorisé** | Connexion depuis un pays inhabituel | Vérifier les credentials IAM |
| **Reconnaissance réseau** | Scan de ports sur VPC | Bloquer l'IP source |

### 📌 Fonctionnement

1. **GuardDuty analyse** en continu les logs (CloudTrail, VPC Flow, DNS)
2. **Machine Learning** détecte les comportements anormaux
3. **Alertes** envoyées via SNS ou affichées dans la console

### 🔗 Documentation officielle
- [Amazon GuardDuty](https://docs.aws.amazon.com/guardduty/latest/ug/what-is-guardduty.html)
- [GuardDuty Finding Types](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-active.html)

---

## 📊 Tableau de synthèse

| Service | Rôle principal | Cas d'usage |
|---------|---------------|-------------|
| **IAM** | Gestion des identités et accès | Créer des utilisateurs, groupes, rôles |
| **Organizations** | Gestion multi-comptes | Consolider plusieurs comptes AWS |
| **Artifact** | Rapports de conformité | Fournir des preuves d'audit (ISO, PCI-DSS) |
| **WAF** | Pare-feu applicatif | Bloquer SQL injection, XSS, HTTP flood |
| **Shield** | Protection DDoS | Protéger contre les attaques volumétriques |
| **KMS** | Gestion des clés de chiffrement | Chiffrer S3, RDS, EBS |
| **GuardDuty** | Détection de menaces | Alertes sur comportements suspects |

---

## ✅ Bonnes pratiques de sécurité AWS

1. ✅ **Activer MFA** sur tous les comptes (surtout root)
2. ✅ **Principe du moindre privilège** : donner uniquement les permissions nécessaires
3. ✅ **Chiffrer les données** sensibles (S3, RDS, EBS) avec KMS
4. ✅ **Utiliser IAM Roles** pour les services AWS (pas de clés en dur)
5. ✅ **Activer CloudTrail** pour auditer toutes les actions AWS
6. ✅ **Activer GuardDuty** pour détecter les menaces automatiquement
7. ✅ **Utiliser AWS WAF** devant les applications web publiques
8. ✅ **Configurer des SCP** dans Organizations pour appliquer des règles de sécurité globales

---

## 🔗 Ressources officielles AWS

### Documentation générale
- [AWS Security Hub](https://docs.aws.amazon.com/securityhub/)
- [AWS Well-Architected Framework - Security Pillar](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/welcome.html)

### Whitepapers (Livres blancs)
- [AWS Security Best Practices](https://docs.aws.amazon.com/whitepapers/latest/aws-security-best-practices/welcome.html)
- [AWS Compliance Programs](https://aws.amazon.com/compliance/programs/)

### Formation AWS
- [AWS Security Fundamentals](https://aws.amazon.com/training/learn-about/security/)
- [AWS Skill Builder - Security Learning Path](https://explore.skillbuilder.aws/learn/public/learning_plan/view/91/security-learning-plan)

---

## 💡 Points clés à retenir

🔐 **Sécurité = Responsabilité partagée** entre AWS et le client  
🔑 **IAM = Contrôle d'accès granulaire** (utilisateurs, groupes, rôles, politiques)  
🏢 **Organizations = Gestion multi-comptes** avec SCP pour centraliser les règles  
📄 **Artifact = Rapports de conformité** à la demande  
🛡️ **WAF = Protection applicative** contre SQL injection, XSS, HTTP flood  
🛡️ **Shield = Protection DDoS** (Standard gratuit, Advanced payant)  
🔐 **KMS = Chiffrement des données** avec gestion centralisée des clés  
🔍 **GuardDuty = Détection intelligente** de menaces avec ML  

---

**📌 Note importante** : Ce résumé couvre les **concepts essentiels** du module sécurité AWS. Pour une compréhension approfondie, consultez la documentation officielle AWS et pratiquez avec les LABs associés.

**Durée de lecture** : 15-20 minutes  
**Niveau** : Débutant à Intermédiaire  
**Prérequis** : Bases AWS (EC2, S3, IAM)
