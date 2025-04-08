---
title: "Connecter un Cluster Kubernetes à un Registre d'Images Privé"
seoTitle: "Kubernetes Cluster: Link to Private Image Registry"
seoDescription: "Connectez Kubernetes à un registre d'images privé avec des secrets docker-registry pour des déploiements sécurisés"
datePublished: Mon Jun 24 2024 23:00:51 GMT+0000 (Coordinated Universal Time)
cuid: clxtkzy2n000109l3450sgvvl
slug: connecter-un-cluster-kubernetes-a-un-registre-dimages-prive
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/YniQ5vouxqg/upload/c2a4a567ac839d0af9e910b3d6299017.jpeg
tags: kubernetes, registry, kubernetes-security

---

Les images sont utilisées comme base pour le déploiement des applications conteneurisées.

Prenons, par exemple, un fichier de définition de pods simple.

```plaintext
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
    name: web-server
    image: nginx
```

L'image mentionnée ici provient de Docker Hub (lorsque les configurations sont par défaut). Ces images sont produites par des équipes officielles et vérifiées par Docker Hub. Il est possible d'utiliser des images provenant de registres privés. Dans ce cas, il faut ajouter le nom du registre, le compte à partir duquel on récupère l'image, puis le nom de l'image.

**Exemple**: [gcr.io/google-containers/cadvisor](http://gcr.io/google-containers/cadvisor)

Pour que notre cluster se connecte à un registre Docker, nous devons utiliser un type de secret `docker-registry`, qui est un type de secret par défaut sur Kubernetes.

```plaintext
kubectl create secret docker-registry regcred 
--docker-server=notre-registre-prive.io \
--docker-username=user-registre \
--docker-password=mdp-registre \
--docker-email=user@email.com
```

***regcred***: est le nom du secret

Une fois fait, nous pouvons désormais connecter notre cluster à notre registre Docker privé et récupérer les images qui s'y trouvent.

Ajoutez `imagePullSecrets` dans le fichier de définition du pod et mettez le nom du secret Docker précédemment créé.

```plaintext
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: web-server
    image: nginx
  imagePullSecrets: 
  - name: regcred 
```

Voilà comment se connecter de manière sécurisée à son registre sous Kubernetes.