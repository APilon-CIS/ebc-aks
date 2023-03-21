# Exercice 3 - Création d'un Deployment

Un Deployment est un objet qui va permettre de déployer **un ou plusieurs Pods** en suivant une stratégie, qui peut être **RollingUpdate** (par défaut) ou **Recreate**. Toutes les modifications que l'on souhaite apporter a nos Pods seront effectuées sur l'objet deployment, par exemple :
* Changement de **version**
* Changement de **variable**
* Changement du nombre de **replicas**
* ...

Des que Kubernetes détecte une modification sur l'objet deployment, il va supprimer/créer des nouveaux Pods qui utiliseront la nouvelle configuration.

Les 2 stratégies de déploiement sont :
* **RollingUpdate** : Ajoute des nouveaux Pods, puis supprime les anciens. Permet des déploiements sans interruption de service. Il est possible de configurer le nombre de Pods à créer / supprimer
* **Recreate** : détruit tous les Pods puis en déploie de nouveaux

## 1. Supprimer le pod créé à l'exercice précédent

Supprimer le pod Wordpress créé précédemment, car c'est maintenant via un Deployment que l'on va créer des Pods :

```bash
kubectl delete pod wordpress -n <mon-namespace>
```

## 2. Créer un fichier pour le Deployment

Dans le dossier que vous avez créé pour votre namespace, créez un nouveau fichier `wordpress-deploy.yml`.

Pour un deployment, on va spécifier les champs habituels, ainsi que :
* Une template de pod, qui sert de modèle pour les pods à déployer
* Le nombre de replicas (le nombre de pods démarrés pour de la HA)
* Un selecteur, qui permet au deployment de savoir quels Pods il gère le cycle de vie

```yaml
---
apiVersion: ...
kind: ...
metadata:
  name: ...
  namespace: ...
spec:
  selector:
    matchLabels:
      # Les labels a matcher au format key: value
  replicas: # Le nombre de replicas souhaité
  template: # La template de Pod
    metadata:
      labels:
        # Attention aux labels
        # On ne spécifier pas de nom ou namespace, c'est le Deployment qui gère
    spec:
      # mêmes données que dans un pod
```

Pour rappel, on souhaite un Deployment avec :
* Nom du Deployment: **wordpress**
* Replicas: **2**
* Namespace: votre namespace créé précédemment
* Labels et selecteurs: *app: wordpress*

Le detail des champs, et notamment la version de l'API est disponible sur la documentation Kubernetes : https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/deployment-v1/

## 3. Vérification

Exécutez les commandes suivantes

```bash
kubectl get deployment -n <mon-namespace>
kubectl get pods –n <mon-namespace> # Remarquez le nombre de pods
```
