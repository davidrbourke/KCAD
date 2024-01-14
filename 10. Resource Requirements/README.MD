# Resource Requirements

The **kube-scheduler** decides which Node a Pod is created on.
It considers the resources required by the Pod and identifies the best Node with sufficient resources available. If there are no Nodes with available resources, the Pod will not be allocated to a Node and the Pod events will show an error.

## Resource Values
### Requests
The minimum amount of resources required, the guaranteed amount the Pod will get:


```
resources:
  requests:
    memory: "4Gi"
    cpu: 2
```

0.1 cpu
100m
Minimum is 1m

1 count of Cpu == 1 virtual CPU, or 1 virtual Core, or 1 hyperthread, depending on the Cloud provider or OS.

Specify in full bytes or using suffix:
- 1 G (Gigabyte) = 1,000,000,000 bytes
- 1 M (Megabyte) = 1,000,000 bytes
- 1 K (Kilobyte) = 1,000 bytes

- 1 Gi (Gibibyte) = 1,073,741,824 bytes
- 1 Mi (Mebibyte) = 1,048,576 bytes
- 1 Ki (Kibibyte) = 1,024 bytes

### Limits
If no limit is specified, the Pod can consume all the resources on the Node. A limit, limits how much resource the **Container** can consume.

```
resources:
  limits:
    memory: "4Gi"
    cpu: 2
```

A Cpu limit will never be exceeded, performance will be throttled.
A memory limit can be exceeded, but if it exceeds constantly the Pod will be terminated (OOM Kill).


### Some rules for CPU
- If limits are specified, but not limits, then requests are automatically set to the same as limits.
- Requests and Limits are set, they won't exceed the limits, means that a Pod may have claimed resources it doesn't need, and prevents other Pods using that resource.
- Requests are set, but not limits, the Pod will be guaranteed its requests amount, but other Pods can use available resources in the Node so a Pod. If not setting limits, you must set requests for all the Pods, to guarantee that the Pod gets some resource.

### Some rules for Memory
- If limits, but no requests, requests are set to limits
- Requests and limits are set, they won't exceed limits
- Request are set, but not limits, any Pod can consume as much memory as available, once memory is assigned to a Pod, the only way to free it is to kill it.

## Limit Ranges
This is to apply a default at the namespace level, so if you don't specify resources in the Pod, the Pods get assigned the limit range by default:

```
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-resource-constraint
spec:
  limits:
  - default:
      cpu: 500m
    defaultRequest:
      cpu: 500m
    max:
      cpu: "1"
    min:
      cpu: 100m
    type: Container
```

max: the maximum **limit** that can be set on a Container
min: the minumu **request** that can be set on a Container

```
apiVersion: v1
kind: LimitRange
metadata:
  name: memory-resource-constraint
spec:
  limits:
  - default:
      memory: 1Gi
    defaultRequest:
      memory: 1Gi
    max:
      memory: 1Gi
    min:
      memory: 500Mi
    type: Container
```

The limit constraints are applied on to new Pod created after the limit range is created or updated.

## Resource Quotes
At a namespace level, limit total cpu and memory resources allowed for all Pods in the namespace.

```
apiVersion: v1
kind: ResourceQuote
metadata:
  name: my-resource-quota
spec:
  hard:
    requests.cpu: 4
    requests.memory: 4Gi
    limits.cpu: 10
    limits.memory: 10Gi
```
