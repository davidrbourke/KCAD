# Kubernetes Overview

## Node
A physical machine, need more than 1 for redundancy.

## Cluster
A set of Nodes grouped together, if one Node fails, the application still runs on other Nodes, also used for scaling load.

## Master Node
A Node with K8s configured as a master, watches the Cluster, manages the other Nodes.

## K8s Components
1. API Server - frontend for K8s, command line, etc
2. etcd - Distributed Key Store to store data used to manage the cluster, stored across the Cluster, implementing logs to prevent conflicts between distrubuted data
3. kubelet - Agent that runs on each Node in the Cluster, responsble for ensuring the Containers are running on the Nodes as expected
4. Container Runtime - The underlying software running the Containers, e.g., Docker
5. Controller - Responsible for monitoring the state of containers, and make decisions to bring up new containers if one goes down
6. Scheduler - scheduling the new containers across the Nodes

## Master vs Worker Nodes
### Worker Nodes
 - where the containers are hosted, the Container runtime is installed on the Worker Nodes (e.g. Docker among others)
 - Kubelet Agent

### Master Node
 - kube-api-server
 - etcd
 - Controller
 - Scheduler

## Kubectl
Command line tool to deploy and manage containers in the cluster.
e.g., kubectl run hello-minikuke
 kubectl cluster-info
 kubectl get nodes

### Output formats
-o json
-o name
-o wide
-o yaml
