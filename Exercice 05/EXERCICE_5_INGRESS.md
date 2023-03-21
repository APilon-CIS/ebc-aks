# Exercice 5 - Exposer Wordpress via une Ingress
---
## Prérequis

* Changez le type de service Wordpress en **ClusterIP**. On va désormais l'exposer via une Ingress
* Activez l'intégration entre **AKS et l'Application Gateway**
---

Les Ingress sont un mécanisme qui permettent d'installer et de configurer des LoadBalancer de niveau 7 au sein du cluster Kubernetes :
* Un pod, appelé **Ingress controller** va écouter l'API Kubernetes, et prendre en charge toutes les resources de type Ingress qui lui sont assignées
* Ce pod est généralement exposé sur un IP externe au cluster (IP Publique par exemple), et sert de point d'entrée à plusieurs applications

Pour permettre l'utilisation d'un Application Gateway, AKS va :
* Créer un Application Gateway
* Déployer un pod avec une identité managée, qui aura les droits de modification sur l'application gateway créée
* Écouter pour la création de resources de type Ingress, qui contiennent l'annotation
```yaml
kubernetes.io/ingress.class: azure/application-gateway
```

Ainsi, le pod configurera l'application gateway en fonction des règles renseignées dans le fichier .yml

## 1. Créez votre Ingress pour exposer Wordpress

Utilisez la documentation Kubernetes pour configurer votre ingress : https://kubernetes.io/docs/reference/kubernetes-api/service-resources/ingress-v1/ (intéressez vous uniquement à la partie **rules.http**, les autres champs ne nous concernent pas pour le moment).

Vous pouvez également utiliser la documentation spécifique à l'ingress controller Azure Application Gateway https://azure.github.io/application-gateway-kubernetes-ingress/annotations/.

Quelques indices :
* Contrairement aux services, qui pointent vers des Pods, les Ingress ciblent des Services
* On souhaite que tous les trafique '/\<votre-nom\>' soit redirigé vers votre service Wordpress

## 2. Vérification

```bash
kubectl get ingress -n <mon-namespace> # Accédez à Wordpress dans votre navigateur !
```