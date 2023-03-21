# Exercice 6 - Création d'une base de données dans Kubernetes
---
## Prérequis
* Repasser le nombre de replicas Wordpress à 1

---

Le but de cet exercice est d'utiliser toutes les compétences que vous avez acquis dans les précédents exercices pour ajouter une base de données à votre site Wordpress.

On souhaite donc :
* Avoir un container de base de données qui utilise MySQL sur Kubernetes
* Avoir un site Wordpress qui se connecte tout seul a la BDD (vous aurez besoin d'ajouter des variables d'environnement à vos Pods https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/pod-v1/#environment-variables)

Je vous met ici quelques ressources utiles, et bon courage ! :)
* https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/deployment-v1/
* https://kubernetes.io/docs/reference/kubernetes-api/service-resources/service-v1/
* https://hub.docker.com/_/mysql
* https://hub.docker.com/_/wordpress

## Validation

Rendez-vous sur votre site Wordpress, terminez le setup, et postez des articles ! :)