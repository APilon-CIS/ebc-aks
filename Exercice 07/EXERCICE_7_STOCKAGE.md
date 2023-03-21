# Exercice 7 - Stockage persistant

Les containers sont par définition **volatiles**. Si un container redémarre, toutes les données sont perdues. Cela ne pose pas de problèmes pour des applications *stateless* (site web statique, proxy, serveur http etc...) mais pour de nombreux cas il est nécessaire de mettre en place un mécanisme de stockage persistant.

Dans notre cas, nous souhaitons mettre un stockage persistant à 2 niveaux :
* Sur la **base de données**, pour conserver les données stockées en base
* Sur **Wordpress**, pour conserver le contenu statique (images de blogs etc...)

Pour la base de donnée, chaque pod de basse de donnée doit disposer de son propre stockage. Si un déploiement en HA est envisagé, il faudra utiliser les mécanismes mysql pour former un cluster de base de données, qui assurera la réplication des données.

Pour Wordpress, il est nécessaire que chaque pod Wordpress partage un même espace de stockage, si un pod Wordpress enregistre une image par exemple, il faut que les autre Pods soient en capacité de retourner l'image au client.

Pour ces besoins, sur Kubernetes, on distingue 3 types d'"access modes" principaux :
- **ReadWriteOnce** : stockage en lecture / écriture pouvant être monté par un seul node. Par exemple, un stockage de type Disk va ajouter un disque à une VM, et ne sera accessible que sur cette VM.
- **ReadOnlyMany** : stockage en lecture seule pouvant être utilisé par plusieurs node. Par exemple, AzureFiles ou stockage NFS.
- **ReadWriteMany** : Lecture / écriture pouvant être utilisé par plusieurs node. Par exemple, AzureFiles ou stockage NFS.

## Quels objets pour le stockage sur Kubernetes

### PersistentVolume (PV)

Kubernetes délègue le stockage à différents backend, par exemple Azure Files, Azure Disk, stockage NFS, CephFS etc... Pour représenter ce stockage externe dans Kubernetes, on utilise des PersistentVolumes.

1. L'administrateur provisionne du stockage externe (par exemple création d'un blob container)
2. L'administrateur créé un objet PersistentVolume (PV) pour représenter ce stockage dans Kubernetes (nom, taille, access mode, etc...)
3. Les Pods vont pouvoir réquisitionner ce stockage

### PersistentVolumeClaim

Pour pouvoir utiliser un PersistentVolume, les Pods peuvent :
- Spécifier directement le nom du volume si il est connu
- Utiliser un objet PersistentVolumeClaim, qui contient les besoin en stockage du Pod

L'objet PersistentVolumeClaim contient ainsi la taille demandée par le Pod, les modes d’accès, éventuellement le type de stockage ou tout autre information qui peut être utile a Kubernetes pour assigner un stockage à un Pod.

Une fois cet objet créé, Kubernetes va assigner un PV disponible au PVC. Tant qu'aucun volume ne correspondant aux critères est disponible, les Pods restent au statut Pending.

### StorageClass

Pour éviter aux administrateurs de devoir provisionner manuellement des PV, Kubernetes propose les objets **StorageClass** : c'est un objet qui va contenir toutes les informations sur un pool de stockage pour permettre la création automatique de PV selon les besoins des PVC.

Aure propose plusieurs StorageClass par défaut (disk, files...). Pour utiliser ces StorageClass, un PVC à juste à renseigner le nom de la StorageClass. Il est également possible de créer ses propres StorageClass.

### Volumes et VolumesMounts

Pour utiliser les PV qui lui sont assignés par un PVC, un Pod va utiliser 2 attributs :
- **Volumes** : dans la definition du pod, on déclare une liste de Volumes qui seront utilisables par les containers. On leur donne un nom ainsi qu'une référence au PV ou PVC.
- **VolumeMounts** : dans la definition d'un container, on monte un des volumes déclarés au niveau du Pod en spécifiant le nom du volume, et le chemin du container dans lequel monter le volume.

Par exemple :

```yaml
  containers:
  - name: mon-container
    image: busybox
    volumeMounts:
    - name: mon-volume
      mountPath: /home # Le dossier /home du container sera sauvegardé dans le PV récupéré par mon-volume
  volumes: # Liste des volumes disponibles pour les pods
  - name: mon-volume
    persistentVolumeClaim: # On référence ici un PVC
      claimName: mon-pvc
```

## Exercice : Ajoutez du stockage persistant pour Wordpress et MySQL

> Attention, une fois le stockage ajouté sur Wordpress, le redémarrage peut être long

Pour Wordpress :
* Utiliser la storageClass **azurefile-csi-premium**
* Faites persister le chemin **/var/www/html** du container Wordpress

Pour MySQL :
* Utilisez la storageClass **managed-csi**
* Faites persister le chemin **/var/lib/mysql**

Utilisez la documentation Kubernetes pour savoir quels champs remplir
* PVC : https://kubernetes.io/docs/reference/kubernetes-api/config-and-storage-resources/persistent-volume-claim-v1/
* Volumes / VolumeMounts : https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/pod-v1/#volumes-1
