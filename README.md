## Meeting Notes

## Resources

* [Docker Certification site w/Study Guide](https://success.docker.com/certification)
* [Docker Certified Associate Exam Prep Guide - w/links from exam bullets](https://github.com/Evalle/DCA)
* [Sample test questions](https://djitz.com/certification/https://djitz.com/certification/)
* [Practice tests on Udemy](https://www.udemy.com/docker-certified-associate-certification-2-practice-exams)

*Exercises*
* [Interactive Scenarios - Kubernetes, Docker, etc - Katakoda](https://www.katacoda.com/courses/docker)
* [Deploy sample app to Swarm](https://github.com/dockersamples/atsea-sample-shop-app)
* [Docker Swarm Workshop, J. Petazzo](https://github.com/jpetazzo/container.training)
* [Linux Container from Scratch, Joshua Hoffman](https://vimeo.com/115073286)

*Sandboxes:*
* [Play with Kubernetes](https://labs.play-with-k8s.com/)
* [Play with Docker Enterprise](https://medium.com/@marcosnils/60-seconds-away-from-docker-ee-13d7cf66713f)

## Docker Deep Dive Notes

1. [Chapter 1-4 notes](README-01-04.md)

    Exercises:

    - Configure Docker daemon to start on boot
    - Which storage driver should be used on what OS? Per node decision. Overlay2 becoming favored.
    - [Configure devicemapper for production](https://docs.docker.com/storage/storagedriver/device-mapper-driver/#configure-direct-lvm-mode-for-production)

5. [Docker Engine](README-05-docker-engine.md)

6. Images

    Exercises 

    - Display layers and create new for writes
    - Tag with multiple and push image to registry
    - Invalidate image cache and optimize for writing files
    - Search Docker Hub with `docker search <name>`

7. Containers

8. [Containerizing an app](README-08-containerizing-an-app.md)

    Exercises

    - create multi-stage build dockerfile

9. Deploying Apps with Docker Compose

10. [Swarm](README-10-swarm.md)

    Exercises

    - Setup a swarm, create 2nd node and join it, run container as service
    - Run app as stack
    - Scale
    - Update app
    - Run replicated and global service
    - Apply node labels to manage placement of tasks
    - Raft consensus to manage cluster state, to keep master replicas have same state - allows (N-1)/2 failures, and requires quorum of (N/2)+1

11. [Docker Networking](README-11-network.md)

12. Docker overlay networking
    
    Exercises

    - create overlay network on 2+ node swarm, attach a service to it

        1. `docker network create -d overlay my-swarm-overlay`
        1. `docker service create --name test --network my-swarm-overlay --replicas=2 ubuntu sleep infinity`

13. Volumes and persistent data

    - 
    - 

14. Deploying apps with Docker Stacks

15. Security in Docker

    - All about layers: Linux & Docker platform security tech
    - Docker has moderately secure defaults

    - Docker tech: secrets management, docker content trust, security scanning
        - Swarm mode is secure by default using things like 
            - cryptographic node IDs, 
            - mutual auth, 
            - auto CA config, 
            - auto cert rotation, 
            - encrypted cluster store, 
            - encrypted networks
        - Docker Content Trust lets you sign images and verify their integrity & publisher
        - Docker Security Scanning analyses images for known vulerabilities
        - Docker secrets are first-class citizens, stored in encrypted data store, encrypted in flight, stored in in-memory filesystems when in use, operate a least privledge model
        - When a Swarm is set up, it becomes the root CA, default of 90 days for cert rotation
        - Swarm token has a pattern you can match to prevent repo check-in
    
    - OS (linux) tech: seccomp, mandatory access control, capabilities, control groups, kernel namespaces
        - Docker containers utilize these namespaces: pid, net, mnt, ipc, user, uts
        - All new containers get a sensible default seecomp profile
        - Docker prevents containers from adding back removed capabilities
        - seccomp - Docker uses in filter mode to limit syscalls a container can make to the host's kernel. All containers get a default seccomp profile with moderate security.

    - Rotate swarm join token, `docker swarm join-token --rotate manager`

16. Tools for the enterprise

    - Components of Docker Enterprise:
        - Docker Trusted Registry - secure on-prem registry
        - Universal Control Plane (UCP) - Enterprise-grade operations UI
        - Docker EE - hardened & certified container engine
        - Certified OSes and cloud platforms - certified infrastructure
    - Planning a UCP installation - 
        - all nodes should have a static IP and stable DNS name
        - Odd number of managers. 5 is best for backup schedule. More than 7 has back-end Raft and cluster reconciliation issues (workers don't participate in Raft. You can have any number.)
        - Manager nodes should be spread acress availability zones in a single region; need high-speed connections
    - Installing and backing up/restoring Swarm, UCP, DTR (need to be done separately. First Swarm, than UCP)
        - DTR - can be made HA using shared storage. DTR backup doesn't include images as the backup of the storage backend is considered separate. 
    - [Disaster Recovery for UCP & DTR](https://docs.docker.com/datacenter/ucp/2.2/guides/admin/backups-and-disaster-recovery/)
    
17. Enterprise-grade features

    - UCP 
        - RBAC
        - LDAP integration
    - Docker Content Trust (DTC)
        - all images are verified
        - can set up build pipelines that only promote the image if it passes scanning
    - HTTP routing mesh: Swarm Routing Mesh is layer 4 so balances load w/o knowledge of the app. UCP implements the HTTP Routing Mesh (HRM) which implements a layer 7 routing mash
