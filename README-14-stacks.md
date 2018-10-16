# Docker Deep Dive - Chapter 14 - Deploying apps with Docker Stacks

## TLDR

Docker Stacks used for managing multi-service apps.
Based on Docker Compose file (but adds a few new things)

The Process
* Define the process in Docker Compose file
* Deploy the stack
* Manage the stack

See Fig 14.1 for a good overview of the relationship of containers, services, and stacks.

## The Sample Application

### Setup
1. Go to https://labs.play-with-docker.com/
2. Click on Login and select Docker  (If you don’t have a Docker login, you will need to get one)
3. Press Start button to start the session
4. Next to the button Instances, click on the wrench icon
5. Select “3 Managers and 2 Workers”
6. On the left hand side, select manager1.   
	Tip:  On Mac, select Alt+Enter  to enlarge the screen
7. Enter the command`git clone https://github.com/nigelpoulton/atsea-sample-shop-app`
8. cd into atsea-sample-shop-app and inspect the file docker-stack.yml. 

### Docker Stack file (at a high level)

Note the four sections:

* version  - must be 3.0 or higher
* services - where we define stack of services
* networks - definition of network
* secrets - define secrets the app uses

### Networks

Networks are created before secrets and services
By default, all networks are created used overlay driver.

### Secrets

In the sample app, secrets are defined as external.  They must exist before the app starts.

Secrets could be defined using file: <filename>, but then the secret must already exist on the host filesystem
as a plaintext file.

### Services

#### reverse_proxy

##### image key
*Important Fact #1*:  The image key is mandatory!!  By default, will pull image from Docker Hub.
*Important Fact #2*:  Stacks do not support builds.

##### ports key

Example: 80:80 maps port 80 on the swarm to port 80 on each service replica

Note the difference between ingress and host mode.
* ingress mode - port will be mapped and accessible from every node in the swarm, even if service is not runnning on a node
* host mode - ports are only mapped on nodes running the service

##### secrets

* Secrets get mounted into service replicas as a regular file.
* Name of file is the value of the target value; file will appear in replica under /run/secrets
* Linux mounts /run/secrets in memory (Windows does not)

#### database service

Note the placement constraint `node.role == worker`
This ensures that the service will only be run on a worker node.

#### appserver service

Note some of the keys under deploy:

* replicas - desired number of service replicas that will be deployed
* update_config - tells Docker what to do during in rolling updates.
    * parallelism - number of replicas to update at a time
    * failure_action - if an update fails, then will perform rollback based on the prior version of the service
* restart-policy - tells Swarm what to do if a container fails

IMPORTANT:  It is recommended to change the stack file first and then re-deploy the stack!

#### visualizer

* stop_grace_period: Amount of time Swarm waits to forcibly kill a terminated container (default is 10s)

#### payment_gateway

* has a constraint to run on a worker 
* Uses custom node label for hardening

## Deploying the Application

1.  Make sure you are on the manager1 node
2.  Enter the command `docker swarm init`  How come it didn't work?
3.  Enter command `docker node ls`
4.  Add a label to worker 1  `docker node update --label-add pcidss=yes worker1`
5.  Verify the node label `docker node inspect worker1`
6.  Set up the secrets

```
openssl req -newkey rsa:4096 -nodes -sha256 -keyout domain.key -x509 -days 365 -out domain.crt

docker secret create revprox_cert domain.crt

docker secret create revprox_key domain.key

docker secret create postgres_password domain.key

echo staging | docker secret create staging_token -
```

7. Deploy the application

`docker stack deploy -c docker-stack.yml seaStack`

To see the application in action, 

To see the list of stacks

`docker stack ls`

To see where services are running.   Gives an overview of every service in the stack.

 `docker stack ps seasStack`

At this point, go back to the main Docker playground page (you may need to hit Alt-Enter)
* Click on 8001 link to see the visualizer.
* Click on 443 link to see the app. Note: You will need to change URL to include https

To see the service logs

`docker service logs seaStack_database`

If more than one replica is running, you will see logs for both.

## Managing the Application

NOTE:  It's possible to use `docker service update` to make updates, but this is not the recommended best practice!

As an exercise, edit docker-stack.yml to update the number of replicas
* services.appserver.deploy.replicas=10

Redeploy by entering the deploy command.  This only update any changed configuration.

`docker stack deploy -c docker-stack.yml seaStack`

Go back to the visualizer to see the updated change.