---
title: "Déploiement d'une Application sur GKE avec Ingress et Cloud Load Balancer : Guide Essentiel DevOps"
seoTitle: "Déploiement d'Applications sur GKE avec Ingress et Load Balancer"
seoDescription: "Apprenez à déployer des applications sur GKE avec Ingress et Cloud Load Balancer pour optimiser performance et sécurité avec des certificats SSL gérés GCP."
datePublished: Fri Feb 21 2025 12:18:22 GMT+0000 (Coordinated Universal Time)
cuid: cm7eqlutk000209l45mxp8ibp
slug: deploiement-dune-application-sur-gke-avec-ingress-et-cloud-load-balancer-guide-essentiel-devops
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/KU9ABpm7eV8/upload/6add9ae052e478025c92bf949f958e68.jpeg
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1740139949741/4ae2d8d1-22ab-49cb-99c4-d69937c63a9d.png
tags: kubernetes, cloud-computing, devops, gcp, gke

---

Dans un environnement cloud en constante évolution, le déploiement d’applications scalables et sécurisées est un défi majeur pour les ingénieurs DevOps. Kubernetes, combiné aux services managés de Google Cloud Platform (GCP), offre une solution robuste pour orchestrer des applications tout en assurant haute disponibilité et flexibilité.

Dans cet article, nous partageons un retour d’expérience sur le déploiement d’une application sur **Google Kubernetes Engine (GKE)**, avec une exposition publique via **Kubernetes Ingress** et un **Cloud Load Balancer**. Pour garantir une accessibilité stable et sécurisée, nous avons utilisé :

* **Une adresse IP statique publique** pour éviter les changements d’IP,
    
* **Cloud Domains** pour la gestion du nom de domaine [`domaine.example.com`](http://dev.cosinetum.com),
    
* **GKE Managed Certificate** pour une gestion automatisée du certificat SSL.
    

Cet article s’adresse principalement aux **ingénieurs DevOps** cherchant à industrialiser leurs déploiements sur GCP. Nous décrirons chaque étape, des choix techniques aux défis rencontrés, afin de fournir une méthodologie reproductible et des bonnes pratiques pour vos propres projets.

# Architecture Cible

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1740094039762/9ef109ce-5d9c-43bb-ac62-4e1fc32bc794.png align="center")

* *Architecture application sur GKE avec Ingress, Cloud Load Balancer et GKE Managed Certificate.*
    
* L’architecture ci-dessus illustre la mise en place d’un déploiement Kubernetes sur **Google Kubernetes Engine (GKE)**, avec une exposition sécurisée via un **External Application Load Balancer** et un **Ingress Kubernetes**. Les composants que nous avons utiliser pour mettre en place l’architecture sont les suivants:
    
* **Nom de domaine et résolution DNS**
    
    * Le domaine [`domaine.example.com`](http://dev.cosinetum.com) est géré via **Cloud Domains** et pointe vers une **adresse IP statique publique** réservée sur Google Cloud Platform.
        
    * Cette IP statique garantit que l’accès à l’application reste stable, même en cas de modification des ressources sous-jacentes.
        
* **Gestion du trafic avec le Load Balancer**
    
    * Un **External Application Load Balancer** de type globale, est provisionné automatiquement par le **Kubernetes Ingress Controller**.
        
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

Note : On peut aussi réserver l'adresse IP statique publique en passant par la console Google Cloud (interface graphique).

## **Enregistrement du domaine sur Cloud Domains**

L’enregistrement du domaine sur **Cloud Domains** permet de gérer et d’héberger les configurations DNS directement sur Google Cloud. Deux scénarios sont possibles : **l’achat d’un nouveau domaine** via Google Domains ou **l’importation d’un domaine existant** depuis un autre registrar.

**Achat ou importation du domaine**

* **Achat d’un domaine** via la cli: Si le domaine n’a pas encore été enregistré, il peut être acheté directement via **Cloud Domains** avec la commande suivante :
    
    ```plaintext
    gcloud domains registrations register domaine.example.com \
        --contact-data-from-file=contact.json \
        --yearly-price=12USD \
        --cloud-dns-zone=my-dns-zone
    ```
    
    L’achat du domaine peut aussi se faire via la console Google Cloud. En entrant "cloud domain" dans la recherche, on arrive sur une page qui ressemble à celle ci-dessous :
    
* ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1739581066158/5e3d9eb4-47d4-4845-9850-c4d476e8fe55.png align="center")
    

On renseigne les informations concernant le domaine, puis nous créons une zone Cloud DNS qui ressemblera à cette capture une fois que vous aurez terminé de créer la zone Cloud associée au nom de domaine que nous avons créé plus haut.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1740060285783/e2ae9638-72ca-410e-b0fa-1bf822a62df7.png align="center")

Nous pouvons aussi créer une zone cloud en utilisant **la ligne de commande** et configurer l’enregistrement **A** pour associer l’adresse IP statique réservée au domaine :

```plaintext
gcloud dns managed-zones create my-dns-zone \
    --dns-name="domaine.example.com." \
    --description="DNS zone for my domain" \
    --visibility="public"
```

Ajout de l’enregistrement **A** pointant vers l’IP statique du Load Balancer :

```plaintext
gcloud dns records-sets transaction start --zone=my-dns-zone

gcloud dns records-sets transaction add 34.120.45.67 \
    --name="domaine.example.com." \
    --ttl=300 \
    --type=A \
    --zone=my-dns-zone

gcloud dns records-sets transaction execute --zone=my-dns-zone
```

on pourra valider nos configurations DNS avec les commandes nslookup ou dig

```plaintext
nslookup domaine.example.com
dig domaine.example.com
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

La création du cluster Kubernetes peut aussi se faire via la console GCP où nous pouvons choisir de créer un cluster avec Autopilot ou un cluster Standard.

Note : La création du cluster peut être automatisée avec Terraform. Le type de machine et la région doivent être choisis en fonction de vos besoins.

Une fois le cluster créé, il faut mettre à jour le contexte de `kubectl` pour exécuter des commandes sur le cluster :

```plaintext
gcloud container clusters get-credentials my-gke-cluster --region=europe-west1
```

puis

```plaintext
kubectl get nodes
```

Pour vérifier que nous sommes dans le cluster Kubernetes que nous venons de créer.

### **Déploiement de l’application sur Kubernetes**

Une application sur Kubernetes est généralement définie via des **manifests YAML**, décrivant les objets Kubernetes nécessaires. Dans notre scénario, une fois notre cluster monté, nous effectuons le déploiement de la solution ClickOnSite en utilisant des fichiers Helm.

Une fois l’application déployée, nous avons des pods, des services et d'autres ressources qui sont déployés sur notre cluster. Nous allons supposer que vous avez les déploiements et les services sur votre cluster.

Nous allons donc passer à la configuration de l’objet Kubernetes **FrontendConfig** sur GKE.

### Configuration FrontendConfig GKE

Nous allons créer un fichier YAML contenant la configuration du frontend pour activer la redirection HTTP vers HTTPS.

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

Une fois le fichier créé, nous allons l’appliquer dans notre cluster avec la commande :

```plaintext
kubectl -n my-namespace apply -f /path/to/frontend-cfg.yaml
```

### Création du certificat

Nous créons le certificat manager GKE pour le domaine que nous avons créé au début. Dans un fichier YAML (managed-cert.yaml), la configuration pour créer un objet SSL lié à notre domaine est la suivante :

```plaintext
apiVersion: networking.gke.io/v1
kind: ManagedCertificate
metadata:
  name: my-managed-cert
spec:
  domains:
    - domaine.example.com
```

On applique (crée) le certificat et on patiente un moment pour que le certificat obtienne un statut actif :

```plaintext
kubectl -n my-namespace apply -f managed-cert.yaml
```

On vérifie la création de notre certificat et son état avec la commande suivante :

```plaintext
kubectl -n my-namespace describe managedcertificates.networking.gke.io my-managed-cert
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1740103260815/f0cba890-8e8f-49f7-ab1b-6465080bdcec.png align="center")

On veillera a avoir le domaine cible dans le domaine, le status du certificat active, le status du domaine active aussi.

## Creation de l’ingress et de l’equilibreur de charge

Nous créons l'ingress, c'est l'élément qui nous permet d'exposer nos services à l'extérieur de notre cluster en respectant des règles de routage définies dans l'ingress. Nous créons un objet ingress GCP qui crée automatiquement un équilibreur de charge d'application externe avec la redirection du trafic HTTP vers HTTPS en utilisant le certificat que nous avons généré plus haut. Créons le fichier d'ingress suivant :

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
  - host: domaine.example.com
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

On enregistre et on crée l’ingress.

```plaintext
kubectl -n my-namespace apply -f mon-ingress.yaml
```

L’ingress déployé va automatiquement créer l’équilibreur de charge d’application externe avec la redirection du trafic HTTP vers HTTPS. Pour voir notre ingress et notre équilibreur de charge, nous pouvons vérifier la console GCP en saisissant "load balancer" ou "équilibreur de charge". Nous arriverons sur une page qui présentera notre équilibreur de charge.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1740103092964/1b6c7560-72d2-4cec-b403-27819fc4456f.png align="center")

On peut aussi afficher notre ingress en ligne de commande en faisant :

```plaintext
kubectl -n cos describe ingress mon-ingress
```

Et nous devons avoir une sortie semblable à celle-ci :

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1740102938454/5e8542cf-8f6c-4014-83f0-e76189ed3622.png align="center")

# Tests et validation

Une fois que notre ingress est créé, entraînant la création de notre équilibreur de charge lié à notre domaine et à notre adresse IP statique, et établissant le lien vers notre service interne dans notre cluster, nous pouvons vérifier en accédant à la page web de notre domaine pour confirmer que tout est en ordre ou en exécutant la commande suivante sur le domaine :

```plaintext
curl -I https://domaine.example.com
```

En ouvrant l'adresse dans le navigateur, nous pouvons accéder à l'application web.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1740102339644/4a758bd3-0199-4d9d-a0da-0f010c03db1c.png align="center")

# **Problèmes rencontrés et solutions apportées**

Nous avons rencontré les problèmes suivants :

* **Délai de propagation DNS et solutions**. Il peut arriver qu'après plus d'une demi-heure, votre domaine ne soit pas encore actif.
    
* **Erreurs liées au certificat SSL et résolution**.
    
* **Problèmes de configuration d'Ingress et débogage**. Il faut s'assurer que l'ingress pointe vers le bon service, que les règles correspondent à ce que nous souhaitons, et que les ports sont corrects.
    

# **Conclusion et recommandations**

Dans cet article, nous avons détaillé le déploiement d’une application sur **Google Kubernetes Engine (GKE)** en configurant un **Ingress Kubernetes** associé à un **Application Load Balancer externe**, une **IP publique statique**, et un **certificat SSL géré par GKE**.

Nous avons suivi plusieurs étapes essentielles :  
\- **Réservation d’une IP statique** pour garantir la stabilité des enregistrements DNS.  
\- **Enregistrement du domaine sur Cloud Domains** et configuration des entrées DNS.  
\- **Déploiement du cluster GKE** et configuration de `kubectl`.  
\- **Déploiement de l’application** avec un **Deployment** et un **Service** Kubernetes.  
\- **Mise en place d’un Ingress** qui a automatiquement créé un **Load Balancer externe** pour exposer l’application.  
\- **Sécurisation avec HTTPS** via un **certificat SSL géré** qui a été généré à partir de l’objet **managed certificate** de Kubernetes et l’objet **FrontendConfig**.

### **Bonnes pratiques**

* **Utiliser une IP statique** permet d’éviter les interruptions en cas de recréation du Load Balancer.
    
* **Surveiller les logs et métriques** du Load Balancer via **Cloud Monitoring** et **Cloud Logging** pour détecter rapidement d’éventuels problèmes.
    
* **Optimiser l’autoscaling** du cluster GKE pour garantir la disponibilité et la gestion efficace des ressources.
    

### **Perspectives d’amélioration**

📌 **Automatisation avec Terraform** : Toutes ces configurations peuvent être automatisées via **Infrastructure as Code (IaC)** pour plus de reproductibilité et de fiabilité.  
📌 **Mise en place d’une CI/CD** : L’intégration avec **Cloud Build, GitHub Actions ou ArgoCD** permettrait d’automatiser les déploiements et les mises à jour de l’application.  
📌 **Utilisation de Cloud Armor** : Pour renforcer la sécurité en appliquant des règles de filtrage et de protection contre les attaques DDoS.

📌**Utilisation de Cloud CDN en frontal** : pour améliorer la performance et la distribution des contenus. Cela dépend de vos besoins.

Avec cette approche, nous avons mis en place une architecture **scalable, sécurisée et optimisée** sur **Google Cloud**, tout en ouvrant la porte à des améliorations continues grâce à l’automatisation et au monitoring.

Liens utiles:

* Equilibreur de charge : [https://cloud.google.com/load-balancing/docs/application-load-balancer](https://cloud.google.com/load-balancing/docs/application-load-balancer)
    
* Configuration de l’ingress : [https://cloud.google.com/kubernetes-engine/docs/how-to/load-balance-ingress?hl=fr](https://cloud.google.com/kubernetes-engine/docs/how-to/load-balance-ingress?hl=fr)
    
* Configuration de l’objet ManagedCertificate : [https://cloud.google.com/kubernetes-engine/docs/how-to/managed-certs](https://cloud.google.com/kubernetes-engine/docs/how-to/managed-certs)
    
* Configuration de la redirection HTTP vers HTTPS ingress FrontendConfig : [https://cloud.google.com/kubernetes-engine/docs/how-to/ingress-configuration#https\_redirect](https://cloud.google.com/kubernetes-engine/docs/how-to/ingress-configuration#https_redirect)
    
* Provisionner un cluster GKE avec Terraform : [https://developer.hashicorp.com/terraform/tutorials/kubernetes/gke](https://developer.hashicorp.com/terraform/tutorials/kubernetes/gke)
    
* Service Kubernetes : [https://kubernetes.io/fr/docs/concepts/services-networking/service/](https://kubernetes.io/fr/docs/concepts/services-networking/service/)
    
* Deploiements Kubernetes : [https://kubernetes.io/fr/docs/concepts/workloads/controllers/deployment/](https://kubernetes.io/fr/docs/concepts/workloads/controllers/deployment/)