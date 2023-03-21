# Exercice 4 - Exposer Wordpress via un Service

Les services sont le mécanisme fourni par Kubernetes pour permettre d'exposer un ou plusieurs Pods au sein du cluster (pour une communication de Pod a Pod), ou bien pour exposer les Pods à l’extérieur du cluster.

Il existe 3 types principaux de Service :
* **ClusterIP** : expose les Pods à l'intérieur du cluster. La création d'un service entraîne en réalité la création d'une adresse IP qui redirigera le trafique vers les Pods, ainsi qu'une entrée DNS qui porte le nom du service. Ainsi, les Pods ciblés par le services sont joignables par les autres Pods via l'adresse **\<nom-du-service\>.\<nom-du-namespace\>.svc.cluster.local**. Au sein d'un même namespace, les Pods peuvent utiliser seulement le nom du service
* **NodePort** : Attribue, en plus de l’IP interne, un port commun a tous les nœuds Kubernetes pour rediriger le trafique vers les Pods. On peut ainsi joindre les Pods sur **http://\<ip-node\>:\<node-port\>/**
* **LoadBalancer**: Attribue en plus une IP Publique au service, pour qu’il soit joignable facilement depuis l’extérieur

Pour savoir vers quels Pods rediriger le trafique, les services :
* Utilisent également un mécanisme de **selecteurs** (la syntaxe est légèrement différente)
* Doivent savoir sur quel port écouter, et vers quel port du Pod cible rediriger le trafique

```yaml
# Champs habituels
# ...
  type: # Type du service
  selector:
    # Quels selectors pour cibler vos Pods ?
  ports:
    - port: # Que quel port écoute le service ?
      targetPort: # Quel est le port cible ?
      name: #Donner un petit nom parlant
```

## 1. Créer un fichier pour le Service Wordpress

Vous devriez avoir déjà toutes les infos pour créer votre Service Wordpress. En cas de doute, la documentation Kubernetes sur les Service peut vous aider : https://kubernetes.io/docs/reference/kubernetes-api/service-resources/service-v1/

Pour rappel, on souhaite :
* Exposer Wordpress sur une IP Publique
* Nom du fichier : **wordpress-service.yml**
* Nom du service: **wordpress**
* Namespace: votre namespace créé précédemment

## 2. Verification

```bash
kubectl get service -n <mon-namespace> #Repérez l’adresse IP 'EXTERNAL-IP'
curl -L http://<ip-publique>/ # Ou bien directement dans votre navigateur !
```