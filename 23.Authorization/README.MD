# Authorization

## Authorisation
What can they do?
- Role Based Authorization (RBAC)
- ABAC - Atribute Based Access Control
- Node Authorization
- Webhook Mode

Communication between the various components of Kubernetes, e.g., Controller, Api Server, Scheduler, Kubelet, etc, is secured by TLS Encryption.  
Communication between Pods is restricted using Network Policies.  

## API Groups
There are 2 API groups in Kubernetes.

- /api /v1 core groups
- /apis named group APIs, going forward all new features will be in named groups.
  /apps
  /extensions
  /networking.k8s.io
  /storage.k8s.io
  etc.

Kubernetes API reference page can tell you the APIs and verbs.

Can access resources with curl, returns a list of all available APIs (need to pass authenticate certificates):
```
curl http://<cluster>:8001 -k
```
Or using kubectl
```
kubectl proxy
```

Each API (/api/v1 or /apis/v1) has resources, and each resource has Verbs for actions on the resource. This pattern can be used to control access to various resources and verbs, e.g.:
- /apps/v1/deployments (list, get, create, delete, update, watch)
- /apps/v1/replicasets
- /apps/v1/statefulsets
- /networking.k8s.io/v1/networkpolicies
- etc

### Authorization in the Cluster
- Node Authorizer
  - This controls the access within the cluster for the Kubelet to read/write information regarding the Pod.
- ABAC - Attribute Based Access Control
  - A Policy definition can be created for each group or user
  - Everytime you need to change the Access Control on the cluster, you edit the Policies and restart the Server, so are difficult to manage
- RBAC - Role Based Access Control
  - Define a Role, e.g., for Developers, then associate the Developers to the Role.
  - A change to the Access Control requires changing the Role and its permissions are applied immediately
- Webhook
  - Outsource the Access Control to a 3rd party tool, e.g. Open Policy Agent
  - Kubernetes will make a call to the Agent to check whether access is granted
- AlwaysAllow
  - Allows any request
- AlwaysDeny
  - Denies all requests

Where to define the Access Control? In the kube-apiserver flags:

```
cat /etc/kubernetes/manifests/kube-apiserver.yaml

--authorization-mode=AlwaysAllow
```
Default is AlwaysAllow, add more with comma separate list:
```
--authorization-mode=Node,RBAC,Webhook
```
Requests are authorised in the order specified, when a module denies a request, it is passed to the next in the list. As soon as a module approves a request, no more checks are done and permission is granted.

### RBAC - Role Based Access Control
1. Ceate a role definition file object
2. Link a user to a role - create a role binding

```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list", "get", "create", "update", "delete"]
- apiGroups: [""]
  resources: ["ConfigMap"]
  verbs: ["create"]
```
If the apiGroups is empty [""], then the API is /api by default.

```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: devuser-developer-binding
subjects:
- kind: User
  name: dev-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
```

The roles scope to namespaces, to limit namespaces, specify the namespace in the metadata: namespace: <namespace-name>.   

```
kubectl get roles
kubectl get rolebindings
kubectl describe role <role-name>
kubectl describe rolebinding <role-binding-name>
```

### Check access
```
kubectl auth can-i <verb> <resource>
kubectl auth can-i create deployments
kubectl auth can-i delete nodes

# Check another users permissions - e.g., dev-user
kubectl auth can-i delete nodes --as dev-user
kubectl auth can-i delete nodes --as dev-user --namespace somenamespace
```

To limit a Rule to a specific resource, e.g. Pod, using resourceNames: [<podname>]
```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list", "get", "create", "update", "delete"]
  resourceNames: ["blue", "orange"]
```

### Imperative Commands
```
k create role developer --verb=list,create,delete --resource=pods
k create rolebinding dev-user-binding --user=dev-user --role=developer
```

## Cluster Roles

Nodes are cluster wide, they cannot be scoped to a namespace.  
Cluster wide resources are not assigned to a namespace.  

To see namespace or cluster resources:
```
kubectl api-resources --namespace=true
kubectl api-resources --namespace=false
```

Cluster Admin roles gives permission to view, create or delete Nodes in the cluster.  
Storage Admin roles gives permissions to view, create and delete Persistent Volumes.  

### Steps
1. Create a Cluster Role
2. Create a cluster role binding to link a user to the cluster role

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-administrator
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["list", "get", "create", "delete"]
```

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin-role-binding
subjects:
- kind: User
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io/v1
roleRef:
  kind: ClusterRole
  name: cluster-administrator
  apiGroup: rbac.authorization.k8s.io/v1
```

Cluster Roles can also be created for namespace resources, the user will have access to the resources across all namespaces.  
e.g. Giving access to all Pods across the cluster in all namespaces.  

### Imperative Commands
```
k create clusterrole developer --verb=list,create,delete --resource=nodes
k create clusterrolebinding dev-admin-binding --user=dev-user --clusterrole=developer

# * is access to all options
k create clusterrole developer --verb=* --resource=nodes
```
