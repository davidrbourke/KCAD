# DaemonSets

A DaemonSet ensures that all (or some Nodes) run a copy of a Pod. As Nodes are added to the Cluster, Pods are added to them.

Examples of usage
- running a logs collection daemon on each Node
- running a monitoring daemon on each Node

The definition is similar to a Deployment:
- The first spec section is the DaemonSet selector
- The second spec section under the template is for the Pod configuration

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
      # these tolerations are to have the daemonset runnable on control plane nodes
      # remove them if your control plane nodes should not run pods
      - key: node-role.kubernetes.io/control-plane
        operator: Exists
        effect: NoSchedule
      - key: node-role.kubernetes.io/master
        operator: Exists
        effect: NoSchedule
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      # it may be desirable to set a high priority class to ensure that a DaemonSet Pod
      # preempts running Pods
      # priorityClassName: important
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
```

- RestartPolicy: must be Always in a DaemonSet, or empty, which defaults to Always

## Running on Specific Nodes
Specify a `.spec.template.spec.nodeSelector` (see Selectors and Affinity section).

## Priority
If a Pod cannot be allocated to a Node because there is not enough capacity, another Pod(s) will be evicted from the Node to make room. 
Set the `.spec.template.spec.priorityClassName` to a higher class:

### Priorty Class

You create the Priority Class resources, and match the name of the Priority Class with the priorityClassName on the DaemonSet. The higher the value, the higher the priority (any 32 bit integer number smaller than or equalt to 1 billion).

```
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000
globalDefault: false
description: "This priority class should be used for XYZ service pods only."
```

- The **globaldefault** field means that this class should be applied for any Pod without a Priority Class.
- A number of tolerations are added automatically when the Pod in a DaemonSet is created, this is to ensure it does get allocated to a Node, even if there are possible Node issues.

### Deleting a DaemonSet
You can delete a DaemonSet, if you use the `--cascade=orphan` flag, the Daemon Set is deleted but the Pods will be left on the Nodes:
```
kubectl delete ds <ds-name> --cascade=orphan
```