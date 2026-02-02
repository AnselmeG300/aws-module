# Tableau comparatif IaaS, PaaS, SaaS (basé sur AWS)

## **Tableau comparatif IaaS, PaaS, SaaS (basé sur AWS)**

| Aspect | **IaaS** | **PaaS** | **SaaS** |
|--------|----------|----------|---------|
| **Définition** | Blocs de base pour IT cloud (réseau, calcul, stockage) | Plateforme gérée pour déployer et gérer vos apps | Logiciel complet géré par le fournisseur |
| **Qui gère l'infrastructure ?** | Vous | Fournisseur | Fournisseur |
| **Qui gère l'OS ?** | Vous | Fournisseur | Fournisseur |
| **Qui gère les patchs ?** | Vous | Fournisseur | Fournisseur |
| **Qui gère le code/l'app ?** | Vous | Vous | Fournisseur |
| **Niveau de contrôle** | **Très élevé** | Moyen | Très faible |
| **Niveau de flexibilité** | **Très élevée** | Moyenne | Faible |
| **Effort de gestion** | **Élevé** | Réduit | Minimal |

---

## **Responsabilités par modèle**

```
Infrastructure    OS    Runtime    Middleware    App    Données
     ✅ Vous      ✅     ✅          ✅         ✅      ✅      ← IaaS
     ❌           ❌     ❌          ❌         ✅      ✅      ← PaaS
     ❌           ❌     ❌          ❌         ❌      ❌      ← SaaS
   (Fournisseur)
```

---

## **Exemples AWS & Comparaison**

| Modèle | Exemple AWS | Ce que vous faites | Ce que AWS fait |
|--------|-------------|-------------------|-----------------|
| **IaaS** | EC2, VPC, EBS | Installer OS, app, patch, scaling manuel | Fournir serveurs, réseau, stockage brut |
| **PaaS** | Elastic Beanstalk, Lambda | Déployer votre code | Gérer OS, patchs, scaling auto, infrastructure |
| **SaaS** | WorkSpaces, Chime | Utiliser l'application | Gérer tout (app, infra, OS, données) |

---

## **Cas d'usage**

| Modèle | Cas d'usage | Exemple réel |
|--------|-----------|-------------|
| **IaaS** | Migration existante, besoins très spécifiques, contrôle total nécessaire | Migrer une app d'un datacenter vers EC2 |
| **PaaS** | Déploiement rapide, sans gestion infrastructure, scaling auto | Déployer une API Node.js sur Beanstalk, scaling automatique |
| **SaaS** | Besoin d'une solution complète, pas de code custom | Utiliser Chime pour les réunions d'équipe |

---

## **Résumé clé**

✅ **IaaS** = Vous louez les **briques de base** (serveur, réseau, stockage) → **maximum de contrôle et de responsabilité**

✅ **PaaS** = Vous louez une **plateforme prête** pour déployer votre code → **équilibre entre contrôle et facilité**

✅ **SaaS** = Vous louez une **application complète** → **minimum de contrôle, pas de maintenance**

---

## **Pour votre formation AWS 3 jours**

- **Jour 1-2**: Focus sur **IaaS** (EC2, VPC, S3, IAM)
- **Jour 3**: Introduction à **PaaS** (Elastic Beanstalk)
- **SaaS**: Hors scope (services tiers)
