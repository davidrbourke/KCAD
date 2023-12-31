# Core Concepts

## Docker vs ContainerD
In the beginning there was just Docker. So it became the dominant to Container tool. Kubernetes was created to manage Docker containers.
Kubernetes was expanded to run other containers, not just Docker.
Kubernetes was created (according Open Container Initiative standards) to run any container runtime that meets the OCI standards (e.g., rkt).

Docker wasn't built to support the OCI Standards, so Docker Shim was created to support Docker outside the OCI standard.
Docker isn't just a Container runtime, it has many other tools for building images, etc. ContainerD was the runtime used by Docker. It can be used as a separate runtime.
Docker Shim was eventually removed, but Docker images continue to work as Docker images now follow the OCI Standard.

ContainerD is a separate project on its own now, without having to install Docker.

## ctr 
Tool for debugging ContainerD. E.g., to pull an Image
$ ctr images pull docker.io/library/redis:alpine
$ ctr run docker.io/library/redis:apline redis

## nerdctl
A better general purpose cmdline tool than ctr for ContainerD, and has access to more features in Docker, can be used almost like the Docker command:
$ nerdctl run --name redis redis:alpine

## crictl (Container Runtime Interface - CRI)
A tool to interact with all the different contanier runtimes not specifically for ContainerD, but other runtimes such as rkt.
Can be used to debug container runtimes, but not used for creating containers. Many similar commands to Docker cmd:
$ crictl pull busybox
$ crictl images
$ crictl ps -a
$ crictl exec -it -t [container] ls
$ crictl logs [container]
$ crictl pods

