# Admission Control

Enforcing rules on the creation of Pods, e.g.:
- Only allow pods with image from certain repositories
- Run only as certain users
- Metadata always contains certain labels

Admission Controllers help to enforce how a cluster is used:
- Change the request itself
- Perform additional operations before the Pods is created
- Allow the Pod to start based on rules

## Prebuilt Controllers
- Always pulls images
- Default storage class, automatically adds PVC to any Pods that doesn't request any.
- Event rate limit, to prevent API server overload
- Namespace exists, won't create a Pod if the namespace doesn't exist (enabled by default)
- Namespace autoprovision controller, will automatically create the namespace if it doesn't exist (not default)


## Check which Controllers are enabled
Check the --enable-admission-plugins setting in /etc/kubernetes/manifests/kube-apiserver.yaml, or:
```
kube-apiserver -h | grep enable-admission-plugins

# If kube-apiserver is running on a Pod (e.g. not cloud hosted k8s):
ps -ef | grep kube-apiserver | grep admission-plugins
```

## To enable Controllers
Update the /etc/kubernetes/manifests/kube-apiserver.yaml `--enable-admission-plugins=<controller>` configuration.  
E.g., `--enable-admission-plugins=NodeRestriction,NamespaceAutoProvision`

## To disable Controllers
`--disable-admission-plugins=NodeRestriction=NodespaceAutoProvision`

**NamespaceAutoProvision (depricated) is now replace by NamespaceLifecycle admissions controller.**

The NamespaceLifecycle admission controller will make sure that requests to a non-existent namespace are rejected and that the default namespaces such as default, kube-system and kube-public cannot be deleted.

To get a list of the default Admission Controller enabled for a Pod, connect to the kube-apiserver-controlplane Pod:
```
kubectl exec -it kube-apiserver-controlplane -n kube-system -- kube-apiserver -h | grep 'enable-admission-plugins'
```

## Validating and Mutating Admission Controllers

The Namespace exists admission controller is a **validating** controller.  
DefaultStorageClass is a **mutating** controller, it will watch for a creating Pod and modifiy the request to add a default storage class to a request if it is missing from the Pod definition.  It can change or mutate the object before it is created.

The mutation controller runs first, as then the configuration can be validated after, including the mutated changes.  
If any admission controller rejects the request, the Pod creation is rejected.

### Custom Admission Controller

Types:
- Mutation admission webhook
- Validating admission webhook

Can point these controllers to our own server running code and logic, the webhook sends an admission review request to your server, and receives back an admission review response object. If the **allowed** field in the response is true, the admission is accepted, otherwise if false, it is rejected.  

Example JSON response:
```
{
  "apiVersion": "admission.k8s.io/v1",
  "kind": "AdmissionReview",
  "response": {
    "uid": "<value from request.uid>",
    "allowed": true,
    "patchType": "JSONPatch",
    "patch": "W3sib3AiOiAiYWRkIiwgInBhdGgiOiAiL3NwZWMvcmVwbGljYXMiLCAidmFsdWUiOiAzfV0="
  }
}
```

#### Steps to setup
1. Create the code for handling the request/response, e.g., in Go to handle the request and return a response. The response has to return the allowed field, and can return a  patch field (using Json patch) to alter the Pod/deployment configuration.
2. Deploy our own webhook server to run the handling request/response code, running on a server, or as a deployment/service in the Kubernetes cluster
3. Configure the cluster to call the service

```
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  name: "pod-policy.example.com"

webhooks:
- name: "pod.policy.example.com"
  clientConfig: 
    service:
      namespace: "webhook-namespace"
      name: "webhook-service"
    caBundle: "<certificate>"
  rules:
  - apiGroups: [""]
    apiVersions: ["v1"]
    operations: ["CREATE"]
    resources: ["pods"]
    scope: "Namespaced"
```

In the above, the webhooks would be called when a new Pod is created (specified in operations and resources).  

If the server is hosted external to the cluster:
```
clientConfig: 
    url: "http://external-server.com"
```
