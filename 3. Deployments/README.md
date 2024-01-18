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

## Updates and Rollbacks
A new deployment rollout creates a new deployment revision. This keeps track of the changes made to our deployment.

See the rollout status of the status:  
`kubectl rollout status deployment/<deployment-name>`

See the rollout history:
`kubectl rollout history deployment/<deployment-name>`  

To see a specific revision:  
`kubectl rollout history deployment/<deployment-name> --revision=1`

To do a deployment and record the change-case:  
`kubectl apply -f <deployment-definition>.yaml --record`


## Deployment Strategy
1. **Recreate** - Destroy all the running instances, then create new instances. During the period before re-deployment, the application is unaccessible.
2. **RollingUpdate** - Default - take down an old version and bring up a new version one by one, the deployment is seemless, the application is always available.

A new deployment can be done through updating the deployment definition file.  
Or, updating the image directly, but this creates a difference between your configuration and the deployment.  
`kubectl set image deployment/<deployment-name> <container-name>=<new-image-name:version> --record`   

**--record** flag is used to record the cause of the change for visibility in the rollout history.  

The deployment event history will show if which strategy was used to deploy.  

For each deployment, a new ReplicaSet is created.  

## Rollback  
A rollback will rollback to the previous replica set:    
`kubectl rollout undo deployment/<deployment-name>`  

To rollback to a specific revision:  
`kubectl rollout undo deployment/<deployment-name> --to-revision=1`  
