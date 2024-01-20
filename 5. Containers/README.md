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

- CMD [] defines the command that runs when the image starts
  - CMD ["<command>", "<parameter>"]
  - e.g., CMD ["sleep", "5"]
- ENTRYPOINT command, allows us to specify which command to run when the container starts, any parameters entered into the docker run command with get appended to the ENTRYPOINT command.
  - ENTRYPOINT ["sleep"]
- Run image passing parameter: `docker run <image> 5`  

Use both ENTRYPOINT followed by CMD to set a default, anything provided in the docker run command will overwite the CMD
- ENTRYPOINT ["sleep]
- CMD ["5"]

Use --entry-point to override the ENTRYPOINT in the dockerfile:  
`dopcker run --entry-point sleep2.0 <image> 10`  

## Commands and Aruguments in Kubernetes
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
      command: ["sleep2.0]
```

### Passing arguments with imperative commands
Anything after a double dash -- will be passed to the image as arguments/CMD, e.g. would pass --color and green  
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
      args: ["python", "app"]
      command: ["--color", "green"]
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
