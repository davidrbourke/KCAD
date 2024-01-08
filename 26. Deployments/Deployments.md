# Deployments

Allows for production deployment, a high level object above ReplicaSet and Pods
- rolling updates
- rolling back changes to a previous deployment
- scaling
- modifying resource allocations to all pods

Similar to ReplicaSet file, but with different Kind: Deployment.


## Commands
`kubectl create -f deployment.yml`

`kubectl get deployments`

### Get all created resources
`kubectl get all`

### Create deployment without yaml
`kubectl create deployment httpd-frontend --image=httpd:2.4-alpine --replicas=3`
