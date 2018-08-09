# Docker Deep Dive - Chapter 10 - Swarm

Two main components:
- A secure cluster
- An orchestration engine

## TLDR

Enterprise-grade secure clouster of Docker hosts

Engine for orchestrating apps in containers
  - encrypted distributed cluster store
  - encrpyted networks
  - mutual TLS
  - secure cluster join tokens
  - PKI that makes managing and rotating certificates easy
  - Add/remove nodes non-disruptively

Swarm API that allows deploying and managing microservice apps with ease

Fully integrated into Docker since 1.12. Enabled in a single command `docker swarm init`

## Deep Dive

Build cluster
Deploy some swarm services
Troubleshooting

### Swarm Primer

- Swarm cluster: 1 or more Docker nodes

- Nodes - servers that are managers or workers

- Managers - look after the control plane - state of the cluster, dispatching tasks to workers

- Workers - accept tasks from managers and execute them. Where apps typically run (not best practice to run on manager to keep it clean)

- Configuration store - etcd, on all managers. In memory and installed automatically.

- TLS tightly integrated - can't build a swarm without it. 
  - encrypt communications
  - authenticate nodes
  - authorize roles
  - automatic key rotation

- Service - atomic unit of scheduling. Wraps advanced features around containers.
  - when deployed, called a task
  - adds things like scaling, rolling updates, simple rollbacks
  - IE. Web front-end service
    - replicas: 5
    - update policy
    - rollback policy

### Building a Swarm

*Full steps: Page 187*

1. Initializing a swarm - initialize first manager node:

`docker swarm init`

On a Docker host, this will convert it from single-engine mode to **swarm mode**. 

Give optional IP for other nodes to communicate with --advertise-addr flag. --listen-addr option useful for when advertised address is a remote IP like a load balancer. *Recommended to use both flags*

Default port swarm mode operates on is 2377.

List the nodes with `docker node ls` - you'll see the manager.

1. Join additional manager nodes

First run the `docker swarm join-token` command on the manager to get the join token & command. **Protect the tokens**.

1. Join worker nodes

### Swarm manager high availability (HA)

Active-passive multi-manager HA - only 1 manager is ever considered active, the *leader*. Only one to issue live commands against the swarm. Passive managers will proxy commands to the leader if they receive them.

Raft consensus algorithm is used for HA. Best practices:
  - odd number of managers to prevent split brain conditions (on network partition with 4 managers then 2 on each side, there could be no consensus, knowing there were 2 on the other side but can no longer communicate)
  - don't deploy too many managers (3-5) to keep time low to achieve consensus

### Locking a swarm

- Autolock - managers must present unlock key when restarted to rejoin, since they can access to Raft log time-series database

Pass --autolock flag when creating swarm or update it with docker swarm update --autolock=true. Record the key.

### Swarm services

Services get improved on by stacks.

Specify name, post, mappings, attaching to networks, images

*Declarative* - feed desired state to docker and let it deploy and manage

`docker service create --name web-fe -p 8080:8080 --replicas=5 nigelpoulton/pluralsight-docker-ci`

All service replicas use the same image and config.

Swarm deploys 5 replicas across swarm and monitors them in *reconciliation loop* comparing actual state to desired state.

Inspect services `docker service ls`, see replicas, `docker service ps web-fe`, `docker service inspect --pretty web-fe` for detailed info & seeing what's under the hood.

Replicated vs global state - replicated is default, runs across nodes as equally as possible. Global deploys 1 on every node.

#### Scaling a service

Up the desired state:

`docker service scale web-fe=10`

- Rolling Updates - updates desired state

`docker service update --image...`

- Ingress mode - default. Every node gets mapping so can redirect requests to node that runs needed service

## Troubleshooting

`docker service logs` 

Log driver must support it.









