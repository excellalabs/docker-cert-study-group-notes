## Meeting Notes

###

- next meeting - by cert bullets?
- make list of exercises that will train on hands-on, prod bugs, etc

## Resources

* [Docker Certification site w/Study Guide](https://success.docker.com/certification)
* [Docker Certified Associate Exam Prep Guide - w/links from exam bullets](https://github.com/Evalle/DCA)

* [Play with Kubernetes](https://labs.play-with-k8s.com/)
* [Play with Docker Enterprise](https://medium.com/@marcosnils/60-seconds-away-from-docker-ee-13d7cf66713f)
* [Docker Swarm Workshop, J. Petazzo](https://github.com/jpetazzo/container.training)

* [Linux Container from Scratch, Joshua Hoffman](https://vimeo.com/115073286)

## Docker Deep Dive Notes

1. Containers from 30,000 feet
    * What are containers, etc
    * Windows vs Linux
1.  Docker
    * Kubernetes - leading orchestrator, needed for managing deployment and running apps
    * Docker Engine - plumbing that runs and orchestrates containers
    * EE vs CE
    * Moby - tools that combine to make Docker daemon. Breaks dDOcekr into modular components. Big infra contributors. Golang.
        * Batteries included but removable
    * Open Container Initiative (OCI) - standardized image format and container runtime
1. Installing Docker: 
    - Windows, Mac, Windows Server 2016, Ubuntu, Docker EE, etc
    - Upgrading Docker: 
        - make sure containers have restart policy, 
        - drain nodes if needed (using Swarm mode). 
        - Steps: 
            - Stop Docker daemon, remove old version, install new version, configure new version to auto start when system boots, ensure containers restarted.
    - Docker and storage drivers
        - Every Docker container gets an area of local storage where image layers are stacked and the container filesystem is mounted. By default this is where all container read/write ops occur, making it integral to the perf and stability of every container.
        - Local storage area has been managed by storage driver, which are sometiomes called the graph driver. Docker supports several different storage drivers, each which implements layering and copy-on-write in its own way - impacting perf and stability.
          - Some available drivers:
              - aufs (the original and oldest)
              - overlay2 (probably best choice for future)
              - devicemapper
              - btrfs
              - zfs
          - Windows only supports windowsfilter driver
1. The Big Picture
    - Ops perspective 
        - Docker client, 
        - daemon/engine, 
        - images, running containers
        - attaching to running containers
      - Dev perspective - containerize app
