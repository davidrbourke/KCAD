# Ingress

- Ingress makes you application externally accessible using a single accessible URL.  
- You can configure routing to different services using a simple URL path in the Ingress.  
- You can implement SSL security.  

Still need to publish the Ingress as a Node Port or Load Balancer, but only need to do it once.


1. Ingress Controller -  First deploy a supported solution, e.g. Nginx, Haproxy, traefik, GCP Httpts/GCE Loadbalancer
2. Ingress Resources -  Specify a set of rules

Cluster does not come with a Ingress Controller by default.


## Ingress Controller
Has a definition template.
E.g. nginx-ingress-controller. 
https://kubernetes.github.io/ingress-nginx/deploy/

A Node Port service has to be created to expose the Ingress Controller externally.  

A Service Account with the right permissions to allow the Controller to monitor for new Ingress configuration updates.  

## Ingress Resource
A set of rules applied for incoming traffic, e.g. to a single application, or routing based on different paths, domain names, etc.  

### Simple Routing to a single backend service
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear
spec:
  backend:
    serviceName: wear-service
    serivcePort: 80
```

### Routing with conditions on paths
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear
spec:
  rules:
  - http:
      paths:
      - path: /wear
        backend:
          serviceName: wear-service
          serivcePort: 80
      - path: /watch
        backend:
          serviceName: watch-service
          serivcePort: 80
```

### Routing with conditions on host url
A unique rule has to be created for each host.
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear
spec:
  rules:
  - host: wear.my-online-store.com
    http:
      paths:
      - path: /wear
        backend:
          serviceName: wear-service
          serivcePort: 80
  - host: watch.my-online-store.com
    http:
    paths:
    - path: /watch
      backend:
        serviceName: watch-service
        serivcePort: 80
```

### Imperative command to create Ingress
```
kubectl create ingress <ingress-name> --rule="host/path=service:port"
kubectl create ingress ingress-test --rule="wear.my-online-store.com/wear*=wear-service:80"
kubectl create ingress ingress-pay -n critical-space --rule="/pay=pay-service:8282"
```

### Annotations
Annotations on the Ingress can be used to alter the rules, e.g. the ingress routes the path /pay to a the pay-service (in the imperative command above).
However, what is the application doesn't have a /pay? And /pay should route to the home url? Use an annotation rewrite:  
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-pay
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /pay
        backend:
          serviceName: pay-service
          serivcePort: 8282
```
