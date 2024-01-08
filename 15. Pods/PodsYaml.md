# Pods
Kubernetes YAML definition file always contains required fields:
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


### Create the pod
`kubectl create -f <filename.yml>`
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
-spec.terminationGracePeriodSeconds

`kubectl edit pod <pod-name>`

# Replication Controllers
To run multiple instances of a Pod, providing Availability, share load (Load Balancing)
Automatically brings up a new Pod if the existing one fails.

In the replication controller yaml file, the pod definition goes under the template section. Can be copied exactly from a pod definition file, but remove the apiVersion and Kind.
Number of replicas is set in replicas property of the replication controller.

### Create replicas
```
kubectl create -f <rc-definition>.yaml
kubectl get replicationcontroller
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
