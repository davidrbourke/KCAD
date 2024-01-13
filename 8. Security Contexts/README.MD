# Security Contexts

## Docker Security

The Docker host has services running, containers running on the host share the same underlying Operating System components. Containers are isolated using namespaces.
Processes running on the container are run on the host in its own namesapce.
The container process cannot see other namespaces processes or containers.
The host can see all processes, included container processes.
The process IDs will not match when viewed from the host or container.

Docker host has a root and non-root users.
By default docker runs processes in containers as the **root** user.

To stop the container using the root user, you must set the USER id of the non-root user.

This can be defined in the docker image:

```
FROM ubuntu

USER 1000
```

Or from the run command:
`docker run --user=1000 ubuntu`

To see which user the pod is running under, run **whoami** on linux.

The root user within the container is limited, not like the root user on the host.
It cannot perform operations that distrupt the host or other containers.
You can enhance these privilages or reduce them using the **cap** command:

(See all capabilities: /usr/include/linux/capability.h)

```
docker run --cap-add MAC_ADMIN ubuntu
docker run --cap-drop KILL ubuntu
```

Or to run the container with all priviledges enabled:
`docker run --privileged ubuntu`

## Security Contexts

The security settings can be configured at container or pod level, 
- If configured on the pod, all the containers in the pod get the settings.
- If configured on the pod AND container, the container settings override the pod settings.

