Parfait 👍
Voici un **résumé clair, structuré et rapidement révisable** de **tout le Module 4 – Networking**, prêt à être **copié-collé tel quel** dans un fichier de révision.

---

# ✅ Module 4 – Networking : Résumé de révision rapide

## 1. Concepts fondamentaux du réseau dans AWS

Le réseau dans AWS repose sur le principe d’**isolation et de contrôle du trafic**. Les ressources ne communiquent que si des règles explicites l’autorisent, selon une architecture pensée pour la **sécurité**, la **performance** et la **scalabilité**.

---

## 2. Amazon Virtual Private Cloud (VPC)

Un **Amazon VPC** est un **réseau virtuel privé** que le client définit dans AWS.

Il permet de :

* Définir des plages d’adresses IP,
* Créer des sous-réseaux (subnets),
* Contrôler les flux réseau entrants et sortants,
* Isoler les ressources.

👉 Le VPC est la **base de toute architecture réseau AWS**.

---

## 3. Subnets (Sous-réseaux)

Un **subnet** est une subdivision d’un VPC.

* **Subnet public**

  * Connecté à internet via une Internet Gateway
  * Héberge des ressources accessibles depuis internet (ex : serveur web)

* **Subnet privé**

  * Pas d’accès direct à internet
  * Héberge des ressources sensibles (ex : bases de données)

---

## 4. Internet Gateway (IGW)

Une **Internet Gateway** permet de connecter un VPC à internet.

* Indispensable pour rendre un subnet **public**
* Sans IGW, aucune communication directe avec internet n’est possible

---

## 5. Virtual Private Gateway (VPG) et VPN

La **Virtual Private Gateway** permet de connecter un **réseau d’entreprise (on-premises)** à un VPC AWS via un **VPN**.

* Le VPN chiffre les données
* Le trafic passe par internet mais de façon sécurisée
* Utilisé pour des architectures **hybrides**

---

## 6. AWS Direct Connect

**AWS Direct Connect** fournit une **connexion privée et dédiée** entre un data center et AWS.

Avantages :

* Latence plus faible et plus stable
* Bande passante prévisible
* Meilleure performance que le VPN
* Idéal pour gros volumes de données ou applications critiques

---

## 7. Sécurité réseau : couches de protection

AWS utilise une **approche multicouche** de la sécurité réseau.

### a) Network ACLs (NACL)

* Pare-feu au **niveau du subnet**
* **Stateless** (entrée et sortie gérées séparément)
* Les règles sont évaluées dans un ordre numéroté
* NACL par défaut : tout autorisé
* NACL personnalisée : tout bloqué jusqu’à configuration

👉 Filtrage large et global du trafic

---

### b) Security Groups (SG)

* Pare-feu au **niveau de l’instance**
* **Stateful** (le retour du trafic est automatiquement autorisé)
* Par défaut :

  * Tout trafic entrant est bloqué
  * Tout trafic sortant est autorisé

👉 Filtrage précis, orienté application

---

## 8. Différence clé entre NACL et Security Group

| Élément | NACL                      | Security Group              |
| ------- | ------------------------- | --------------------------- |
| Niveau  | Subnet                    | Instance                    |
| État    | Stateless                 | Stateful                    |
| Règles  | Entrée et sortie séparées | Retour automatique autorisé |
| Usage   | Filtrage global           | Sécurité fine               |

👉 **Les deux se complètent** et doivent être utilisés ensemble.

---

## 9. Réseau global AWS

AWS fournit des services pour rendre les applications **accessibles mondialement** avec performance et fiabilité.

---

## 10. DNS (Domain Name System)

Le **DNS** permet de traduire un **nom de domaine** en **adresse IP**.

Exemple :
`www.exemple.com → 192.0.2.0`

Sans DNS, les utilisateurs devraient mémoriser des adresses IP.

---

## 11. Amazon Route 53

**Amazon Route 53** est le service DNS managé d’AWS.

Il permet de :

* Gérer des noms de domaine,
* Créer des enregistrements DNS,
* Diriger les utilisateurs vers des ressources AWS ou externes,
* Fournir une haute disponibilité.

---

## 12. Amazon CloudFront

**Amazon CloudFront** est un **CDN (Content Delivery Network)**.

* Met en cache le contenu dans des **edge locations** partout dans le monde
* Réduit la latence
* Améliore l’expérience utilisateur
* Allège la charge des serveurs d’origine

Souvent utilisé avec **Route 53**.

---

## 13. Architecture typique globale

1. Route 53 résout le nom de domaine
2. CloudFront distribue le contenu depuis le point le plus proche
3. Le trafic est dirigé vers un Load Balancer et des instances EC2
4. Les règles réseau sécurisent chaque couche

---

## 14. Points clés à retenir pour l’examen

* Le **VPC** est la base du réseau AWS
* Subnet public ≠ subnet privé
* **Internet Gateway** = accès internet
* **VPN** = connexion sécurisée via internet
* **Direct Connect** = connexion privée dédiée
* **NACL = stateless / subnet**
* **Security Group = stateful / instance**
* **Route 53 = DNS**
* **CloudFront = CDN**

