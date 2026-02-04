# LAB 05 — Service de Calcul : EC2 et Types d'Instances

## 🎯 Objectifs

À la fin de ce lab, vous serez capable de :
- ✅ Comprendre les différents **types d'instances EC2** (T2, T3, T3 small, etc.)
- ✅ Créer une **instance EC2** via la console AWS
- ✅ Configurer une instance avec **Docker** via User Data Script
- ✅ Déployer une **application containerisée** sur EC2
- ✅ Accéder à l'application via l'**adresse IP publique**

## 📋 Prérequis

- ✅ Accès à la console AWS (voir credentials_[PRENOM].txt)
- ✅ Région : **Virginia (us-east-1)**
- ✅ Permissions pour créer des ressources EC2

## 📚 Vue d'ensemble : EC2 et Types d'Instances

### Qu'est-ce qu'EC2 ?

**EC2** (Elastic Compute Cloud) est le service de **calcul** d'AWS. Il fournit des serveurs virtuels (instances) à la demande.

### Types d'Instances

AWS propose plusieurs **familles de types d'instances**, chacune optimisée pour un cas d'usage :

| Type | Usage | Exemple |
|------|-------|---------|
| **T2/T3** | Usage général, burstable, débutants | Applications web, tests, petits projets |
| **M5/M6** | Usage général, performances équilibrées | Serveurs d'applications stables |
| **C5/C6** | Calcul haute performance | Traitement de données, IA/ML |
| **R5/R6** | Mémoire optimisée | Bases de données, caches |
| **I3/I4** | Stockage optimisé (I/O rapide) | Data warehouses |
| **G4/G5** | GPU (calcul graphique) | ML, jeux, rendering |

### T2 vs T3 vs T3 Small

- **T2** : Ancienne génération, burstable, économique
- **T3** : Nouvelle génération, burstable, plus performant que T2
- **T3 Small** (t3.small) : Taille très petite de la génération T3
  - 1 vCore, 2 Go RAM, $0.0104/heure

**Burstable** = L'instance accumule des "credits CPU" au repos et peut les utiliser en pic d'activité.

## 🚀 Étape par Étape : Créer une Instance EC2

### Étape 1 : Accéder à la Console EC2

1. Connectez-vous à la [Console AWS](https://console.aws.amazon.com)
   - Utilisateur : `M2i_[VOTRE_PRENOM]`
   - Mot de passe : Voir credentials_[PRENOM].txt
   - Région : **Virginia (us-east-1)**

2. Accédez au service **EC2**
   - Recherchez "EC2" dans la barre de recherche
   - Cliquez sur "EC2"

3. Accédez à l'onglet **Instances**
   - Menu gauche > "Instances"
   - Bouton orange **"Lancer une instance"** (ou "Launch instances")

### Étape 2 : Configurer le Nom et l'Image

1. **Étape 1 : Nom et images**
   - Nom : `M2i_[VOTRE_PRENOM]`
     ```
     Exemple : M2i_John
     ```

2. **Sélectionnez une AMI (Amazon Machine Image)**
   - Sélectionnez **"Ubuntu Server 24.04 LTS"** (gratuit, léger, et compatible Docker)
   - Assurez-vous que c'est marqué **"Éligible à la couche gratuite"**
   - Cliquez sur le bouton bleu "Sélectionner"

### Étape 3 : Choisir le Type d'Instance

1. **Type d'instance**
   - Sélectionnez **t3.small**
   - Vérifiez qu'il est marqué **"Éligible à la couche gratuite"** (si vous êtes nouveau compte)

   **Alternative** : Si vous voulez tester plusieurs types :
   - Créez d'abord une instance avec **t3.small**
   - Puis une deuxième avec **t2.micro** pour comparer

2. Cliquez sur **"Suivant : Détails de l'instance"**

### Étape 4 : Configurer Docker via User Data Script

1. Allez à la section **"Détails avancés"** (en bas)

2. Trouvez le champ **"Données utilisateur"** (User data)

3. Collez le **Docker Script officiel** :

```bash
#!/bin/bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Ajout de l'utilisateur ubuntu au groupe docker
sudo usermod -aG docker ubuntu

# Démarrage de Docker
sudo systemctl enable docker
sudo systemctl start docker

# Attendre que Docker soit prêt
sleep 10

# Déploiement d'une application de test (Nginx)
sudo docker run -d --name my-app -p 80:80 nginx:latest

echo "✅ Docker installé avec succès via Docker Script !"
echo "✅ Nginx en cours d'exécution sur le port 80"
```

**Important - Points clés de ce script** :
- ✅ Utilise le **Docker Script officiel** (https://get.docker.com)
- ✅ Cette méthode est l'approche **recommandée par Docker**
- ✅ Compatible avec Ubuntu et la plupart des distributions Linux
- ✅ **Fonctionne du premier coup** 
- ✅ Ajoute l'utilisateur `ubuntu` au groupe docker
- ✅ Lance un conteneur **Nginx** sur le port 80

**Note sur la containerisation** : Nous n'apprenons PAS la containerisation ici. Nous l'utilisons simplement comme outil pour déployer rapidement une application. Focalisez-vous sur le déploiement, pas sur les détails de Docker.

**Lien officiel Docker Script** : https://get.docker.com/

### Étape 5 : Configurer le Stockage

1. **Stockage (EBS)**
   - Volume : **8 Go** (gratuit pour les 12 premiers mois)
   - Type : **gp3 (SSD général)**
   - Suppression à la fin : Cocher "Oui"

2. Cliquez sur **"Suivant : Ajouter des tags"**

### Étape 6 : Ajouter des Tags

1. Ajoutez un tag pour identifier votre instance :
   - Clé : `Name`
   - Valeur : `M2i_[VOTRE_PRENOM]`

2. Cliquez sur **"Suivant : Configurer le groupe de sécurité"**

### Étape 7 : Configurer le Groupe de Sécurité

1. **Groupe de sécurité**
   - Créer un nouveau groupe de sécurité : `M2i-[VOTRE_PRENOM]-sg`

2. **Règles d'entrée (Inbound rules)**
   - Règle 1 : **SSH (port 22)**
     - Type : SSH
     - Source : **Anywhere (0.0.0.0/0)** ⚠️ Déconseillé en production, OK pour ce lab
   
   - Règle 2 : **HTTP (port 80)**
     - Type : HTTP
     - Source : Anywhere (0.0.0.0/0)
   
   - Règle 3 : **HTTPS (port 443)** (optionnel pour ce lab)
     - Type : HTTPS
     - Source : Anywhere (0.0.0.0/0)

3. Cliquez sur **"Vérifier et lancer"**

### Étape 8 : Vérifier et Lancer

1. **Vérification récapitulatif**
   - Nom : M2i_[VOTRE_PRENOM] ✅
   - AMI : Ubuntu Server 24.04 LTS ✅
   - Type : t3.small ✅
   - Région : Virginia (us-east-1) ✅
   - User Data : Script Docker Ubuntu ✅
   - Groupe de sécurité : Ports 22 (SSH), 80 (HTTP), 443 (HTTPS) ouverts ✅

2. Cliquez sur **"Lancer l'instance"**

3. **Sélectionner ou créer une clé SSH** (optionnel pour ce lab, non nécessaire si connexion via console)

4. Cliquez sur **"Lancer l'instance"** ✅

### Étape 9 : Vérifier que l'Instance Démarre

1. Vous verrez un écran de confirmation avec un ID d'instance (`i-xxxxxxx`)

2. Cliquez sur cet ID ou allez dans **"Instances"** pour voir l'état

3. Attendez que l'instance passe à l'état **"running"** (vert)
   - Colonnes importantes :
     - **État** : Doit être "running"
     - **Adresse IP publique** : Copie cette adresse !
     - **Vérifications de statut** : Doit être "2/2 passed"

### Étape 10 : Accéder à Nginx

1. Une fois l'instance **running**, copiez son **Adresse IP publique**
   - Exemple : `54.123.45.67`

2. Ouvrez un navigateur et accédez à :
   ```
   http://[ADRESSE_IP_PUBLIQUE]
   ```
   Exemple : `http://54.123.45.67`

3. Vous devriez voir la **page de bienvenue Nginx** ! 🎉

   ```
   Welcome to nginx!
   If you see this page, the nginx web server is successfully installed and 
   working. Further configuration is required.
   ```

---

## 🐳 Étape 11 : Déployer une Application Personnalisée avec Dockerfile

Maintenant que Docker est installé, nous allons **construire et déployer notre propre image Docker** au lieu d'utiliser l'image Nginx par défaut.

### Étape 11.1 : Se connecter à l'instance EC2

1. **Allez dans EC2 > Instances**

2. **Sélectionnez votre instance** `M2i_[VOTRE_PRENOM]`

3. **Cliquez sur "Connect"** (en haut à droite)

4. **Onglet "EC2 Instance Connect"** → Cliquez sur **"Connect"**
   - Une fenêtre de terminal s'ouvre dans le navigateur ✅

### Étape 11.2 : Arrêter le conteneur Nginx par défaut

```bash
# Vérifier les conteneurs en cours d'exécution
sudo docker ps

# Arrêter et supprimer le conteneur Nginx par défaut
sudo docker stop my-app
sudo docker rm my-app
```

### Étape 11.3 : Créer le Dockerfile

1. **Créez un fichier Dockerfile** avec l'éditeur `vi` :
   ```bash
   vi Dockerfile
   ```

2. **Appuyez sur `i`** pour passer en mode INSERT (insertion)

3. **Copiez et collez le contenu suivant** :

```dockerfile
FROM ubuntu:22.04 AS files
LABEL maintainer='ANSELME'
RUN apt-get update
RUN DEBIAN_FRONTEND=noninteractive apt-get install git -y
RUN mkdir /opt/files
RUN git clone https://github.com/daviddias/static-webpage-example.git /opt/files/ 
RUN ls /opt/files/

FROM nginx:stable-alpine3.17-slim
LABEL maintainer='ANSELME'
COPY --from=files /opt/files/src/ /usr/share/nginx/html/
ENTRYPOINT ["/docker-entrypoint.sh"]
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

4. **Appuyez sur `ESC`** pour quitter le mode INSERT

5. **Tapez `:wq`** et appuyez sur **ENTER** pour sauvegarder et quitter
   - `:w` = write (sauvegarder)
   - `:q` = quit (quitter)

6. **Vérifiez que le fichier a été créé** :
   ```bash
   cat Dockerfile
   ```

**📝 Explication du Dockerfile** :
- **Stage 1 (ubuntu:22.04)** : Clone un site web statique depuis GitHub
- **Stage 2 (nginx)** : Copie le site web dans une image Nginx légère
- **Multi-stage build** : Image finale optimisée et légère
- **Expose 80** : L'application écoute sur le port 80

### Étape 11.4 : Construire l'image Docker

```bash
# Construire l'image avec le tag "my-webapp"
sudo docker build -t my-webapp .
```

**Sortie attendue** :
```
[+] Building 45.2s (14/14) FINISHED
=> [internal] load build definition from Dockerfile
=> [files 1/6] FROM docker.io/library/ubuntu:22.04
=> [files 2/6] RUN apt-get update
=> [files 3/6] RUN DEBIAN_FRONTEND=noninteractive apt-get install git -y
=> [files 4/6] RUN mkdir /opt/files
=> [files 5/6] RUN git clone https://github.com/daviddias/static-webpage-example.git /opt/files/
=> [stage-1 1/2] FROM docker.io/library/nginx:stable-alpine3.17-slim
=> [stage-1 2/2] COPY --from=files /opt/files/src/ /usr/share/nginx/html/
=> exporting to image
=> => naming to docker.io/library/my-webapp
```

**⏱️ Durée** : Environ 1-2 minutes (téléchargement des images de base)

### Étape 11.5 : Vérifier que l'image a été créée

```bash
# Lister les images Docker
sudo docker images
```

**Sortie attendue** :
```
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
my-webapp    latest    abc123def456   10 seconds ago   45.2MB
nginx        latest    xyz789ghi012   2 weeks ago      187MB
ubuntu       22.04     klm345nop678   3 weeks ago      77.8MB
```

### Étape 11.6 : Lancer le conteneur en mode détaché

```bash
# Lancer le conteneur en arrière-plan (-d = detached)
sudo docker run -d --name my-custom-app -p 80:80 my-webapp
```

**Options expliquées** :
- `-d` : Mode détaché (le conteneur tourne en arrière-plan)
- `--name my-custom-app` : Nom du conteneur
- `-p 80:80` : Mappe le port 80 de l'hôte vers le port 80 du conteneur
- `my-webapp` : Nom de l'image à utiliser

**Sortie attendue** :
```
a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0u1v2w3x4y5z6
```
(ID du conteneur)

### Étape 11.7 : Vérifier que le conteneur est en cours d'exécution

```bash
# Vérifier l'état du conteneur
sudo docker ps
```

**Sortie attendue** :
```
CONTAINER ID   IMAGE       COMMAND                  STATUS         PORTS                NAMES
a1b2c3d4e5f6   my-webapp   "/docker-entrypoint.…"   Up 5 seconds   0.0.0.0:80->80/tcp   my-custom-app
```

✅ **STATUS** doit être **"Up"** (en cours d'exécution)

### Étape 11.8 : Accéder à l'application web personnalisée

1. **Récupérez l'adresse IP publique** de votre instance EC2
   - Retournez dans la console EC2 > Instances
   - Copiez l'**Adresse IPv4 publique**

2. **Ouvrez un navigateur** et accédez à :
   ```
   http://[VOTRE_IP_PUBLIQUE]
   ```
   Exemple : `http://54.123.45.67`

3. **Vous devriez voir le site web statique** cloné depuis GitHub ! 🎉

**Résultat attendu** :
- ✅ Une page web avec un design personnalisé (pas la page par défaut de Nginx)
- ✅ Le site provient du repository `static-webpage-example`

### 🔍 Dépannage

**Problème : La page ne s'affiche pas**

1. **Vérifiez que le conteneur est bien en cours d'exécution** :
   ```bash
   sudo docker ps
   ```
   - Le conteneur `my-custom-app` doit apparaître avec STATUS "Up"

2. **Vérifiez les logs du conteneur** :
   ```bash
   sudo docker logs my-custom-app
   ```

3. **Vérifiez le groupe de sécurité** :
   - EC2 Console > Instances > Sélectionnez votre instance
   - Onglet "Sécurité" > Groupe de sécurité
   - Vérifiez que le **port 80 (HTTP)** est ouvert pour `0.0.0.0/0`

4. **Attendez quelques secondes** :
   - Docker peut prendre 5-10 secondes pour démarrer complètement

---

## ✅ Validation du Lab

Avant de nettoyer, répondez aux questions :

1. **Quel est votre type d'instance ?**
   - Allez dans l'onglet "Détails" de votre instance
   - Notez le type exact (ex: t3.small)

2. **Combien de vCPU et de RAM votre instance a-t-elle ?**
   - Type t3.small : 1 vCore, 2 Go RAM
   - Type t3.medium : 1 vCore, 4 Go RAM
   - Type t3.large : 2 vCore, 8 Go RAM

3. **Quel est le coût horaire approximatif de votre instance ?**
   - Consulter [AWS Pricing Calculator](https://calculator.aws/)

4. **Pouvez-vous accéder à l'application Nginx ?**
   - Oui / Non

5. **Quels ports sont ouverts dans le groupe de sécurité ?**
   - 22 (SSH), 80 (HTTP), 443 (HTTPS)

## 🧹 Nettoyage — ⚠️ TRÈS IMPORTANT

⚠️ **VOUS DEVEZ SUPPRIMER VOTRE INSTANCE** pour éviter les frais AWS !

**Important** : Ne pas seulement l'arrêter, mais la **TERMINER** (supprimer définitivement).

### Étapes de suppression :

1. Allez dans **"Instances"**

2. Sélectionnez votre instance `M2i_[VOTRE_PRENOM]`

3. Cliquez sur le menu **"État de l'instance"** > **"Terminer l'instance"** (Terminate)
   - ⚠️ **"Arrêter"** (Stop) = L'instance est suspendue mais continue de coûter
   - ✅ **"Terminer"** (Terminate) = L'instance est supprimée définitivement ✅ **C'EST CE QU'IL FAUT FAIRE**

4. Une boîte de dialogue vous demandera confirmation :
   ```
   Cette action ne peut pas être annulée. 
   Êtes-vous certain de vouloir terminer cette instance ?
   ```

5. Tapez `delete` (ou cochez la case) et cliquez sur **"Terminer"**

6. L'instance passera à l'état **"shutting-down"** puis **"terminated"** (gris)

### Vérification

Une fois terminée :
- L'instance affichera l'état **"terminated"** (gris, barrée)
- Les coûts **cesseront immédiatement**
- Les données seront **définitivement supprimées** (EBS aussi)

**⚠️ RAPPEL** : Cette action est **définitive et irréversible**. Assurez-vous que vous n'en avez plus besoin avant de terminer l'instance.

## 🔑 Concepts Clés Retenus

| Concept | Explication |
|---------|------------|
| **EC2** | Service de serveurs virtuels (instances) d'AWS |
| **Types d'instances** | T2, T3, M5, C5, etc. - chacun optimisé pour un cas d'usage |
| **User Data** | Script qui s'exécute au démarrage de l'instance |
| **Groupe de sécurité** | Pare-feu pour contrôler les ports entrants/sortants |
| **Adresse IP publique** | IP temporaire pour accéder à l'instance depuis Internet |
| **AMI** | Image préconfigurée du système d'exploitation |

## 📚 Ressources Supplémentaires

- [Documentation EC2](https://docs.aws.amazon.com/ec2/)
- [Types d'instances EC2](https://aws.amazon.com/ec2/instance-types/)
- [Groupes de sécurité EC2](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html)
- [Docker (bonus, non obligatoire)](https://docs.docker.com/)

---

**Durée estimée** : 30-45 minutes

**Prochaine étape** : Essayez différents types d'instances (t3.medium, t3.large) et comparez les performances !

🚀 Bon lab !
