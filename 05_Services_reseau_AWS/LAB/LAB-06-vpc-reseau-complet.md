# LAB 06 — VPC et Services Réseau AWS Complet

## 🎯 Objectifs

À la fin de ce lab, vous serez capable de :
- ✅ Créer un **VPC personnalisé** avec sous-réseaux publics et privés
- ✅ Configurer des **tables de routage** pour rendre les sous-réseaux accessibles
- ✅ Mettre en place des **NACL** (Network Access Control Lists)
- ✅ Déployer un **Application Load Balancer (ALB)**
- ✅ Configurer un **Auto Scaling Group (ASG)** avec Launch Template
- ✅ Tester le déploiement dans des sous-réseaux privés **avec et sans NAT Gateway**
- ✅ Comprendre l'architecture réseau AWS

---

## 📋 Prérequis

- ✅ Accès à la console AWS
- ✅ Région : **Virginia (us-east-1)**
- ✅ **Fichier `network-configurations.txt`** fourni par le formateur
- ✅ Connaissances de base en réseau (CIDR, routage)

---

## 🗺️ Architecture Cible

```
┌─────────────────────────────────────────────────────────────────┐
│                    VPC 10.X.0.0/16                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  AZ us-east-1a              │       AZ us-east-1b              │
│                              │                                  │
│  ┌──────────────────┐        │       ┌──────────────────┐      │
│  │ Public Subnet 1  │        │       │ Public Subnet 2  │      │
│  │   10.X.1.0/24    │◄───────┼──────►│   10.X.2.0/24    │      │
│  │   [NACL]         │        │       │   [NACL]         │      │
│  │   [EC2 + App]  │        │       │   [EC2 + App]  │      │
│  └──────────────────┘        │       └──────────────────┘      │
│           ▲                  │                ▲                 │
│           └──────────────────┼────────────────┘                 │
│                   [ALB + ASG]│                                  │
│                              │                                  │
│  ┌──────────────────┐        │       ┌──────────────────┐      │
│  │ Private Subnet 1 │        │       │ Private Subnet 2 │      │
│  │  10.X.11.0/24    │        │       │  10.X.12.0/24    │      │
│  │  [EC2 + App]   │        │       │  [EC2 + App]   │      │
│  └──────────────────┘        │       └──────────────────┘      │
│                              │                                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                    [Internet Gateway]
                              │
                         [Internet]
```

---

## 📖 PARTIE 1 : Créer le VPC et les Sous-réseaux Publics

### Étape 1.1 : Récupérer votre configuration réseau

1. **Ouvrez le fichier `network-configurations.txt`**

2. **Trouvez votre section** en cherchant votre prénom
   - Exemple pour Hannah :
     ```
     APPRENANT 1 : Hannah
     VPC CIDR               : 10.1.0.0/16
     Subnet Public 1        : 10.1.1.0/24  (AZ: us-east-1a)
     Subnet Public 2        : 10.1.2.0/24  (AZ: us-east-1b)
     Subnet Private 1       : 10.1.11.0/24 (AZ: us-east-1a)
     Subnet Private 2       : 10.1.12.0/24 (AZ: us-east-1b)
     ```

3. **Notez vos plages IP** quelque part (bloc-notes, papier)
   - **⚠️ IMPORTANT** : N'utilisez QUE vos plages IP, jamais celles d'un autre apprenant !

---

### Étape 1.2 : Créer le VPC

1. **Accédez au service VPC**
   - Console AWS → Recherchez "VPC" → Cliquez sur "VPC"

2. **Créez le VPC**
   - Cliquez sur **"Create VPC"** (bouton orange)

3. **Configuration du VPC**
   - **Resources to create** : `VPC only` (pour l'instant)
   - **Name tag** : `M2i-[VOTRE_PRENOM]-VPC`
     - Exemple : `M2i-Hannah-VPC`
   
   - **IPv4 CIDR block** : Utilisez VOTRE plage VPC
     - Exemple pour Hannah : `10.1.0.0/16`
   
   - **IPv6 CIDR block** : No IPv6 CIDR block
   
   - **Tenancy** : Default

4. **Cliquez sur "Create VPC"**

**✅ Résultat** : Vous avez maintenant votre VPC personnel !

---

### Étape 1.3 : Créer une Internet Gateway

1. **Menu gauche → "Internet Gateways"**

2. **Cliquez sur "Create internet gateway"**

3. **Configuration**
   - **Name tag** : `M2i-[VOTRE_PRENOM]-IGW`
     - Exemple : `M2i-Hannah-IGW`

4. **Cliquez sur "Create internet gateway"**

5. **Attacher l'IGW au VPC**
   - Une fois créée, cliquez sur **"Actions" → "Attach to VPC"**
   - Sélectionnez votre VPC : `M2i-[VOTRE_PRENOM]-VPC`
   - Cliquez sur **"Attach internet gateway"**

**✅ Résultat** : Votre VPC peut maintenant communiquer avec Internet !

---

### Étape 1.4 : Créer les Sous-réseaux Publics

#### Sous-réseau Public 1

1. **Menu gauche → "Subnets"**

2. **Cliquez sur "Create subnet"**

3. **Configuration**
   - **VPC ID** : Sélectionnez `M2i-[VOTRE_PRENOM]-VPC`
   
   - **Subnet name** : `M2i-[VOTRE_PRENOM]-Public-1a`
   
   - **Availability Zone** : `us-east-1a`
   
   - **IPv4 CIDR block** : Utilisez votre plage Public 1
     - Exemple pour Hannah : `10.1.1.0/24`

4. **Cliquez sur "Create subnet"**

#### Sous-réseau Public 2

1. **Répétez la création avec** :
   - **Subnet name** : `M2i-[VOTRE_PRENOM]-Public-1b`
   - **Availability Zone** : `us-east-1b`
   - **IPv4 CIDR block** : Votre plage Public 2
     - Exemple pour Hannah : `10.1.2.0/24`

**✅ Résultat** : Vous avez 2 sous-réseaux publics dans 2 AZ différentes !

---

### Étape 1.5 : Configurer la Table de Routage Publique

1. **Menu gauche → "Route Tables"**

2. **Créez une nouvelle table de routage**
   - Cliquez sur **"Create route table"**
   - **Name** : `M2i-[VOTRE_PRENOM]-Public-RT`
   - **VPC** : Sélectionnez votre VPC
   - Cliquez sur **"Create route table"**

3. **Ajoutez une route vers Internet**
   - Sélectionnez votre table de routage : `M2i-[VOTRE_PRENOM]-Public-RT`
   - Onglet **"Routes"** → Cliquez sur **"Edit routes"**
   - Cliquez sur **"Add route"**
     - **Destination** : `0.0.0.0/0` (tout Internet)
     - **Target** : `Internet Gateway` → Sélectionnez votre IGW
   - Cliquez sur **"Save changes"**

4. **Associez la table de routage aux sous-réseaux publics**
   - Onglet **"Subnet associations"** → Cliquez sur **"Edit subnet associations"**
   - ✅ Cochez vos 2 sous-réseaux publics :
     - `M2i-[VOTRE_PRENOM]-Public-1a`
     - `M2i-[VOTRE_PRENOM]-Public-1b`
   - Cliquez sur **"Save associations"**

**✅ Résultat** : Vos sous-réseaux publics sont maintenant accessibles depuis Internet !

---

### Étape 1.6 : Activer l'Auto-assign Public IP

⚠️ **Crucial** : Les instances dans les sous-réseaux publics doivent avoir une IP publique automatiquement.

1. **Menu gauche → "Subnets"**

2. **Sélectionnez `M2i-[VOTRE_PRENOM]-Public-1a`**
   - Cliquez sur **"Actions" → "Edit subnet settings"**
   - ✅ Cochez **"Enable auto-assign public IPv4 address"**
   - Cliquez sur **"Save"**

3. **Répétez pour `M2i-[VOTRE_PRENOM]-Public-1b`**

**✅ Résultat** : Les instances EC2 dans ces sous-réseaux auront automatiquement une IP publique !

---

## 🛡️ PARTIE 2 : Configurer les NACL (Network Access Control Lists)

### Étape 2.1 : Créer une NACL pour les Sous-réseaux Publics

1. **Menu gauche → "Network ACLs"**

2. **Cliquez sur "Create network ACL"**

3. **Configuration**
   - **Name** : `M2i-[VOTRE_PRENOM]-Public-NACL`
   - **VPC** : Sélectionnez votre VPC
   - Cliquez sur **"Create network ACL"**

### Étape 2.2 : Configurer les Règles Entrantes (Inbound)

1. **Sélectionnez votre NACL** : `M2i-[VOTRE_PRENOM]-Public-NACL`

2. **Onglet "Inbound rules" → "Edit inbound rules"**

3. **Ajoutez les règles suivantes** :

   | Rule # | Type | Protocol | Port Range | Source | Allow/Deny |
   |--------|------|----------|------------|--------|------------|
   | 100 | HTTP | TCP | 80 | 0.0.0.0/0 | Allow |
   | 110 | HTTPS | TCP | 443 | 0.0.0.0/0 | Allow |
   | 120 | SSH | TCP | 22 | 0.0.0.0/0 | Allow |
   | 130 | Custom TCP | TCP | 1024-65535 | 0.0.0.0/0 | Allow |
   | * | All traffic | All | All | 0.0.0.0/0 | Deny |

4. **Cliquez sur "Save changes"**

**Explication** :
- Règle 100-120 : Autoriser le trafic entrant (HTTP, HTTPS, SSH)
- Règle 130 : Autoriser les réponses éphémères (ports dynamiques)
- Règle * : Tout le reste est bloqué par défaut

### Étape 2.3 : Configurer les Règles Sortantes (Outbound)

1. **Onglet "Outbound rules" → "Edit outbound rules"**

2. **Ajoutez les règles suivantes** :

   | Rule # | Type | Protocol | Port Range | Destination | Allow/Deny |
   |--------|------|----------|------------|-------------|------------|
   | 100 | HTTP | TCP | 80 | 0.0.0.0/0 | Allow |
   | 110 | HTTPS | TCP | 443 | 0.0.0.0/0 | Allow |
   | 120 | Custom TCP | TCP | 1024-65535 | 0.0.0.0/0 | Allow |
   | * | All traffic | All | All | 0.0.0.0/0 | Deny |

3. **Cliquez sur "Save changes"**

### Étape 2.4 : Associer la NACL aux Sous-réseaux Publics

1. **Onglet "Subnet associations" → "Edit subnet associations"**

2. **✅ Cochez vos 2 sous-réseaux publics** :
   - `M2i-[VOTRE_PRENOM]-Public-1a`
   - `M2i-[VOTRE_PRENOM]-Public-1b`

3. **Cliquez sur "Save changes"**

**✅ Résultat** : Vos sous-réseaux publics sont protégés par une NACL !

---

## 🚀 PARTIE 3 : Créer le Launch Template et l'Auto Scaling Group

### Étape 3.1 : Créer un Groupe de Sécurité pour les Instances

1. **Menu gauche → "Security Groups"**

2. **Cliquez sur "Create security group"**

3. **Configuration**
   - **Security group name** : `M2i-[VOTRE_PRENOM]-Web-SG`
   - **Description** : `Security group for web servers`
   - **VPC** : Sélectionnez votre VPC

4. **Inbound rules** → Cliquez sur "Add rule"
   
   | Type | Protocol | Port | Source | Description |
   |------|----------|------|--------|-------------|
   | HTTP | TCP | 80 | 0.0.0.0/0 | Allow HTTP from anywhere |
   | HTTPS | TCP | 443 | 0.0.0.0/0 | Allow HTTPS from anywhere |
   | SSH | TCP | 22 | 0.0.0.0/0 | Allow SSH (lab only) |

5. **Outbound rules** : Laisser par défaut (All traffic)

6. **Cliquez sur "Create security group"**

---

### Étape 3.2 : Créer le Launch Template

1. **EC2 → Menu gauche → "Launch Templates"**

2. **Cliquez sur "Create launch template"**

3. **Configuration du template**
   
   **Launch template name and description** :
   - **Name** : `M2i-[VOTRE_PRENOM]-App-LT`
   - **Description** : `Launch template for App web servers`

   **Application and OS Images (AMI)** :
   - **AMI** : `Ubuntu Server 24.04 LTS`

   **Instance type** :
   - **Type** : `t3.micro` (tier gratuit)

   **Key pair** :
   - Sélectionnez votre clé SSH existante (ou "Proceed without a key pair")

   **Network settings** :
   - **Subnet** : Ne pas inclure (sera défini par l'ASG)
   - **Security groups** : Sélectionnez `M2i-[VOTRE_PRENOM]-Web-SG`

   **Advanced details** :
   - Scrollez jusqu'à **"User data"**
   - Collez le script suivant :

```bash
#!/bin/bash
# Installation de Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker ubuntu
sudo systemctl enable docker
sudo systemctl start docker

# Attendre que Docker soit prêt
sleep 10

# Déployer App avec Docker
sudo docker run -d --name web-app -p 80:8080 mmumshad/simple-webapp-color

echo "✅ App déployé avec succès via Docker"
```

4. **Cliquez sur "Create launch template"**

**✅ Résultat** : Vous avez un modèle de lancement réutilisable !

---

### Étape 3.3 : Créer un Groupe de Sécurité pour l'ALB

1. **VPC → Security Groups → "Create security group"**

2. **Configuration**
   - **Name** : `M2i-[VOTRE_PRENOM]-ALB-SG`
   - **Description** : `Security group for Application Load Balancer`
   - **VPC** : Votre VPC

3. **Inbound rules** :
   
   | Type | Protocol | Port | Source |
   |------|----------|------|--------|
   | HTTP | TCP | 80 | 0.0.0.0/0 |
   | HTTPS | TCP | 443 | 0.0.0.0/0 |

4. **Cliquez sur "Create security group"**

---

### Étape 3.4 : Créer l'Application Load Balancer (ALB)

1. **EC2 → Menu gauche → "Load Balancers"**

2. **Cliquez sur "Create load balancer"**

3. **Sélectionnez "Application Load Balancer"** → Cliquez sur "Create"

4. **Configuration de base**
   - **Name** : `M2i-[VOTRE_PRENOM]-ALB`
   - **Scheme** : `Internet-facing`
   - **IP address type** : `IPv4`

5. **Network mapping**
   - **VPC** : Sélectionnez votre VPC
   - **Mappings** :
     - ✅ Cochez `us-east-1a` → Sélectionnez `M2i-[VOTRE_PRENOM]-Public-1a`
     - ✅ Cochez `us-east-1b` → Sélectionnez `M2i-[VOTRE_PRENOM]-Public-1b`

6. **Security groups**
   - Sélectionnez : `M2i-[VOTRE_PRENOM]-ALB-SG`

7. **Listeners and routing**
   - **Listener** : HTTP:80
   - **Default action** : Créer un nouveau Target Group
     - Cliquez sur "Create target group" (s'ouvre dans un nouvel onglet)

---

### Étape 3.5 : Créer le Target Group

1. **Dans le nouvel onglet "Create target group"** :

2. **Choose a target type** : `Instances`

3. **Configuration**
   - **Target group name** : `M2i-[VOTRE_PRENOM]-TG`
   - **Protocol** : `HTTP`
   - **Port** : `80`
   - **VPC** : Votre VPC

4. **Health checks**
   - **Protocol** : `HTTP`
   - **Path** : `/`
   - **Healthy threshold** : `2`
   - **Unhealthy threshold** : `2`
   - **Timeout** : `5`
   - **Interval** : `30`

5. **Cliquez sur "Next"**

6. **Ne sélectionnez PAS d'instances pour l'instant** (l'ASG les ajoutera automatiquement)

7. **Cliquez sur "Create target group"**

8. **Retournez dans l'onglet de création de l'ALB**
   - Rafraîchissez la liste des Target Groups
   - Sélectionnez : `M2i-[VOTRE_PRENOM]-TG`

9. **Cliquez sur "Create load balancer"**

**✅ Résultat** : Votre ALB est créé et attend des instances !

---

### Étape 3.6 : Créer l'Auto Scaling Group (ASG)

1. **EC2 → Menu gauche → "Auto Scaling Groups"**

2. **Cliquez sur "Create Auto Scaling group"**

3. **Step 1 : Choose launch template**
   - **Name** : `M2i-[VOTRE_PRENOM]-ASG`
   - **Launch template** : Sélectionnez `M2i-[VOTRE_PRENOM]-App-LT`
   - Cliquez sur **"Next"**

4. **Step 2 : Choose instance launch options**
   - **VPC** : Votre VPC
   - **Availability Zones and subnets** :
     - ✅ Cochez `M2i-[VOTRE_PRENOM]-Public-1a`
     - ✅ Cochez `M2i-[VOTRE_PRENOM]-Public-1b`
   - Cliquez sur **"Next"**

5. **Step 3 : Configure advanced options**
   - **Load balancing** :
     - ✅ Cochez "Attach to an existing load balancer"
     - **Choose from your load balancer target groups** :
       - Sélectionnez `M2i-[VOTRE_PRENOM]-TG`
   
   - **Health checks** :
     - ✅ Cochez "ELB" (en plus d'EC2)
     - **Health check grace period** : `300` secondes
   
   - Cliquez sur **"Next"**

6. **Step 4 : Configure group size and scaling**
   - **Desired capacity** : `2`
   - **Minimum capacity** : `2`
   - **Maximum capacity** : `4`
   
   - **Scaling policies** : Laisser par défaut (None)
   
   - Cliquez sur **"Next"**

7. **Step 5 : Add notifications**
   - Cliquez sur **"Next"** (laisser vide)

8. **Step 6 : Add tags**
   - Cliquez sur **"Add tag"**
     - **Key** : `Name`
     - **Value** : `M2i-[VOTRE_PRENOM]-ASG-Instance`
   - Cliquez sur **"Next"**

9. **Step 7 : Review**
   - Vérifiez la configuration
   - Cliquez sur **"Create Auto Scaling group"**

**✅ Résultat** : L'ASG va automatiquement créer 2 instances EC2 dans vos sous-réseaux publics !

---

### Étape 3.7 : Vérifier le Déploiement

1. **Attendez 3-5 minutes** que les instances démarrent et que Docker installe App

2. **Vérifiez les instances** :
   - EC2 → Instances
   - Vous devriez voir 2 instances avec le tag `M2i-[VOTRE_PRENOM]-ASG-Instance`
   - État : `Running`

3. **Vérifiez le Target Group** :
   - EC2 → Target Groups
   - Sélectionnez `M2i-[VOTRE_PRENOM]-TG`
   - Onglet **"Targets"**
   - Les 2 instances doivent avoir le statut `healthy` (après ~2 minutes)

4. **Testez l'ALB** :
   - EC2 → Load Balancers
   - Sélectionnez `M2i-[VOTRE_PRENOM]-ALB`
   - Copiez le **DNS name**
     - Exemple : `M2i-Hannah-ALB-1234567890.us-east-1.elb.amazonaws.com`
   - Ouvrez ce DNS dans un navigateur

**✅ Résultat attendu** : Vous devriez voir la page "Hello color!" 🎉

---

## 🔒 PARTIE 4 : Créer les Sous-réseaux Privés

### Étape 4.1 : Créer les Sous-réseaux Privés

#### Sous-réseau Privé 1

1. **VPC → Subnets → "Create subnet"**

2. **Configuration**
   - **VPC** : Votre VPC
   - **Subnet name** : `M2i-[VOTRE_PRENOM]-Private-1a`
   - **Availability Zone** : `us-east-1a`
   - **IPv4 CIDR block** : Votre plage Private 1
     - Exemple pour Hannah : `10.1.11.0/24`

3. **Cliquez sur "Create subnet"**

#### Sous-réseau Privé 2

1. **Répétez avec** :
   - **Subnet name** : `M2i-[VOTRE_PRENOM]-Private-1b`
   - **Availability Zone** : `us-east-1b`
   - **IPv4 CIDR block** : Votre plage Private 2
     - Exemple pour Hannah : `10.1.12.0/24`

**✅ Résultat** : Vous avez 2 sous-réseaux privés !

---

### Étape 4.2 : Créer une Table de Routage Privée (SANS NAT)

1. **VPC → Route Tables → "Create route table"**

2. **Configuration**
   - **Name** : `M2i-[VOTRE_PRENOM]-Private-RT`
   - **VPC** : Votre VPC

3. **⚠️ NE PAS AJOUTER DE ROUTE vers Internet** (pas de NAT Gateway pour l'instant)

4. **Associer aux sous-réseaux privés**
   - Onglet **"Subnet associations" → "Edit subnet associations"**
   - ✅ Cochez :
     - `M2i-[VOTRE_PRENOM]-Private-1a`
     - `M2i-[VOTRE_PRENOM]-Private-1b`
   - Cliquez sur **"Save associations"**

**✅ Résultat** : Vos sous-réseaux privés sont isolés d'Internet !

---

### Étape 4.3 : Créer une Instance Bastion (Jump Host)

**💡 Pourquoi une bastion ?**
Pour accéder aux instances privées (sans IP publique), nous avons besoin d'une instance intermédiaire dans le sous-réseau public. C'est une pratique courante en production.

1. **EC2 → Instances → "Launch instances"**

2. **Configuration de la Bastion**
   - **Name** : `M2i-[VOTRE_PRENOM]-Bastion`
   - **AMI** : Ubuntu Server 24.04 LTS
   - **Instance type** : t3.micro
   - **Key pair** : ⚠️ **IMPORTANT** : Sélectionnez ou créez une paire de clés SSH
     - Si vous n'en avez pas, cliquez sur "Create new key pair"
     - Nom : `M2i-[VOTRE_PRENOM]-key`
     - Type : RSA
     - Format : `.pem`
     - **Téléchargez et conservez le fichier .pem** (vous en aurez besoin !)
   
   - **Network settings** :
     - **VPC** : Votre VPC
     - **Subnet** : `M2i-[VOTRE_PRENOM]-Public-1a` ⚠️ (PUBLIC !)
     - **Auto-assign public IP** : `Enable`
     - **Security group** : Créez un nouveau SG
       - Nom : `M2i-[VOTRE_PRENOM]-Bastion-SG`
       - Règles entrantes :
         - Type : SSH, Port : 22, Source : `0.0.0.0/0` (ou votre IP pour plus de sécurité)

3. **Cliquez sur "Launch instance"**

4. **Attendez que la bastion soit en état "Running"**

**✅ Résultat** : Vous avez maintenant une bastion accessible depuis Internet via SSH !

---

### Étape 4.4 : Modifier le Security Group des Instances Privées

**⚠️ IMPORTANT** : Pour que la bastion puisse se connecter aux instances privées, il faut autoriser le trafic SSH depuis la bastion.

1. **VPC → Security Groups**

2. **Sélectionnez `M2i-[VOTRE_PRENOM]-Web-SG`** (le SG des instances privées)

3. **Onglet "Inbound rules" → "Edit inbound rules"**

4. **Ajoutez une règle** :
   - **Type** : SSH
   - **Protocol** : TCP
   - **Port** : 22
   - **Source** : `Custom` → Sélectionnez `M2i-[VOTRE_PRENOM]-Bastion-SG`
     - (Cela autorise uniquement la bastion à se connecter en SSH)

5. **Cliquez sur "Save rules"**

**✅ Résultat** : Les instances privées accepteront maintenant les connexions SSH depuis la bastion !

---

### Étape 4.5 : Tester le Déploiement SANS NAT Gateway

#### Créer une Instance Manuellement dans le Sous-réseau Privé

1. **EC2 → Instances → "Launch instances"**

2. **Configuration**
   - **Name** : `M2i-[VOTRE_PRENOM]-Private-Test`
   - **AMI** : Ubuntu Server 24.04 LTS
   - **Instance type** : t3.micro
   - **Key pair** : ⚠️ **Utilisez la MÊME clé** que pour la bastion (`M2i-[VOTRE_PRENOM]-key`)
   
   - **Network settings** :
     - **VPC** : Votre VPC
     - **Subnet** : `M2i-[VOTRE_PRENOM]-Private-1a` ⚠️ (PRIVÉ !)
     - **Auto-assign public IP** : `Disable`
     - **Security group** : Utilisez `M2i-[VOTRE_PRENOM]-Web-SG`
   
   - **User data** (même script Docker/App) :

```bash
#!/bin/bash
echo "=== Début du script ===" > /var/log/lab-test.log
date >> /var/log/lab-test.log

echo "Tentative de téléchargement de Docker..." >> /var/log/lab-test.log
curl -fsSL https://get.docker.com -o get-docker.sh 2>&1 >> /var/log/lab-test.log

if [ $? -eq 0 ]; then
    echo "✅ Docker téléchargé avec succès" >> /var/log/lab-test.log
    sudo sh get-docker.sh >> /var/log/lab-test.log 2>&1
    sudo usermod -aG docker ubuntu
    sudo systemctl enable docker
    sudo systemctl start docker
    sleep 10
    sudo docker run -d --name web-app -p 80:8080 mmumshad/simple-webapp-color 2>&1 >> /var/log/lab-test.log
    echo "✅ App déployé" >> /var/log/lab-test.log
else
    echo "❌ ÉCHEC : Impossible de télécharger Docker (pas d'accès Internet)" >> /var/log/lab-test.log
fi

echo "=== Fin du script ===" >> /var/log/lab-test.log
```

3. **Cliquez sur "Launch instance"**

---

#### Observer l'Échec du Déploiement via la Bastion

**💡 Méthode** : Connexion SSH en 2 étapes (Bastion → Instance Privée)

1. **Attendez 3-5 minutes** que l'instance démarre

2. **Récupérez les informations nécessaires** :
   - **IP publique de la bastion** :
     - EC2 → Instances → Sélectionnez `M2i-[VOTRE_PRENOM]-Bastion`
     - Notez l'"Public IPv4 address"
     - Exemple : `54.123.45.67`
   
   - **IP privée de l'instance de test** :
     - EC2 → Instances → Sélectionnez `M2i-[VOTRE_PRENOM]-Private-Test`
     - Notez l'"Private IPv4 address"
     - Exemple : `10.1.11.25`

3. **Sur votre ordinateur, ouvrez un terminal** (PowerShell, CMD, ou Git Bash)

4. **Connectez-vous à la bastion** :

   **Windows PowerShell** :
   ```powershell
   # Positionnez-vous dans le dossier où se trouve votre clé .pem
   cd C:\Users\[VOTRE_NOM]\Downloads
   
   # Connectez-vous à la bastion
   ssh -i "M2i-[VOTRE_PRENOM]-key.pem" ubuntu@[IP_PUBLIQUE_BASTION]
   ```

   **Exemple** :
   ```powershell
   ssh -i "M2i-Hannah-key.pem" ubuntu@54.123.45.67
   ```

   ⚠️ Si vous avez une erreur de permissions, ignorez-la sous Windows (ou utilisez Git Bash).

5. **Depuis la bastion, connectez-vous à l'instance privée** :

   **⚠️ PROBLÈME** : Votre clé .pem n'est pas sur la bastion !
   
   **Solution** : Utilisez le **SSH Agent Forwarding**
   
   - **Déconnectez-vous de la bastion** (tapez `exit`)
   
   - **Reconnectez-vous avec l'option `-A`** :
   ```powershell
   ssh -A -i "M2i-[VOTRE_PRENOM]-key.pem" ubuntu@[IP_PUBLIQUE_BASTION]
   ```

6. **Maintenant, depuis la bastion, connectez-vous à l'instance privée** :
   ```bash
   ssh ubuntu@[IP_PRIVEE_INSTANCE_TEST]
   ```

   **Exemple** :
   ```bash
   ssh ubuntu@10.1.11.25
   ```

7. **Vérifiez les logs** :
   ```bash
   # Notre log personnalisé
   sudo cat /var/log/lab-test.log
   
   # Log cloud-init
   sudo cat /var/log/cloud-init-output.log
   ```

**❌ Résultat attendu dans `/var/log/lab-test.log`** :
```
=== Début du script ===
[Date]
Tentative de téléchargement de Docker...
❌ ÉCHEC : Impossible de télécharger Docker (pas d'accès Internet)
=== Fin du script ===
```

**Explication** :
- Le script `curl -fsSL https://get.docker.com` **ÉCHOUE**
- Raison : Pas d'accès à Internet (pas de NAT Gateway dans la route table)
- Docker ne peut pas être téléchargé
- App ne peut pas être déployé

8. **Déconnectez-vous** (tapez `exit` deux fois) :
   ```bash
   exit  # Quitte l'instance privée
   exit  # Quitte la bastion
   ```

**✅ Vous avez confirmé** : Sans NAT Gateway, les instances privées ne peuvent PAS accéder à Internet !

---

### Étape 4.6 : Créer un NAT Gateway

1. **VPC → NAT Gateways → "Create NAT gateway"**

2. **Configuration**
   - **Name** : `M2i-[VOTRE_PRENOM]-NAT`
   - **Subnet** : `M2i-[VOTRE_PRENOM]-Public-1a` ⚠️ (doit être dans un sous-réseau PUBLIC)
   - **Elastic IP allocation ID** : Cliquez sur "Allocate Elastic IP"

3. **Cliquez sur "Create NAT gateway"**

4. **Attendez que le statut passe à "Available"** (~2 minutes)

---

### Étape 4.5 : Ajouter une Route vers le NAT Gateway

1. **VPC → Route Tables**

2. **Sélectionnez `M2i-[VOTRE_PRENOM]-Private-RT`**

3. **Onglet "Routes" → "Edit routes"**

4. **Cliquez sur "Add route"**
   - **Destination** : `0.0.0.0/0`
   - **Target** : `NAT Gateway` → Sélectionnez `M2i-[VOTRE_PRENOM]-NAT`

5. **Cliquez sur "Save changes"**

**✅ Résultat** : Les sous-réeaux privés peuvent maintenant accéder à Internet (sortie uniquement) !

---

### Étape 4.7 : Retester le Déploiement AVEC NAT Gateway

1. **Terminez l'instance précédente** `M2i-[VOTRE_PRENOM]-Private-Test`

2. **Créez une NOUVELLE instance** avec la même configuration
   - Même User Data (Docker + App)
   - Même sous-réseau privé

3. **Attendez 5 minutes**

4. **Connectez-vous via la bastion** (même méthode qu'avant) :
   ```powershell
   # Sur votre PC
   ssh -A -i "M2i-[VOTRE_PRENOM]-key.pem" ubuntu@[IP_PUBLIQUE_BASTION]
   ```
   
   ```bash
   # Depuis la bastion
   ssh ubuntu@[IP_PRIVEE_NOUVELLE_INSTANCE]
   ```

5. **Vérifiez les logs** :
   ```bash
   # Notre log personnalisé
   sudo cat /var/log/lab-test.log
   
   # Vérifier que Docker tourne
   sudo docker ps
   ```

**✅ Résultat attendu dans `/var/log/lab-test.log`** :
```
=== Début du script ===
[Date]
Tentative de téléchargement de Docker...
✅ Docker téléchargé avec succès
[...logs d'installation...]
✅ App déployé
=== Fin du script ===
```

**Vérification Docker** :
```bash
ubuntu@ip-10-1-11-XX:~$ sudo docker ps
CONTAINER ID   IMAGE                                  STATUS          PORTS
XXXXXXXXXXXX   mmumshad/simple-webapp-color          Up 2 minutes    0.0.0.0:80->8080/tcp
```

**✅ Résultat** :
- Docker est installé avec succès
- App tourne dans un conteneur
- Le conteneur est accessible sur le port 80 (en interne)

**💡 Différence** :
- **SANS NAT** : ❌ Échec (pas d'accès Internet)
- **AVEC NAT** : ✅ Succès (téléchargement possible)

---

## 🌐 PARTIE 5 : Accéder aux Instances Privées via ALB

### Étape 5.1 : Créer un Nouveau Target Group pour le Sous-réseau Privé

1. **EC2 → Target Groups → "Create target group"**

2. **Configuration**
   - **Target type** : `Instances`
   - **Target group name** : `M2i-[VOTRE_PRENOM]-Private-TG`
   - **Protocol** : `HTTP:80`
   - **VPC** : Votre VPC
   - **Health check path** : `/`

3. **Cliquez sur "Next"**

4. **Register targets** :
   - ✅ Cochez votre instance privée
   - Cliquez sur "Include as pending below"

5. **Cliquez sur "Create target group"**

---

### Étape 5.2 : Ajouter un Listener à l'ALB

1. **EC2 → Load Balancers**

2. **Sélectionnez `M2i-[VOTRE_PRENOM]-ALB`**

3. **Onglet "Listeners" → Sélectionnez le listener HTTP:80**

4. **Cliquez sur "View/edit rules"**

5. **Ajoutez une règle de forwarding** :
   - Path : `/private*` → Forward to `M2i-[VOTRE_PRENOM]-Private-TG`
   - Default : Continue forwarding to `M2i-[VOTRE_PRENOM]-TG` (public)

6. **Testez** :
   ```
   http://[DNS_ALB]/          → Instances publiques
   http://[DNS_ALB]/private   → Instances privées
   ```

**✅ Résultat** : L'ALB route le trafic vers les instances publiques ET privées !

---

## 🧹 PARTIE 6 : Nettoyage — ⚠️ TRÈS IMPORTANT

**⚠️ ATTENTION** : Supprimez TOUTES les ressources pour éviter les coûts AWS !

### Ordre de Suppression (important !) :

1. **Instances de test manuelles** :
   - EC2 → Instances
   - Sélectionnez `M2i-[VOTRE_PRENOM]-Private-Test` (toutes les versions)
   - Sélectionnez `M2i-[VOTRE_PRENOM]-Bastion`
   - Instance state → Terminate

2. **Auto Scaling Group** :
   - EC2 → Auto Scaling Groups
   - Sélectionnez votre ASG → Actions → Delete
   - Confirmez

3. **Application Load Balancer** :
   - EC2 → Load Balancers
   - Sélectionnez votre ALB → Actions → Delete

4. **Target Groups** :
   - EC2 → Target Groups
   - Sélectionnez vos TG → Actions → Delete

5. **Launch Template** :
   - EC2 → Launch Templates
   - Sélectionnez votre template → Actions → Delete

6. **NAT Gateway** :
   - VPC → NAT Gateways
   - Sélectionnez votre NAT → Actions → Delete NAT gateway
   - Confirmez

7. **Elastic IP** :
   - VPC → Elastic IPs
   - Sélectionnez l'EIP du NAT → Actions → Release Elastic IP address

8. **Internet Gateway** :
   - VPC → Internet Gateways
   - Sélectionnez votre IGW → Actions → Detach from VPC
   - Puis Actions → Delete internet gateway

9. **Subnets** :
   - VPC → Subnets
   - Sélectionnez tous vos subnets → Actions → Delete subnet

10. **Route Tables** :
    - VPC → Route Tables
    - Sélectionnez vos RT (sauf la main) → Actions → Delete route table

11. **Security Groups** :
    - VPC → Security Groups
    - Sélectionnez vos SG (sauf default) : Web-SG, ALB-SG, Bastion-SG
    - Actions → Delete security groups

12. **Network ACLs** :
    - VPC → Network ACLs
    - Sélectionnez vos NACL (sauf default) → Actions → Delete network ACL

13. **VPC** (en dernier) :
    - VPC → Your VPCs
    - Sélectionnez votre VPC → Actions → Delete VPC

---

## ✅ Validation du Lab

Avant de nettoyer, répondez aux questions :

1. **Quelle est votre plage VPC CIDR ?**
   - Exemple : `10.1.0.0/16`

2. **Combien de sous-réseaux publics avez-vous créés ?**
   - Réponse : 2 (dans 2 AZ différentes)

3. **Quelle est la différence entre un sous-réseau public et privé ?**
   - Public : Route vers Internet Gateway (accessible depuis Internet)
   - Privé : Pas de route directe vers Internet (isolé)

4. **Pourquoi le déploiement a échoué SANS NAT Gateway ?**
   - Pas d'accès à Internet pour télécharger Docker/App

5. **Quel est le rôle de l'ALB dans cette architecture ?**
   - Répartir le trafic entre plusieurs instances
   - Haute disponibilité (multi-AZ)
   - Health checks automatiques

6. **Combien d'instances l'ASG a-t-il créé automatiquement ?**
   - Réponse : 2 (selon Desired capacity)

---

## 🎓 Concepts Clés Retenus

| Concept | Explication |
|---------|-------------|
| **VPC** | Réseau virtuel isolé dans AWS |
| **Subnet Public** | Sous-réseau avec route vers Internet Gateway |
| **Subnet Private** | Sous-réseau isolé, sans accès direct à Internet |
| **Internet Gateway** | Passerelle pour accéder à Internet (bidirectionnel) |
| **NAT Gateway** | Passerelle pour accès Internet sortant UNIQUEMENT (depuis privé) |
| **Route Table** | Table de routage définissant où envoyer le trafic |
| **NACL** | Pare-feu stateless au niveau du sous-réseau |
| **Security Group** | Pare-feu stateful au niveau de l'instance |
| **ALB** | Application Load Balancer (Layer 7) |
| **ASG** | Auto Scaling Group (gestion automatique des instances) |
| **Launch Template** | Modèle de configuration pour instances EC2 |
| **Target Group** | Groupe d'instances cibles pour le load balancer |

---

## 📊 Comparaison : Public vs Private Subnet

| Aspect | Public Subnet | Private Subnet |
|--------|---------------|----------------|
| **Route vers Internet** | ✅ Via Internet Gateway | ❌ Pas directement |
| **IP Publique** | ✅ Auto-assignée | ❌ Non |
| **Accès entrant** | ✅ Depuis Internet | ❌ Non (sauf via ALB/NAT) |
| **Accès sortant** | ✅ Direct | ✅ Via NAT Gateway |
| **Cas d'usage** | Web servers, ALB | Bases de données, App servers |
| **Sécurité** | ⚠️ Exposé | ✅ Isolé |

---

## 📚 Ressources Supplémentaires

- [VPC Documentation](https://docs.aws.amazon.com/vpc/)
- [NAT Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html)
- [Application Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/)
- [Auto Scaling Groups](https://docs.aws.amazon.com/autoscaling/ec2/userguide/AutoScalingGroup.html)

---

**Durée estimée** : 2-3 heures

🚀 **Bon lab !** N'oubliez pas de tout supprimer à la fin !