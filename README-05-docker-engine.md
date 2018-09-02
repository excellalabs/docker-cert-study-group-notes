# The Docker Engine

Docker used to be made up of:

- **Docker client**: a cli or lib that talks to the Docker daemon.

- **Monolithic Docker daemon**: performed __***all***__ docker image workflow tasks and container creation tasks

- **LXC**: a system-container solution tailored to the linux kernel

This was not only a bad design, but there were some pretty terrible consequences (e.g. if you stop the docker daemon, all of your running containers are killed!). This lead to a major overhaul to break out the functionality into smaller components (using the UNIX philosophy).

Docker is now made up of:

- **Docker client** aka "docker": (unchanged)

- **Docker daemon** aka "dockerd": responsible for the API, docker workflow tasks, security features, core networking, and volume management. (at least it's not "everything" like it used to be)

- **containerd**: is a **container execution supervisor** which can manage a complete container lifecycle and enable runtime telemetry.

- **containerd-shim**: a small process for invoking runc. This is used to keep the STDIO (and other FDs) open for the container incase containerd and/or docker both die. The Shim also reaps the container's exit status and reports the value back to containerd, all without having the be the actual parent of the container's process and do a wait4. This solves the "zombie" reaping problem that has plauged Docker for years.

- **runc**: is lightweight universal container runtime. Runc is used by containerd for spawning and running containers according to OCI spec. It is essentially a CLI wrapper around libcontainer, the library that replaced LXC. Once the user process starts the runc process exits, allowing for a daemonless container configuration.


How you start a container with this engine from the ground up:

```
Client <==REST==> Daemon <==GRPC==> containerd > (fork+exec) > containerd-shim + runc > (exec) > your processes

```

Want more? Go nuts: https://ops.tips/blog/run-docker-with-forked-runc/
