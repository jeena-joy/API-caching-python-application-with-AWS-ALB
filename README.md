# API-caching-python-application-with-AWS-ALB

Dockerization of an api caching project in python. This project is designed with an nginx proxy loadbalancer to loadbalance the traffic among api containers and a Redis cache container for api caching. All the communications between the containers are through a bridge network inside the host server.

## Architecture

![api 002](https://user-images.githubusercontent.com/91469974/145708526-db34dd94-53b0-488e-9c87-3bc548670095.png)


## Features
IP-Location finding website (Python)
Easy to migrate everywhere
AWS ALB - load balanced
All containers are spin up with a single command

## Includes
Launch Configuration with the userdata provided.
Auto scaling group with capacity as 2
Target Group(worker server attached)
Application Load balancer
Elastic Cache

## How to install docker and docker-compose.

### Docker installation

```sh
yum install docker -y
yum install git -y
```

### docker-compose installation

```sh
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/bin/docker-compose
sudo chmod +x /usr/bin/docker-compose
docker-compose version 
```

## How to get IPstack API
Please go through the ipstack and click "GET FREE API KEY" on the top right corner.

![ipstack](https://user-images.githubusercontent.com/91469974/145708769-2c7561df-2fb3-48f8-a84d-c8b81c84af90.png)

A swarm is a cluster of one or more computers running Docker. It can consist of one or more machines, so you can use swarm functionality on your local machine. A swarm consists of multiple Docker hosts which run in swarm mode and act as managers and workers.

## Initializing Docker Swarm in an instance - Manager

```sh
docker swarm init
```

```sh
Swarm initialized: current node (h3hfjwi6wswhni0wo4ajh1yp5) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-123tuk27j2s4lgz18yu2h2oxy3wuie5g2c8n7ftfv3309581bl-1d4my4irhqq3omqqvbya2q5qb 172.31.36.35:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

To make the workers join in the swarm/cluster need to enter the command/token in the worker instance.

Here, I have provided this command/token in the userdata which is defined in the Launch Configuration.

## Userdata

```sh
#!/bin/bash

echo "ClientAliveInterval 60" >> /etc/ssh/sshd_config
echo "LANG=en_US.utf-8" >> /etc/environment
echo "LC_ALL=en_US.utf-8" >> /etc/environment

echo "password123" | passwd root --stdin
sed  -i 's/#PermitRootLogin yes/PermitRootLogin yes/' /etc/ssh/sshd_config
sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
service sshd restart

amazon-linux-extras install docker -y
systemctl restart docker.service
systemctl enable docker.service
docker -a -G docker ec2-user

docker swarm join --token SWMTKN-1-123tuk27j2s4lgz18yu2h2oxy3wuie5g2c8n7ftfv3309581bl-1d4my4irhqq3omqqvbya2q5qb 172.31.36.35:2377
```

## How to Use

- Installing service network

```sh
docker network create ipstackapp --driver=overlay
```

- Elastic cache is used in AWS for api caching. However, instead of Elastic cache can use a Redis container.

```sh
docker service create \
--name redis \
--replicas 1 \
--network alb \
redis:latest
```

- Creating ipstack container. These containers are created in any of the worker instance or even in the Manager instance.
- 
```sh
docker service create \
--name api \
--replicas 4 \
--network ipstackapp \
-p 80:8080 \
-e CACHING_SERVER=caching \
-e IPSTACK_KEY=418e**************************** \ # ==> Your ipstack API KEY
fujikomalan/ipstack:v1
```

```sh
# docker service ls
ID             NAME      MODE         REPLICAS   IMAGE                    PORTS
vt0mct1xjwcn   api     replicated   4/4       fujikomalan/ipstack:v1   *:80->8080/tcp

```

## Conclusion
Here is a simle API caching python application with AWS ALB
