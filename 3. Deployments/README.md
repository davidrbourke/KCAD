# Deployments

Allows for production deployment, a high level object above ReplicaSet and Pods
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

A Service Mesh like Istio are more specific about Canary deployments, e.g. you could route 1% of traffic.

