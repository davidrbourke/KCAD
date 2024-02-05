# Exam Tips

1. Use imperative commands (e.g. run) to create a resource quickly, then you can edit the generate yaml to save time having to type out the entire yaml
2. Use --dry-run=client to test your command/file
3. -o yaml when creating a resource to see what has been created
4. Use the above in combination to generate a file, e.g.
  `kubectl run mypod --image=nginx --namespace=dev --dry-run -o yaml > pod-def.yml`


# Imperative commnads
Create a pod
`kubectl run redis --image=redis:alpine`

Generate the yaml to a file
`kubectl run redis --image=redis:alpine --dry-run -o yaml > a3.yml`

Create a service
`kubectl expose pod redis --port=6379 --name=redis-service`

**Use --help to check parameters for a command**
`kubectl run --help`

`kubectl create deployment webapp --image=kodekloud/webapp-color --replicas=3`
`kubectl run custom-nginx --image=nginx --port=8080`
`kubectl create namespace dev-ns`
`kubectl create deployment redis-deploy --replicas=2 --namespace=dev-ns --image=redis`

Create a pod and expose a service
`kubectl run httpd --image=httpd:alpine --port=80`
`kubectl expose pod httpd --name httpd --port=80 --type=ClusterIP --dry-run=client -o yaml`
Or in one command
`kubectl run httpd --image=httpd:alpine --port=80 --expose=true`

## Count of line:
Use pipe wc -l  
This count will include any headers so -1  
```
kubectl get clusterroles | wc -l
```

## Identify the OS
```
cat /etc/os-release
cat /etc/*release*
```

