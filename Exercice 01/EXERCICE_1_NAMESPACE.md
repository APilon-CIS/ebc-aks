# Exercice 1 - Création d'un namespace

On communique avec un cluster Kubernetes en écrivant les resources que l'on souhaite déployer dans des fichiers YAML, que l'on envoie au cluster. Le but de cet exercice est de créer un namespace tout simple, pour découvrir comment communiquer avec le cluster Kubernetes.

## 1. Créer un fichier pour le namespace

Créez un dossier ou vous allez placer tous les fichiers des différents exercices. Dans ce dossier, créez un fichier et nommez le `namespace.yml`

Le contenu du fichier sera le suivant (pensez à remplacer **\<MON-NAMESPACE\>** par une valeur de votre choix, contenant votre nom par exemple)

```yaml
---
apiVersion: v1  # Détermine la version de l'API à utiliser
kind: Namespace # Le type de resource que l'on déploie
metadata:       # Des métadonnées sur la resources, nom tag etc...
  name: <MON-NAMESPACE>
```

Les champs **apiVersion**, **kind** et **metadata** sont communs à la quasi totalité des resources Kubernetes. Généralement vient s'ajouter à ses champs un champ **spec**. Pour savoir comment remplir ces champs, la documentation vous renseigne : https://kubernetes.io/docs/reference/kubernetes-api/

Ici, pour la création d'un namespace : https://kubernetes.io/docs/reference/kubernetes-api/cluster-resources/namespace-v1/

## 2. Création du namespace

### Appliquer le fichier

Pour appliquer les modification sur le cluster telles que décrites dans le fichier .yml, rien de plus simple :

```sh
kubectl apply -f /chemin/vers/mon/fichier.yml
```

Il est également possible de spécifier un dossier, kubectl appliquera tous les fichiers YAML qu'il trouvera dans ce dossier.

### Modification

En cas de modification de certains champs, un `kubectl apply` suffit à appliquer uniquement les modifications apportées au fichier. Certains champs ne peuvent pas être modifiés, et nécessitent donc la suppression et recréation de la ressource.

### Suppression

Pour supprimer une ressource, `kubectl delete -f /chemin/vers/mon/fichier.yml`. Il est possible de supprimer aussi via `kubectl delete <TYPE-RESOURCE> <NOM-RESOURCE>`

## 3. Vérification

Exécutez la commande 

```sh
kubectl get namespace
```

Et vous devriez voir votre namespace apparaître.