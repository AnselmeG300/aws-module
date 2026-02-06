# LAB 08 — Politiques S3 et Transferts de Fichiers entre Buckets

## 🎯 Objectifs

À la fin de ce lab, vous serez capable de :
- ✅ Comprendre les **politiques de bucket S3** (Bucket Policies)
- ✅ Configurer les **politiques IAM** pour accéder à S3
- ✅ **Copier des fichiers** d'un bucket S3 à un autre
- ✅ Mettre en place la **réplication S3** (même région et cross-region)
- ✅ Gérer les **permissions cross-account** (accès entre comptes AWS)
- ✅ Automatiser les transferts avec **AWS CLI** et **scripts**
- ✅ Configurer les **lifecycle policies** pour gérer le cycle de vie des objets

---

## 📋 Prérequis

- ✅ Accès à la console AWS
- ✅ Région : **N. Virginia (us-east-1)** et **Ohio (us-east-2)** pour cross-region
- ✅ Accès à **CloudShell** (AWS CLI déjà configuré)
- ✅ Permissions IAM suffisantes (S3FullAccess pour ce lab)
- ✅ Navigateur web (pour la console + CloudShell)

---

## 📚 Concepts clés

### 🔐 Types de politiques S3

| Type | Niveau | Cas d'usage |
|------|--------|-------------|
| **Bucket Policy** | Bucket | Contrôler l'accès au bucket entier ou à des préfixes |
| **IAM Policy** | Utilisateur/Rôle | Définir ce qu'un utilisateur peut faire sur S3 |
| **ACL (Access Control List)** | Objet/Bucket | Legacy, éviter si possible |
| **S3 Block Public Access** | Compte/Bucket | Bloquer tout accès public |

### 🔄 Méthodes de copie entre buckets

| Méthode | Cas d'usage | Temps réel | Historique |
|---------|-------------|------------|------------|
| **AWS CLI (`aws s3 cp`)** | Copie manuelle/scriptée | Non | Non |
| **S3 Batch Operations** | Copie massive (millions d'objets) | Non | Oui |
| **S3 Replication** | Copie automatique continue | Oui | Optionnel |
| **DataSync** | Migration massive avec validation | Non | Non |
| **SDK (Boto3, etc.)** | Automatisation custom | Selon code | Non |

---

## 🏗️ Architecture du lab

```
┌─────────────────────────────────────────────────────────────┐
│                        Compte AWS                           │
│                                                             │
│  Region: us-east-1 (N. Virginia)                            │
│  ┌──────────────────────────┐                               │
│  │  Bucket Source           │                               │
│  │  m2i-source-[nom]-bucket │                               │
│  │                          │                               │
│  │  📄 fichier1.txt         │                               │
│  │  📄 fichier2.json        │                               │
│  │  📁 dossier/             │                               │
│  │     📄 fichier3.csv      │                               │
│  └──────────┬───────────────┘                               │
│             │                                                │
│             │ Copie manuelle (AWS CLI)                       │
│             │ ou Réplication automatique                     │
│             ▼                                                │
│  ┌──────────────────────────┐                               │
│  │  Bucket Destination 1    │                               │
│  │  m2i-dest1-[nom]-bucket  │ (même région)                 │
│  │                          │                               │
│  │  📄 fichier1.txt (copie) │                               │
│  └──────────────────────────┘                               │
│                                                             │
│  Region: us-east-2 (Ohio)                                   │
│  ┌──────────────────────────┐                               │
│  │  Bucket Destination 2    │                               │
│  │  m2i-dest2-[nom]-bucket  │ (cross-region)                │
│  │                          │                               │
│  │  📄 fichier1.txt (copie) │                               │
│  └──────────────────────────┘                               │
└─────────────────────────────────────────────────────────────┘
```

---

## 🚀 PARTIE 1 : Créer les buckets et uploader des fichiers

### Étape 1.1 : Créer les buckets S3

#### Via la console AWS

1. **Console AWS → S3 → Create bucket**

2. **Bucket source (us-east-1)** :
   - **Bucket name** : `m2i-source-[votre-prenom]-bucket`
   - Exemple : `m2i-source-anselme-bucket`
   - **Region** : `US East (N. Virginia) us-east-1`
   - **Block Public Access** : ✅ Laissez tout coché (sécurité)
   - **Versioning** : ✅ Enable (important pour la réplication)
   - **Default encryption** : Server-side encryption with Amazon S3 managed keys (SSE-S3)
   - **Cliquez sur "Create bucket"**

3. **Bucket destination 1 (même région)** :
   - **Bucket name** : `m2i-dest1-[votre-prenom]-bucket`
   - **Region** : `US East (N. Virginia) us-east-1`
   - **Versioning** : ✅ Enable
   - **Create bucket**

4. **Bucket destination 2 (cross-region)** :
   - **Bucket name** : `m2i-dest2-[votre-prenom]-bucket`
   - **Region** : `US East (Ohio) us-east-2` ← Différent !
   - **Versioning** : ✅ Enable
   - **Create bucket**

#### Via CloudShell (AWS CLI)

```bash
# Variables
prenom="anselme"  # ← Changez avec votre prenom
source_bucket="m2i-source-$prenom-bucket"
dest1_bucket="m2i-dest1-$prenom-bucket"
dest2_bucket="m2i-dest2-$prenom-bucket"

# Creer le bucket source (us-east-1)
aws s3 mb "s3://$source_bucket" --region us-east-1

# Activer le versioning
aws s3api put-bucket-versioning \
  --bucket "$source_bucket" \
  --versioning-configuration Status=Enabled

# Creer bucket destination 1 (meme region)
aws s3 mb "s3://$dest1_bucket" --region us-east-1
aws s3api put-bucket-versioning \
  --bucket "$dest1_bucket" \
  --versioning-configuration Status=Enabled

# Creer bucket destination 2 (cross-region)
aws s3 mb "s3://$dest2_bucket" --region us-east-2
aws s3api put-bucket-versioning \
  --bucket "$dest2_bucket" \
  --versioning-configuration Status=Enabled

echo "Buckets crees avec succes."
```

---

### Étape 1.2 : Créer des fichiers de test

```bash
# Creer un dossier de travail dans CloudShell
mkdir -p ~/s3-lab/dossier
cd ~/s3-lab

# Creer fichier1.txt
cat <<EOF > fichier1.txt
Ceci est un fichier de test pour le LAB S3.
Date de creation: $(date)
Auteur: M2i Formation
EOF

# Creer fichier2.json
cat <<EOF > fichier2.json
{
  "lab": "LAB-08-S3",
  "objectif": "Politiques et transferts S3",
  "date": "$(date +%F)",
  "region": "us-east-1"
}
EOF

# Creer un sous-dossier avec fichier3.csv
cat <<EOF > dossier/fichier3.csv
Nom,Prenom,Age,Ville
Dupont,Jean,30,Paris
Martin,Marie,25,Lyon
Bernard,Luc,35,Marseille
EOF

# Creer fichier4.log (gros fichier)
for i in $(seq 1 1000); do
  echo "[$i] Log entry at $(date +%H:%M:%S) - Random data: $(uuidgen)"
done > fichier4.log

echo "Fichiers de test crees dans ~/s3-lab"
ls -R
```

---

### Étape 1.3 : Uploader les fichiers dans le bucket source

```bash
# Variables
source_bucket="m2i-source-anselme-bucket"  # ← Votre bucket

# Uploader tous les fichiers
aws s3 sync . "s3://$source_bucket/"

# Verifier l'upload
echo "\nContenu du bucket source :"
aws s3 ls "s3://$source_bucket/" --recursive --human-readable

# Resultat attendu :
# 2026-02-05 10:30:00   150 Bytes fichier1.txt
# 2026-02-05 10:30:00   180 Bytes fichier2.json
# 2026-02-05 10:30:00   120 Bytes dossier/fichier3.csv
# 2026-02-05 10:30:00    50 KB fichier4.log
```

---

## 🔐 PARTIE 2 : Configurer les politiques de bucket

### Étape 2.1 : Créer une politique de bucket (Bucket Policy)

#### Politique 1 : Permettre la lecture depuis le bucket destination

**Scénario** : Autoriser le bucket destination à lire les objets du bucket source.

```bash
# Creer le fichier de politique
source_bucket="m2i-source-anselme-bucket"  # ← Votre bucket

cat <<EOF > bucket-policy.json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowDestinationBucketRead",
      "Effect": "Allow",
      "Principal": {
        "Service": "s3.amazonaws.com"
      },
      "Action": [
        "s3:GetObject",
        "s3:GetObjectVersion"
      ],
      "Resource": "arn:aws:s3:::$source_bucket/*",
      "Condition": {
        "StringEquals": {
          "s3:x-amz-server-side-encryption": "AES256"
        }
      }
    },
    {
      "Sid": "AllowListBucket",
      "Effect": "Allow",
      "Principal": {
        "Service": "s3.amazonaws.com"
      },
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::$source_bucket"
    }
  ]
}
EOF

# Appliquer la politique au bucket source
aws s3api put-bucket-policy \
  --bucket "$source_bucket" \
  --policy file://bucket-policy.json

echo "Politique de bucket appliquee."
```

**💡 Explication de la politique** :

| Élément | Valeur | Explication |
|---------|--------|-------------|
| **Principal** | `"Service": "s3.amazonaws.com"` | Service S3 autorisé |
| **Action** | `s3:GetObject`, `s3:ListBucket` | Lire les objets et lister le bucket |
| **Resource** | `arn:aws:s3:::bucket/*` | Tous les objets du bucket |
| **Condition** | Chiffrement AES256 | Seulement si objets chiffrés |

---

#### Politique 2 : Permettre l'écriture dans le bucket destination

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowReplicationWrite",
      "Effect": "Allow",
      "Principal": {
        "Service": "s3.amazonaws.com"
      },
      "Action": [
        "s3:ReplicateObject",
        "s3:ReplicateDelete",
        "s3:ReplicateTags"
      ],
      "Resource": "arn:aws:s3:::m2i-dest1-anselme-bucket/*"
    }
  ]
}
```

```bash
# Appliquer au bucket destination
aws s3api put-bucket-policy \
  --bucket m2i-dest1-anselme-bucket \
  --policy file://dest-bucket-policy.json
```

---

### Étape 2.2 : Créer une politique IAM pour l'utilisateur

**Scénario** : Permettre à votre utilisateur IAM de copier des fichiers entre buckets.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3ListAllBuckets",
      "Effect": "Allow",
      "Action": "s3:ListAllMyBuckets",
      "Resource": "*"
    },
    {
      "Sid": "AllowReadSourceBucket",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:GetObjectVersion",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::m2i-source-anselme-bucket",
        "arn:aws:s3:::m2i-source-anselme-bucket/*"
      ]
    },
    {
      "Sid": "AllowWriteDestBuckets",
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:PutObjectAcl",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::m2i-dest1-anselme-bucket",
        "arn:aws:s3:::m2i-dest1-anselme-bucket/*",
        "arn:aws:s3:::m2i-dest2-anselme-bucket",
        "arn:aws:s3:::m2i-dest2-anselme-bucket/*"
      ]
    }
  ]
}
```

**Appliquer la politique** :

```bash
# Via console : IAM → Policies → Create policy → JSON → Copier/coller
# Nom : S3-Copy-Between-Buckets-Policy

# Ou via CLI :
aws iam create-policy \
  --policy-name S3-Copy-Between-Buckets-Policy \
  --policy-document file://iam-policy.json

# Attacher à votre utilisateur
aws iam attach-user-policy \
  --user-name benoit \
  --policy-arn arn:aws:iam::123456789012:policy/S3-Copy-Between-Buckets-Policy
```

---

## 📋 PARTIE 3 : Copier des fichiers entre buckets

### Méthode 1 : Copie manuelle avec AWS CLI

#### Copier un fichier unique

```bash
# Copier fichier1.txt du source vers destination 1
aws s3 cp \
  s3://m2i-source-anselme-bucket/fichier1.txt \
  s3://m2i-dest1-anselme-bucket/fichier1.txt

# Vérifier la copie
aws s3 ls s3://m2i-dest1-anselme-bucket/

# ✅ Résultat : 2026-02-05 10:35:00   150 Bytes fichier1.txt
```

#### Copier un dossier entier

```bash
# Copier le dossier "dossier/" avec tous ses fichiers
aws s3 cp \
  s3://m2i-source-anselme-bucket/dossier/ \
  s3://m2i-dest1-anselme-bucket/dossier/ \
  --recursive

# Vérifier
aws s3 ls s3://m2i-dest1-anselme-bucket/dossier/ --recursive
```

#### Synchroniser deux buckets (sync)

```bash
# Synchroniser tous les fichiers (copie uniquement ce qui a changé)
aws s3 sync \
  s3://m2i-source-anselme-bucket/ \
  s3://m2i-dest1-anselme-bucket/

# Avec filtres
aws s3 sync \
  s3://m2i-source-anselme-bucket/ \
  s3://m2i-dest1-anselme-bucket/ \
  --exclude "*" \
  --include "*.txt" \
  --include "*.json"

# ✅ Ne copie que les fichiers .txt et .json
```

#### Copie cross-region

```bash
# Copier vers bucket dans une autre région (us-east-2)
aws s3 sync \
  s3://m2i-source-anselme-bucket/ \
  s3://m2i-dest2-anselme-bucket/ \
  --region us-east-2

echo "Copie cross-region terminee."
```

---

### Methode 2 : Script bash automatise (CloudShell)

```bash
# Creer un script de copie avec rapport
cat <<'EOF' > copy_s3_buckets.sh
#!/usr/bin/env bash
set -euo pipefail

SOURCE_BUCKET="${1:-m2i-source-anselme-bucket}"
DEST_BUCKET="${2:-m2i-dest1-anselme-bucket}"
PREFIX="${3:-}"
DRY_RUN="${4:-false}"

echo "Demarrage de la copie S3"
echo "Source: s3://$SOURCE_BUCKET/$PREFIX"
echo "Destination: s3://$DEST_BUCKET/$PREFIX"

objects_json=$(aws s3api list-objects-v2 --bucket "$SOURCE_BUCKET" --prefix "$PREFIX" --output json)
total_objects=$(echo "$objects_json" | python -c 'import json,sys;print(len(json.load(sys.stdin).get("Contents",[])))')

if [ "$total_objects" -eq 0 ]; then
  echo "Aucun objet trouve dans le bucket source"
  exit 1
fi

copied=0
errors=0

echo "$objects_json" | python - <<'PY'
import json,sys
data=json.load(sys.stdin)
for obj in data.get("Contents",[]):
    print(obj["Key"])
PY
| while read -r key; do
  if [ "$DRY_RUN" = "true" ]; then
    echo "[DRY RUN] Copie: $key"
  else
    if aws s3 cp "s3://$SOURCE_BUCKET/$key" "s3://$DEST_BUCKET/$key" --quiet; then
      copied=$((copied+1))
      echo "OK: $key"
    else
      errors=$((errors+1))
      echo "Erreur copie: $key"
    fi
  fi
done

echo "Rapport: copies=$copied erreurs=$errors"

dest_count=$(aws s3api list-objects-v2 --bucket "$DEST_BUCKET" --prefix "$PREFIX" --query 'length(Contents)' --output text)
if [ "$dest_count" = "$total_objects" ]; then
  echo "Verification OK: $dest_count/$total_objects"
else
  echo "Attention: $dest_count/$total_objects"
fi
EOF

chmod +x copy_s3_buckets.sh
```

**Utilisation** :

```bash
# Copie normale
./copy_s3_buckets.sh "m2i-source-anselme-bucket" "m2i-dest1-anselme-bucket"

# Copie avec prefixe (seulement le dossier "dossier/")
./copy_s3_buckets.sh "m2i-source-anselme-bucket" "m2i-dest1-anselme-bucket" "dossier/"

# Mode dry-run (simulation)
./copy_s3_buckets.sh "m2i-source-anselme-bucket" "m2i-dest1-anselme-bucket" "" true
```

---

## 🔄 PARTIE 4 : Configurer la réplication S3 automatique

### Étape 4.1 : Créer un rôle IAM pour la réplication

```bash
# Politique de confiance (Trust Policy)
cat <<'EOF' > trust-policy.json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "s3.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF

# Creer le role
aws iam create-role \
  --role-name S3-Replication-Role \
  --assume-role-policy-document file://trust-policy.json

# Politique de permissions
cat <<EOF > replication-policy.json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetReplicationConfiguration",
        "s3:ListBucket"
      ],
      "Resource": "arn:aws:s3:::m2i-source-anselme-bucket"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObjectVersionForReplication",
        "s3:GetObjectVersionAcl",
        "s3:GetObjectVersionTagging"
      ],
      "Resource": "arn:aws:s3:::m2i-source-anselme-bucket/*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:ReplicateObject",
        "s3:ReplicateDelete",
        "s3:ReplicateTags"
      ],
      "Resource": [
        "arn:aws:s3:::m2i-dest1-anselme-bucket/*",
        "arn:aws:s3:::m2i-dest2-anselme-bucket/*"
      ]
    }
  ]
}
EOF

# Creer et attacher la politique
aws iam put-role-policy \
  --role-name S3-Replication-Role \
  --policy-name S3-Replication-Policy \
  --policy-document file://replication-policy.json

echo "Role IAM cree : S3-Replication-Role"
```

---

### Étape 4.2 : Configurer la réplication

#### Configuration de réplication (Same-Region Replication - SRR)

```bash
# Configuration de replication
account_id=$(aws sts get-caller-identity --query Account --output text)

cat <<EOF > replication-config.json
{
  "Role": "arn:aws:iam::${account_id}:role/S3-Replication-Role",
  "Rules": [
    {
      "ID": "ReplicateAll",
      "Status": "Enabled",
      "Priority": 1,
      "DeleteMarkerReplication": {
        "Status": "Enabled"
      },
      "Filter": {},
      "Destination": {
        "Bucket": "arn:aws:s3:::m2i-dest1-anselme-bucket",
        "ReplicationTime": {
          "Status": "Enabled",
          "Time": {
            "Minutes": 15
          }
        },
        "Metrics": {
          "Status": "Enabled",
          "EventThreshold": {
            "Minutes": 15
          }
        }
      }
    }
  ]
}
EOF

# Appliquer la configuration
aws s3api put-bucket-replication \
  --bucket m2i-source-anselme-bucket \
  --replication-configuration file://replication-config.json

echo "Replication configuree."
```

#### Configuration Cross-Region Replication (CRR)

```json
{
  "Role": "arn:aws:iam::123456789012:role/S3-Replication-Role",
  "Rules": [
    {
      "ID": "ReplicateCrossRegion",
      "Status": "Enabled",
      "Priority": 2,
      "Filter": {
        "Prefix": "cross-region/"
      },
      "Destination": {
        "Bucket": "arn:aws:s3:::m2i-dest2-anselme-bucket",
        "StorageClass": "STANDARD_IA"
      }
    }
  ]
}
```

```bash
# Appliquer CRR
aws s3api put-bucket-replication \
  --bucket m2i-source-anselme-bucket \
  --replication-configuration file://crr-config.json
```

---

### Étape 4.3 : Tester la réplication

```bash
# Uploader un nouveau fichier dans le bucket source
echo "Test replication" > test-replication.txt

aws s3 cp test-replication.txt s3://m2i-source-anselme-bucket/

# Attendre quelques secondes
sleep 10

# Verifier dans le bucket destination
if aws s3 ls s3://m2i-dest1-anselme-bucket/test-replication.txt >/dev/null 2>&1; then
  echo "Replication reussie."
else
  echo "Replication en cours (peut prendre jusqu'a 15 min)."
fi

# Verifier le statut de replication
aws s3api head-object \
  --bucket m2i-dest1-anselme-bucket \
  --key test-replication.txt \
  --query 'ReplicationStatus'
```

---

## 📦 PARTIE 5 : Gestion du cycle de vie (Lifecycle Policies)

### Étape 5.1 : Créer une politique de cycle de vie

**Scénario** : 
- Fichiers non accédés depuis 30 jours → Transition vers STANDARD_IA
- Fichiers non accédés depuis 90 jours → Transition vers GLACIER
- Supprimer après 365 jours

```json
{
  "Rules": [
    {
      "Id": "ArchiveOldFiles",
      "Status": "Enabled",
      "Filter": {},
      "Transitions": [
        {
          "Days": 30,
          "StorageClass": "STANDARD_IA"
        },
        {
          "Days": 90,
          "StorageClass": "GLACIER"
        }
      ],
      "Expiration": {
        "Days": 365
      },
      "NoncurrentVersionTransitions": [
        {
          "NoncurrentDays": 30,
          "StorageClass": "GLACIER"
        }
      ],
      "NoncurrentVersionExpiration": {
        "NoncurrentDays": 90
      }
    },
    {
      "Id": "DeleteTempFiles",
      "Status": "Enabled",
      "Filter": {
        "Prefix": "temp/"
      },
      "Expiration": {
        "Days": 7
      }
    }
  ]
}
```

```bash
# Appliquer la politique de cycle de vie
aws s3api put-bucket-lifecycle-configuration \
  --bucket m2i-source-anselme-bucket \
  --lifecycle-configuration file://lifecycle-policy.json

echo "Politique de cycle de vie appliquee."

# Vérifier
aws s3api get-bucket-lifecycle-configuration \
  --bucket m2i-source-anselme-bucket
```

---

## 🌐 PARTIE 6 : Accès cross-account (Bonus)

### Scénario : Permettre à un autre compte AWS de lire votre bucket

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCrossAccountRead",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::999999999999:root"
      },
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::m2i-source-anselme-bucket",
        "arn:aws:s3:::m2i-source-anselme-bucket/*"
      ]
    }
  ]
}
```

**💡 Note** : Remplacez `999999999999` par l'Account ID du compte destinataire.

---

## 📊 PARTIE 7 : Monitoring et logs

### Activer les logs d'accès S3

```bash
# Créer un bucket pour les logs
aws s3 mb s3://m2i-logs-anselme-bucket --region us-east-1

# Configurer les logs d'accès
cat <<'EOF' > logging-config.json
{
  "LoggingEnabled": {
    "TargetBucket": "m2i-logs-anselme-bucket",
    "TargetPrefix": "source-bucket-logs/"
  }
}
EOF

aws s3api put-bucket-logging \
  --bucket m2i-source-anselme-bucket \
  --bucket-logging-status file://logging-config.json

echo "Logs d'acces S3 actives."
```

### Analyser les logs avec CloudWatch Insights

```sql
-- Requête CloudWatch Logs Insights pour analyser les accès S3
fields @timestamp, operation, key, remoteip
| filter bucket = "m2i-source-anselme-bucket"
| filter operation = "REST.GET.OBJECT"
| stats count() by key
| sort count desc
| limit 20
```

---

## 🧹 PARTIE 8 : Nettoyage

### Supprimer tous les objets et buckets

```bash
# ⚠️ ATTENTION : Cette opération est irréversible !

buckets=(
  "m2i-source-anselme-bucket"
  "m2i-dest1-anselme-bucket"
  "m2i-dest2-anselme-bucket"
  "m2i-logs-anselme-bucket"
)

for bucket in "${buckets[@]}"; do
  echo "Suppression de $bucket..."

  # Desactiver le versioning
  aws s3api put-bucket-versioning \
    --bucket "$bucket" \
    --versioning-configuration Status=Suspended >/dev/null 2>&1 || true

  # Supprimer tous les objets (y compris les versions)
  aws s3 rm "s3://$bucket" --recursive --quiet || true

  # Supprimer toutes les versions et delete markers
  versions=$(aws s3api list-object-versions --bucket "$bucket" \
    --query '{Objects: Versions[].{Key:Key,VersionId:VersionId}}' --output json)
  if [ "$versions" != '{"Objects": []}' ]; then
    aws s3api delete-objects --bucket "$bucket" --delete "$versions" >/dev/null 2>&1 || true
  fi

  markers=$(aws s3api list-object-versions --bucket "$bucket" \
    --query '{Objects: DeleteMarkers[].{Key:Key,VersionId:VersionId}}' --output json)
  if [ "$markers" != '{"Objects": []}' ]; then
    aws s3api delete-objects --bucket "$bucket" --delete "$markers" >/dev/null 2>&1 || true
  fi

  # Supprimer le bucket
  aws s3 rb "s3://$bucket" --force || true

  echo "$bucket supprime"
done

# Supprimer le role IAM
aws iam delete-role-policy \
  --role-name S3-Replication-Role \
  --policy-name S3-Replication-Policy

aws iam delete-role --role-name S3-Replication-Role

echo "Nettoyage termine."
```

---

## 📚 Ressources et sources

### Documentation AWS officielle

1. **Amazon S3 User Guide**
   - Source: https://docs.aws.amazon.com/s3/
   - Guide complet sur Amazon S3

2. **S3 Bucket Policies**
   - Source: https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucket-policies.html
   - Documentation sur les politiques de bucket

3. **S3 Replication**
   - Source: https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication.html
   - Guide de réplication (SRR et CRR)

4. **S3 Lifecycle Configuration**
   - Source: https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lifecycle-mgmt.html
   - Configuration du cycle de vie des objets

5. **Cross-Account Access**
   - Source: https://docs.aws.amazon.com/AmazonS3/latest/userguide/example-walkthroughs-managing-access-example2.html
   - Accès entre comptes AWS

### Workshops et tutoriels AWS

1. **AWS Workshops - Storage**
   - Source: https://workshops.aws/categories/Storage
   - Catalogue des workshops sur le stockage

2. **AWS Skill Builder - S3 Courses**
   - Source: https://explore.skillbuilder.aws/learn/course/internal/view/elearning/53/introduction-to-amazon-simple-storage-service-s3
   - Cours gratuits sur S3

3. **AWS Samples - S3 Examples**
   - Source: https://github.com/aws-samples/amazon-s3-examples
   - Exemples de code pour S3

### Vidéos et présentations

- **AWS re:Invent - S3 Deep Dive Sessions**
  - Rechercher "S3 best practices" sur YouTube AWS Channel
  - Sessions techniques approfondies sur S3

### Outils recommandés

- **AWS CLI** : https://aws.amazon.com/cli/
- **S3 Browser** (GUI) : https://s3browser.com/
- **CloudBerry Explorer** : https://www.cloudberrylab.com/explorer/amazon-s3.aspx

---

## 🎯 Points clés à retenir

✅ **Bucket Policies** : Contrôle d'accès au niveau du bucket  
✅ **IAM Policies** : Contrôle d'accès au niveau de l'utilisateur/rôle  
✅ **S3 Replication** : Copie automatique (SRR et CRR)  
✅ **Lifecycle Policies** : Gestion automatique du cycle de vie  
✅ **Versioning** : Obligatoire pour la réplication  
✅ **Cross-Account** : Partage sécurisé entre comptes AWS  

---

## 💰 Estimation des coûts

| Ressource | Quantité | Coût estimé |
|-----------|----------|-------------|
| Stockage S3 (STANDARD) | 1 GB | ~$0.023/mois |
| Requêtes PUT/COPY | 1000 | ~$0.005 |
| Transfert cross-region | 1 GB | ~$0.02 |
| Réplication (RTC) | Si activée | ~$0.015/GB |
| **Total (lab complet)** | | **~$0.10** |

💡 **Note** : Coûts très faibles pour un lab, pensez à nettoyer après !

---

**🎉 Félicitations !** Vous maîtrisez maintenant les politiques S3 et les transferts de fichiers entre buckets ! 🚀
