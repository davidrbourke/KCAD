# Namespaces
Namespaces provide isolation. A 'default' namespace is setup automatically.  
Some internal namespaces are setup by K8s also, e.g., 

```
NAME              STATUS   AGE
default           Active   28m
kube-node-lease   Active   28m
kube-public       Active   28m
kube-system       Active   28m
```

Resources quotas can be assigned to a namespace.  
Resources within a namespace can refer to each other by name.  
To reach a service in another namespace, the namespace must be added.  

e.g.,
- In same namespace:  
`mysql.connect('db-service')`
- In a differnet namespace called 'dev':  
`mysql.connect('db-service.dev.svc.cluster.local')`

## The parts of the service url
```
db-service - service
dev - namespace
svc - subdomain
cluster.local - domain
```

## Commands
`kubectl create -f pod-definition.yml --namespace=dev`  
or, put namespace in yaml file.

```
metadata:
  kind: myapp-pod
  namespace: dev
  label:
    app: myapp
    type: frontend  
```

### Create a namespace
Use a namespace definition file:  
`kubectl create -f namespace-dev.yml`  
Or with command:  
`kubectl create namespace dev`  

```
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

### Get objects within a namespace
```
kubectl get pods --namespace=dev
kubectl get pods -n=dev
kubectl get pods (Uses default namespace)
kubectl get pods --all-namespaces
kubectl get pods -A
```

### Set current context namespace
So you don't have to specify the namespace with each command:  
`kubectl config set-context $(kubectl config current-context) --namespace=dev`  

## Resource Quota
A Resource Quota sets cpu and memory limits for the sum of all the resources in the namespace.  

```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "2"
    requests.memory: "1Gi"
    limits.cpu: "4"
    limits.memory: "2Gi"
```