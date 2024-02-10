# ConfigMaps

ConfigMaps are used to pass key value pairs of configuration data to the Pod.  
The Config Map is injected into the Pod, and the environment variables are setup.  

## Steps

1. Setup the ConfigMap
2. Inject into the Pod

## Create the ConfigMap

### Imperative

```
kubectl create configmap \
  app-config --from-literal=APP_COLOR=blue \
             --from-literal=ANOTHER_APP_COLOR=blue
```

**--from-literal** is used to specify the key value pairs. **--from-file** would be used to provide the key value pairs from a file:

```
kubectl create configmap \
  <config-name> --from-file=<path-to-file>

kubectl create configmap \
  app-config --from-file=app_config.properties
```

### Declarative
`kubectl create -f <configmap>.yaml`

Notice that spec is not used, rather the **data** property.
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: webapp-config-map
data:
  APP_COLOR: blue
  APP_MODE: prod
```

### Other commands
```
kubectl get configmaps
kubectl describe configmap <name>
```

## Using ConfigMap in Container - whole file
The **envFrom** property is used, this is a list, so many configMapRefs can be passed, all the key value pairs in the config map will get added as Environment Variables.

```
envFrom:  
  - configMapRef:  
      name: <config-map-name>
```

### Full Example
```
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
spec:
  containers:
  - name: simple-webapp-container
    image: some-image
    ports:
    - containerPort: 8080
    envFrom:
    - configMapRef:
      name: webapp-config-map
```

## Using ConfigMap in Container - specific values
```
spec:  
  containers:  
  - env:  
    - name: APP_COLOR  
      valueFrom:  
        configMapKeyRef:  
          name: webapp-config-map  
          key: APP_COLOR  
    image: kodekloud/webapp-color
```

### Full Example
```
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
spec:
  containers:
  - name: simple-webapp-container
    image: some-image
    ports:
      - containerPort: 8080
    env:
    - name: APP_COLOR
      valueFrom:
        configMapKeyRef:
          name: webapp-config-map
          key: APP_COLOR
```

## Mount ConfigMap from Volume
The Config Map data can be mounted from a Volume, if done this way, each key-value pair in the config map gets it's own unique file created into the volume directory, the file name is the *key*, the content of the file is the *value*.

### Example - Deployment with mounted volume
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-3
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app-3
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: my-app-3
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: nginx
        volumeMounts:
        - name: config-volume
          mountPath: /etc/config
      volumes:
        - name: config-volume
          configMap:
            name: my-app-config
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
```
### Example - Config Map
```
apiVersion: v1
data:
  key1: config1
  key2: config2
kind: ConfigMap
metadata:
  name: my-app-config
```

### Example Directory Structure
```
# cd /etc/config
# ls
key1  key2
# cat key1
config1
# cat key2
config2
#
```
