# Deployments

Allows for production deployment, a high level object above ReplicaSet and Pods. Features:
- rolling updates
- rolling back changes to a previous deployment
- scaling
- modifying resource allocations to all pods

Similar to ReplicaSet file, but with different Kind: Deployment.

## Commands
`kubectl create -f deployment.yml`

`kubectl get deployments`

### Get all created resources
`kubectl get all`

### Create deployment without yaml
`kubectl create deployment httpd-frontend --image=httpd:2.4-alpine --replicas=3`

## Updates and Rollbacks
A new deployment rollout creates a new deployment revision. This keeps track of the changes made to our deployment.

See the rollout status of the status:  
`kubectl rollout status deployment/<deployment-name>`

See the rollout history:  
`kubectl rollout history deployment/<deployment-name>`  

To see a specific revision:  
`kubectl rollout history deployment/<deployment-name> --revision=1`

To do a deployment and record the change-case:  
`kubectl apply -f <deployment-definition>.yaml --record`


## Deployment Strategy
1. **Recreate** - Destroy all the running instances, then create new instances. During the period before re-deployment, the application is inaccessible.
2. **RollingUpdate** - Default - take down an old version and bring up a new version one by one, the deployment is seemless, the application is always available.

A new deployment can be done through updating the deployment definition file.  
Or, updating the image directly, but this creates a difference between your configuration and the deployment.  
`kubectl set image deployment/<deployment-name> <container-name>=<new-image-name:version> --record`   

**--record** flag is used to record the cause of the change for visibility in the rollout history.  

The deployment event history will show which strategy was used to deploy.  

For each deployment, a new ReplicaSet is created.  

## Rollback  
A rollback will rollback to the previous replica set:    
`kubectl rollout undo deployment/<deployment-name>`  

To rollback to a specific revision:  
`kubectl rollout undo deployment/<deployment-name> --to-revision=1`  

## Blue/Green and Cannary Deployment

### Blue/Green
Both new and old version are deployed, 100% of traffic routes to Blue/old version.  
Tests can be run against the Green/new version.  
Traffic can be switched to the Green/new deployment.  

The label on the Blue deployment service, e.g. version: v1, matches the label on the Blue Deployment.  
The Green deployment is deployed, and the Green service label points to the Green deployment labels, e.g. version: v2.  
To switch the routing of the traffic, the Blue deployment service label is updated to version: v2, and traffic switches to the Green/v2 deployment Pods.  

#### Blue Deployment

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-blue
  labels:
    app: myapp
    type: frontend
spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        version: v1
    spec:
      containers:
        - name: my-nginx
          image: nginx
  replicas: 3
  selector:
      matchLabels:
        version: v1
```

#### Service pointing to V1/Blue

```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    version: v1
```

#### Green Deployment
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-green
  labels:
    app: myapp
    type: frontend
spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        version: v2
    spec:
      containers:
        - name: my-nginx
          image: nginx
  replicas: 3
  selector:
      matchLabels:
        version: v2
```

#### Update Service to point to V2/Green

```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    version: v2
```

### Canary Update
Deploy a new version and route only a small percentage of traffic to the new version.  
Once the testing is complete, route all traffic to the new deployment.  

1. Deploy the version 2 Deployment
2. Setup a common label from the Service to both the new and old deployments, e.g. app: front-end
3. Reduce the number of Pods in the new/v2 deployment, e.g.:
  - 5 Pods in v1 deployment, 5 pods in v2 deployment - will split load 50/50
  - 5 Pods in v1 deployment, **1** pod in v2 deployment - will split load 83/17 - 17% of traffic is going to the new deployment
4. Once tests are complete, increase the Pods in the v2 deployment, reduce the pods in the v1 deployment down to zero

A Service Mesh like Istio is more specific about Canary deployments, e.g. you could route 1% of traffic.

## Stateful Sets

Similar to Deployments, it creates Pods, scale-up/down. But Pods are deployed in a **sequential order**, the next Pod won't start up until the current one is running. Each Pod gets a unique Pod name based on the index of the start sequence, there are no random Pod names. E.g., `<pod-name>-0`, `<pod-name>-1`, etc.

A scenario where you would need this, if running a database, e.g. Mongo/mySQL with a master-slave configuration. You want the master Pod to start up first, the first slave Pod next, which will clone from master, the second slave next which will clone from slave-1. You also need consistent Pod names (not random suffix), so that you know that master will be `<pod-name>-0` always.

```
apiVersion: apps/v1
kind: StatefulSets
metadata:
  name: mysql
  labels:
    app: mysql
spec:
  template:
    metadata:
      name: mysql
    spec:
      containers:
        - name: mysql
          image: mysql
  replicas: 3
  selector:
      matchLabels:
        app: mysql
  serviceName: mysql-h
```

### Notes
- **kind** is **StatefulSet**
- **serviceName** must be specified with a headless service name
- to override sequential start of the Pods, use **podManagementPolicy: Parallel**

### Headless service
To access Pods with a normal Deployment that is scaled up, we would create a Service. Other applications can access the scaled up Pods using the Service, which acts as a Load Balancer, and directs traffic to one of the Pods.
In the Stateless service example of using a scaled database across multiple Pods, we need to make the MySQL Pods accessible to the applications in the cluster. However, writes should go to only the master Pod (`<pod-name>-0`), reads could come from any Pods. If we use a standard Service, writes would be Load Balanced to any Pod in the Stateful Sets. A Headless Service allows control of this, to write to only the master Pod. It creates DNS entries for each Pod with the Podname and the index sequence, this allows references to specific Pods from our applications. E.g.:

```
<pod-name>-<index>.<headless-svc-name>.<namespace>.svc.cluster.local
mysql-0.mysql-h.default.svc.cluster.local
mysql-1.mysql-h.default.svc.cluster.local
mysql-2.mysql-h.default.svc.cluster.local
```

```
apiVersion: v1
kind: Service
metadata:
  name: mysql-h
spec:
  ports:
    - port: 3306
  selector:
    app: mysql
  clusterIP: None
```

Setting **clusterIP: None** is the setting that creates a headless service.  

If using a Deployment, to register multiple DNS A Name records for each Pod in the Deployment, you must then specificy the sub domain and host name to the name of the Service.

```
subdomain: mysql-h
hostname: mysql-pod
```

However in the above scenario, all Pods would get the same A Name record. A Stateful doesn't require you to do this, it automatically creates the DNS A Name records with the unique name for each pod: `<pod-name>-0`, `<pod-name>-1`, etc. That is why you must specify the serviceName in the Stateful: to link the Pods in the Stateful set to the Service Name, so the headless service creates the unqiue DNS entries for each Pod.

### Storage in Stateful Sets
By default with Storage Volumes in a Deployment, all pods would use the same Persistent Volume. For Stateful sets, you might want each Pod to have unique storage. Each Pod needs its own Persistent Volume Claim.

Use a Volume Claim Template - simply you would move a standard Persistent Volume Claim file yaml into a section called **volumeClaimTemplates** within the Stateful Set definition:

```
apiVersion: apps/v1
kind: StatefulSets
metadata:
  name: mysql
  labels:
    app: mysql
spec:
  template:
    metadata:
      name: mysql
    spec:
      containers:
        - name: mysql
          image: mysql
          volumeMounts:
            - mountPath: /var/lib/mysql
              name: data-volume
  replicas: 3
  selector:
      matchLabels:
        app: mysql
  serviceName: mysql-h
  volumeClaimTemplates:
  - metadata:
      name: data-volume
    spec:
      accessModes:
        - ReadWriteOnce
      storageClassName: google-storage
      resources:
        requests:
          strorage: 500Mi
```

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: google-storage
provisioner: kubernetes.io/gce-pd
```

When a Pod is restarted or recreated, it is attached to the same Storage Class as before, so it provides stable storage.
