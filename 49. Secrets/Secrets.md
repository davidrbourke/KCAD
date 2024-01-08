# Secrets

Configmap stores data in plaintext. 
Secrets are for sensitive information, they are stored encoded.

## Steps
1. Create the secret
2. Inject it

## Imperative

From-literal:
`kubectl create secret <name> --from-literal=<key>=<value>`
`kubectl create secret generic --from-literal=DB_HOST=mysql --from-literal=DB_PASSWORD=pswrd`

From-file:
`kubectl create secret <name> --from-file=<path-to-file>`
`kubectl create secret generic --from-file=secrets.properties`

## Declarative
(See file secret-definition.yaml)

`kubectl apply -f <file-name>.yaml`

Key value pairs in secret files must be base64 encoded.

### Encode on Linux
`echo -n 'mysql' | base64`
### Decode on Linux
`echo -n 'bXlzcWw=' | base64 --decode`

## Secret commands
kubectl get secrets
kubectl describe secrets
kubectl describe secrets <secret-name> -o yaml

## Using Secrets in a Container
The **envFrom** property is used, this is a list, so many secretRef can be passed, all the key value pairs in the Secret will get added as ENVironment Variables.

```
envFrom:  
  - secretRef:
    name: <secret-name>
```

If mounting a file, each key-value pair in the file is mounted as its own file in the /opt/<secret-name>-volumes


### Using ConfigMap in Container - specific values
````
spec:  
  containers:  
  - env:  
    - name: DB_Password
      valueFrom:  
        secretKeyRef:  
          name: app-secret
          key: DB_Password
```

## Note
1. Secrets are not encrypted, only encoded, so don't check secrets into code repo.
2. Secrets are not encrypted in etcd, so consider enabled encryption-at-rest. There is an EncryptionConfiguration kind object to do this.
3. Anyone able to creates pods/deployments in the Namespace can access those secrets in the Namespace.
4. Consider Role Base Access Control
5. Consider 3rd party secret store providers, e.g. Vault, or cloud provider
6. A secret is only sent to a Node if a Pod on that Node requires it
7. Kubectl stores the secret into a tmpfs so that secret is not written to disk storage
8. Once a Pod that depends on a secret is deleted, kubelet will delete the local copy of there secret
