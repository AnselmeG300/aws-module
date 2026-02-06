# LAB 09 — Configuration des Classes de Stockage S3 via la Console

## 🎯 Objectifs

À la fin de ce lab, vous serez capable de :
- ✅ Comprendre les **différentes classes de stockage S3** et leurs cas d'usage
- ✅ **Configurer la classe de stockage** lors de l'upload via la console
- ✅ **Modifier la classe** d'objets existants dans S3
- ✅ Mettre en place des **règles de transition automatique** (Lifecycle)
- ✅ Comparer les **coûts et performances** de chaque classe
- ✅ Choisir la **classe optimale** selon le cas d'usage

---

## 📋 Prérequis

- ✅ Accès à la console AWS
- ✅ Région : **N. Virginia (us-east-1)** ou région de votre choix
- ✅ Permissions IAM : `S3FullAccess` pour ce lab
- ✅ Navigateur web (Chrome, Firefox, Edge)

---

## 📚 Concepts clés : Les Classes de Stockage S3

### 🗂️ Vue d'ensemble des classes de stockage

Amazon S3 propose **8 classes de stockage** différentes, chacune optimisée pour des patterns d'accès spécifiques :

| Classe | Cas d'usage | Durabilité | Disponibilité | Latence | Coût |
|--------|-------------|------------|---------------|---------|------|
| **S3 Standard** | Accès fréquent | 99.999999999% (11 9's) | 99.99% | ms | $$$ |
| **S3 Intelligent-Tiering** | Accès imprévisible | 99.999999999% | 99.9% | ms | $$-$$$ |
| **S3 Standard-IA** | Accès peu fréquent | 99.999999999% | 99.9% | ms | $$ |
| **S3 One Zone-IA** | Données non critiques, accès rare | 99.999999999% | 99.5% | ms | $ |
| **S3 Glacier Instant Retrieval** | Archive, accès trimestriel | 99.999999999% | 99.9% | ms | $ |
| **S3 Glacier Flexible Retrieval** | Archive, accès annuel | 99.999999999% | 99.99% | min-heures | ¢¢ |
| **S3 Glacier Deep Archive** | Archive long terme (7-10 ans) | 99.999999999% | 99.99% | 12-48h | ¢ |
| **S3 Express One Zone** | Haute performance, latence ms | 99.999999999% | 99.95% | µs-ms | $$$$ |

---

### 💰 Comparaison des coûts (us-east-1, février 2026)

| Classe | Stockage ($/GB/mois) | Requête GET ($/1000) | Récupération ($/GB) | Durée min. |
|--------|---------------------|---------------------|---------------------|------------|
| **Standard** | $0.023 | $0.0004 | Gratuit | Aucune |
| **Intelligent-Tiering** | $0.023-$0.0125 | $0.0004 | Gratuit | 30 jours |
| **Standard-IA** | $0.0125 | $0.001 | $0.01 | 30 jours |
| **One Zone-IA** | $0.01 | $0.001 | $0.01 | 30 jours |
| **Glacier Instant** | $0.004 | $0.01 | $0.03 | 90 jours |
| **Glacier Flexible** | $0.0036 | $0.0004 | $0.02-$0.10 | 90 jours |
| **Glacier Deep Archive** | $0.00099 | $0.0004 | $0.02 | 180 jours |
| **Express One Zone** | $0.16 | $0.0008 | Gratuit | Aucune |

---

### 🔍 Quand utiliser chaque classe ?

#### ✅ S3 Standard
- **Usage** : Données fréquemment accédées
- **Exemples** : Sites web actifs, CDN, applications mobiles, analytics temps réel
- **Avantage** : Latence minimale, haute disponibilité
- **Inconvénient** : Coût de stockage le plus élevé

#### 🧠 S3 Intelligent-Tiering
- **Usage** : Données avec patterns d'accès **imprévisibles**
- **Exemples** : Data lakes, logs d'applications, backups récents
- **Avantage** : Optimisation automatique des coûts sans frais de récupération
- **Inconvénient** : Frais de monitoring ($0.0025/1000 objets)

#### 📦 S3 Standard-IA (Infrequent Access)
- **Usage** : Données accédées moins d'une fois par mois
- **Exemples** : Backups, logs anciens, DR (Disaster Recovery)
- **Avantage** : 50% moins cher que Standard
- **Inconvénient** : Frais de récupération, durée minimale 30 jours

#### 🌍 S3 One Zone-IA
- **Usage** : Données non critiques, reproductibles
- **Exemples** : Thumbnails, données transformées, copies secondaires
- **Avantage** : 20% moins cher que Standard-IA
- **Inconvénient** : Une seule AZ (risque de perte si AZ défaillante)

#### ❄️ S3 Glacier Instant Retrieval
- **Usage** : Archives rarement accédées (1 fois/trimestre)
- **Exemples** : Images médicales, médias archivés, données de conformité
- **Avantage** : 68% moins cher que Standard-IA, récupération en ms
- **Inconvénient** : Frais de récupération élevés, durée min. 90 jours

#### 🧊 S3 Glacier Flexible Retrieval
- **Usage** : Archives annuelles ou moins fréquentes
- **Exemples** : Données réglementaires, backups annuels, historiques
- **Avantage** : 84% moins cher que Standard
- **Inconvénient** : Récupération en 1-5 min (accéléré), 3-5h (standard), 5-12h (bulk)

#### 🏔️ S3 Glacier Deep Archive
- **Usage** : Archive long terme (7-10 ans), conformité légale
- **Exemples** : Archives médicales, données financières, backups anciens
- **Avantage** : Coût de stockage le plus faible ($1/TB/mois)
- **Inconvénient** : Récupération en 12-48h, durée min. 180 jours

#### ⚡ S3 Express One Zone
- **Usage** : Applications nécessitant des **milliers de requêtes/seconde**
- **Exemples** : ML training, analytics haute performance, traitement vidéo temps réel
- **Avantage** : Latence ultra-faible (millisecondes à microsecondes)
- **Inconvénient** : Coût très élevé, une seule AZ

---

## 🚀 PARTIE 1 : Créer un bucket et configurer la classe par défaut

### Étape 1.1 : Créer un bucket S3 via la console

1. **Connexion à la console AWS**
   - Ouvrez https://console.aws.amazon.com
   - Région : **US East (N. Virginia) us-east-1**

2. **Création du bucket**
   - Service : **S3** (barre de recherche ou menu Services)
   - Cliquez sur **Create bucket**

3. **Configuration du bucket**
   - **Bucket name** : `m2i-storage-classes-<votre-prenom>-bucket`
     - Exemple : `m2i-storage-classes-anselme-bucket`
   - **Region** : `US East (N. Virginia) us-east-1`
   - **Block Public Access** : ✅ Laissez tout coché
   - **Versioning** : Désactivé (pour ce lab)
   - **Default encryption** : Server-side encryption with Amazon S3 managed keys (SSE-S3)
   - Cliquez sur **Create bucket**

---

## 📤 PARTIE 2 : Uploader des fichiers avec différentes classes de stockage

### Étape 2.1 : Préparer des fichiers de test (local)

Créez 5 fichiers de test sur votre ordinateur :

1. **fichier-standard.txt** (actif, fréquent)
   ```
   Fichier accédé fréquemment - Classe STANDARD
   Date: 2026-02-06
   Usage: Site web actif
   ```

2. **fichier-intelligent.txt** (pattern imprévisible)
   ```
   Fichier avec accès imprévisible - Classe INTELLIGENT-TIERING
   Date: 2026-02-06
   Usage: Data lake
   ```

3. **fichier-standard-ia.txt** (accès rare)
   ```
   Fichier accédé rarement - Classe STANDARD-IA
   Date: 2026-02-06
   Usage: Backup mensuel
   ```

4. **fichier-glacier.txt** (archive)
   ```
   Fichier archivé - Classe GLACIER Instant Retrieval
   Date: 2026-02-06
   Usage: Archive trimestrielle
   ```

5. **fichier-deep-archive.txt** (archive long terme)
   ```
   Fichier archive long terme - Classe GLACIER Deep Archive
   Date: 2026-02-06
   Usage: Conformité réglementaire 10 ans
   ```

---

### Étape 2.2 : Uploader avec S3 Standard (classe par défaut)

1. **Ouvrir le bucket**
   - Console S3 → Buckets → Cliquez sur `m2i-storage-classes-<votre-prenom>-bucket`

2. **Upload du fichier**
   - Cliquez sur **Upload**
   - **Add files** → Sélectionnez `fichier-standard.txt`
   - **Storage class** : Par défaut, `Standard` est sélectionné
   - ✅ **Ne modifiez rien** pour ce premier fichier
   - Cliquez sur **Upload** (en bas de page)

3. **Vérification**
   - Une fois l'upload terminé, cliquez sur **Close**
   - Vous verrez `fichier-standard.txt` dans la liste
   - Cliquez sur le nom du fichier
   - Onglet **Properties** → Section **Storage class** : `Standard`

---

### Étape 2.3 : Uploader avec Intelligent-Tiering

1. **Upload du fichier**
   - Retour à la liste des objets (bouton retour ou breadcrumb)
   - Cliquez sur **Upload** → **Add files** → `fichier-intelligent.txt`

2. **Modifier la classe de stockage**
   - **Descendez** jusqu'à la section **Storage class**
   - Cliquez sur **Change default storage class** (ou développez la section)
   - Sélectionnez **Intelligent-Tiering**
   - 📖 **Lisez la description** : 
     > "Automatically optimizes storage costs by moving objects between access tiers"

3. **Finaliser l'upload**
   - Cliquez sur **Upload**
   - **Vérification** : Propriétés → Storage class = `Intelligent-Tiering`

---

### Étape 2.4 : Uploader avec Standard-IA

1. **Upload**
   - **Upload** → **Add files** → `fichier-standard-ia.txt`
   - **Storage class** : Sélectionnez **Standard-IA**
   - 📖 Description :
     > "Infrequent Access - For data accessed less frequently but requires rapid access when needed"

2. **Options supplémentaires** (optionnel)
   - Vous pouvez définir des **tags** : `Backup=Monthly`
   - **Metadata** : `Classification=Backup`

3. **Upload** et vérification

---

### Étape 2.5 : Uploader avec Glacier Instant Retrieval

1. **Upload**
   - **Upload** → **Add files** → `fichier-glacier.txt`
   - **Storage class** : Sélectionnez **Glacier Instant Retrieval**
   - 📖 Description :
     > "Archive data accessed once a quarter with instant retrieval in milliseconds"

2. **⚠️ Important** : Notez les limitations
   - Durée minimale : **90 jours**
   - Taille minimale : **128 KB** par objet
   - Frais de récupération : $0.03/GB

3. **Upload** et vérification

---

### Étape 2.6 : Uploader avec Glacier Deep Archive

1. **Upload**
   - **Upload** → **Add files** → `fichier-deep-archive.txt`
   - **Storage class** : Sélectionnez **Glacier Deep Archive**
   - 📖 Description :
     > "Lowest-cost storage for long-term archive, retrieval time of 12 hours"

2. **⚠️ Attention** :
   - Durée minimale : **180 jours**
   - Récupération : 12-48 heures
   - Coût le plus bas : $0.00099/GB/mois

3. **Upload** et vérification

---

### 📊 Résumé des fichiers uploadés

Vous devriez maintenant avoir **5 fichiers** avec différentes classes :

| Fichier | Classe | Icône Console | Coût/mois (1 GB) |
|---------|--------|---------------|------------------|
| fichier-standard.txt | Standard | 🟢 | $0.023 |
| fichier-intelligent.txt | Intelligent-Tiering | 🔵 | $0.023-$0.0125 |
| fichier-standard-ia.txt | Standard-IA | 🟡 | $0.0125 |
| fichier-glacier.txt | Glacier Instant Retrieval | 🟠 | $0.004 |
| fichier-deep-archive.txt | Glacier Deep Archive | 🔴 | $0.00099 |

---

## 🔄 PARTIE 3 : Modifier la classe d'un objet existant

### Étape 3.1 : Changer Standard → Standard-IA

1. **Sélectionner l'objet**
   - Console S3 → Votre bucket
   - ✅ **Cochez** `fichier-standard.txt`

2. **Modifier la classe**
   - Menu **Actions** (en haut à droite)
   - Sélectionnez **Edit storage class**

3. **Choisir la nouvelle classe**
   - **New storage class** : `Standard-IA`
   - 📖 Note :
     > "Cette opération est gratuite, mais l'objet sera facturé au minimum 30 jours en Standard-IA même si vous le supprimez avant"

4. **Confirmer**
   - Cliquez sur **Save changes**
   - **Vérification** : Propriétés → Storage class = `Standard-IA`

---

### Étape 3.2 : Changer Standard-IA → Glacier Flexible Retrieval

1. **Sélectionner** `fichier-standard-ia.txt`
2. **Actions** → **Edit storage class**
3. **New storage class** : `Glacier Flexible Retrieval`
4. ⚠️ **Avertissement console** :
   > "Objects transitioned to Glacier cannot be accessed immediately. Retrieval time: 1-5 minutes (Expedited), 3-5 hours (Standard), 5-12 hours (Bulk)"

5. **Save changes**

---

### Étape 3.3 : Restaurer un objet depuis Glacier

Si vous avez déplacé un fichier vers Glacier Flexible Retrieval, vous devez le **restaurer** avant de pouvoir le télécharger :

1. **Sélectionner** l'objet en Glacier
2. **Actions** → **Initiate restore**
3. **Restore options** :
   - **Tier** : 
     - `Bulk` (5-12h, $0.0025/GB) ← Le moins cher
     - `Standard` (3-5h, $0.01/GB)
     - `Expedited` (1-5 min, $0.03/GB) ← Le plus rapide
   - **Days available** : Nombre de jours où la copie restaurée sera disponible (ex: 7 jours)

4. **Confirmer** : **Initiate restore**
5. **Statut** : L'objet affichera "Restoration in progress"
6. Une fois restauré, vous pourrez le télécharger pendant la durée spécifiée

---

## ⏰ PARTIE 4 : Configurer des règles de transition automatique (Lifecycle)

### Scénario : Transition automatique vers des classes moins coûteuses

**Objectif** : Automatiser la gestion du cycle de vie des objets pour optimiser les coûts.

**Règle à créer** :
- Objets non accédés depuis **30 jours** → Transition vers `Standard-IA`
- Objets non accédés depuis **90 jours** → Transition vers `Glacier Instant Retrieval`
- Objets non accédés depuis **180 jours** → Transition vers `Glacier Flexible Retrieval`
- Suppression après **365 jours**

---

### Étape 4.1 : Créer une règle de cycle de vie

1. **Accéder à la configuration Lifecycle**
   - Console S3 → Votre bucket
   - Onglet **Management**
   - Section **Lifecycle rules**
   - Cliquez sur **Create lifecycle rule**

2. **Nom de la règle**
   - **Lifecycle rule name** : `Optimize-Storage-Costs`
   - **Description** (optionnel) : `Transition automatique vers classes moins coûteuses`

3. **Scope (Portée de la règle)**
   - **Apply to all objects in the bucket** ✅ Cochez cette option
   - Ou **Limit the scope using filters** : Appliquer uniquement à un préfixe (ex: `backups/`)

4. **Acknowledge** : Cochez la case de confirmation

---

### Étape 4.2 : Configurer les transitions

1. **Lifecycle rule actions**
   - ✅ **Transition current versions of objects between storage classes**
   - ✅ **Expire current versions of objects** (suppression automatique)

2. **Transition current versions of objects**
   - Cliquez sur **Add transition**

   **Transition 1** :
   - **Days after object creation** : `30`
   - **Storage class** : `Standard-IA`
   - Cliquez sur **Add transition** pour ajouter une autre

   **Transition 2** :
   - **Days after object creation** : `90`
   - **Storage class** : `Glacier Instant Retrieval`
   - **Add transition**

   **Transition 3** :
   - **Days after object creation** : `180`
   - **Storage class** : `Glacier Flexible Retrieval`

3. **Expire current versions of objects**
   - **Days after object creation** : `365`
   - ⚠️ Cela supprimera définitivement les objets après 1 an

---

### Étape 4.3 : Timeline de la règle (Visualisation)

La console affiche un **diagramme de timeline** :

```
Jour 0        Jour 30         Jour 90              Jour 180                Jour 365
│             │               │                    │                       │
│ Standard    │ Standard-IA   │ Glacier Instant    │ Glacier Flexible      │ Suppression
│             │               │ Retrieval          │ Retrieval             │
└─────────────┴───────────────┴────────────────────┴───────────────────────┴──────────→
```

---

### Étape 4.4 : Créer la règle

1. **Review** : Vérifiez tous les paramètres
2. **Create rule**
3. **Vérification** : Onglet **Management** → **Lifecycle rules** → Statut = **Enabled**

---

### Étape 4.5 : Tester la règle (Simulation)

⚠️ **Important** : Les règles de lifecycle s'exécutent à **minuit UTC** le lendemain de leur création.

Pour tester immédiatement :

1. **Console** : Vous ne pouvez pas forcer l'exécution immédiate
2. **CloudShell / AWS CLI** : Utilisez cette commande pour simuler

   ```bash
   # Modifier manuellement la date de création (simulation)
   aws s3api copy-object \
     --bucket m2i-storage-classes-anselme-bucket \
     --copy-source m2i-storage-classes-anselme-bucket/fichier-standard.txt \
     --key fichier-standard.txt \
     --storage-class STANDARD_IA \
     --metadata-directive COPY
   ```

3. **Vérification** : Propriétés de l'objet → Storage class devrait être `Standard-IA`

---

## 🎯 PARTIE 5 : Cas d'usage pratiques et exercices

### Exercice 1 : Optimiser les coûts d'un bucket de logs

**Scénario** : Vous avez un bucket qui stocke des logs d'application (10 GB/jour).

**Besoins** :
- Accès fréquent pendant **7 jours**
- Accès occasionnel entre **7-30 jours**
- Archive après **30 jours** (accès rare)
- Suppression après **90 jours**

**À faire** :
1. Créer un bucket `m2i-logs-<prenom>-bucket`
2. Créer une règle de lifecycle avec ces transitions :
   - 0-7 jours : `Standard`
   - 7-30 jours : `Standard-IA`
   - 30-90 jours : `Glacier Instant Retrieval`
   - > 90 jours : Suppression

**Configuration Lifecycle** :

1. **Lifecycle rule name** : `Logs-Retention-Policy`
2. **Scope** : `Apply to all objects`
3. **Transitions** :
   - Transition 1 : 7 jours → `Standard-IA`
   - Transition 2 : 30 jours → `Glacier Instant Retrieval`
4. **Expiration** : 90 jours
5. **Create rule**

**💰 Économies estimées** (pour 300 GB/mois) :

| Classe | Coût/mois |
|--------|-----------|
| **Sans lifecycle (tout en Standard)** | $6.90 |
| **Avec lifecycle** | $2.15 |
| **Économie** | **68% (~$4.75/mois)** |

---

### Exercice 2 : Archiver des backups anciens

**Scénario** : Backups de base de données (backup complet quotidien de 50 GB).

**Besoins** :
- Backups récents (< 30 jours) : Accès rapide → `Standard`
- Backups mensuels (30-365 jours) : → `Standard-IA`
- Backups annuels (> 1 an) : → `Glacier Deep Archive` (conservation 7 ans)

**À faire** :
1. Créer un bucket `m2i-backups-<prenom>-bucket`
2. Créer un préfixe pour les backups quotidiens : `daily/`
3. Créer une règle de lifecycle :

**Configuration** :

```
Lifecycle rule name: Daily-Backup-Archive
Scope: Prefix = "daily/"

Transitions:
  - 30 jours → Standard-IA
  - 365 jours → Glacier Deep Archive

Expiration:
  - 2555 jours (7 ans)
```

**💰 Coût annuel estimé** (50 GB/jour × 365 jours = 18.25 TB/an) :

| Classe | Coût annuel |
|--------|-------------|
| **Tout en Standard** | $5,037 |
| **Avec lifecycle optimisé** | $1,245 |
| **Économie** | **$3,792 (75%)** |

---

### Exercice 3 : Data Lake avec Intelligent-Tiering

**Scénario** : Data lake avec des données d'analytics (pattern d'accès imprévisible).

**Besoins** :
- Certains fichiers sont accédés fréquemment pendant des semaines, puis jamais
- Impossible de prédire quels fichiers seront accédés
- Optimisation automatique des coûts

**Solution** : **S3 Intelligent-Tiering**

**À faire** :
1. Créer un bucket `m2i-datalake-<prenom>-bucket`
2. Configurer **Intelligent-Tiering** par défaut
3. Activer les **Archive tiers** (optionnel)

**Configuration Intelligent-Tiering avec Archive** :

1. **Console S3** → Votre bucket → **Management**
2. **Intelligent-Tiering Archive configurations** → **Create configuration**
3. **Configuration name** : `Auto-Archive-Old-Data`
4. **Scope** : All objects ou prefix `raw-data/`
5. **Archive access tiers** :
   - ✅ **Archive Access tier** : Déplacer vers archive après **90 jours** sans accès
   - ✅ **Deep Archive Access tier** : Déplacer vers deep archive après **180 jours** sans accès
6. **Create**

**Fonctionnement** :
- Fréquent Access (< 30 jours) : Coût Standard
- Infrequent Access (30-90 jours) : Coût Standard-IA automatiquement
- Archive (90-180 jours) : Coût Glacier Instant (si activé)
- Deep Archive (> 180 jours) : Coût Glacier Deep Archive (si activé)

**💡 Avantage** : Pas de frais de récupération, contrairement aux classes Glacier classiques.

---

## 📊 PARTIE 6 : Monitoring et analyse des coûts

### Étape 6.1 : Voir les métriques de stockage

1. **Console S3** → Votre bucket → **Metrics**
2. **Storage** : Voir la répartition par classe de stockage
   - Graphique : **Storage bytes** (en GB) par classe
   - Graphique : **Number of objects** par classe

3. **Request metrics** (si activés) :
   - GET requests
   - PUT requests
   - Latence moyenne

---

### Étape 6.2 : Utiliser S3 Storage Lens

**S3 Storage Lens** = Dashboard d'analyse avancée des coûts et usages S3.

1. **Console S3** → **Storage Lens** (menu gauche)
2. **View default dashboard**
3. **Insights** :
   - Répartition du stockage par classe
   - Objets sans lifecycle configuré
   - Buckets avec accès public
   - Recommandations d'optimisation

4. **Recommandations** :
   - "80% de vos objets en Standard n'ont pas été accédés depuis 90 jours → Suggéré : Transition vers Standard-IA"
   - "Bucket X : 500 GB en Glacier Instant non accédés depuis 1 an → Suggéré : Deep Archive"

---

### Étape 6.3 : Simuler les coûts avec AWS Pricing Calculator

1. Accédez à https://calculator.aws/
2. **Add service** → **Amazon S3**
3. **Configuration** :
   - **Region** : US East (N. Virginia)
   - **S3 Standard** : 100 GB
   - **S3 Standard-IA** : 200 GB
   - **S3 Glacier Instant** : 500 GB
   - **Requests** : 10,000 GET, 5,000 PUT
4. **Estimate** : Voir le coût mensuel total

**Exemple de résultat** :
```
S3 Standard (100 GB):          $2.30
S3 Standard-IA (200 GB):       $2.50
S3 Glacier Instant (500 GB):   $2.00
Requests:                      $0.10
Total/mois:                    $6.90
```

---

## 🧹 PARTIE 7 : Nettoyage

### Supprimer tous les objets et le bucket

**Via la Console** :

1. **Console S3** → Votre bucket
2. **Vider le bucket** :
   - Onglet **Objects**
   - Cliquez sur **Empty** (bouton orange en haut à droite)
   - Tapez `permanently delete` pour confirmer
   - Cliquez sur **Empty**

3. **Supprimer le bucket** :
   - Retour à la liste des buckets
   - Sélectionnez votre bucket
   - Cliquez sur **Delete**
   - Tapez le nom du bucket pour confirmer
   - Cliquez sur **Delete bucket**

4. **Supprimer les règles lifecycle** (si vous gardez le bucket) :
   - Onglet **Management** → **Lifecycle rules**
   - Sélectionnez la règle
   - **Delete**

---

## 📚 Ressources et documentation

### Documentation AWS officielle

1. **Amazon S3 Storage Classes**
   - https://aws.amazon.com/s3/storage-classes/
   - Guide complet sur toutes les classes

2. **S3 Lifecycle Configuration**
   - https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lifecycle-mgmt.html
   - Configuration détaillée du cycle de vie

3. **S3 Intelligent-Tiering**
   - https://aws.amazon.com/s3/storage-classes/intelligent-tiering/
   - Optimisation automatique des coûts

4. **S3 Glacier**
   - https://aws.amazon.com/s3/storage-classes/glacier/
   - Archive long terme

5. **AWS Pricing Calculator**
   - https://calculator.aws/
   - Estimation des coûts

---

## 🎯 Points clés à retenir

✅ **8 classes de stockage** adaptées à différents besoins  
✅ **Standard** : Données fréquemment accédées (coût élevé)  
✅ **Intelligent-Tiering** : Pattern d'accès imprévisible (optimisation auto)  
✅ **Standard-IA / One Zone-IA** : Accès peu fréquent (coût moyen)  
✅ **Glacier** : Archive (coût faible, récupération variable)  
✅ **Lifecycle policies** : Automatiser les transitions (économies jusqu'à 75%)  
✅ **Restauration** : Nécessaire pour accéder aux objets en Glacier  
✅ **Durées minimales** : 30 jours (IA), 90 jours (Glacier Instant), 180 jours (Deep Archive)  

---

## 💡 Bonnes pratiques

1. **Analyser les patterns d'accès** avant de choisir une classe
2. **Utiliser Intelligent-Tiering** si le pattern est imprévisible
3. **Toujours configurer des lifecycle policies** pour une gestion automatique
4. **Attention aux durées minimales** : Supprimer un objet avant = Facturation complète
5. **Tester les restaurations Glacier** avant la mise en production
6. **Monitorer avec S3 Storage Lens** pour identifier les optimisations
7. **Utiliser One Zone-IA** uniquement pour données reproductibles (risque AZ)
8. **Préférer Glacier Instant** si besoin d'accès rapide occasionnel (vs Flexible)

---

## 🎓 Quiz de validation

**Question 1** : Quelle classe choisir pour un site web actif avec 1000 requêtes/seconde ?
<details>
<summary>Réponse</summary>
✅ **S3 Express One Zone** (latence µs-ms, haute performance) ou **S3 Standard** (latence ms, multi-AZ)
</details>

**Question 2** : Vous avez 1 TB de logs non accédés depuis 2 ans à archiver pour 10 ans. Quelle classe ?
<details>
<summary>Réponse</summary>
✅ **S3 Glacier Deep Archive** ($0.00099/GB/mois = $1/TB/mois)
</details>

**Question 3** : Données d'analytics avec accès imprévisible. Quelle classe ?
<details>
<summary>Réponse</summary>
✅ **S3 Intelligent-Tiering** (optimisation automatique sans frais de récupération)
</details>

**Question 4** : Backups accédés 1 fois/mois. Standard-IA ou Glacier Instant ?
<details>
<summary>Réponse</summary>
✅ **Standard-IA** (accès mensuel = fréquent, Glacier Instant pour accès trimestriels)
</details>

**Question 5** : Vous supprimez un objet en Standard-IA après 15 jours. Facturation ?
<details>
<summary>Réponse</summary>
✅ **30 jours** (durée minimale de facturation pour Standard-IA, même si supprimé avant)
</details>

---

**🎉 Félicitations !** Vous maîtrisez maintenant les classes de stockage S3 et leur configuration via la console ! 🚀

---

## 💰 Estimation des économies (exemple réel)

**Scénario** : Entreprise avec 10 TB de données S3
- 2 TB : Données actives (< 30 jours)
- 3 TB : Données occasionnelles (30-90 jours)
- 5 TB : Archives (> 90 jours)

| Configuration | Coût mensuel | Économie |
|---------------|--------------|----------|
| **Tout en Standard** | $230 | - |
| **Avec Lifecycle optimisé** | $65 | **$165/mois (72%)** |

**Économie annuelle** : **$1,980** 💸

---

## 📞 Support

Questions ou problèmes ?
- Consultez la [documentation S3](https://docs.aws.amazon.com/s3/)
- AWS Support (si vous avez un plan support)
- Forums AWS : https://forums.aws.amazon.com/

Bon lab ! 🚀
