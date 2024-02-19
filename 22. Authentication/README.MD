# Authentication

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
  - This has the --server specification and --certificate-authority from the Kubectl command above.
- Users
  - User accounts you use to access the cluster, users may have different priviledges
  - This has the --client-key and --client-certificate from the Kubectl command above.
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
kubectl config view --kubeconfig=my-custom-config # this is it not using the default config file $HOME/.kube/config.
kubectl config use-context <context-name>
kubectl get config current-context
```

Instead of the passing the path to the certificates in the config file, you can use the raw certificate data based64 encoded. Use **certificate-authority-data: <base-64-raw-cert>** instead of **certificate-authority: ca.crt**.
