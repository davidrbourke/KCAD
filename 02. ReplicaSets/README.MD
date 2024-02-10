# Replication Controllers
- To run multiple instances of a Pod, providing Availability, share load (Load Balancing)
- Automatically brings up a new Pod if the existing one fails.
- In the replication controller yaml file, the pod definition goes under the template section. Can be copied exactly from a pod definition file, but remove the apiVersion and Kind.
- Number of replicas is set in replicas property of the replication controller.

### Create replicas
```
kubectl create -f <rc-definition>.yaml
kubectl get replicationcontroller
```

```
apiVersion: v1
kind: ReplicationController
metadata:
  name: myapp-replication-controller
  labels:
    app: myapp
    type: front-end
spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx

replicas: 3
```

## Replication Controllers are replaced by **Replica Sets**
Differences in definition file:
```
apiVersion: apps/v1
kind: ReplicaSet
selector:
  matchLabels:
    type: front-end (label from pod definition)
```
Selector must be set for ReplicaSet as it can control pods created not with the ReplicaSet definition.

### Create replica set
```
kubectl create -f <rs-definition>.yaml
kubectl get replicaset
```

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replica-set
  labels:
    app: myapp
    type: front-end
spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx

replicas: 3
selector:
  matchLabels:
    type: front-end
```

### Scale up or down
A number of ways
1. Update the number of replicas in the definition file, and apply the update: kubectl replace -f <rs-definition>.yaml
2. `kubectl scale --replicas=6 <rs-definition>.yml`
3. `kubectl scale --replicas=6 replicaset <replica set name>`


# Labels and Selectors
Used by replica set to know which pods it is responsible for monitoring - a filter. Used in many ways in K8s.


`kubectl explain <object>`
e.g. kubectl explain replicaset

When updating a replica set the pods are not automatically updated, need to delete the pods, or scale down-up.
