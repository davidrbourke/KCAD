# Network Policies

By default, all traffic within the cluster is allowed, any Pod can talk to any other Pod or Service.

Network Policies allow limiting traffic, e.g. to prevent a frontend application connecting directly to the database.

Once an Network Policy is applied to a Pod, all traffic other than what is allowed it blocked.

Use Labels and Selectors to allow traffic.
The Network Policy is controlled by the Network Solution, e.g., Kube-router, Calio, etc. Not all Network Solutions support Network Policies.


This example below would allow traffic from a Pod with a Label name=api-pod to the port 3306:  

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchlabels:
      role: db
  policyTypes: 
  - Ingress
  ingress:
  - from:
    - podSelector:
      matchLabels:
        name: api-pod
    ports:
    - protocol: TCP
      port: 3306
```
## Network Policy explanation
- Policy Types: Ingress and Egress, because only the **Ingress** policy type is specified, Egress is Allow-All.
  - Traffic from a Request will always be allowed back by default, so the Egress rule is not required.
  - The calling Pod does not need an Ingress rule to return a request from the Db Pod either, responses to requests are allowed, however, if the DB Pod was making its own request to the calling Pod, the calling Pod may need a rule to Allow Ingress from the DB Pod (unless no Ingress is specified, then it is Open to Allow-All).
- The podSelector specifies which Pods to protect with the Network Policy, in this case, all Pods with the label role=db.
- **From** the source of the request that is allowed
- **Port** the target port on the Pod 


To limit the Ingress by Pod and Namespace, use the namespaceSelector:
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchlabels:
      role: db
  policyTypes: 
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: api-pod
    - namespaceSelector:
        matchLabels:
          name: prod
    ports:
    - protocol: TCP
      port: 3306
```

To limit by IP addresses, e.g. for resources not within the cluster:  

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchlabels:
      role: db
  policyTypes: 
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: api-pod
      namespaceSelector:
        matchLabels:
          name: prod
    - ipBlock:
        cidr: 192.168.5.10/32
    ports:
    - protocol: TCP
      port: 3306
```

The above **from** block has 2 rules, for podSelector and ipBlock. 
- The ipBlock is in its own rule, so this works like an OR, rule 1 (podSelector && namespaceSelector) || ipBlock
- The 1st **from** rule for podSelector also has the namespaceSelector within it, that that works like an AND.

## Egress example
This would allow external traffic to the IP address.

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchlabels:
      role: db
  policyTypes: 
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: api-pod
    ports:
    - protocol: TCP
      port: 3306
  egress:
  - to:
    - ipBlock:
        cidr: 192.168.5.10/32
    ports:
    - protocol: TCP
      port: 80
```

## Multiple rules with different ports

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchlabels:
      role: db
  policyTypes: 
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          name: payroll
    ports:
    - protocol: TCP
      port: 8080
  - to:
    - podSelector:
        matchLabels:
          name: mysql
    ports:
    - protocol: TCP
      port: 3306
```
