# Exercice 8 - Secrets et intégration KeyVault

Pour utiliser des valeurs sensibles, Kubernetes met à disposition des objets de type **Secret**. Les Secrets Kubernetes vont stocker les valeurs sensibles en *base64* pour les mettre à disposition des Pods, qui pourront les utiliser dans des volumes ou des variables d'environnement.

Cependant, étant donné que les secrets sont en base64, **il est primordial de ne pas mettre sur GIT** les objets de type Secret, mais de stocker les valeurs sur un stockage fait pour, comme l'Azure KeyVault.

Pour créer des secrets depuis des valeurs contenues dans Azure KeyVault, nous allons :
1. Activer l'intégration de notre cluster Kubernetes avec l'Azure KeyVault (cluster configuration -> enable secret store CSI driver)
2. Créer des objets de type *SecretProviderClass*

Le premier point va déployer des Pods dans le cluster qui s'occuperont de tout ce qui concerne les SecretProviderClass. Le second contiendra toutes les informations nécessaires à la récupération d'un secret dans le KeyVault (nom du secret, nom du KeyVault, identité managée etc...). Cet objet permettra aux Pods de monter le secret dans un volume et de créer un Secret Kubernetes pour être utilisé par variable d'environnement.

## 1 - Activer secret store CSI driver

Dans les options de votre cluster Kubernetes, allez dans **Cluster Configuration**, et cochez **enable secret store CSI driver**. Validez.

## 2 - Création d'un KeyVault

* Créez un Azure KeyVault.
* Dans la création, choisissez **Azure role-based access control**

## 3 - Ajout des droits sur le KeyVault

Une fois le KeyVault créé, il faut donner des droits à :
* **Vous**, pour créer des secrets
* **Au cluster Kubernetes**, pour qu'il puisse lire les secrets

Rendez vous dans
* **IAM**
* **Add Role Assignement**
* Role : **Key Vault Secrets Officer**
* Members : **sélectionnez votre compte**

Pour autoriser le cluster Kubernetes à lire les secrets, il faut assigner des droits à la bonne identité managée. L'activation du Secret CSI Driver **créé automatiquement une identité mangée qui est attachée à toutes les VMs du cluster**.

Rendez vous dans
* **IAM**
* **Add Role Assignement**
* Role : **Key Vault Secrets User**
* Members : cochez **'managed identity'** et choisissez l'**identité managée pour les secrets de votre cluster**.

## 4 - Créez un secret dans le keyvault

Sur Azure Keyvault, **créez un secret** pour stocker le **mot de passe de la base de données**.

**Très important !!** -> Utilisez le même mot de passe que vous aviez définis dans les variables d'environnement.

## 5 - Créez un objet de type SecretProviderClass

Créez un objet de type **SecretProviderClass**. La documentation de tous les champs possible pour cet objet est disponible ici : https://azure.github.io/secrets-store-csi-driver-provider-azure/docs/getting-started/usage/

Ce qu'il est important de savoir a propos de cet objet :
* Par défaut, permet uniquement d'être **utilisé comme volume**
* Il peut également **créer un objet Secret Kubernetes**
* Pour créer un Secret, il doit **obligatoirement être utilisé comme volume par au moins 1 pod**.


```yaml
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: # Au choix
  namespace: # Votre Namespace
spec:
  provider: azure
  secretObjects: # Contient les informations sur les secrets Kubernetes à créer
  - secretName: # Nom du secret qui sera créé
    type: Opaque # Type du secret
    data:
    - objectName: # Nom de l'objet KeyVault a utiliser
      key: # Nom du champ dans le secret qui sera créé
  parameters:
    keyvaultName: # Nom de votre keyvault
    useVMManagedIdentity: "true" # L'identité pour les secrets est liée aux VMs
    userAssignedIdentityID: #Le client-id de votre identité managée
    objects: |
      array:
        - |
          objectName: # Nom du secret dans le keyvault
          objectType: secret
    tenantID: # Tenant ID
```

## 6 - Montez l'objet créé sur le pod MySQL

Ajoutez dans votre Deployment un **volume** qui va référencer votre **SecretProviderClass**. La syntaxe pour ce type de volume est :

```yaml
volumes:
- name: # Nom du volume
  csi:
    driver: secrets-store.csi.k8s.io
    readOnly: true
    volumeAttributes:
      secretProviderClass: # Nom de votre SecretProviderClass
```

Montez ce volume dans le Pod **MySQL**, sur le chemin */mnt/secrets-store*.

Le fait d'avoir monté un volume a parti de cette SecretProviderClass **a créé un secret Kubernetes**, vérifiez :


```bash
kubectl get secrets -n <mon-namespace>
```

## 7 - Utilisez le nouveau secret dans les variables d'environnement Wordpress et MySQL

Utilisez le nouveau secret dans les variables d'environnement Wordpress et MySQL, pour remplacer les valeurs écrites en dur. La syntaxe pour utilise un secret dans une variable d'environnement est :

```yaml
env:
- name: # Nom de la variable
  valueFrom:
    secretKeyRef:
      name: # Nom du secret
      key: # Clé du secret (voir 'key' dans le SecretProviderClass)
```