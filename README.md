# Zookeeper Cluster with Docker

# Prerequisite

On docker host

```
sudo apt-get update && sudo apt-get install -y git curl wget build-essential
curl -sSL https://get.docker.com/ | sh
sudo usermod -aG docker ubuntu
```

# Build

Build docker image

`docker build -t zookeeper ./zookeeper`

```
Sending build context to Docker daemon 5.632 kB
Sending build context to Docker daemon 
Step 0 : FROM ubuntu:vivid
 ---> 013f3d01d247
Step 1 : RUN apt-get update  && apt-get -y install git ant openjdk-8-jdk  && apt-get clean
 ---> Using cache
 ---> 7a34981b848f
Step 2 : RUN mkdir /tmp/zookeeper
 ---> Using cache
 ---> 2990200607ee

...

Step 13 : ADD supervisor.conf /etc/supervisor/conf.d/supervisor.conf
 ---> 62881361e5fd
Removing intermediate container e66ba950ebfa
Step 14 : CMD "/usr/bin/supervisord"
 ---> Running in f1100af587b0
 ---> 29b0fbb36256
Removing intermediate container f1100af587b0
Successfully built 29b0fbb36256
```

# Deploy

Create first container

`docker run -d -e "ZK_NODE=1" --name zk1 zookeeper`

Create environment parameter

`ZK_IP=docker inspect --format '{{ .NetworkSettings.IPAddress }}' zk1`

Create your second container using zk1 instance IP and incrementing each node

`docker run -d -e "ZK_NODE=2" -e "ZK_IP=${ZK_IP}" --name zk2 zookeeper`

`docker run -d -e "ZK_NODE=3" -e "ZK_IP=${ZK_IP}" --name zk3 zookeeper`

`docker run -d -e "ZK_NODE=4" -e "ZK_IP=${ZK_IP}" --name zk4 zookeeper`

Check number of nodes using zkCli

`docker exec -it zk1 bin/zkCli.sh -server ${ZK_IP}:2181 config| grep ^server`

```
server.1=172.17.0.24:2888:3888:participant;0.0.0.0:2181
server.2=172.17.0.25:2888:3888:participant;0.0.0.0:2181
server.3=172.17.0.26:2888:3888:participant;0.0.0.0:2181
server.4=172.17.0.27:2888:3888:participant;0.0.0.0:2181
```

Check container processes spawned

`docker ps -a`

```
CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS              PORTS               NAMES
fc69330fbedd        zookeeper           "/bin/sh -c \"/usr/b   5 minutes ago       Up 5 minutes                            zk4                 
468c86763a25        zookeeper           "/bin/sh -c \"/usr/b   9 minutes ago       Up 9 minutes                            zk3                 
299fce0e974a        zookeeper           "/bin/sh -c \"/usr/b   9 minutes ago       Up 9 minutes                            zk2                 
13ed2e9a4297        zookeeper           "/bin/sh -c \"/usr/b   12 minutes ago      Up 12 minutes                           zk1                 
```

# Validate

Check logs on first zk1 node

`ZK_HOST=docker inspect --format '{{ .Config.Hostname }}'`

`docker exec -it zk1 tail -f /var/log/zookeeper--server-${ZK_HOST}.log`

Check for Leader Election

`docker exec -it zk1 grep 'LEADER ELECTION' /var/log/zookeeper--server-${ZK_HOST}.log`

```
2015-09-16 04:53:43,045 [myid:1] - INFO  [QuorumPeer[myid=1](plain=/0:0:0:0:0:0:0:0:2181)(secure=disabled):Leader@412] - LEADING - LEADER ELECTION TOOK - 33
2015-09-16 04:56:46,869 [myid:1] - INFO  [QuorumPeer[myid=1](plain=/0:0:0:0:0:0:0:0:2181)(secure=disabled):Follower@66] - FOLLOWING - LEADER ELECTION TOOK - 9
```
