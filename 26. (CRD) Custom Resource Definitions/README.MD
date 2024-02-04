# Custom Resource Definitions (CRD)

When a definition YAML file is applied, it creates or updates an entry in the etcd database for the resource. A Controller then creates the actual resources. The Deployment Controller is built into Kubernetes, its job is to continuously minitor the condition of the cluster and the specification in the etcd. It is written in Go and is part of the K8s source code. There are various controllers, for each Resource. 

## Custom Resources
Custom resources are non-standard resources that are not part of Kubernetes. E.g. A resource of type FlightTicket, a custom controller would be required to interpret the resource definition and perform expected set up, e.g. purchase a Flight Ticket. It would read the state of the definition from the etcd database. 

### Custom Resource YAML
```
apiVersion: flights.com/v1
kind: FlightTicket
metadata:
  name: my-flight-ticket
spec:
  from: Mumbai
  to: London
  number: 2
```

### Applying the YAML
```
kubectl create -f flightticket.yaml
```

## Configure the Custom Resource Definition (CRD)
```
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  # This name must match the <spec.names.plural>.<spec.group>
  name: flighttickets.flights.com 
spec:
  scope: Namespaced
  group: flights.com
  names:
    kind: FlightTicket
    singular: flightticket
    plural: flighttickets
    shortnames:
      - ft
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties: 
            spec:
              type: object
              properties:
                from:
                  type: string
                to:
                  type: string
                number:
                  type: integer
                  minimum: 1
                  maximum: 10
```

- **scope** defines if the resource type is scoped to namespaces (e.g. like a Pod) or not (e.g. persistent volume, nodes), options are Namespaced or Cluster.
- **group** the api version for the resource definition 
- **names** is the name, singular, plural, short name for interacting with the resource definition via kubectl
- **versions** are the valid api version that can be used in the resource definition file (served through the API server - options true/false, storage version in etcd, true means only 1 version is stored)
- **schema** the support fields in the spec section of the resource definition, uses OpenAPI v3.

Once the CRD is created, you can start creating resources. The above configuration only allows you to create the resource definition in etcd. Without a Controller it doesn't actually do anything.

## Create Custom Controllers
A Controller is a process running in a loop listening for objects of a specific kind changing in etcd. This can be written in different languages. Developing in Go with the Kubernetes Go Client is the easiest way. 

### Steps
1. Clone this repo: https://github.com/kubernetes/sample-controller
2. Customise the controller.go class with custom logic, e.g. making call the Flight Booking API (continuing the example above)
3. Build the code and publish using the kube config.
4. You can now build and deploy the custom controller and run it in a Docker container in the cluster.

In the exam, will not be expected to build a custom controller, but more likely to be asked about setting up the CRD.

## Operator Framework
The Operator Framework allows you to build and deploy the CRD and the customer Controller logic at the same time. E.g., The etcd operator.
