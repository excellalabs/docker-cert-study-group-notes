# Docker Deep Dive - Chapter 11 - Networking

## TLDR

Docker networking based on Container Network Model (CNM)
libnetwork is Docker's real-world implementation of CNM

## The Deep Dive

### Container Network Model (CNM)

Full spec: https://github.com/docker/libnetwork/blob/master/docs/design.md

Defines three building blocks

* Sandbox - isolated network stack.  Includes Ehternet interfaces, ports, routing tables, and DNS config
* Endpoints - virtual network interface; responsible for making connections
* Networks - software implementation of 802.1d bridge

Sandboxes are placed inside containers to provide network connectivity

### Libnetwork

Implementation of CNM
Written in Go and cross-platform
Originally, existed inside daemon and eventually refactored out as external library

### Drivers

Docker ships with several built-in drivers, known as native drivers 

On Linux: bridge, overlay, macvlan
On Windows: nat, overlay, transparent, l2bridge

Each driver charge of creation and management of all resources.

## Single-host Bridge Networks

What is it?
* single-host refers to single docker host and can only connect to containers on that host
* Bridge is a reference to 802.1d bridge (layer 2)

In terms of networking:  "a bridge network is a Link Layer device which forwards traffic between network segments. A bridge can be a hardware device or a software device running within a host machine’s kernel."
In terms of Docker, "a bridge network uses a software bridge which allows containers 
connected to the same bridge network to communicate, while providing isolation from 
containers which are not connected to that bridge network."

Docker on Linux uses built-in bridge driver for single host networks

Every Docker host gets default single-host bridge network.

With the command: `docker network ls`, you will see a bridge network listed with bridge driver.

The command: `docker network inspect bridge`, contains a lot of low-level details.

Default bridge network with Docker based on Linux bridge in the kernel called docker0.
* Container communicate with Docker bridge
* ...which communicates with docker0
* ...which communicates with Ethernet interface

You can create a new single-host bridge network.  Example `docker network create -d bridge localhost`
This will create a new Docker network that you can inspect; you can also use Linux brctl tool to inspect this new Linux bridge.
Containers created on a user-defined bridge network can ping each other by name.

**WARNING**:  Default bridge network does not support name resolution via Docker DNS service. 
(Is this why we define networks in docker-compose file?)

**IMPORTANT FOR EXAM**: Containers can communicate outside of bridge network using port mappings.
With port mapping, you can map container port to a port on Docker host.  For example, container running
webserver on port 80 can map the port 5000 on the Docker host.  

Example: `docker container run -d --name web --network localnet --publish 50000:80`

## Multi-host Overlay Networks

Allows single network to span multiple hosts
    * Allows containers on different hosts to communicate at layer 2
    * Ideal for container-to-container communication
    
 MACVLAN Driver
   * Purpose:  Allows containerized apps to communicate to external systems
   * Gives containers its own MAC and IP address (makes it appear as a physical device)
   * Doesn't require port mappings; container is connected through to host interface
   * *Negative*: Requires host NIC to be in promiscuous mode (not allowed on public cloud)
   * Good for corporate data centers; not good for public cloud
   
## Container and Service Logs for troubleshooting

General Advice: If connectivity issues, check daemon and container logs

See book on different locations logs are stored

Container Logs:
* To configure verbosity of daemon logs, edit daemon.json so that debug is set to true.  
* Restart Docker whenever daemon.json is changed.

Daemon Logs:


### Useful Links

1) Use Bridge Networks: https://docs.docker.com/network/bridge/
2) Overlay Networks: https://docs.docker.com/network/overlay/
3) Macvlan Networks: https://docs.docker.com/network/macvlan/
4) Layer 2 vs. Layer 3 Switch:  
https://community.fs.com/blog/layer-2-switch-vs-layer-3-switch-which-one-do-you-need.html

### Notes

Promiscuous Mode - _In networking terms, a computer having its network interface card set 
to “promiscuous mode” receives all packets on the same network segment. In “normal mode,” 
a network card accepts only packets addressed to its MAC Address._