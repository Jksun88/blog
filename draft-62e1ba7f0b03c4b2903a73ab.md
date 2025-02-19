---
title: "Commande utiles: KUBERNETES "
slug: commande-utiles-kubernetes

---

Kubernetes est un outils d'orchestration deja bien utilise dans la couche technologique des entreprises. 
Nous allons repertoriez dans cet article quelque commandes qui sont frequemment utilisees. 

1. Security 

kubectl get pods --kubeconfig config
kubectl config use-context name_of_context: pour changer de contexte. 


The kubeconfig file is usually located at **$HOME/.kube/config**. That file have three parts. 
- clusters
- users 
- contexts 

In the config file we can specify the certificate file in 2 ways : 
- first by specifying the name of the certificate or the full path to the certificate file in the *certificate-authority* field. 
- Use the *certificate-authority-data* to specify the exact data value of the certificate. 