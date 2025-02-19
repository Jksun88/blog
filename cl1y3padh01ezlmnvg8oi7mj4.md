---
title: "Connexion a Terraform Cloud via la CLI"
seoTitle: "Connexion a terraform cloud"
seoDescription: "se connecter a terraform cloud"
datePublished: Wed Apr 13 2022 21:44:57 GMT+0000 (Coordinated Universal Time)
cuid: cl1y3padh01ezlmnvg8oi7mj4
slug: connexion-a-terraform-cloud-via-la-cli
tags: cloud, terraform

---

**Terraform Cloud** c'est l'offre de service gerees de Hashicorp. Il supprime le besoin d'outils et documentation non utile pour les organisations et professionelle qui utilisent Terraform en production. 

Terraform est un outil qui permet de provisionner une infrastructure situe dans un environnement distant et qui est capable de supporter le flux de travail de Terraform (bon nombre d'environnement prennent en charge le flux de travail de Terraform). 

Terraform Cloud sauvegarde le fichier d'etat (*state file*) a distance de maniere securise. 

### Connexion a Terraform Cloud

Pour utiliser Terraform Cloud a travers la ligne de commande (CLI), vous devez: 

1. Creer un compte [Terraform Cloud](https://cloud.hashicorp.com/products/terraform)
Puis dans la ligne de commande (CLI) de la machine que vous voulez connecter a Terraform Cloud
2. Entrez la commande 
```terraform login
``` 
et tapez ***"yes"*** a la question et allez dans votre navigateurs (la page s'ouvre automatiquement apres votre confirmation, sinon aller a l'url indique dans votre console) 
3. Generation du token d'authentification
Generer le token, vous pouvez garder le nom par defaut terraform login ou pas. 
4. Copiez le token generer et coller le une fois dans votre ligne de commande sur votre machine. 
Le token etant sensible il ne s'affichera pas dans la ligne de commande. 

Une fois que vous avez fait cela, vous pouvez desormais vous connecter a votre compte Terraform Cloud. Rassurer vous de specifier le bloc cloud dans le sous bloc terraform de votre fichier de configuration. 

