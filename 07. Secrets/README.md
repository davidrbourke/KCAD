# Secrets

Configmaps store data in plaintext. Secrets are for **sensitive information**, they are stored encoded.

## Steps
1. Create the secret
2. Inject it

## Imperative

From-literal:  
*Note: generic keyword is a type of secret, there are different types.*
```
kubectl create secret generic <name> --from-literal=<key>=<value>
kubectl create secret generic db-creds --from-literal=DB_HOST=mysql --from-literal=DB_PASSWORD=pswrd
```

From-file:
```
kubectl create secret generic <name> --from-file=<path-to-file>
kubectl create secret generic --from-file=secrets.properties
```

Imperative command will automatically base64 encode the secret.

## Declarative
```
apiVersion: v1
kind: Secret
metadata:
  name: some-secrets
data:
  DB_Host: bXlzcWw=
  DB_User: cm9vdA==
  DB_Password: cHN3cmQ=
```

`kubectl apply -f <file-name>.yaml`

 Key value pairs in secret files must be base64 encoded.

### Encode on Linux
`echo -n 'mysql' | base64`
### Decode on Linux
`echo -n 'bXlzcWw=' | base64 --decode`

## Secret commands
```
kubectl get secrets
kubectl describe secrets
kubectl describe secrets <secret-name> -o yaml
```

## Using Secrets in a Container
The **envFrom** property is used, this is a list, so many secretRefs can be passed, all the key value pairs in the Secret will get added as Environment Variables.

```
envFrom:  
  - secretRef:
      name: <secret-name>
```

If mounting a file, each key-value pair in the file is mounted as its own file in the /opt/<secret-name>-volumes.  
**When I did this in the lab, it load the secrets to the Environment Variables, not files - need to double check this**

### Using Secrets in Container - specific values
```
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
2. Secrets are not encrypted in etcd, so consider enabling encryption-at-rest. There is an EncryptionConfiguration kind object to do this.
3. Anyone able to create pods/deployments in the Namespace can access those secrets in the Namespace.
4. Consider Role Based Access Control
5. Consider 3rd party secret store providers, e.g. Vault, or a cloud provider
6. A secret is only sent to a Node if a Pod on that Node requires it
7. Kubectl stores the secret into a tmpfs so that secret is not written to disk storage
8. Once a Pod that depends on a secret is deleted, kubelet will delete the local copy of the secret

## Lab
** Lab used envFrom to load secrets, but this has gone from the Kubernetes documents - is envFrom still valid in 1.28? **

## Encrypting Secret data at Rest

https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#:~:text=Data%20is%20encrypted%20when%20written,contents%20of%20your%20secret%20data.

Secrets stored in the configuration as base64 encoded, not encrypted by default.

Secrets in config will still be base64, but it can be encrypted on etcd server.

Need etcd-client installed to query etcd.
`apt-get install etcd-client`

### Check if encryption-at-rest is enabled 
On the pod:

`ps -auz | grep kube-api | grep "encryption-provider-config"`
or
`cat /etc/kubernetes/manifests/kube-apiserver.yaml`  
If enabled there will be a setting for --encryption-provider-config

### Set encryption at reset
1. Create a configuration file
   - specify which types of resources to encrypt, e.g., secrets
   - specify the encryption algorithm
   - remove the identity {} provider - this means no encryption
   - specify which keys the encryption algorithm will use
2. Pass in --encryption-provider-config setting
   - mount in the new encryption config file to the kube-apiserver static pod (have to use volume mount)
   - pass the mounted encryption config file to the --encryption-provider-config setting in the kube-apiserver startup command
kind: EncryptionConfiguration

After encryption is enabled, only new secrets will be encrypted, not existing secrets, so they need to be replaced to be encrypted.
