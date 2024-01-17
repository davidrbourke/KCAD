# Logging and Monitoring

## Logging

Read Logs with live tail:
`docker logs -f <pod-name>`

If there are multiple containers in the Pod, specify the container or the command will fail:
`docker logs -f <pod-name> -c <container-name>`

## Monitoring

Node level & Pod level
- No. of Nodes
- CPU
- Memory
- Network etc

Kubernetes doesn't come with a built in solution.
Need to use other 3rd party tools, such a Metrics Server, Prometheus, Datadog, Elastic, etc.

Metrics Server is slimmed down version of Heapster

It records basic metrics, and stores them in the memory - you cannot see historic performance data.

Kubernetes runs a kubelet on each Node, the kubelet contains a sub-component, the cAdvisor to make metrics available to the kubelet.

To deploy, clone the Metrics Server deployment files github repository and deploy the components.

Repository
`https://github.com/kubernetes-sigs/metrics-server`

Deploy
`kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml`

Once all the components are installed, 

View metrics of nodes
`kubectl top node`

View metrics of pods
`kubectl top pod`

