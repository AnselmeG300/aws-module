# LAB 03 — Interactions avec AWS : Console, CLI et SDK

## Objectif
Maîtriser les **3 moyens d'interaction** avec AWS à travers différents cas d'usage :

### 🖥️ **EXERCICE 1 & 2 : Console et CLI**
Manipuler des **instances EC2** (infrastructure) :
1. **Créer** une instance EC2
2. **Se connecter** à l'instance
3. **Détruire** l'instance

### 🐍 **EXERCICE 3 : SDK (Python Boto3)**
Manipuler des **données DynamoDB** (applications) :
1. **Créer** une table DynamoDB (via Console)
2. **CRUD** : Create, Read, Update, Delete de produits (via SDK)
3. **Vérifier** les opérations dans la Console

**⚠️ Important** : Le SDK est utilisé pour la manipulation de **données**, pas pour créer de l'infrastructure.

**Tags** : Toutes les ressources EC2 doivent être taguées avec votre prénom.

---

## 📋 Préparation

### ⚠️ **IMPORTANT — Clé SSH `aws-training-key`**
**La clé SSH `aws-training-key` doit être créée en AMONT par le formateur et fournie à chaque apprenant.**
- La clé est créée dans AWS (EC2 > Key pairs)
- Le fichier `.pem` est téléchargé et distribué aux apprenants
- Les apprenants ne créent PAS leur propre clé (ils utilisent la clé commune)

### Prérequis
- Accès à un compte AWS (compte de labo partagé ou personnel)
- **Fichier `aws-training-key.pem` fourni par le formateur** ✅ (pour EC2 uniquement)
- AWS CLI installée et configurée (pour l'exercice CLI)
- Python 3.8+ et Boto3 (pour l'exercice SDK avec DynamoDB)

### Paramètres communs pour les exercices Console et CLI (EC2)
| Paramètre | Valeur |
|-----------|--------|
| **Région** | us-east-1 (Virginie) |
| **AMI** | Amazon Linux 2 (ami-0c02fb54eef1ca2e6 ou plus récente) |
| **Type d'instance** | t3.micro (gratuit dans le tier gratuit) |
| **Groupe de sécurité** | Autoriser SSH (port 22) depuis votre IP |
| **Tag : Name** | `EC2-[VOTRE_PRENOM]` |
| **Tag : Owner** | `[VOTRE_PRENOM]` |
| **Tag : Classroom** | `AWS-Training-Jour2` |

**Note** : L'exercice SDK (EXERCICE 3) utilise DynamoDB, pas EC2.

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

## 🚀 EXERCICE 3 — AWS SDK (Python Boto3) avec DynamoDB

### 🎯 Objectif de l'exercice

**Le SDK n'est PAS utilisé pour créer des ressources d'infrastructure** (EC2, VPC, etc.) mais pour **communiquer avec les services AWS** et **manipuler des données**.

Dans cet exercice, vous allez :
1. ✅ Créer une table DynamoDB via la **console**
2. ✅ Renseigner des produits via la **console**
3. ✅ Utiliser le **SDK Python (Boto3)** pour effectuer des opérations **CRUD** (Create, Read, Update, Delete) sur la table

---

### 📍 Deux scénarios d'exécution

Le code SDK (Boto3) peut être exécuté dans **deux contextes différents** :

| Scénario | Lieu d'exécution | Credentials | Usage |
|----------|------------------|-------------|-------|
| **LOCAL** | Votre machine personnelle | Access Key + Secret Key (explicites) | Tests, développement, prototypage |
| **AWS Lambda** | Fonction Lambda (serveur AWS) | Access Key + Secret Key (variables d'environnement) | Production, automatisation serverless |

---

## 🖥️ PARTIE 1 : Créer la table DynamoDB (Console)

### Étape 1.1 : Créer la table via la console

1. **Allez à [DynamoDB Console](https://console.aws.amazon.com/dynamodbv2)**

2. **Cliquez sur "Create table"**

3. **Remplissez les informations** :
   - **Table name** : `Produits`
   - **Partition key** : `ProductID` (Type: **String**)
   - **Settings** : Laissez les paramètres par défaut (On-demand)

4. **Cliquez sur "Create table"**

5. **Attendez** que la table soit créée (statut : `Active`)

---

### Étape 1.2 : Renseigner des produits via la console

1. **Allez dans votre table** `Produits`

2. **Onglet "Explore table items"**

3. **Cliquez sur "Create item"**

4. **Ajoutez 3 produits manuellement** :

**Produit 1** :
```json
{
  "ProductID": "P001",
  "Name": "Laptop Dell XPS",
  "Description": "Ordinateur portable professionnel 15 pouces",
  "Price": 1299.99,
  "Stock": 15
}
```

**Produit 2** :
```json
{
  "ProductID": "P002",
  "Name": "Souris Logitech",
  "Description": "Souris sans fil ergonomique",
  "Price": 39.99,
  "Stock": 50
}
```

**Produit 3** :
```json
{
  "ProductID": "P003",
  "Name": "Clavier mécanique",
  "Description": "Clavier RGB gaming",
  "Price": 89.99,
  "Stock": 25
}
```

5. **Vérifiez** que les 3 produits sont bien enregistrés

---

## 🖥️ PARTIE 2 : Scénario LOCAL (Votre machine)

### Étape 2.1 : Installer Boto3

```bash
pip install boto3
```

---

### Étape 2.2 : CREATE - Ajouter 2 nouveaux produits via SDK

**Créez un script** `dynamo_create.py` :

```python
import boto3
from decimal import Decimal

# ⚠️ Configuration des credentials AWS
AWS_ACCESS_KEY_ID = "AKIA2XXXXXXXXXXX"      # ← Remplacez
AWS_SECRET_ACCESS_KEY = "xxxxxxxxxxxxxxxxxx" # ← Remplacez

REGION = "us-east-1"
TABLE_NAME = "Produits"

# Créez un client DynamoDB
dynamodb = boto3.resource(
    'dynamodb',
    region_name=REGION,
    aws_access_key_id=AWS_ACCESS_KEY_ID,
    aws_secret_access_key=AWS_SECRET_ACCESS_KEY
)

table = dynamodb.Table(TABLE_NAME)

# 🔹 Produit 4 : Écran
produit4 = {
    'ProductID': 'P004',
    'Name': 'Écran Samsung 27"',
    'Description': 'Écran 4K UHD pour professionnels',
    'Price': Decimal('349.99'),
    'Stock': 12
}

# 🔹 Produit 5 : Webcam
produit5 = {
    'ProductID': 'P005',
    'Name': 'Webcam Logitech HD',
    'Description': 'Webcam 1080p pour visioconférence',
    'Price': Decimal('79.99'),
    'Stock': 30
}

# Insérer les produits
table.put_item(Item=produit4)
print(f"✅ Produit créé : {produit4['Name']}")

table.put_item(Item=produit5)
print(f"✅ Produit créé : {produit5['Name']}")

print("\n✓ 2 nouveaux produits ajoutés via SDK!")
```

**Exécutez** :
```bash
python dynamo_create.py
```

---

### Étape 2.3 : READ - Lire un produit par ID

**Créez un script** `dynamo_read.py` :

```python
import boto3

AWS_ACCESS_KEY_ID = "AKIA2XXXXXXXXXXX"
AWS_SECRET_ACCESS_KEY = "xxxxxxxxxxxxxxxxxx"

REGION = "us-east-1"
TABLE_NAME = "Produits"

dynamodb = boto3.resource(
    'dynamodb',
    region_name=REGION,
    aws_access_key_id=AWS_ACCESS_KEY_ID,
    aws_secret_access_key=AWS_SECRET_ACCESS_KEY
)

table = dynamodb.Table(TABLE_NAME)

# Lire un produit par ID
product_id = "P004"
response = table.get_item(Key={'ProductID': product_id})

if 'Item' in response:
    item = response['Item']
    print(f"📦 Produit trouvé :")
    print(f"  ID: {item['ProductID']}")
    print(f"  Nom: {item['Name']}")
    print(f"  Description: {item['Description']}")
    print(f"  Prix: {item['Price']} €")
    print(f"  Stock: {item['Stock']}")
else:
    print(f"❌ Produit {product_id} introuvable")
```

**Exécutez** :
```bash
python dynamo_read.py
```

---

### Étape 2.4 : UPDATE - Mettre à jour un produit par ID

**Créez un script** `dynamo_update.py` :

```python
import boto3
from decimal import Decimal

AWS_ACCESS_KEY_ID = "AKIA2XXXXXXXXXXX"
AWS_SECRET_ACCESS_KEY = "xxxxxxxxxxxxxxxxxx"

REGION = "us-east-1"
TABLE_NAME = "Produits"

dynamodb = boto3.resource(
    'dynamodb',
    region_name=REGION,
    aws_access_key_id=AWS_ACCESS_KEY_ID,
    aws_secret_access_key=AWS_SECRET_ACCESS_KEY
)

table = dynamodb.Table(TABLE_NAME)

# Mettre à jour le prix du produit P004
product_id = "P004"
new_price = Decimal('299.99')  # Prix réduit !

response = table.update_item(
    Key={'ProductID': product_id},
    UpdateExpression='SET Price = :val',
    ExpressionAttributeValues={':val': new_price},
    ReturnValues='UPDATED_NEW'
)

print(f"✅ Produit {product_id} mis à jour !")
print(f"  Nouveau prix : {response['Attributes']['Price']} €")
```

**Exécutez** :
```bash
python dynamo_update.py
```

---

### Étape 2.5 : DELETE - Supprimer un produit par ID

**Créez un script** `dynamo_delete.py` :

```python
import boto3

AWS_ACCESS_KEY_ID = "AKIA2XXXXXXXXXXX"
AWS_SECRET_ACCESS_KEY = "xxxxxxxxxxxxxxxxxx"

REGION = "us-east-1"
TABLE_NAME = "Produits"

dynamodb = boto3.resource(
    'dynamodb',
    region_name=REGION,
    aws_access_key_id=AWS_ACCESS_KEY_ID,
    aws_secret_access_key=AWS_SECRET_ACCESS_KEY
)

table = dynamodb.Table(TABLE_NAME)

# Supprimer le produit P005
product_id = "P005"

table.delete_item(Key={'ProductID': product_id})
print(f"🗑️  Produit {product_id} supprimé avec succès")
```

**Exécutez** :
```bash
python dynamo_delete.py
```

---

### Étape 2.6 : Vérification finale dans la console

1. **Allez dans DynamoDB Console**
2. **Vérifiez** :
   - ✅ P004 et P005 ont été créés (P005 devrait être supprimé ensuite)
   - ✅ P004 a un nouveau prix (299.99€)
   - ✅ P005 n'existe plus

---

## ☁️ PARTIE 3 : Scénario LAMBDA (Serverless)

### Étape 3.1 : Créer une fonction Lambda pour CREATE

1. **Allez à [AWS Lambda](https://console.aws.amazon.com/lambda)**

2. **Créez une fonction** :
   - Runtime : **Python 3.11**
   - Nom : `DynamoCreate`
   - Role : Créer un rôle avec politique **AmazonDynamoDBFullAccess**

3. **Code de la fonction** :

```python
import boto3
import json
import os
from decimal import Decimal

AWS_ACCESS_KEY_ID = os.environ.get('AWS_ACCESS_KEY_ID')
AWS_SECRET_ACCESS_KEY = os.environ.get('AWS_SECRET_ACCESS_KEY')

REGION = "us-east-1"
TABLE_NAME = "Produits"

def lambda_handler(event, context):
    try:
        dynamodb = boto3.resource(
            'dynamodb',
            region_name=REGION,
            aws_access_key_id=AWS_ACCESS_KEY_ID,
            aws_secret_access_key=AWS_SECRET_ACCESS_KEY
        )
        
        table = dynamodb.Table(TABLE_NAME)
        
        # Créer un nouveau produit (reçu via event)
        product = {
            'ProductID': event.get('ProductID', 'P006'),
            'Name': event.get('Name', 'Produit Lambda'),
            'Description': event.get('Description', 'Créé via Lambda'),
            'Price': Decimal(str(event.get('Price', 99.99))),
            'Stock': event.get('Stock', 10)
        }
        
        table.put_item(Item=product)
        
        return {
            'statusCode': 200,
            'body': json.dumps({
                'message': 'Produit créé avec succès',
                'product': event
            })
        }
    
    except Exception as e:
        return {
            'statusCode': 500,
            'body': json.dumps({'error': str(e)})
        }
```

4. **Configurez les variables d'environnement** :
   - `AWS_ACCESS_KEY_ID` : Votre Access Key
   - `AWS_SECRET_ACCESS_KEY` : Votre Secret Key

5. **Testez avec cet événement** :
```json
{
  "ProductID": "P006",
  "Name": "Casque Bluetooth",
  "Description": "Casque sans fil noise cancelling",
  "Price": 149.99,
  "Stock": 20
}
```

---

### Étape 3.2 : Créer une fonction Lambda pour READ

```python
import boto3
import json
import os

AWS_ACCESS_KEY_ID = os.environ.get('AWS_ACCESS_KEY_ID')
AWS_SECRET_ACCESS_KEY = os.environ.get('AWS_SECRET_ACCESS_KEY')

REGION = "us-east-1"
TABLE_NAME = "Produits"

def lambda_handler(event, context):
    try:
        dynamodb = boto3.resource(
            'dynamodb',
            region_name=REGION,
            aws_access_key_id=AWS_ACCESS_KEY_ID,
            aws_secret_access_key=AWS_SECRET_ACCESS_KEY
        )
        
        table = dynamodb.Table(TABLE_NAME)
        
        # Lire un produit par ID
        product_id = event.get('ProductID', 'P001')
        response = table.get_item(Key={'ProductID': product_id})
        
        if 'Item' in response:
            # Convertir Decimal en float pour JSON
            item = response['Item']
            if 'Price' in item:
                item['Price'] = float(item['Price'])
            if 'Stock' in item:
                item['Stock'] = float(item['Stock'])
                
            print("Produit trouvé:", item)
            return {
                'statusCode': 200,
                'body': json.dumps({
                    'product': item
                })
            }
        else:
            return {
                'statusCode': 404,
                'body': json.dumps({
                    'error': f'Produit {product_id} introuvable'
                })
            }
    
    except Exception as e:
        return {
            'statusCode': 500,
            'body': json.dumps({'error': str(e)})
        }
```

**Testez avec** :
```json
{
  "ProductID": "P006"
}
```

---

### 📊 Comparaison : LOCAL vs LAMBDA

| Aspect | LOCAL | LAMBDA |
|--------|-------|--------|
| **Lieu** | Votre machine | Serveur AWS |
| **Credentials** | En dur dans le code | Variables d'environnement |
| **Démarrage** | Manuel (`python script.py`) | Déclenché par événement/API |
| **Durée max** | Illimitée | 15 minutes |
| **Coût** | Gratuit | Gratuit (1M requêtes/mois) |
| **Usage** | Dev/test | Production/automatisation |
| **Sécurité** | ⚠️ Locale seulement | ✅ Isolée en AWS |

---

### ⚠️ Important : Sécurité des credentials

**LOCAL** :
- ✅ Acceptable en développement
- ❌ JAMAIS en production ou dans Git
- Supprimez après le test

**LAMBDA** :
- ✅ Les variables d'environnement isolées dans AWS
- ✅ Ne sont pas visibles publiquement
- ⚠️ Toujours préférable d'utiliser IAM Roles

**Bonne pratique en production** :
```python
# Production avec IAM Role (pas de credentials)
dynamodb = boto3.resource('dynamodb', region_name=REGION)  # ← Pas de credentials !
```

---

## 📊 Tableau de synthèse

| Aspect | Console (EC2) | CLI (EC2) | SDK (DynamoDB) |
|--------|---------------|-----------|----------------|
| **Service AWS** | EC2 (instances) | EC2 (instances) | DynamoDB (base de données) |
| **Type d'opération** | Infrastructure | Infrastructure | Données |
| **Créer** | Clics > Launch instance | `aws ec2 run-instances` | `table.put_item()` (ajouter produit) |
| **Lire** | Console > Instance details | `aws ec2 describe-instances` | `table.get_item()` (lire produit) |
| **Mettre à jour** | Console > Modify | - | `table.update_item()` (modifier prix) |
| **Supprimer** | Clics > Terminate | `aws ec2 terminate-instances` | `table.delete_item()` (supprimer produit) |
| **Durée approx.** | 10 min | 10 min | 20-30 min |

---

## 🏆 Livrables attendus

✅ **EXERCICE 1 & 2 (Console et CLI - EC2)** :
- Instance EC2 créée et taguée avec votre prénom
- Connexion SSH réussie (EC2 Connect, Session Manager ou SSH)
- Commandes exécutées sur l'instance
- Instance détruite proprement
- Capture d'écran ou log de chaque étape

✅ **EXERCICE 3 (SDK - DynamoDB)** :
- Table DynamoDB `Produits` créée
- 3 produits ajoutés manuellement via Console
- 2 produits ajoutés via SDK (P004, P005)
- Opérations CRUD réussies (Create, Read, Update, Delete)
- Screenshots des résultats dans la console DynamoDB
- Code Python fonctionnel pour chaque opération

✅ **Rapport final** (1-2 pages) :
- Tableau comparatif des 3 méthodes (Console, CLI, SDK)
- Cas d'usage de chaque approche (infrastructure vs données)
- Pourquoi SDK pour DynamoDB et pas pour EC2 ?
- Quelle méthode préférez-vous pour quel contexte ?

---

## 💡 Remarques importantes

### Tagging obligatoire (EC2 uniquement)
- **Name** : `EC2-[VOTRE_PRENOM]` (pour identifier facilement)
- **Owner** : `[VOTRE_PRENOM]` (suivi des ressources)
- **Classroom** : `AWS-Training-Jour2` (tracking pédagogique)

### Sécurité
- ⚠️ **EC2** : Jamais de clé SSH dans le code, ne pas partager votre `.pem`
- ⚠️ **SDK** : Jamais de credentials AWS dans Git, supprimer après le lab
- ⚠️ **DynamoDB** : Utilisez IAM Roles en production (pas de credentials en dur)

### Coûts
- **EC2** : Instances t3.micro = gratuit dans le tier gratuit AWS (750h/mois)
- **DynamoDB** : On-demand = gratuit pour petites tables (25 GB storage gratuit)
- **⚠️ Nettoyage obligatoire** :
  - Terminez vos instances EC2 après chaque exercice
  - Supprimez la table DynamoDB `Produits` en fin de LAB

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
2. **Dans quel contexte utiliseriez-vous chaque méthode ?** 
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
