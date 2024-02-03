# API Versions

Resources in Kubernetes are split into the the core group /api and named groups /apis.
Each API Group has different versions, e.g. for named groups:
- /apis
- /apps/v1
- /extensions/v1
- /networking.k8s.io/v1
- etc

Commands to see all resource types and API version:
```
kubectl api-resources
kubectl api-versions
```

## Versions
- /v1 - the stable version
- /v1alpha1 - when the api is first released it is released as alpha, it is not enabled by default, it may have bugs, and no guarantee it is going to be available in future releases, it may be dropped.
- /v1beta1 - after alpha stage and testing, it is moved to beta, this is enabled by default, for beta testing, and for users to provide feedback, it may still have bugs.
- /ga - this is stable version, e.g. v1, highly reliable and present in many future releases and available to all users

An API group can support multiple groups at the same time, you can create the resource with any API, but only 1 can be the **preferred version**.  e.g., running `kubectl get deployments` will only get the preferred version.  To see the preferred version run `kubectl explain <resource>, e.g., kubectl explain deployment`.  

The **Storage Version** is when you create a resource with one of the non-preferred versions, e.g. v1beta1, it will be stored in etcd with the storage version, e.g. v1. Only one version is stored as the storage version.

The preferred version will be listed if you go to the REST API of the resource, e.g. <clusterIP>:<port>/apis/batch (will show the preferred version for the **batch** resource). Cannot currently see the Storage Version. You would have to query the ectd database itself.

### Get the preferred version
1. Proxy to the kubernetes REST API  
2. Send API request to `localhost:<port>/apis/<api-group>`, in the example below, the api-group is for the authorization.k8s.io group.

```
kubectl proxy 8001&

curl localhost:8001/apis/authorization.k8s.io
```

## Enabling API alpha version
The version must be added to the kube-apiserver config yaml (/etc/kubernetes/manifest/kube-apiserver.yaml), e.g. below would enable the v2alpha1 version for the batch resource.
```
--runtime-config=batch/v2alpha1\\
```

## API Deprecations
### Deprecation Policy
- **API elements may only be removed by incrementing the version of the API group.** e.g, functionality will continue to be present in /v1alpha1 version of the resource, but can be removed in /v1alpha2. Users can still create the resource using the /v1alpha1 version, but internally it will be converted and stored as /v1alpha2.
- **API objects must be able to round-trip between API versions in a given release without information loss, with the exception of whole REST resources that do not exist in some versions.** E.g. resource is created in /v1alpha1, but converted to /v1alpha2 for storage it may change a field. Converting back to /v1alpha1 must have the original /v1alpha1 configuration.
- **Other than the most recent API versions in each track, older API versions must be supported after their announced deprecation for a duration of no less than:** 
  - **GA: 12 months or 3 releases (whichever is longer)**
  - **Beta: 9 months or 3 releases (whichever is longer)**
  - **Alpha: 0 releases**
This might potentially break things for users, so the release notes for each Kubernetes release version will detail breaking changes.
- **The "preferred" API version and the "storage version" for a given group may not advance until after a release has been made that supports both the new version and the previous version.** 

## Kubectl Convert
When an old API is removed, the yaml definitions need to be updated to the new versions. To convert yaml files from the old to new versions, you can use the kubectl convert function.

```
kubectl convert -f <old-file> --output-version <new-api>
kubectl convert -f nginx-pod.yaml --output-version apps/v1
```

Need to install the kubectl convert command as a separate plugin, not part of kubectl by default.
