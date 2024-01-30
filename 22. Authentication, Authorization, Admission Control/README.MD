# Authentication, Authorization, Admission Control

## Authentication
Who can access the API Server? Methods for authentication:
- Files
  - Usernames & passwords
  - Username & token
- Certificates
- LDAP (External Providers)
- Service Accounts

Users accessing the cluster; Admins, Developers, End Users, Bots (3rd party services).  
Focusing on Admins and Developers and Bots (Service Accounts).  
Users are not created in a cluster. Only Service Accounts can be created within the Clusters.  

All user access is managed by the API Server. Authentication mechanisms:
 - Static Password or Token file
 - Certificates
 - 3rd party protocols

 ### Static Files
 These are basic approaches, not recommended for production environments (depricated in v1.19 of Kuberentes).  
 Create a list of users in a csv file, with columns: password, user, userid, group (optional). Pass the filename as option to the kube api server:
 ```
 --basic-auth-file=user-details.csv
 ```
Restart the server for the options to take effect.

Authenticate the user using the REST Api:
```
curl -v -k https://master-node-ip:6443/api/v1/pods -u "user1:password123"
```

For using a token instead of a password in the .csv file:
columns: token, user, userid, group (optional)
```
--token-auth-file=user-token-details.csv
```

Authenticate the user using the REST Api:
```
curl -v -k https://master-node-ip:6443/api/v1/pods --header "Authorization: Bearer <token>
```

If using Kubeadm, the csv files can be loaded via Volume Mounts.

## Using a Certificate
Using REST:
```
curl https://<clustername>:6443/api/v1/pods \
  --key admin.key
  --cert admin.crt
  --cacert ca.crt
```

Using Kubectl:
```
kubectl get pods
  --server <clustername>:6443
  --client-key admin.key
  --client-certificate admin.crt
  --certificate-authority ca.crt
```

Move this configuration to kube config file, and specify the file options in the kube command. Default looks in $HOME/.kube/config, so don't need to specify the path to the file.

### KubeConfig Format
- Clusters
  - The clusters you need access to, e.g., dev, test, prod
  - This has the --server specification from the Kubectl command above.
- Users
  - User accounts you use to access the cluster, users may have different priviledges
  - This has the --client-key, --client-certificate, --certificate-authority from the Kubectl command above.
- Contexts 
  - Which Users to use for which Clusters

This file uses existing users with existing priviledges to access the cluster, it doesn't configure any user accounts. 
All the sections are arrays, so the file can have many clusters, contexts and users.

```
apiVersion: v1
kind: Config

current-context: my-kube-admin@my-kube-playground

clusters:
- name: my-kube-playground
  cluster:
    certificate-authority: ca.crt
    server: https://my-kube-playground:6443

contexts:
- name: my-kube-admin@my-kube-playground
  context:
    cluster: my-kube-admin
    user: my-kube-admin
    namespace: <namespace-name> # Optional: will set the default namespace when you use the config.

users:
- name: my-kube-admin
  user:
    client-certificate: admin.crt
    client-key: admin.key
```

Set the default context: see the **current-context** section at the top. Also can use the command line:
```
kubectl config view
kubectl config view --kubeconfig=my-custom-confing # this is it not using the default config file $HOME/.kube/config.
kubectl config use-context <context-name>
kubectl get config current-context
```

Instead of the passing the path to the certificates in the config file, you can use the raw certificate data based64 encoded. Use **certificate-authority-data: <base-64-raw-cert>** instead of **certificate-authority: ca.crt**.

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

Each API (/api/v1 or /apis/v1) has resources, and each resources has Verbs for actions on the resource. This pattern can be used to control access to various resources and verbs, e.g.:
- /apps/v1/deployments (list, get, create, delete, update, watch)
- /apps/v1/replicasets
- /apps/v1/statefulsets
- /networking.k8s.io/v1/networkpolicies
- etc

### Authorization in the Cluster
- Node Authorizer
 - This controls the access within the cluster for the Kubelet to read/write information regarding the Pod.
- ABAC
  - A Policy definition can be created for each group or user
  - Everytime you need to change the Access Control on the cluster, you edit the Policies and restart the Server, so are difficult to manage
- RBAC
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

To limit to a Rule to a specific resource, e.g. Pod, using resourceNames: [<podname>]
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
