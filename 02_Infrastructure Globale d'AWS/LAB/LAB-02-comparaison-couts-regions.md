# LAB 02 — Infrastructure Globale AWS : Comparaison des coûts par région

## Objectif
Utiliser **AWS Pricing Calculator** pour comparer les coûts des services AWS dans différentes régions et identifier la région la plus économique.

## Instructions

### Étape 1 : Accéder à AWS Pricing Calculator

Rendez-vous sur le lien suivant :

**[AWS Pricing Calculator](https://calculator.aws/)**

---

### Étape 2 : Configurer les services à comparer

Pour chaque région listée ci-dessous, vous allez calculer le coût total de **trois services** :

#### **Services à configurer :**

1. **AWS Amplify**
   - Configuration standard (hosting, déploiement continu)
   - Garder les paramètres par défaut

2. **Amazon EC2 — Instance t3.medium**
   - Type d'instance : **t3.medium**
   - Système d'exploitation : **Linux**
   - Utilisation : **On-Demand** (à la demande)
   - Durée : **1 mois** (730 heures)

3. **Amazon CloudFront**
   - **Data out to Internet** : 100 GB/mois (distribution de contenu)
   - **HTTP/HTTPS requests** : 1 million de requêtes/mois
   - Garder les paramètres par défaut

---

### Étape 3 : Comparer les régions

Pour chaque région ci-dessous, **créez un devis distinct** et **notez le coût total mensuel** (Amplify + EC2 t3.medium + CloudFront) :

#### 🌍 **Régions à comparer :**

| N° | Région | Code région | Coût Amplify | Coût EC2 t3.medium | Coût CloudFront | **Coût total** |
|-----|--------|-------------|--------------|-------------------|-----------------|----------------|
| 1 | **Virginie (N. Virginia)** | us-east-1 | | | | |
| 2 | **Irlande (Ireland)** | eu-west-1 | | | | |
| 3 | **Paris (Paris)** | eu-west-3 | | | | |
| 4 | **Oregon (N. California)** | us-west-2 | | | | |

---

### Étape 4 : Analyse et conclusions

Après avoir rempli le tableau, répondez aux questions suivantes :

**Q1. Quelle région offre le coût total le moins cher ?**
```
Réponse : ________________________
```

**Q2. Quel est l'écart de coût entre la région la moins chère et la plus chère (en USD et en %) ?**
```
Réponse : ________________________
```

**Q3. Pourquoi les coûts varient-ils entre les régions ? (Nommez au moins 2 raisons)**
```
Réponse : 
1. ________________________________________________________________
2. ________________________________________________________________
```

**Q4. En tant qu'entreprise européenne, quelle région choisiriez-vous et pourquoi ?**
```
Réponse : ________________________________________________________________
```

---

## Mode d'emploi AWS Pricing Calculator

### 📋 **Créer un devis :**
1. Cliquez sur **"Create estimate"**
2. Sélectionnez la **région** en haut à gauche
3. Cliquez sur **"Add service"**

### 🔍 **Ajouter Amplify :**
1. Recherchez **"Amplify"**
2. Sélectionnez **"AWS Amplify"**
3. Configurez les paramètres de déploiement
4. Cliquez sur **"Add to estimate"**

### 🖥️ **Ajouter EC2 :**
1. Cliquez sur **"Add service"**
2. Recherchez **"EC2"**
3. Sélectionnez **"EC2 instances"**
4. Configurez :
   - **Instance type** : t3.medium
   - **Operating system** : Linux
   - **Pricing model** : On-Demand
   - **Quantity** : 1 instance
   - **Monthly hours** : 730 (24h × 30 jours)
5. Cliquez sur **"Add to estimate"**

### � **Ajouter CloudFront :**
1. Cliquez sur **"Add service"**
2. Recherchez **"CloudFront"**
3. Sélectionnez **"CloudFront"**
4. Configurez :
   - **Data out to Internet** : 100 GB/mois
   - **HTTP/HTTPS requests** : 1,000,000 requêtes/mois
5. Cliquez sur **"Add to estimate"**

### �💾 **Récupérer le coût total :**
- Consultez la section **"Estimate of your monthly bill"** en bas de page
- Notez le coût total en USD

---

## Livrables attendus

✅ Tableau complété avec 4 régions et leurs coûts respectifs (Amplify + EC2 + CloudFront)  
✅ Réponses aux 4 questions d'analyse  
✅ Justification de votre choix de région  

---

## Durée estimée
**15 minutes**

## Niveau
**Débutant – Intermédiaire** — Compréhension des modèles de tarification AWS

---

## 💡 Remarques importantes

- **Les prix fluctuent** : Les coûts affichés peuvent varier selon la date de consultation
- **Comparaison équitable** : Assurez-vous que tous les paramètres sont identiques pour chaque région
- **Facturation mensuelle** : Les calculs tiennent compte d'une utilisation d'1 mois complet
- **Réductions non incluses** : Les devis ne tiennent pas compte des réductions (Reserved Instances, Savings Plans)

---

## Ressources complémentaires
- [AWS Pricing Calculator](https://calculator.aws/)
- [Régions AWS](https://aws.amazon.com/fr/about-aws/global-infrastructure/regions_availability-zones/)
- [Pricing AWS Amplify](https://aws.amazon.com/fr/amplify/pricing/)
- [Pricing Amazon EC2](https://aws.amazon.com/fr/ec2/pricing/)
- [Pricing Amazon CloudFront](https://aws.amazon.com/fr/cloudfront/pricing/)
