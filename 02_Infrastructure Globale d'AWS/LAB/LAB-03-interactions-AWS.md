# LAB 03 — Interactions avec AWS : Console, CLI et SDK

## Objectif
Maîtriser les **3 moyens d'interaction** avec AWS en effectuant le même cycle de vie pour une ressource :
1. **Créer** une ressource EC2 (instance de calcul)
2. **Se connecter** et utiliser la ressource
3. **Détruire** la ressource

**Important** : Chaque ressource doit être **taguée avec votre prénom** pour le suivi et l'identification.

---

## 📋 Préparation

### ⚠️ **IMPORTANT — Clé SSH `aws-training-key`**
**La clé SSH `aws-training-key` doit être créée en AMONT par le formateur et fournie à chaque apprenant.**
- La clé est créée dans AWS (EC2 > Key pairs)
- Le fichier `.pem` est téléchargé et distribué aux apprenants
- Les apprenants ne créent PAS leur propre clé (ils utilisent la clé commune)

### Prérequis
- Accès à un compte AWS (compte de labo partagé ou personnel)
- **Fichier `aws-training-key.pem` fourni par le formateur** ✅
- AWS CLI installée et configurée (pour les exercices CLI et SDK)
- Python 3.8+ et Boto3 (pour l'exercice SDK)

### Paramètres communs pour tous les exercices
| Paramètre | Valeur |
|-----------|--------|
| **Région** | us-east-1 (Virginie) |
| **AMI** | Amazon Linux 2 (ami-0c02fb54eef1ca2e6 ou plus récente) |
| **Type d'instance** | t3.micro (gratuit dans le tier gratuit) |
| **Groupe de sécurité** | Autoriser SSH (port 22) depuis votre IP |
| **Tag : Name** | `EC2-[VOTRE_PRENOM]` |
| **Tag : Owner** | `[VOTRE_PRENOM]` |
| **Tag : Classroom** | `AWS-Training-Jour2` |

---

## 🖥️ EXERCICE 1 — AWS Management Console (Interface graphique)

### Étape 1.1 : Créer une instance EC2 via la console

1. **Accédez à la console AWS** :
   - Allez sur [AWS Management Console](https://console.aws.amazon.com)
   - Sélectionnez la région **Virginie (us-east-1)**

2. **Lancez une instance EC2** :
   - Allez à **EC2 > Instances**
   - Cliquez sur **"Launch instances"**
   - **Nom et étiquettes** :
     - Name: `EC2-John` (remplacez "John" par votre prénom)
     - Owner: `John`
     - Classroom: `AWS-Training-Jour2`
   
3. **Sélectionnez une AMI** :
   - Choisissez **Amazon Linux 2**
   - Architecture : **64-bit (x86)**

4. **Sélectionnez le type d'instance** :
   - Type : **t3.micro** (eligible au tier gratuit)

5. **Configurez la paire de clés** :
   - Si vous n'avez pas de clé, créez-en une :
     - Nom : `aws-training-key`
     - Format : `.pem`
   - Téléchargez et **sauvegardez la clé en lieu sûr**

6. **Configurez le groupe de sécurité** :
   - Créez ou sélectionnez un groupe de sécurité autorisant :
     - **SSH (port 22)** depuis votre IP
     - **HTTP (port 80)** (optionnel)
   
7. **Lancez l'instance** :
   - Cliquez sur **"Launch instance"**
   - Notez l'ID d'instance (ex: `i-0abc1234def5678`)

### Étape 1.2 : Se connecter à l'instance

**OPTION A : EC2 Connect (Navigateur — Recommandé)**

1. **Allez à EC2 > Instances**
2. Sélectionnez votre instance
3. Cliquez sur le bouton **"Connect"** (en haut à droite)
4. Onglet **"EC2 Instance Connect"**
5. **"Connect"** → Une fenêtre de terminal s'ouvre dans le navigateur ✅

**Avantage** : Pas de clé SSH, accès direct via navigateur

---

**OPTION B : Session Manager (AWS Systems Manager)**

1. **Allez à [AWS Systems Manager](https://console.aws.amazon.com/systems-manager)**
2. Menu gauche → **"Session Manager"**
3. Cliquez sur **"Start session"**
4. Sélectionnez votre instance `EC2-[VOTRE_PRENOM]`
5. **"Start session"** → Accès au terminal ✅

**Avantage** : Connexion sécurisée sans clé SSH, gérée par IAM

---

**OPTION C : SSH classique (si vous avez une clé)**

```bash
chmod 400 aws-training-key.pem
ssh -i aws-training-key.pem ec2-user@<PUBLIC_IP>
```

---

### Étape 1.3 : Vérifier la connexion

Une fois connecté (par EC2 Connect, Session Manager ou SSH), exécutez :

```bash
whoami
uname -a
df -h
```

### Étape 1.4 : Déconnexion

```bash
exit
```

### Étape 1.3 : Détruire l'instance via la console

1. **Allez à EC2 > Instances**
2. **Sélectionnez** votre instance `EC2-[VOTRE_PRENOM]`
3. **Instance State > Terminate instance**
4. **Confirmez** la suppression
5. **Attendez** que l'état passe à "Terminated"

---

## ⌨️ EXERCICE 2 — AWS Command Line Interface (CLI)

### Étape 2.1 : Créer une instance EC2 via la CLI

1. **Ouvrez un terminal** et configurez la région :
   ```bash
   export AWS_REGION=us-east-1
   export MY_NAME="EC2-John"  # Remplacez par votre EC2-prénom
   export STORAGE=100  # Remplacez par votre EC2-prénom
   ```

2. **Créez l'instance** :
   ```bash
   aws ec2 run-instances --image-id "ami-068c0051b15cdb816" --count 1 --instance-type t3.micro --key-name "aws-training-key" --security-group-ids "sg-096869bc076d1c94a" --block-device-mappings DeviceName=/dev/sda1,Ebs={VolumeSize=$STORAGE} --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value='$MY_NAME'}]'
   ```

3. **Notez l'ID d'instance** (sortie : `InstanceId`)

4. **Vérifiez l'état de l'instance** :
   ```bash
   aws ec2 describe-instances \
     --filters "Name=tag:Name,Values=$MY_NAME" \
     --query 'Reservations[0].Instances[0].[InstanceId,State.Name,PublicIpAddress]' \
     --region us-east-1
   ```

### Étape 2.2 : Se connecter à l'instance via CLI

1. **Récupérez l'adresse IP publique** :
   ```bash
   IP=$(aws ec2 describe-instances \
     --filters "Name=tag:Name,Values=$MY_NAME" \
     --query 'Reservations[0].Instances[0].PublicIpAddress' \
     --output text \
     --region us-east-1)
   
   echo "Adresse IP : $IP"
   ```

2. **Televerser la clé et donner les droits en lecture** :
   ```bash
   chmod 400 aws-training-key.pem
   ```

3. **Connectez-vous via SSH** :
   ```bash
   ssh -i aws-training-key.pem ec2-user@$IP
   ```

4. **Exécutez des commandes** :
   ```bash
   whoami
   uptime
   exit
   ```

### Étape 2.3 : Détruire l'instance via CLI

1. **Récupérez l'ID d'instance** :
   ```bash
   INSTANCE_ID=$(aws ec2 describe-instances \
     --filters "Name=tag:Name,Values=$MY_NAME" \
     --query 'Reservations[0].Instances[0].InstanceId' \
     --output text \
     --region us-east-1)
   
   echo "ID à supprimer : $INSTANCE_ID"
   ```

2. **Terminez l'instance** :
   ```bash
   aws ec2 terminate-instances --instance-ids $INSTANCE_ID --region us-east-1
   ```

3. **Vérifiez la suppression** :
   ```bash
   aws ec2 describe-instances \
     --instance-ids $INSTANCE_ID \
     --query 'Reservations[0].Instances[0].State.Name' \
     --region us-east-1
   ```

---

## � SECTION IMPORTANTE : Récupérer vos Access Key et Secret Key

### Où trouver vos credentials ?

1. **Allez à [IAM Console](https://console.aws.amazon.com/iamv2)**

2. **Menu gauche → "Users"**

3. **Sélectionnez votre utilisateur** (ex: `M2i_John`)

4. **Onglet "Security credentials"**

5. **Section "Access keys"** → Cliquez sur **"Create access key"**

6. **Application name** : `lab03-sdk`

7. Vous recevrez :
   - ✅ **Access Key ID** (commence par `AKIA...`)
   - ✅ **Secret Access Key** (chaîne longue de caractères)

8. **Copiez ces deux valeurs** et collez-les dans vos scripts Python

### ⚠️ Important : Sécurité

- **JAMAIS** partager vos credentials
- **JAMAIS** committer dans Git
- **Supprimez les credentials** après le lab
  - IAM > Users > Security credentials > Supprimer l'access key

---

## 🚀 EXERCICE 3 — AWS SDK (Python Boto3) avec Credentials



### Étape 3.1 : Créer une instance EC2 via SDK

1. **Installez Boto3** :
   ```bash
   pip install boto3
   ```

2. **Créez un script Python** (`lab03-create-instance.py`) :
   ```python
   import boto3
   import time
   
   # ⚠️ Configuration des credentials AWS
   # IMPORTANT: Récupérez vos Access Key ID et Secret Access Key depuis IAM
   # IAM > Users > Votre utilisateur > Security credentials > Access keys
   
   AWS_ACCESS_KEY_ID = "AKIA2XXXXXXXXXXX"      # ← Remplacez par votre Access Key
   AWS_SECRET_ACCESS_KEY = "xxxxxxxxxxxxxxxxxx" # ← Remplacez par votre Secret Key
   
   # Configuration
   MY_NAME = "John"  # Remplacez par votre prénom
   REGION = "us-east-1"
   
   # Créez un client EC2 avec vos credentials
   ec2 = boto3.client(
       'ec2',
       region_name=REGION,
       aws_access_key_id=AWS_ACCESS_KEY_ID,
       aws_secret_access_key=AWS_SECRET_ACCESS_KEY
   )
   
   # Créez une instance
   response = ec2.run_instances(
       ImageId='ami-0c02fb54eef1ca2e6',  # Amazon Linux 2
       InstanceType='t3.micro',
       KeyName='aws-training-key',
       SecurityGroups=['SSH-Access'],
       TagSpecifications=[
           {
               'ResourceType': 'instance',
               'Tags': [
                   {'Key': 'Name', 'Value': f'EC2-{MY_NAME}'},
                   {'Key': 'Owner', 'Value': MY_NAME},
                   {'Key': 'Classroom', 'Value': 'AWS-Training-Jour2'}
               ]
           }
       ],
       MinCount=1,
       MaxCount=1
   )
   
   # Récupérez l'ID d'instance
   instance_id = response['Instances'][0]['InstanceId']
   print(f"✓ Instance créée : {instance_id}")
   
   # Attendez que l'instance soit en cours d'exécution
   print("⏳ Attente du démarrage de l'instance...")
   ec2.get_waiter('instance_running').wait(InstanceIds=[instance_id])
   
   # Récupérez l'adresse IP publique
   instances = ec2.describe_instances(InstanceIds=[instance_id])
   public_ip = instances['Reservations'][0]['Instances'][0]['PublicIpAddress']
   print(f"✓ Adresse IP publique : {public_ip}")
   print(f"✓ Connectez-vous avec EC2 Connect ou Session Manager")
   ```

3. **Exécutez le script** :
   ```bash
   python lab03-create-instance.py
   ```

4. **Notez l'ID d'instance et l'adresse IP**

### Étape 3.2 : Se connecter à l'instance

1. **Option A : EC2 Connect (Navigateur)**
   - EC2 > Instances > Sélectionnez votre instance
   - Bouton **"Connect"** > Onglet **"EC2 Instance Connect"**
   - Cliquez sur **"Connect"** pour accéder au terminal

2. **Option B : Session Manager**
   - AWS Systems Manager > Session Manager > Start session
   - Sélectionnez votre instance
   - Cliquez sur **"Start session"**

3. **Option C : SSH (si vous avez la clé)**
   ```bash
   ssh -i aws-training-key.pem ec2-user@<PUBLIC_IP>
   ```

### Étape 3.3 : Détruire l'instance via SDK

1. **Créez un script Python** (`lab03-terminate-instance.py`) :
   ```python
   import boto3
   
   # ⚠️ Configuration des credentials AWS (même que pour create)
   AWS_ACCESS_KEY_ID = "AKIA2XXXXXXXXXXX"      # ← Même clé que create-instance.py
   AWS_SECRET_ACCESS_KEY = "xxxxxxxxxxxxxxxxxx" # ← Même secret que create-instance.py
   
   # Configuration
   MY_NAME = "John"  # Même prénom qu'à la création
   REGION = "us-east-1"
   
   # Créez un client EC2 avec vos credentials
   ec2 = boto3.client(
       'ec2',
       region_name=REGION,
       aws_access_key_id=AWS_ACCESS_KEY_ID,
       aws_secret_access_key=AWS_SECRET_ACCESS_KEY
   )
   
   # Trouvez l'instance par son tag Name
   instances = ec2.describe_instances(
       Filters=[
           {'Name': 'tag:Name', 'Values': [f'EC2-{MY_NAME}']},
           {'Name': 'instance-state-name', 'Values': ['running', 'stopped']}
       ]
   )
   
   if instances['Reservations']:
       instance_id = instances['Reservations'][0]['Instances'][0]['InstanceId']
       print(f"Instance trouvée : {instance_id}")
       
       # Terminez l'instance
       ec2.terminate_instances(InstanceIds=[instance_id])
       print(f"✓ Instance {instance_id} en cours de suppression...")
       
       # Attendez la suppression
       ec2.get_waiter('instance_terminated').wait(InstanceIds=[instance_id])
       print(f"✓ Instance {instance_id} supprimée")
   else:
       print("Aucune instance trouvée avec ce tag")
   ```

2. **Exécutez le script** :
   ```bash
   python lab03-terminate-instance.py
   ```

---

## 📊 Tableau de synthèse

| Étape | Console | CLI | SDK |
|-------|---------|-----|-----|
| **Créer** | Clics dans l'interface | `aws ec2 run-instances` | `boto3.client('ec2').run_instances()` |
| **Récupérer infos** | Affichage direct | `aws ec2 describe-instances` | `ec2.describe_instances()` |
| **Se connecter** | Copier l'IP et SSH | Script Bash + SSH | Script Python + SSH |
| **Détruire** | Clics > Terminate | `aws ec2 terminate-instances` | `ec2.terminate_instances()` |
| **Durée approx.** | 5-10 min | 5-10 min | 5-10 min |

---

## 🏆 Livrables attendus

✅ **Pour chaque méthode (Console, CLI, SDK)** :
- Instance créée et taguée avec votre prénom
- Connexion SSH réussie
- Commandes exécutées sur l'instance
- Instance détruite
- Capture d'écran ou log de chaque étape

✅ **Rapport final** (1 page) :
- Tableau comparatif des 3 méthodes
- Avantages/inconvénients de chaque approche
- Quelle méthode préférez-vous et pourquoi ?

---

## 💡 Remarques importantes

### Tagging obligatoire
- **Name** : `EC2-[VOTRE_PRENOM]` (pour identifier facilement)
- **Owner** : `[VOTRE_PRENOM]` (suivi des ressources)
- **Classroom** : `AWS-Training-Jour2` (tracking pédagogique)

### Sécurité
- ⚠️ **Jamais** de clé SSH dans le code
- ⚠️ **Ne pas** partager votre fichier `.pem`
- ⚠️ Limitez l'accès SSH à votre IP

### Coûts
- Instances t3.micro = **gratuit** dans le tier gratuit AWS (750h/mois)
- **⚠️ Terminez vos instances après chaque exercice**
- N'oubliez pas les snapshots/volumes orphelins

---

## 🔗 Ressources complémentaires

### Documentation officielle
- [Amazon EC2 Console](https://console.aws.amazon.com/ec2/)
- [AWS CLI — EC2 Commands](https://docs.aws.amazon.com/cli/latest/reference/ec2/)
- [Boto3 EC2 Client](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/ec2.html)

### Tutoriels
- [Getting Started with EC2](https://aws.amazon.com/ec2/getting-started/)
- [AWS CLI Configuration](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html)
- [Boto3 Quickstart](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/quickstart.html)

---

## ❓ Questions de réflexion

1. **Quelle méthode (Console, CLI, SDK) vous semble la plus simple ? Pourquoi ?**
2. **Dans quel contexte utiliseriez-vous chaque méthode ?** (Ex: Console pour exploration, CLI pour automatisation)
3. **Quel est l'avantage d'utiliser des tags sur les ressources ?**
4. **Comment pourriez-vous automatiser la création de plusieurs instances ?**

---

## 🚀 Cas d'usage réels en production

Les 3 méthodes (Console, CLI, SDK) que vous venez de pratiquer sont largement utilisées en production. Voici des exemples concrets :

### **1️⃣ Cas SDK — Application de notification (Python Boto3)**

**Projet** : Système de notification basé sur les transactions AWS  
**Repository** : [notification-transactions-aws-sdk](https://github.com/AnselmeG300/notification-transactions-aws-sdk)

**Ce qu'il fait** :
- Utilise **AWS SDK (Boto3)** pour interagir avec plusieurs services AWS
- Gère les transactions et envoie des notifications
- Exemple parfait d'intégration AWS dans une application Python

**Apprentissage** :
- Comment utiliser le SDK pour gérer plusieurs services
- Bonnes pratiques d'intégration AWS en code applicatif
- Gestion d'erreurs et retry logic

---

### **2️⃣ Cas AWS CLI — Pipeline CI/CD GitLab avec infrastructure dynamique**

**Projet** : Environnement dynamique créé par CI/CD avec AWS CLI  
**Repository** : [gitlab-ci-training](https://github.com/AnselmeG300/gitlab-ci-training/blob/main/TP5%20-%20Environnement%20dynamique/EC2/.gitlab-ci.yml)

**Ce qu'il fait** :
- Utilise **AWS CLI** dans une pipeline GitLab CI/CD
- Crée/configure dynamiquement des instances EC2 pour chaque build
- Détruit l'infrastructure après les tests

**Apprentissage** :
- Automatisation complète avec CLI en CI/CD
- Gestion du cycle de vie des ressources (create → test → destroy)
- Intégration AWS dans des workflows DevOps

---

### **3️⃣ Cas IaC (Infrastructure as Code) — Déploiement Jenkins avec Terraform/CloudFormation**

**Projet** : Pipeline Jenkins pour déployer une application Spring Boot avec IaC  
**Repository** : [jenkins-CICD-spring-boot-app](https://github.com/AnselmeG300/jenkins-CICD-spring-boot-app/blob/iac/Jenkinsfile)

**Ce qu'il fait** :
- Utilise **Terraform ou CloudFormation** (Infrastructure as Code)
- Provisionne l'infrastructure AWS de manière déclarative
- Intègre la gestion IaC dans Jenkins pour déploiement cohérent

**Apprentissage** :
- Infrastructure définie en code (versionnable et reproductible)
- Déploiement infrastructure + application en même pipeline
- Scalabilité et gestion d'environnements multiples (dev/staging/prod)

---

## 📊 Synthèse : Quand utiliser quoi en production ?

| Contexte | Méthode | Exemple |
|----------|--------|---------|
| **Exploration/prototype** | Console | Tester une nouvelle feature AWS |
| **Automatisation/CI-CD** | CLI | Pipeline GitLab/Jenkins créant ressources |
| **Application cloud-native** | SDK | Microservice interagissant avec AWS |
| **Infrastructure scalable** | IaC (Terraform) | Provisionner VPC + EC2 + RDS de manière reproductible |

---

## 💡 À retenir

✅ **Console** : visuelle, learning curve faible  
✅ **CLI** : powerful, idéale pour l'automatisation et les scripts  
✅ **SDK** : flexibilité maximale, intégration code applicatif  
✅ **IaC** : scalabilité, versionning, reproductibilité  

Ces 4 approches sont **complémentaires** et souvent utilisées ensemble en production ! 

