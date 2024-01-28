# Volumes

Docker containers are transient, they do not save data between restarts.  
Volumes are attached to persist data.  

This Pod writes a volume on the host's file system:
```
apiVersion: v1
type: Pod
metadata:
  name: mypod
spec:
  containers
  - image: alpine
    name: alpine
    command: ["/bin/sh", "-c"]
    args: ["shuf -i 0-100 -n 1 >> /opt/number.out;"]
    
    volumeMounts:
    - mountPath: /opt
      name: data-volume

  volumes:
  - name: data-volume
    hostPath:
      path: /data
      type: Directory
```

The Volume is created under spec.volumes[*].  
The Volume is then mounted to a directory in the container.  

## Volume Storage Options
1. Host Path - stores on a single Node, not across multiple nodes
2. Other supported:
  - NFS, AWS, etc

AWS Elastic volume example:
```
volumes:
- name: data-volume
  awsElasticBlockStore:
    volumeID: <volume-id>
    fsType: ext4
```

## Persistent Volumes
Persisent Volumes allow storage to be managed more centrally, without this, each Pod/Deployment must have its Storage Volumes configured.  
A Persistent Volume is cluster wide storage.  Users can select storage from the cluster using Persisent Volume Claims (PVC).  

PV has accessModes:
- ReadOnlyMany
- ReadWriteOnce
- ReadWriteMany

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol1
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 1Gi
  hostPath:
    path: /tmp/data
```

**hostPath** is not to be used in Production, use some other cloud storage provider for example.

## Persistent Volume Claims (PVC)
An Adminstrator creates the Persistent Volume, user applications request the Persistent Volume with a Claim.  
Every PVC is bound to a single PV, there is a 1-2-1 relationship between the PV and the PVC. This is matched on various properties, but labels and selectors can be used if there is a need to be more specific.  

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      strorage: 500Mi
  persistentVolumeReclaimPolicy: Retain
```


```
kubectl get persistentvolumeclaim
kubectl delete persistentvolumeclaim myclaim
```

By default, deleteing a PVC is set to Retain the PV until it is manually reclaimed by an Administrator. You can control the delete behaviour, to release the PV for reallocation automatically with the property: **persistentVolumeReclaimPolicy: Recyle**. Optios:
- Retain: Have to manually delete the PV.
- Delete: Deletes the PV when the PVC is delete.
- Recycyle: Releases the PV and deletes any data on it, the PV can be allocated to a new PVC.

## Using Persistent Volume Claims in Pods

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


## Storage Classes
If using cloud storage, you first have to provision the storage in the cloud. This is static provision. Storage classes allow automatic provisioning (dynamic provisioning). 

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: google-storage
provisioner: kubernetes.io/gce-pd
```

The Persistent Volume is no longer needed to be created manually, the Persistent Volume Claim has a **storageClassName** matching the Storage class, when the PVC is created, the storage class is associated with it, and will provision the disk in the cloud and bind the PVC to the Volume. A PV is still created, but it is created automatically by the Storage Class.

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: google-storage
  resources:
    requests:
      strorage: 500Mi
```

There are many other provisions for other clouds, e.g. AWS, Azure, etc.

### Volume Binding Modes
By default, when a PVC is created with a StorageClass, the PVC is Bound to the new PV immediately. There is an options to delay the binding until a Pod is using the PVC, this option is **volumeBindingMode: WaitForFirstCustomer**.  

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: google-storage
provisioner: kubernetes.io/gce-pd
volumeBindingMode: WaitForFirstCustomer
```
