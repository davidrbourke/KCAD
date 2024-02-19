# Readiness and Liveness Probes

## Readiness Probe

### Pod lifecycle
The Pod Status describes the Pod lifecycle stage:
- Pending Status - when the scheduler tries to place the Pod, if the scheduler cannot find a Node, it remains in Pending status.
- ContainerCreating - scheduler has found a place for the Pod, and the Pod is being created.
- Running - the Pod is Running.

**Conditions** complement the Pod Status:
- PodScheduler
- Initialized
- ContainersReady
- Ready - the Pod is running and ready to accept user traffic. However, the http server or start-up tasks on the Container may still take a few seconds to warm up.

`kubectl describe pod <name>` will show the Status and Conditions.

The Ready state may not actually be ready to receive traffic. Kubernetes by default will serve user traffic once the Ready condition is met.

Ways to decide an application on a Pod is ready:
- HTTP Tests
- TCP Test for Database connection
- Execute a custom command/script that can check something in the Container

### Http Test

Any HTTP response code >= 200 and < 400 is a success, any other code is an error.

```
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
spec:
  containers:
  - name: simple-webapp
    image: simple-webapp
    ports:
      - containerPort: 8080
    readinessProbe:
      httpGet:
        path: /api/ready
        port: 8080
```

Other Http Request options can be set:
- host - default to Pod IP
- scheme - http/s (skips certificate validation)
- httpHeaders - to set custom headers


### TCP Test

The TCP test attempts to open a socket on the container at the Port number, if it can establish a connection, it suceeds, otherwise it is a failure.

```
readinessProbe:
  tcpSocket:
    port: 3306
```

### Exec Command

Some custom command or script can be executed that if it successfully completes, then the application is ready.
The command succeeds if it returns a 0, any other result is an error.

```
readinessProbe:
  exec:
    command:
      - cat
      - /app/is_ready
```

### Probe configuration

- **initialDelaySeconds** is how long to wait from the when the Pod is Ready until runnning the readinessProbe test, default is 0 seconds, minimum is 1.
- **periodSeconds** is how long to wait between retries of the test if the first test fails (e.g., because the application is not ready), default is 10 seconds, minimum is 1.
- **failureThreshold**, by default the probe will make 3 attempts, the failureThreshold will increase/decrease the number of attempts.
- **successThreshold**, by default, 1 successful test will result in passing the test, you can require more consequtive successful tests with the success threshold.
- **timeoutSeconds** is how long before the probe timesout, default is 1 seconds, minimum is 1.
- **terminationGracePeriodSeconds** is how long to wait before the kubelet triggers a Pod shutdown, default is 30 seconds, minimum is 1.

```
readinessProbe:
  httpGet:
    path: /api/ready
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
  failureThreshold: 8
```

## Liveness Probes

Kubernetes will restart a crashed container automatically. However, if the application is not really working, but the container is still running, Kubernetes assumes the application is up as the container is still running. Traffic will still be routed. A Liveness Probe will test if the application is actually healthy. If the Liveness Probe finds an unhealthy application, it will restart a Pod and traffic will not be routed to it until it is up and running again.

### Http Test

```
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
spec:
  containers:
  - name: simple-webapp
    image: simple-webapp
    ports:
      - containerPort: 8080
    livenessProbe:
      httpGet:
        path: /api/ready
        port: 8080
      initialDelaySeconds: 10
      periodSeconds: 5
      failureThreshold: 8
```

### TCP Test

```
livenessProbe:
  tcpSocket:
    port: 3306
```

### Exec Command

```
livenessProbe:
  exec:
    command:
      - cat
      - /app/is_ready
```
