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

    Docker works with its own and latest OS tech for security.

    - Docker tech: secrets management, docker content trust, security scanning
    - OS (linux) tech: seccomp, mandatory access control, capabilities, control groups, kernel namespaces
        - Docker utilizes these namespaces: pid, net, mnt, ipc, user, uts
        - All new containers get a sensible default seecomp profile
    - Rotate swarm join token, `docker swarm join-token --rotate manager`

16. Tools for the enterprise

    - Installing and backing up/restoring Swarm, UCP, DTR
    - [Disaster Recovery for UCP & DTR](https://docs.docker.com/datacenter/ucp/2.2/guides/admin/backups-and-disaster-recovery/)
    
17. Enterprise-grade features

    - UCP RBAC, Docker Content Trust (DTC), HTTP routing mesh