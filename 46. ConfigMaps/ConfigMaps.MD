# ConfigMaps

ConfigMaps are used to pass key value pairs of configuration data to the Pod.
The Config Map is injected into the Pod, and the environment variables are setup.

1. Setup the ConfigMap
2. Inject into the Pod

## Create the ConfigMap
### Imperative
`kubectl create configmap \
  app-config --from-literal=APP_COLOR=blue \
             --from-literal=ANOTHER_APP_COLOR=blue`

**from-literal** is used to specify the key value pairs. **from-file** would be used to provide the key value pairs from a file:
`kubectl create configmap \
  <config-name> --from-file=<path-to-file>`

`kubectl create configmap \
  app-config --from-file=app_config.properties`

### Declarative
`kubectl create -f <configmap>.yaml`

See configmap.yaml.
Notice that spec is not used, rather the **data** property.

### Other commands
`kubectl get configmaps  
kubectl describe configmap <name>`

## Using ConfigMap in Container
The **envFrom** property is used, this is a list, so many configMapRef can be passed:
`envFrom:  
  - configMapRef:  
    name: <config-map-name>`

### Example loading environment varible into pod from config map
`spec:  
  containers:  
  - env:  
    - name: APP_COLOR  
      valueFrom:  
        configMapKeyRef:  
          name: webapp-config-map  
          key: APP_COLOR  
    image: kodekloud/webapp-color`

  