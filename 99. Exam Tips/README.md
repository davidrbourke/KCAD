# Exam Tips

1. Use imperative commands (e.g. run) to create a resource quickly, then you can edit the generated yaml to save time having to type out the entire yaml
2. Use --dry-run=client to test your command/file
3. -o yaml when creating a resource to see what has been created
4. Use the above in combination to generate a file, e.g.
  `kubectl run mypod --image=nginx --namespace=dev --dry-run=client -o yaml > pod-def.yml`


# Imperative commnads
Create a pod
`kubectl run redis --image=redis:alpine`

Generate the yaml to a file
`kubectl run redis --image=redis:alpine --dry-run=client -o yaml > a3.yml`

Create a service
`kubectl expose pod redis --port=6379 --name=redis-service`

**Use --help to check parameters for a command**
```kubectl run --help

kubectl create deployment webapp --image=kodekloud/webapp-color --replicas=3
kubectl run custom-nginx --image=nginx --port=8080
kubectl create namespace dev-ns
kubectl create deployment redis-deploy --replicas=2 --namespace=dev-ns --image=redis
```

Create a pod and expose a service
```
kubectl run httpd --image=httpd:alpine --port=80
kubectl expose pod httpd --name httpd --port=80 --type=ClusterIP --dry-run=client -o yaml
```
Or in one command
`kubectl run httpd --image=httpd:alpine --port=80 --expose=true`  

Change the default namespace from default  
(Useful so you don't have to keep typing it)
```
 kubectl config set-context --current --namespace <namespace>
```

## Count of line:
Use pipe wc -l  
This count will include any headers so -1  
--no-headers will remove the column headers, so the header row won't get included in the line count.  
```
kubectl get clusterroles --no-headers | wc -l
```

## Identify the OS
```
cat /etc/os-release
cat /etc/*release*
```


## Time Management
2 hours & 19 questions. 

1. It is vital to attempt all the questions, do not get stuck early on in difficult questions, skip the tough ones to get through the easy ones first, then go back over the skipped ones.
2. Do not get stuck on any question, even on easy questions, if you can't make sense of the error, mark it for review and come back later if there is time. Troubleshooting issues is for after you have attempted all the questions. The goal is to get through ALL the EASY questions first.
3. Be really good with YAML. If you have to fix e.g, yaml indentations, etc, then you will not get through it, they don't have to look pretty. Only the end result is evaluated (e.g., the created object).
4. Use alias names and shortcuts, e.g. ns for namespace, etc.

Watch this: https://www.youtube.com/watch?v=rnemKrveZks


## Kube API Server YAML
/etc/kubernetes/manifest/kube-apiserver.yaml
