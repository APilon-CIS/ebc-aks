# Exercice 10 - Intégration avec un ACR privé

Depuis le début du TP, nos images de container sont récupérées depuis le docker hub. Ce n'est pas une bonne pratique :
* Le docker hub contient tout et n'importe quoi
* Le docker hub est désormais payant au bout d'un certain nombre de requetes
* C'est publique, il ne faut pas y pousser nos images privées (déploiements internes)
* Nécessite une connexion internet

Généralement, en entreprise, on va avoir un registre privé de container ou stocker les images, et permettre ainsi un accès sécurisé, rapide, permettre de scanner les images pour failles de sécurité etc... Quelques exemples de registres privés : Nexus3, Harbor, Artifactory...

Azure met a disposition une ressource ACR qui est un registre de container hébergé sur les datacenter Azure.

## Un mot sur la connexion aux registres privés

Les registres privés sont protégés par user/password. Pour permettre à Kubernetes de s'identifier auprès des registres, on créé un secret Kubernetes de type ImagePullSecret, que l'on peut référencer dans les deployments.

Azure permet d'intégrer nativement un ACR au cluster Kubernetes en utilisant des identités managées, pas besoin d'utilisateur/mot de passe. Pour ce faire, Azure ajoute une identité à Kubelet, et autorise cette identité à Pull les images du registre.

## Exercice

### Créez un ACR

Sur le portail Azure, créez un ACR, ave les options par défaut

### Copiez une image depuis le dockerhub

Dans un cas classique, notre serveur de CI (gitlab-ci, AzureDevOps), utilise les commandes docker/podman pour pousser l'image sur notre registre. L'AzureCLI permet de copier une image depuis le docker hub sur notre registre. Copiez l'image Wordpress sur votre registre :

```bash
az acr import -g <rg> -n <acr-name> --source docker.io/library/wordpress:latest --image wordpress:v1
```

L'image s’appellera Wordpress:v1 sur votre registre

### Attachez votre ACR au cluster

Pour attacher un ACR au cluster Kubernetes, utilisez la commande 

```bash
az aks update -n <nom-cluster> -g <resource-group> --attach-acr <nom-acr>
```

```
The az aks update --attach-acr command uses the permissions of the user running the command to create the ACR role assignment. This role is assigned to the kubelet managed identity. For more information on AKS managed identities, see Summary of managed identities.
```

### Utilisez votre image privée

Dans votre deployment Wordpress, remplacez l'image pour utiliser l'image privée *<nom-registre>.azurecr.io/wordpress:v1*

Vérifiez que l'image a change :

```bash
kubectl get pods -n <mon-namespace> # Identifiez un pod Wordpress
kubectl describe pod -n <mon-namespace> <mon-pod> # Regardez le champ Image
```