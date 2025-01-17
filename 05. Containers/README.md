# Containers

Steps with docker, example with python

1. Base OS Operating system
2. Update apt repo
3. Install dependencies using apt
4. Install python dependencies using pip
5. Copy source folder to /opt folder
6. Run the web server using flask command

Build image  
Push to registry

## Dockerfile
Format is: INSTRUCTION argument
```
FROM Ubunutu

RUN apt-get update
# etc
```

Base image can be an OS image, or another image created from a Base OS image.
All Dockerfiles must start with a FROM instruction.

### Command to see image size
`docker history <image-name>`  

Each layer in the Dockerfile is cached, so changes/failures may not require rebuilding the entire image.

# Commands
```
docker build -t <imagename> -f <path to Dockerfile> .
docker run -p 8282:8080 -d webapp-color
```
To find underlying OS, run the image, then `cat /etc/os-release`  

Run and exits as soon a the process is complete:  

`docker run <image>`  

## ENTRYPOINT and CMD defaults

- ENTRYPOINT command, specifies the default process that is executed when your container starts:
  - ENTRYPOINT ["sleep"]
- CMD [] defines the arguments that get passed to the ENTRYPOINT process (note: if ENTRYPOINT is not set, then defaults to the shell):
  - CMD ["<command>", "<parameter>"]
  - CMD ["sleep", "5"]
  - If ENTRYPOINT ["sleep] & CMD ["5"], then the container will run `sleep 5`

## Override CMD when running the container
- Run image passing parameter: `docker run <image> 5`  

Use both ENTRYPOINT followed by CMD to set a default, anything provided in the docker run command will overwite the CMD:
- ENTRYPOINT ["sleep]
- CMD ["5"]

## Override ENTRYPOINT when running the container
Use --entry-point to override the ENTRYPOINT in the dockerfile:  
`docker run --entry-point sleep2.0 <image> 10`  

## Commands and Arguments in Kubernetes
*Note: as this is confusing, command in Kubernetes definition is not CMD in the Dockerfile, command == ENTRYPOINT*  
- args: array - override the docker CMD
- command: array - override the ENTRYPOINT

```
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
      args: ["5"]
      command: ["sleep2.0"]
```

### Passing arguments with imperative commands
Anything after a double dash -- will be passed to the image as args/CMD, e.g. below example would pass `--color` and `green`:  
`kubectl run webapp-green --image=kodekloud/webapp-color -- --color green`
To pass in commands to ENTRYPOINT, use --command, e.g.,  
`kubectl run webapp-green --image=kodekloud/webapp-color --command python app -- --color green`

Would be same as:
```
apiVersion: v1
kind: Pod
metadata:
  name: webapp-green
spec:
  containers:
    - name: webapp-green
      image: kodekloud/webapp-color
      args: ["--color", "green"]
      command: ["python", "app"]
```


## Environment Variables
3 ways to set environment variables in Kubernetes:  
1 - In the Pod or Deployment
```
env:
  - name: APP_COLOR
    value: green
```

2 - In the ConfigMap
```
env:
  - name: APP_COLOR
    valueFrom: 
      configMapKeyRef:
```

3 - In the Secrets
```
env:
  - name: APP_COLOR
    valueFrom: 
      secretKeyRef:
```

# Example using my Minikube and docker hub

My docker file called 'dockerfile'

```
FROM busybox:latest

ENV SLEEP_SECONDS=10
ENV LOOPS=10

ENTRYPOINT ["sh", "-c"]
CMD ["echo $SLEEP_SECONDS; sleep $SLEEP_SECONDS;"]
```

Build the docker file
```
 docker build -t mysleeper:0.0.a .
```

Run the image locally - runs for 10 seconds and logs the echo commands
```
 docker run mysleeper:0.0.a
```

Override the environment variable for SLEEP_SECONDS
```
 docker run -e SLEEP_SECONDS=5 mysleeper:0.0.a
```

Override the CMD
```
docker run mysleeper:0.0.a -- "echo hi"
```

Tag the image with my hub repo, and push to the docker hub
```
docker image tag mysleeper:0.0.a polarcoder/mysleeper:0.0.a
docker push polarcoder/mysleeper:0.0.a
```

Create a pod on the Minikube and get the logs
```
 kubectl run mysleeper --image=polarcoder/mysleeper:0.0.a
 kubectl logs mysleeper

 # Outputs:
 10
```

Pod Yaml to override the environment variable
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: mysleeper
  name: mysleeper
spec:
  containers:
  - image: polarcoder/mysleeper:0.0.a
    name: mysleeper
    env:
      - name: SLEEP_SECONDS
        value: "20"
  dnsPolicy: ClusterFirst
  restartPolicy: Never
```

Pod Yaml to override the CMD
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: mysleeper2
  name: mysleeper2
spec:
  containers:
  - image: polarcoder/mysleeper:0.0.a
    name: mysleeper2
    args: ["echo 'Override sleep seconds to 20'; echo $SLEEP_SECONDS; sleep 20;"]
  dnsPolicy: ClusterFirst
  restartPolicy: Never
```
