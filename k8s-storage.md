# K8s - Storage

### Introduction

### Docker Storage
1. Storage Drivers
2. Volume Drivers

#### Storage in Docker
File System -> var/lib/docker (aufs, containers, image and volume)

R/W Layer
Copy on write

Persistent volume (volume/data_volume)

Volume mounting
`docker run -v data_volume:/var/lib/mysql mysql`

Bind Mounting <br>
Mounting any directory to volume
`docker run -v /data/mysql:/var/lib/mysql mysql`

`--mount`

Storage drivers enables all the files writing and moving around.. 


Volume Drivers <br>
Volumes are handled by volume drivers such as local, afs, NetApp ...Vm Ware 


Volume Drivers 

#### CSI

1. CRI Supports different runtime. 
2. CNI 
3. CSI -> Standard

### <u>K8s Storage</u>

#### Volume 
you have to specify volume at pod level
```

```
#### Persistent Volume 
Create volume at cluster-wide level, so that in pod-spec or other spec, you can claim the volume. 


Access modes <br>
1. ReadOnlyMany
2. ReadWriteOnce
3. ReadWriteMany


```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0003
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  mountOptions:
    - hard
    - nfsvers=4.1
  hostPath:
    path: /tmp
```

with nfs 

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0003
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: slow
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /tmp
    server: 172.17.0.2
```


#### Persistent Volume Claim
To make the storage available to the node.
Single persistent volume

1:1 volume to volume claim

Whenever the claim is deleted then, the underlying volume has the following options. 

PersistentVolumeReclaimPolicy:
`Delete` `Recycle` `Retain`

##### <b> Using PVCs in POD</b> #####
```
apiVersion: v1
kind: Pod
metadata:
    name: mypod
spec:
    containers:
    - name: myfrontend
        image: nginx
        volumeMounts:
        - mountPath: "/var/www/html"
        name: mypd
    volumes:
    - name: mypd
        persistentVolumeClaim:
        claimName: myclaim
```

The same is true for ReplicaSets or Deployments. Add this to the pod template section of a Deployment on ReplicaSet.


Reference URL: https://kubernetes.io/docs/concepts/storage/persistent-volumes/#claims-as-volumes


```
    volumeMounts:
    - mountPath: /log
      name: log-volume

  volumes:
  - name: log-volume
    hostPath:
      # directory location on host
      path: /var/log/webapp
      # this field is optional
      type: Directory
```

??? Can you create volume imperatively ???
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-log
spec:
  accessModes:
    - ReadWriteMany
  capacity:
    storage: 100Mi
  hostPath: /pv/log
  ---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: claim-log-1
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Mi
 ---
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - name: event-simulator
    image: kodekloud/event-simulator
    env:
    - name: LOG_HANDLERS
      value: file
    volumeMounts:
    - mountPath: /log
      name: log-volume

  volumes:
  - name: log-volume
    persistentVolumeClaim:
      claimName: claim-log-1
  ```
#### Storage Class

1. Static 
2. Dynamic

You dont have to create a volume claim defn separately, the pvc will automatically claim the pv. (In GCP)

```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: local-pvc
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: local-storage
  resources:
    requests:
      storage: 500Mi
```

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    volumeMounts:
      - name: local-persistent-storage
        mountPath: /var/www/html
  volumes:
    - name: local-persistent-storage
      persistentVolumeClaim:
        claimName: local-pvc
```

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: delayed-volume-sc
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```