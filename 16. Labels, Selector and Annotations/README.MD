# Labels, Selectors and Annotations

Labels and Selectors are a way to group things together, e.g., a class or kind. You want to be able to filter them on criteria or multiple criteria.

- **Labels** are properties attached to each item.
- **Selectors** help you filter these items.

In Kubernetes a lot of different objects are created, Pods, Services, Deployments.  
You can group and select objects using labels and selectors in different ways, e.g. App, Function, etc.

Specify labels in the Definition file under metadata, labels are case-sensitive:

```
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    app: App1
    function: Front-End
spec:
  containers:
    - name: simple-webapp
      image: simple-webapp
```

Select pods with the labels:  
`kubectl get pods --selector app=App1`

Multiple selectors:  
`kubectl get all --selector bu=finance,env=prod,tier=frontend`

Kubernetes uses labels and selectors internally, e.g. creating a replica set.
- The Labels configured in the **template** section are the labels for the Pods.
- The Labels configured in the ReplicaSet metadata are the labels for the ReplicaSet.

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: simple-webapp
  labels:
    app: App1
    type: Front-End
spec:
  replicas: 3
  selector:
    matchLabels:
      app: App1
  template:
    metadata:
      labels:
        app: App1
        type: Front-End
    spec:
      containers:
        - name: simple-webapp
          image: simple-webapp
```

To get the ReplicaSet to discover the Pods, configure the **selector** under the ReplicaSet **spec** to match the labels on the Pod.
You can specify more labels to match to be more specific.

Another example, is the relationship between a Service and Pod. The Service has a selector to identify the Pods it is a Service for:

```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: App1
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
```

## Annotations
Used to record other details for information purposes. E.g. Build Version, contact details, etc.

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: simple-webapp
  labels:
    app: App1
    type: Front-End
  annotations:
    buildversion: 1.34
spec:
  replicas: 3
  selector:
    matchLabels:
      app: App1
  template:
    metadata:
      labels:
        app: App1
        type: Front-End
    spec:
      containers:
        - name: simple-webapp
          image: simple-webapp
```
