# Taints and Tolerations

## Taint
A Node can have a **taint**. A restriction on what Pods can be deployed on which Nodes.
With no restrictions, the scheduler can balance the Pods across all Nodes.
There may be scenarios where you need specific pods on a Node, e.g., they share some specific services or resources.

- A **Taint** applied to a Node prevents **all** Pods being assigned to a Node.
- A **Toleration** allows certain Pods to be tolerant of the Taint, so Pods with the toleration can be assigned to the Node with the Taint.

`kubectl taint nodes <node-name> key=value:taint-effect`

e.g.
`kubectl taint nodes node1 app=blue:NoSchedule`

## Taint effects
Defines what happens to the Pods if they do not tolerate the taint:
- NoSchedule - do not schedule the Pod on the Node
- PreferNoSchedule - try to avoid placing the Pod on the Node but not guaranteed
- NoExecute - no new Pods will applied to the Nodes, and existing Pods on the Node that do not tolerate the taint will be evicted.

Pod definition for `kubectl taint nodes node1 app=blue:NoSchedule`

*All of the toleration values need to be wrapped in double-quotes*

```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: nginx-controller
    image: nginx
  tolerations:
  - key: "app"
    operator: "Equal"
    value: "blue"
    effect: "NoSchedule"
```

## Notes
A taint doesn't tell the Pod which node to go to (this is Node affinity).
The Pod with a Toleration can still be scheduled to a Node that doesn't have the Taint, it just is used to prevent Nodes being scheduled to a Pod.

The scheduler doesn't schedule any Pods to the Master node, that is because when the Cluster is created, a Taint is created on the Master Node.
To see the Master Node taint:

`kubectl describe node kubemaster | grep Taint`

## Remove a Taint
Use the taint command with a hyphen - at the end:
`kubectl taint nodes controlplane  node-role.kubernetes.io/control-plane:NoSchedule-`
