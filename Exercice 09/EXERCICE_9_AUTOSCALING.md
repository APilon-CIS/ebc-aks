# Exercice 9 - Scaling automatique

Kubrenetes dispose d'un mécanisme natif pour permettre d'augmenter ou diminuer le nombre de replicas d'un deployment en fonction de la charge. Ce type d'objet est appelé un **HorizontalPodAutoscaler**.

Dans cet objet, on va définir un liste de critères de scalabilité, qui vont se baser sur les **métriques des Pods (CPU par exemple)**, le nombre de replicas maximum (pour éviter une explosion des coûts) et d'autres critères éventuel.

> Les HorizontalPodAutoscaler nécessitent que le cluster ait metric-server d'installé. C'est le cas sur AKS.
## Un point sur les resources des Pods

Dans un pod, il est possible de définir une liste de ressources, comme le CPU et la RAM. Il ya 2 types de ressources : les **requests** et les **limits**.

### Requests

La partie Requests va définir les ressources **nécessaires** au Pod pour fonctionner. Ces données sont utilisées par le **scheduler** pour placer le Pod au mieux. Par exemple, si un pod a besoin de **1 CPU**, Kubernetes placera le Pod sur un node qui a **au moins 1 CPU disponible**.

### Limits

Les limits correspondent aux ressources que le pod **n'est pas censé dépasser** dans des conditions normales. Si la limite de CPU est atteinte, le Pod ne pourra pas consommer plus de CPU (il sera bridé). En revanche, pour la RAM, la limité peut être dépassée par le Pod. Si le pod dépasse la limite de RAM, Kubernetes va, au bout d'un moment, **supprimer le Pod et le recréer**. On peut avoir des logs de type **OOMKilled**.

## Prérequis

Les HorizontalPodAutoscaler vont nous permettre de définir une règle de scalabilité si un pod atteints **un certain pourcentage du CPU** qu'il a dans ses requests. Il faut donc ajouter une request CPU dans notre deployment.

Dans le deployment Wordpress, ajoutez les ressources suivantes dans la partie container :

```yaml
resources:
  requests:
    cpu: 50m
```

## Créez un HorizontalPodAutoscaler

Créez un objet de type HorizontalPodAutoscaler. LA syntaxe est la suivante :

```yaml
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: # Un nom
  namespace: # Votre namespace
spec:
  maxReplicas: # A voir, mettez 4 par exemple
  scaleTargetRef:
    kind: Deployment
    name: # Le nom du déploiement que l'on souhaite scaler
    apiVersion: apps/v1
  metrics: # Ici, le type de métrique sur lequel on souhaite se baser
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

Un HorizontalPodAutoscaler déclenchera une scalabilité lorsque l'utilisation **moyenne de CPU dépassera les 50% des requests**.

Vérifiez que votre HPA est créé :

```bash
kubectl get hpa -n <mon-namespace>
```

Maintenant, il s'agit de simuler une montée en charge. Signalez-moi que vous avez terminé et j'utiliserai un outil pour le faire. Les différents moyens d'effectuer des tests de montée en charge seront échangés à l'oral. Quelques exemples :
* Locus
* Gatling
* Azure Load Testing
* Chaos Mesh