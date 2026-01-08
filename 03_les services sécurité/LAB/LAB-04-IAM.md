# LAB 04 — Sécurité AWS : IAM (Identity and Access Management)

## 🎯 Objectifs

À la fin de ce lab, vous comprendrez et maîtriserez les **4 concepts clés d'IAM** :

1. ✅ **Utilisateur** (User) — Identité individuelle
2. ✅ **Groupe** (Group) — Agrégation d'utilisateurs
3. ✅ **Permission** (Policy) — Autorisations granulaires
4. ✅ **Rôle** (Role) — Délégation de permissions aux services/ressources

**Ordre d'apprentissage** : Du simple au complexe
- Utilisateur (1 concept : 1 identité)
- Groupe (1 conteneur : N utilisateurs)
- Permission (Attachement de politiques)
- Rôle (Cas avancé : services/délégation)

---

## 📋 Préparation

### Prérequis
- ✅ Accès à la console AWS
- ✅ Région : **Virginie (us-east-1)**
- ✅ Permissions d'administrateur IAM

### Compte de test
- Vous allez créer plusieurs utilisateurs/groupes pour **ce lab uniquement**
- **Important** : Nettoyage à la fin (suppression de toutes les ressources créées)

---

## 🏗️ Architecture du Lab

```
┌─────────────────────────────────────────┐
│            IAM (Sécurité)                │
├─────────────────────────────────────────┤
│                                         │
│  Utilisateur    Groupe    Permission   │
│  (Identity)  (Container)  (Policy)     │
│                                         │
│  John          DevOps      EC2-Full     │
│  Alice         DevOps      S3-Read      │
│  Bob           Admin       All-Services │
│                                         │
└─────────────────────────────────────────┘
```

---

## ⭐ EXERCICE 1 — Créer un Utilisateur IAM (Concept simple)

### Étape 1.1 : Accéder à IAM

1. **Allez à [AWS Management Console](https://console.aws.amazon.com)**
   - Région : **Virginie (us-east-1)**

2. **Accédez au service IAM**
   - Barre de recherche : `IAM`
   - Cliquez sur **"IAM"**

3. **Menu gauche → "Users"**

### Étape 1.2 : Créer un utilisateur

1. **Cliquez sur "Create user"** (bouton orange)

2. **Configurez l'utilisateur**
   - **User name** : `lab-john`
   - **Provide user access to the AWS Management Console?** : ✅ OUI
   - **Console password** : `LabPassword123!`
   - **Require password change** : ✅ OUI (recommandé)

3. **Cliquez sur "Next"**

### Étape 1.3 : Configurer les permissions (pour l'instant : RIEN)

1. **Set permissions screen**
   - **Select permission option** : `"Attach policies directly"` (mais on n'en ajoute pas encore)
   - Cliquez sur **"Next"** sans ajouter de permissions

2. **Vérifiez et créez** → **"Create user"**

### Résultat

✅ Utilisateur `lab-john` créé sans permissions (donc sans accès)

**Explication** :
- Un **utilisateur** = Une identité AWS
- Seul = Il a **0 permissions**
- Pour avoir du pouvoir, il doit être dans un **groupe** ou avoir des **permissions directes**

---

## 👥 EXERCICE 2 — Créer un Groupe IAM (Agrégation)

### Étape 2.1 : Créer un groupe

1. **IAM → Menu gauche → "User groups"**

2. **Cliquez sur "Create group"** (bouton orange)

3. **Configurez le groupe**
   - **Group name** : `lab-devops`
   - **Add users to the group** : `lab-john` (sélectionnez l'utilisateur créé)

4. **Attachez une permission au groupe**
   - **Attach permissions policies** :
     - Recherchez `AmazonEC2FullAccess`
     - ✅ Sélectionnez-la
   - Cliquez sur **"Create group"**

### Résultat

✅ Groupe `lab-devops` créé avec :
- ✅ Utilisateur `lab-john` à l'intérieur
- ✅ Permission `AmazonEC2FullAccess` attachée

**Maintenant `lab-john` a des permissions** (via le groupe) !

**Explication** :
- Un **groupe** = Un conteneur d'utilisateurs + des permissions partagées
- Les utilisateurs dans le groupe **héritent des permissions**
- **Avantage** : Gérer les permissions au niveau groupe (N utilisateurs à la fois)

---

## 📋 EXERCICE 3 — Comprendre les Permissions (Policies)

### Étape 3.1 : Examiner une permission existante

1. **IAM → Menu gauche → "Policies"**

2. **Recherchez `AmazonEC2FullAccess`**
   - Cliquez dessus

3. **Consultez le document de permissions**
   - Onglet : **"Permissions"**
   - Vous voyez le **JSON document** :

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "ec2:*",
            "Resource": "*"
        }
    ]
}
```

**Ce que cela signifie** :
- `"Effect": "Allow"` = Autoriser
- `"Action": "ec2:*"` = TOUTES les actions EC2 (`*` = wildcard)
- `"Resource": "*"` = Sur TOUTES les ressources

### Étape 3.2 : Créer une permission personnalisée (Optionnel)

1. **IAM → "Policies"** → **"Create policy"**

2. **Créez une permission restrictive**
   ```json
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Effect": "Allow",
               "Action": [
                   "s3:GetObject",
                   "s3:ListBucket"
               ],
               "Resource": [
                   "arn:aws:s3:::my-bucket",
                   "arn:aws:s3:::my-bucket/*"
               ]
           }
       ]
   }
   ```

3. **Policy name** : `lab-s3-read-only`

4. **"Create policy"**

**Explication** :
- Les **permissions** = Documents JSON
- Elles définissent **QUI peut faire QUOI sur QUELLES ressources**
- **Fine-grained** = Très granulaire (au niveau action/ressource)

---

## 🔐 EXERCICE 4 — Créer un Rôle IAM (Concept avancé)

### Cas d'usage réel

Vous avez une **instance EC2** qui héberge une application web. Cette application doit **lire des fichiers depuis S3** (images, documents, etc.).

**Problème** : Comment donner accès à S3 sans mettre des clés AWS en dur dans le code ?

**Solution** : Créer un **rôle IAM** qui donne accès à S3, et l'attacher à l'instance EC2.

### Étape 4.1 : Créer un rôle

1. **IAM → Menu gauche → "Roles"**

2. **Cliquez sur "Create role"** (bouton orange)

3. **Sélectionnez le type de confiance**
   - **Trusted entity type** : `"AWS service"`
   - **Service** : `"EC2"` (Elastic Compute Cloud)
   - Cliquez sur **"Next"**

4. **Attachez une permission**
   - Recherchez `AmazonS3ReadOnlyAccess`
   - ✅ Sélectionnez-la
   - Cliquez sur **"Next"**

5. **Configurez le rôle**
   - **Role name** : `lab-ec2-s3-readonly-role`
   - **Description** : `Permet aux instances EC2 de lire les buckets S3`
   - Cliquez sur **"Create role"**

### Résultat

✅ Rôle `lab-ec2-s3-readonly-role` créé pour donner accès S3 aux instances EC2

**Explication** :
- Un **rôle** = Une "permission portative" pour un **service AWS**
- Les **utilisateurs** = Accès à la console (humains)
- Les **rôles** = Accès pour les services (EC2, Lambda, etc.)
- **Cas d'usage** : Instance EC2 qui lit/écrit dans S3, Lambda qui envoie des emails via SES

---

## 🔗 EXERCICE 5 — Attacher un Rôle à une Instance EC2

### Étape 5.1 : Créer une instance EC2 avec le rôle

1. **EC2 → "Instances"** → **"Launch instances"**

2. **Configurez l'instance**
   - **Name** : `lab-instance`
   - **AMI** : Amazon Linux 2
   - **Instance type** : t3.micro
   - **Key pair** : Sélectionnez votre clé SSH existante

3. **⚠️ IMPORTANT : Détails avancés** (Advanced details)
   - Scrollez tout en bas jusqu'à **"IAM instance profile"**
   - Sélectionnez `lab-ec2-s3-readonly-role`

4. **Groupe de sécurité**
   - **SSH (port 22)** depuis votre IP

5. **Lancez l'instance** → **"Launch instance"**

### Résultat

✅ Instance EC2 avec le **rôle IAM attaché** donnant accès à S3

**Avantage** :
- L'instance EC2 peut **automatiquement accéder à S3**
- **Sans avoir besoin de clés d'accès** en dur dans le code
- **Plus sécurisé** : Les credentials sont temporaires et gérés par AWS

### Étape 5.2 : Vérifier l'accès S3 depuis l'instance

1. **Connectez-vous à l'instance EC2**
   
   **Option A : EC2 Instance Connect** (Recommandé - pas besoin de clé SSH)
   - Console EC2 → Sélectionnez votre instance
   - Bouton **"Connect"** → Onglet **"EC2 Instance Connect"**
   - Cliquez sur **"Connect"**
   
   **Option B : SSH classique**
   ```bash
   ssh -i your-key.pem ec2-user@<PUBLIC_IP>
   ```

2. **Testez l'accès S3 (SANS configurer AWS CLI)**
   ```bash
   # Liste tous les buckets S3 de votre compte
   aws s3 ls
   ```

   **Résultat attendu** :
   - ✅ **Succès** : Vous voyez la liste de vos buckets S3
   - ✅ **Preuve** : Le rôle IAM fonctionne, l'instance a accès à S3 automatiquement
   - ❌ **Si erreur** : Vérifiez que le rôle est bien attaché (EC2 Console → Instance → Security tab → IAM Role)

3. **Test avancé : Créer un bucket et lister**
   ```bash
   # Essayer de créer un bucket (devrait échouer - ReadOnly)
   aws s3 mb s3://test-bucket-lab-$(date +%s)
   
   # Résultat attendu : AccessDenied (normal, le rôle est ReadOnly)
   ```

**💡 Point clé** :
- Vous n'avez **JAMAIS** fait `aws configure` sur l'instance
- Vous n'avez **JAMAIS** entré de clés AWS
- L'accès fonctionne grâce au **rôle IAM attaché** !
- C'est la **bonne pratique** pour donner des permissions aux instances EC2

---

## 📊 Tableau Comparatif : Utilisateur vs Groupe vs Rôle

| Concept | Utilisateur | Groupe | Rôle |
|---------|-------------|--------|------|
| **Qu'est-ce que c'est ?** | Identité individuelle | Conteneur d'utilisateurs | Permission portable |
| **Utilisé par** | Personnes (console) | Agrégation de personnes | Services AWS (EC2, Lambda) |
| **Permissions** | Héritées du groupe ou directes | Partagées par tous les membres | Attachées via instance profile |
| **Accès** | Console AWS | Console AWS | Métadonnées (sans clés en dur) |
| **Créer** | IAM > Users | IAM > User Groups | IAM > Roles |
| **Exemple** | john@company.com | DevOps team | Instance EC2 accédant à S3 |
| **Sécurité** | Identifiants = risque | Gestion centralisée | Credentials temporaires (renouvelés automatiquement) |
| **Cas d'usage réel** | Développeur se connectant à la console | Équipe DevOps avec permissions EC2 | Application web lisant des images depuis S3 |

---

## 🧹 Nettoyage — ⚠️ TRÈS IMPORTANT

### À la fin du lab, supprimez TOUTES les ressources créées :

#### 1️⃣ Supprimer l'instance EC2

1. **EC2 → Instances**
2. Sélectionnez `lab-instance`
3. **Instance State → Terminate instance**
4. Confirmez

#### 2️⃣ Supprimer l'utilisateur IAM

1. **IAM → Users**
2. Sélectionnez `lab-john`
3. **Delete user** (confirmez que pas d'accès)

#### 3️⃣ Supprimer le groupe IAM

1. **IAM → User groups**
2. Sélectionnez `lab-devops`
3. **Delete group**

#### 4️⃣ Supprimer le rôle IAM

1. **IAM → Roles**
2. Sélectionnez `lab-ec2-s3-readonly-role`
3. **Delete role**

#### 5️⃣ Supprimer la permission personnalisée (optionnel)

1. **IAM → Policies**
2. Sélectionnez `lab-s3-read-only`
3. **Delete policy**

### Vérification finale

Allez dans chaque section et vérifiez qu'il n'y a plus de ressources `lab-*` créées.

---

## ✅ Livrables attendus

✅ **Pour chaque concept (User, Group, Permission, Role)** :
- Ressource créée
- Permissions attachées (ou héritées)
- Captures d'écran de la configuration

✅ **Rapport final** (1-2 pages) :
- Tableau comparatif des 4 concepts
- Cas d'usage réels pour chacun
- Questions de réflexion répondues

---

## 💡 Cas d'usage réels en production

### Entreprise avec équipes multi-projets

```
📦 Équipe DevOps
├── Users: Alice, Bob
├── Group: devops
└── Policy: EC2FullAccess, S3FullAccess

📦 Équipe DataScience
├── Users: Charlie, Diana
├── Group: datascience
└── Policy: SageMakerFullAccess, S3ReadOnly

📦 Application Finance (Service)
├── Role: finance-app-role
└── Policy: DynamoDBAccess, S3ReadWrite
```

### Microservices avec rôles par service

```
🔐 Service d'authentification (Lambda)
├── Role: auth-service-role
└── Policy: CognitoAccess, DynamoDBWrite

🔐 Service de notification (Lambda)
├── Role: notification-service-role
└── Policy: SESAccess, SNSPublish

🔐 Instance worker (EC2)
├── Role: worker-instance-role
└── Policy: SQSReceive, S3ReadWrite, DynamoDBUpdate
```

---

## ❓ Questions de réflexion

1. **Quelle est la différence entre un utilisateur et un rôle ?**
   - Réponse : Utilisateur = Personne (console), Rôle = Service (métadonnées)

2. **Pourquoi attacher une permission au groupe plutôt qu'à chaque utilisateur ?**
   - Réponse : Scalabilité - gérer 1 groupe > gérer 100 utilisateurs

3. **Quel est l'avantage du rôle par rapport aux clés d'accès en dur dans le code ?**
   - Réponse : Credentials temporaires auto-rotated, plus sûr

4. **Pouvez-vous avoir un utilisateur dans plusieurs groupes ?**
   - Réponse : OUI ! Un utilisateur = N groupes

5. **Qu'est-ce qu'une ARN (Amazon Resource Name) ?**
   - Réponse : Format unique pour identifier une ressource AWS

---

## 🔗 Ressources complémentaires

### Documentation officielle
- [AWS IAM Documentation](https://docs.aws.amazon.com/iam/)
- [IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [IAM Policies Documentation](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html)

### Outils pratiques
- [AWS IAM Policy Simulator](https://policysim.aws.amazon.com/)
- [AWS IAM Access Analyzer](https://console.aws.amazon.com/iamv2/home#/access-analyzer)

### Tutoriels
- [Getting Started with IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/getting-started.html)
- [Create IAM Users](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html)
- [Create and Manage Groups](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_groups_manage.html)

---

## 🚀 Points clés à retenir

| Concept | Points clés |
|---------|------------|
| **Utilisateur** | Identité unique, accès console, 1 identifiant = 1 personne |
| **Groupe** | Conteneur, agrégation, permissions partagées, scalable |
| **Permission** | Document JSON, granulaire (action/ressource), Allow/Deny |
| **Rôle** | Service AWS, credentials temporaires, plus sûr que clés |

---

## ⏱️ Durée estimée
**45-60 minutes**

## 📊 Niveau
**Intermédiaire** — Concepts fondamentaux de sécurité AWS

---

## 🎓 Pédagogie

Ce lab suit une **progression du simple au complexe** :

1. ⭐ **Utilisateur** (1 concept)
   - "Qui êtes-vous ?"
   - Réponse : Un utilisateur

2. ⭐⭐ **Groupe** (1 concept + agrégation)
   - "Quelle équipe êtes-vous ?"
   - Réponse : Un groupe contenant N utilisateurs

3. ⭐⭐⭐ **Permission** (Documents JSON)
   - "Qu'avez-vous le droit de faire ?"
   - Réponse : Une politique attachée au groupe/utilisateur

4. ⭐⭐⭐⭐ **Rôle** (Cas avancé)
   - "Et si c'est un service AWS qui a besoin d'accès ?"
   - Réponse : Un rôle avec credentials temporaires

**Cette progression garantit une compréhension progressive et durable !** 🚀