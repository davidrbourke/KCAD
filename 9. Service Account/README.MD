# Service Accounts

## User account
Used by humans

## Service account
Used by machines
- a monitoring application (e.g. prometheus) to get metrics from the Kubernetes server API
- deployment pipeline (e.g. Jenkins) to deploy to the cluster
- a web UI querying the kubernetes API

### Create a service account
`kubectl create serviceaccount <sa-name>`

A token must also be created with the same name as the service account:
`kubectl create token <sa-name>`

The token, stored as a secret object is used by the service account for authentication. The secret object is linked to the service.

`kubectl get serviceaccount`

To see the token:
`kubectl describe secret <token-secret-name>`

The token can be used as a REST Bearer token in an authorisation header to the kubernetes API.

`curl https://192.168.56.70:6443/api -insecure --header "Authorization: Bearer <token>`

### Default Service
When a pod is created, the default service account token is mounted. This allows the pod to run basic queries against the kubernetes API, but its permissions are limited.

You can override the default service account with another service account, e.g., dashboard-sa:

```
apiVersion: v1
kind: Pod
metadata:
  name: my-kubernetes-dashboard
spec:
  containers:
    - name: my-kubernetes-dashboard
      image: my-kubernetes-dashboard
  serviceAccountName: dashboard-sa
```

Changes to the service account will recreate the deployment, restarting the pods.

Use this pod config to prevent mounting the default service account (not needed if overriding the default account):
`automountServiceAccountToken: false``

### Version changes
- 1.22, the default token associated with the pod (mounted in the pod) when the pod starts became **audience bound**, **object bound** and **time bound**. The TokenRequestAPI was introduced to handle generating the service account token.
- 1.24, when creating a service account, you now have to explicitly create the token, it doesn't happen automatically. Now you must run these commands:  
  - `kubectl create serviceaccount <sa-name>`  
  - `kubectl create token <sa-name>`  
You can also specify the expiry length of the token, otherwise the default is 1hr (e.g., for 10 minutes with argument --duration 10m)

The secret object is associated with the service account using an annotation in the secret:

```
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: mysecretname
  annotation:
    kubernetes.io/service-account.name: <sa-name>
```
So the service account must be created first before creating the token.
