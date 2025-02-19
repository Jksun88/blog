---
title: "Comprendre les Comptes de Service dans Kubernetes"
seoTitle: "Comptes de Service Kubernetes Explications"
seoDescription: "Guide sur les comptes de service Kubernetes: Créez, affichez et associez des comptes pour sécuriser l'accès aux API"
datePublished: Mon Jun 24 2024 14:29:11 GMT+0000 (Coordinated Universal Time)
cuid: clxt2pxy000010al425azghwx
slug: comprendre-les-comptes-de-service-dans-kubernetes
tags: kubernetes, kubernetes-security, kubernetes-service-account

---

Ce sont des comptes réservés aux applications et outils pour accéder à l'API Kubernetes. Par exemple, nous pouvons avoir l'application de monitoring **Prometheus** qui interroge le cluster pour récupérer les métriques de performance, ou encore l'application **Jenkins** qui déploie sur le cluster k8s.

Commande pour créer un service account :

`kubectl create serviceaccount nom-sa`

Pour voir un service account (compte de service) on utilise la commande:

`kubectl get serviceaccount`

Lorsque le service account est créé, un token est également généré. Ce token sera utilisé par l'application pour accéder au cluster. Le token ainsi généré est stocké comme un secret.

Dans la ***version 1.22***, la création d'un token se fait comme expliqué plus haut. Par contre, dans la ***version 1.24***, la création d'un service account n'implique plus la création d'un token. Dès lors, les tokens ne sont plus montés automatiquement sur tous les pods créés dans le namespace.

Pour associer un compte de service et son token à un déploiement, on peut utiliser la commande suivante :

`kubectl set serviceaccount deploy/web-dashboard dashboard-sa` 

**web-dashboard** est le nom du déploiement auquel nous voulons associer le compte de service.

**dashboard-sa** est le nom du compte de service à associer