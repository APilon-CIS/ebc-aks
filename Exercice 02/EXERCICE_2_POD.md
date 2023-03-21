# Exercice 2 - Création d'un pod

Un pod est la plus petite entité gérée par Kubernetes. En effet, c'est au travers des Pods que Kubernetes va gérer les containers. Un pod est un ensemble de 1 ou plusieurs containers. Pour créer un Pod, on va spécifier les containers que l'on souhaite déployer, ainsi que des information supplémentaires si besoin (variables d'environement, ports, volumes etc...)

## 1. Créer un fichier pour le pod

Dans le dossier que vous avez créé pour votre namespace, créez un nouveau fichier `wordpress-pod.yml`.

Comme pour un namespace, on va définir tout d'abord les champs `apiVersion`, `kind` et `metadata`.


```yaml
---
apiVersion: v1                  # Détermine la version de l'API à utiliser
kind: Pod                       # Le type de resource que l'on déploie
metadata:                       # Des métadonnées sur la resources, nom tag etc...
  namespace: <MON-NAMESPACE>    # Déployer le Pod dans votre namespace
  name: wordpress               # Nom du pod
```

Par la suite, pour définir les spécification des containers, on ajoute un champ `spec` dans lequel on remplis les informations de nos containers. La liste et la syntaxe des différente champs est disponible sur la documentation Kubernetes (https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/pod-v1/)

Pour remplir le fichier, référez-vous à la documentation, ou bien utilisez le fichier donné en exemple, en remplissant les champs manquants. Pour rappel, on souhaite :
* 1 container appelé **wordpress**
* Le container utilise l'image **wordpress**
* Le pod est déployé dans votre namespace

## 2. Création du pod

### Appliquer le fichier

Comme pour le namespace, appliquer le fichier grâce à la commande `kubectl apply -f`

## 3. Vérification

Exécutez la commande 

```sh
kubectl get pods -n <mon-namespace>
```

Et vous devriez voir votre pod apparaître. Pour l'instant, il n'est pas possible de joindre notre pod depuis l’extérieur. On peut cependant vérifier que l'application Wordpress fonctionne ouvrant un terminal dans le pod, et en faisant un appel web sur localhost :

```bash
kubectl get pods -n <mon-namespace> # Identifiez le nom de votre pod, normalement Wordpress
kubectl exec -it -n <mon-namespace> wordpress -- bash # Ouvrez un terminal dans le pod
curl -L http://localhost/ # Wordpress devrait répondre !
```