---
title: " Déploiement d'une application sur GKE avec Ingress et Cloud Load Balancer pour dev.cosinetum.com"
slug: deploiement-dune-application-sur-gke-avec-ingress-et-cloud-load-balancer-pour-devcosinetumcom

---

### **Plan détaillé**

Dans un environnement cloud en constante évolution, le déploiement d’applications scalables et sécurisées est un défi majeur pour les ingénieurs DevOps. Kubernetes, combiné aux services managés de Google Cloud Platform (GCP), offre une solution robuste pour orchestrer des applications tout en assurant haute disponibilité et flexibilité.

Dans cet article, nous partageons un retour d’expérience sur le déploiement d’une application sur **Google Kubernetes Engine (GKE)**, avec une exposition publique via **Kubernetes Ingress** et un **Cloud Load Balancer**. Pour garantir une accessibilité stable et sécurisée, nous avons utilisé :

* **Une adresse IP statique publique** pour éviter les changements d’IP,
    
* **Cloud Domains** pour la gestion du nom de domaine [`dev.cosinetum.com`](http://dev.cosinetum.com),
    
* **GKE Managed Certificate** pour une gestion automatisée du certificat SSL.
    

Cet article s’adresse principalement aux **ingénieurs DevOps** cherchant à industrialiser leurs déploiements sur GCP. Nous décrirons chaque étape, des choix techniques aux défis rencontrés, afin de fournir une méthodologie reproductible et des bonnes pratiques pour vos propres projets.

# Architecture Cible

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1739577669148/e761f725-ce2e-4620-9cee-ba4ad2e01c3c.png align="center")

* Titre: Architecture application sur GKE avec ingress, cloud load balancer et certificat manager gke.
    
* L’architecture ci-dessus illustre la mise en place d’un déploiement Kubernetes sur **Google Kubernetes Engine (GKE)**, avec une exposition sécurisée via un **External Application Load Balancer** et un **Ingress Kubernetes**. Les composants que nous avons utiliser pour mettre en place l’architecture sont les suivants:
    
* **Nom de domaine et résolution DNS**
    
    * Le domaine [`dev.cosinetum.com`](http://dev.cosinetum.com) est géré via **Cloud Domains** et pointe vers une **adresse IP statique publique** réservée sur Google Cloud Platform.
        
    * Cette IP statique garantit que l’accès à l’application reste stable, même en cas de modification des ressources sous-jacentes.
        
* **Gestion du trafic avec le Load Balancer**
    
    * Un **External Application Load Balancer** est provisionné automatiquement par le **Kubernetes Ingress Controller**.
        
    * Il reçoit les requêtes entrantes et redirige le trafic HTTP vers HTTPS pour assurer la sécurité des communications.
        
* **Routage du trafic au sein du cluster GKE**
    
    * Le **Ingress Kubernetes** est configuré pour gérer le routage des requêtes vers l’application déployée sur le cluster GKE.
        
    * Un **GKE Managed Certificate** est associé à l’Ingress pour gérer automatiquement le certificat SSL et sécuriser le domaine avec HTTPS.
        
* **Application déployée sur GKE**
    
    * L’application est hébergée sur GKE et exposée via un **Service Kubernetes** permettant la communication entre l’Ingress et le déploiement de l’application.
        

Cette architecture tire parti des services managés de Google Cloud pour **simplifier la gestion des certificats SSL**, **assurer une haute disponibilité** et **garantir une connexion sécurisée** aux utilisateurs finaux.

# **Configuration et mise en place**

## **Création d’une adresse IP statique publique sur GCP**

#### **Commande gcloud pour créer une adresse IP statique**

Exécute la commande suivante pour créer une **adresse IP statique globale**, qui sera utilisée par le Load Balancer :

```plaintext
gcloud compute addresses create mon-ip-statique \
    --global \
    --description="Static IP for External Load Balancer" \
    --network-tier=PREMIUM
```

Une fois l’adresse creer on peut la verifier en utilisant la commande suivante :

```plaintext
gcloud compute addresses describe mon-ip-statique --global
```

Cela va retourner une sortie qui sera semblable a :

```plaintext
address: 34.120.45.68
name: mon-ip-statique
networkTier: PREMIUM
```

Notre adresse ip statique etant creer nous devons l’associer au load balancer. Mais nous le ferons plus bas.

Note : On peut aussi reserver l’adresse ip statique publique en passant la console google cloud (interface graphique).

## **Enregistrement du domaine sur Cloud Domains**

L’enregistrement du domaine sur **Cloud Domains** permet de gérer et d’héberger les configurations DNS directement sur Google Cloud. Deux scénarios sont possibles : **l’achat d’un nouveau domaine** via Google Domains ou **l’importation d’un domaine existant** depuis un autre registrar.

**Achat ou importation du domaine**

* **Achat d’un domaine** via la cli: Si le domaine n’a pas encore été enregistré, il peut être acheté directement via **Cloud Domains** avec la commande suivante :
    
    ```plaintext
    gcloud domains registrations register dev.cosinetum.com \
        --contact-data-from-file=contact.json \
        --yearly-price=12USD \
        --cloud-dns-zone=my-dns-zone
    ```
    
    L’achat du domaine peut aussi se faire via la console google cloud. En entrant dans la recherche “cloud domain“ on va se retrouver sur une page qui ressemble a celle ci desssous:
    
* ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1739581066158/5e3d9eb4-47d4-4845-9850-c4d476e8fe55.png align="center")
    

On renseigne les informations concernant le domaine et par la suite on va creer une zone cloud DNS qui va ressembler a cette capture ci lorsque vous finissez de creer la zone cloud associer au noms de domaine que nous avons creer plus haut.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1739581892560/e55c0d17-bf4f-4fc8-a992-bf449c56b345.png align="center")

Nous pouvons aussi creer une zone cloud en utilisant **la ligne de commande** et configurer l’enregistrement **A** pour associer l’adresse IP statique réservée au domaine :

```plaintext
gcloud dns managed-zones create my-dns-zone \
    --dns-name="dev.cosinetum.com." \
    --description="DNS zone for my domain" \
    --visibility="public"
```

Ajout de l’enregistrement **A** pointant vers l’IP statique du Load Balancer :

```plaintext
gcloud dns records-sets transaction start --zone=my-dns-zone

gcloud dns records-sets transaction add 34.120.45.67 \
    --name="dev.cosinetum.com." \
    --ttl=300 \
    --type=A \
    --zone=my-dns-zone

gcloud dns records-sets transaction execute --zone=my-dns-zone
```

on pourra valider nos configurations DNS avec les commandes nslookup ou dig

```plaintext
nslookup dev.cosinetum.com
dig dev.cosinetum.com
```

## Déploiement du cluster Kubernetes sur GKE

L’hébergement de l’application sur **Google Kubernetes Engine (GKE)** nécessite la création d’un **cluster Kubernetes** managé. Une fois le cluster créé, il faut configurer `kubectl` pour interagir avec celui-ci.

### **Création du cluster GKE**

La commande suivante permet de créer un **cluster GKE régional** avec des nœuds de type **e2-standard-4**, ayant **2 nœuds minimum et 5 maximum** en mode autoscaling :

```plaintext
gcloud container clusters create-auto my-gke-cluster \
    --region=europe-west1 \
    --release-channel=stable \
    --network=default \
    --subnetwork=default \
    --enable-autoscaling \
    --min-nodes=2 \
    --max-nodes=5 \
    --machine-type=e2-standard-4
```

La creation du cluster kubernetes peut aussi se faire via la console GCP ou nous pouvons choisir de creer un cluster avec Autopilot ou un cluster Standard.

Note: La creation du cluster peut etre automatiser avec Terraform. Le type de machine, la region doit etre choisi en fonction de vos besoins.

Une fois le cluster créé, il faut mettre à jour le contexte de `kubectl` pour exécuter des commandes sur le cluster :

```plaintext
gcloud container clusters get-credentials my-gke-cluster --region=europe-west1
```

puis

```plaintext
kubectl get nodes 
```

Pour verifier que nous sommes dans le cluster kubernetes que nous venons de creer.

### **Déploiement de l’application sur Kubernetes**

Une application sur Kubernetes est généralement définie via des **manifests YAML**, décrivant les objets Kubernetes nécessaires. Dans notre scenarios une fois notre cluster monte nous avons effectuer le deploiement de la solution clickonsite en utilisant des fichiers helm.

Une fois l’application deployer, nous avons des pods, des services et autres resources qui sont deployer sur notre cluster. Nous allons supposer que vous avez les deploiements et des services sur votre cluster.

Nous allons donc passer a la configuration de l’objet kubernetes FrontendConfig sur GKE.

Configuration FrontendConfig GKE

Nous allons creer un fichier yaml qui aura la configuration du frontend pour activer la redirection http vers https.

```plaintext
apiVersion: networking.gke.io/v1beta1
kind: FrontendConfig
metadata:
  name: my-frontend-config
  namespace: my-namespace
spec:
  redirectToHttps:
    enabled: true
```

Une fois le fichier creer nous allons l’appliquer dans notre cluster avec la commande :

```plaintext
kubectl -n my-namespace apply -f /path/to/frontend-cfg.yaml
```

Creation du certifica

Nous allons creer le certificat manager gke pour le domaine que nous avons creer au debut. Dans un fichier yaml (managed-cert.yaml) nous allons mettre la configuration suivante pour creer un objet ssl lier a notre domaine :

```plaintext
apiVersion: networking.gke.io/v1
kind: ManagedCertificate
metadata:
  name: my-managed-cert
spec:
  domains:
    - dev.cosinetum.com
```

Nous allons appliquer la creation du certificat et patienter un moment pour que le certificat passe ait un statut actif:

```plaintext
kubectl -n my-namespace apply -f managed-cert.yaml
```

On va verifier la creation de notre certificat et son etat avec la commande suivante :

```plaintext
kubectl -n my-namespace describe managedcertificates.networking.gke.io
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1739585769648/245e18e0-462b-48d2-8701-4da0fb05b65f.png align="center")

On veillera a avoir le domaine cible dans le domaine, le status du certificat active, le status du domaine active aussi.

## Creation de l’ingress et de l’equilibreur de charge

Nous allons creer l’ingress, c’est l’element qui nous permet d’exposer nos services a l’exterieur de notre cluster en respectant des regles de routages defini dans l’ingress. Nous allons creer un objet ingress gcp qui va automatiquement creer un equilibreur de charge d’application externe ayant la redirection de traffic http vers https en utilisant le certificat que nous avons generer plus haut. Creons le fichier d’ingress suivant:

```plaintext
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    kubernetes.io/ingress.class: gce
    kubernetes.io/ingress.allow-http: "true" # Allow http traffic over the ingress load balancer
    kubernetes.io/ingress.global-static-ip-name: "mon-ip-statique"  # Associe l'IP statique
    networking.gke.io/managed-certificates: "my-managed-cert"
    networking.gke.io/v1beta1.FrontendConfig: "my-frontend-config" # Associe la configuration frontend qui active la redirection http-vers-https l'objet FrontendConfig doit etre creer
spec:
  rules:
  - host: dev.cosinetum.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-service
            port:
              number: 80
```

On enregistre et on creer l’ingress

```plaintext
kubectl -n my-namespace apply -f my-ingress.yaml 
```

L’ingress deployer va automatiquement creer l’equilibreur de charge d’application externe avec la redirection du traffic HTTP vers HTTPS. Pour voir notre ingress et notre equilibreur de charge nous pouvons verifier la console GCP. saisir load balancer ou equilibreur de charge. Nous allons arriver sur une page qui presentera notre equilibreur de charge.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1739587005647/4c52c68e-64a0-4c72-a526-f10315320f2a.png align="center")

On peut aussi afficher notre ingress en ligne de commande en faisant :

```plaintext
kubectl -n cos describe ingress mon-ingress
```

Et nous devons avoir une sortie semblable a celle ci:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1739587270577/ee9d2ac7-5c61-4d4c-a9e8-8cdc69568cd0.png align="center")

# Tests et validation

Une fois que notre ingress est creer entrainant la creation de notre equilibreur de charge liee a notre domaine et a notre adresse ip statique et faisant le lien vers notre service interne dans notre cluster, nous pouvons verifier en accedant a la page web de notre domaine pour confirmer que tout est Ok ou en faisant la commande suivante sur le domaine :

```plaintext
curl -I https://dev.cosinetum.com
```

En ouvrant l’addresse dans le navigateur nous pouvons acceder a l’application web.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1739587780056/84b83b38-1cb0-4e73-9069-8b265f8675b9.png align="center")

# **Problèmes rencontrés et solutions apportées**

Nous avons rencontres les problemes suivant:

* **Délai de propagation DNS et solutions**. il peut arriver qu’apres plus d’une demi heure votre domaine ne soit pas encore en mode actif.
    
* **Erreurs liées au certificat SSL et résolution**.
    
* **Problèmes de configuration d’Ingress et debug**.
    

# **Conclusion et recommandations**

Dans cet article, nous avons détaillé le déploiement d’une application sur **Google Kubernetes Engine (GKE)** en configurant un **Ingress Kubernetes** associé à un **Application Load Balancer externe**, une **IP statique publique**, et un **certificat SSL géré par GKE**.

Nous avons suivi plusieurs étapes essentielles :  
\- **Réservation d’une IP statique** pour garantir la stabilité des enregistrements DNS.  
\- **Enregistrement du domaine sur Cloud Domains** et configuration des entrées DNS.  
\- **Déploiement du cluster GKE** et configuration de `kubectl`.  
\- **Déploiement de l’application** avec un **Deployment** et un **Service** Kubernetes.  
\- **Mise en place d’un Ingress** et d’un **Load Balancer externe** pour exposer l’application.  
\- **Sécurisation avec HTTPS** via un **certificat SSL géré**.

### **Bonnes pratiques**

* **Utiliser une IP statique** permet d’éviter les interruptions en cas de recréation du Load Balancer.
    
* **Surveiller les logs et métriques** du Load Balancer via **Cloud Monitoring** et **Cloud Logging** pour détecter rapidement d’éventuels problèmes.
    
* **Optimiser l’autoscaling** du cluster GKE pour garantir la disponibilité et la gestion efficace des ressources.
    

### **Perspectives d’amélioration**

📌 **Automatisation avec Terraform** : Toutes ces configurations peuvent être automatisées via **Infrastructure as Code (IaC)** pour plus de reproductibilité et de fiabilité.  
📌 **Mise en place d’une CI/CD** : L’intégration avec **Cloud Build, GitHub Actions ou ArgoCD** permettrait d’automatiser les déploiements et les mises à jour de l’application.  
📌 **Utilisation de Cloud Armor** : Pour renforcer la sécurité en appliquant des règles de filtrage et de protection contre les attaques DDoS.

Avec cette approche, nous avons mis en place une architecture **scalable, sécurisée et optimisée** sur **Google Cloud**, tout en ouvrant la porte à des améliorations continues grâce à l’automatisation et au monitoring.