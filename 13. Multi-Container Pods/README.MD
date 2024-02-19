# Multi-Container Pods

Sometimes you need multiple services to work together, that can scale together.  
Multi-container Pods are created and destroyed together, they have access to the same resources and Storage Volume.  
In the Pod definition, the containers property is an array - to add mulitple containers.  

## Patterns
### Sidecar
Sidecar pattern consists of a main application plus a 'helper' container.
Example: Deploying a logging agent alongside a web server to collect and forward logs to a central logging server.

### Adapter
Adapter pattern is used to standardise and normalise application output for monitoring.
Example: Adapter container processes the logs to a correct format before sending them to a logging server, the main application development doesn't then have to be concerned with this task. Central logging server also now doesn't need to format logs.

### Ambassador
Ambassador pattern is a way to connect a Pod with the outside world, essentially a proxy to connect with outside services as if they are running on localhost.
Example: A different database for different environments, (e.g., dev, test, prod). The Ambassador Pod is responsible for which database is connected to, the main application Container is just accessing the database on the localhost, the Ambassador container is then proxying the request to the right database.

## Simple example running 2 containers

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: yellow
  name: yellow
spec:
  containers:
  - image: busybox
    name: lemon
    command: ["sleep", "1000"]
  - image: redis
    name: gold
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```
A sleep command had to be added to prevent a crash loop.

## Example running a sidecar to forward logs to elastic

Logs from the the 1st container (app) are written to a volume mount called log-volume - in the /log directory on the container.
These logs are used by the 2nd container (sidecar) to forward to elastic. The 2nd container also has the log-volume mounted, the path is /var/log/event-simulator/.
The content from the /log directory in container 1, is available in container 2 in the path /var/log/event-simulator/. This is where container 2 is expecting them to forward them to Kibana.

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2024-01-15T18:41:47Z"
  labels:
    name: app
  name: app
  namespace: elastic-stack
  resourceVersion: "783"
  uid: d04c10b8-4601-4c65-bd65-989015caa711
spec:
  containers:
  - image: kodekloud/event-simulator
    imagePullPolicy: Always
    name: app
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /log
      name: log-volume
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-h589q
      readOnly: true
  - name: sidecar
    image: kodekloud/filebeat-configured
    volumeMounts:
    - mountPath: /var/log/event-simulator/
      name: log-volume
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: controlplane
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - hostPath:
      path: /var/log/webapp
      type: DirectoryOrCreate
    name: log-volume
  - name: kube-api-access-h589q
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
```

## Init Containers

Multi-container pods run for the lifecycle of the container. In some scenarios, an additional container may only need to run at startup, e.g., to load secrets from a vault. For this, there are initContainers. There can be multiple initContainers, they run sequentially, one at a time, and are terminated on completion. If an initContainer fails, the Pod will restart until it succeeds. The containers will not start running until the initContainers have completed.

```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
```
