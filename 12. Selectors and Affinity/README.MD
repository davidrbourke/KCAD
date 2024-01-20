# Selectors and Affinity

## Node Selectors

May wish to assign certain Pods to certain Nodes, e.g., a Node may have more resources.
By default, a Pod can be scheduled on any Node.

There are two ways to limit a Pods to a Node.
- Node Selectors (easiest way)

```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: data-processor
    image: data-processor
  nodeSelector:
    size: Large
```

`size: Large` is a key=value pair label and selectors.
The Nodes must be labelled prior to creating the Pod.

Command to label a node:
`kubectl label nodes <node-name> <label-key>=<label-value>`

e.g.,
`kubectl label nodes node01 size=large`

### Limitations of Selectors
- For simple scenarios, can only assign Pod to one Node via a label, cannot have more nuanced rules, so instead use Node Affinity.

## Node Affinity
Ensures Pods are hosted on particular nodes.
Affinity is more complex - below is the same assignment as the Node Selector above:

```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: data-processor
    image: data-processor
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: In
            values:
            - Large
            - Medium
```

- **nodeSelectorTerms** is an array, so multiple matches can be set.
- **values** is an array, so you can add more labels to match to the key.
- **operator** is the match type: In (=), NotIn (!=), Exists (checks the label exists, doesn't compare the values), NotExists, Gt (>), Lt (<) = Gt/Lt compare label as integer.

### Type of Node Affinity 

#### requiredDuringSchedulingIgnoredDuringExecution

When a pod is scheduled, no matching label means the Scheduler will not schedule the Pod. If a Pod is running and some Node Affinity rule changes the Affinity, the Pod will continue run on the Node.

#### preferredDuringSchedulingIgnoredDuringExecution
When a pod is scheduled, no matching label means the Scheduler will ignore node affinity rules, and use any Node. If a Pod is running and some Node Affinity rule changes the Affinity, the Pod will continue run on the Node.

#### requiredDuringSchedulingRequiredDuringExecution (new?)
First part of rule applies as above rules, but this rule will evict any Pods running on a Node where a label change impacts the Affinity.

## Taints and Tolerations vs Selectors and Affinity
A combination of taints and tolerations and affinity can be used to ensure pods only ever run on the specific nodes.
