# Pods
A Kubernetes YAML definition file always contains required fields:
- apiVersion - version of kubernetes api used to create the object - must use right version, e.g. v1 (string value)
- kind - type object, e.g. Pod (string value)
- metadata - data about the object, e.g., name, labels in dictionary form
  - labels (also a dictionary), can add any key-value pairs, used to identify specific objects
- spec - (dictionary) information pertaining to the specific object/kind


## Yaml
Yaml can have types
- string
- list
- dictionary (object)


### Create the pod from a definition file
`kubectl create -f <filename.yml>`   

```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
    - name: nginx-container
      image: nginx
```



### Create a pod inline
```
kubectl run <podname> --image=<imagename>
kubectl run <podname> --image=<imagename> --dry-run=client -o yaml
```
### Get the pods
```
kubectl get pods
kubectl get pods -o wide
```
### Specific pod detials
`kubectl describe pod <podname>`
### Output pod definition into new yaml file
`kubectl get pod <pod-name> -o yaml > pod-definition.yaml`
### Edit a pod
Only certain properties can be edited:
- spec.containers[*].image
- spec.initContainers[*].image
- spec.activeDeadlineSeconds
- spec.tolerations
- spec.terminationGracePeriodSeconds

`kubectl edit pod <pod-name>`

