# Services

Services enable connectivity between services within and outside the Cluster.  
- Enable Frontend -> Backend
- Enable External -> Frontend

## IPs
- A Node has an IP Address  (192.168.1.2)  
- The internal Pod network has a range (10.244.0.0)  
- The Pod has an IP from within the range (10.244.0.2)  

You cannnot access the Pod via the internal IP address externally, but you could from the Node itself.  

Something in the middle needs to map an external connection to the Pod. The Kubernetes Service does this.  

## Types of Service
1. **Node Port** listens to a Port on the Node and Forwards a Request to a Port on the Pod running a web application.
2. **Cluster IP** the Service creates a virtual IP in the cluster to enable communication between different services.
3. **Load Balancer** creates a load balancer in supported cloud providers to distribute load in your cluster.

## Node Port Service
- **target port** is the Port on the Pod
- **port** is the Port on the Service
- **node port** is the Node port to access the Service externally (valid range is 30000-32767)

```
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  ports:
    - targetPort: 80
      port: 80
      nodePort: 30008
  selector:
    app: myapp
    type: front-end
```
Only **port** is required, nodePort can be assigned automatically if not specified. You can have multiple port mappings in a Service.  
The label selectors define which Pods to connect to by using the matching labels.  

To access using curl:  
`curl https://<node-id>:<nodePort>`  
The Service can map to multiple Pods using the matching labels, and Service acts as a built in Load Balancer using a random algorithm. This works across Pods across all Nodes.  

## Cluster IP Service
To connect internal layers of the application, e.g. frontend to backend, internal Pod IPs are not static, Pods restarting gets a new IP. How to group the backend Pods together and create a single endpoint for the frontend to connect to the backend? Deploy a Cluster IP Service between the layers. Each Service gets a name and IP, the name can be used by other Pods to access the service's Pods. **Endpoints** are visible in the 'describe' command for the Service, it will show how many Pods the Service is routing traffic to.

```
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  type: ClusterIP
  ports:
    - targetPort: 80
      port: 80
  selector:
    app: myapp
    type: backend-end
```

- **target port** is the Port where the backend is exposed
- **port** is the Port where the Service is exposed.
- Selectors are use to map the Pods to the Service.  

## Imperative Commnd
```
kubectl create service <type> <name> --tcp=<port>:<target-port>
kubectl create service nodeport my-service --tcp=8080:8080
```
