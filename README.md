# Vagrantbox Ubuntu DockerSwarm

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## Prerequisites
* Vagrant - https://releases.hashicorp.com/vagrant
* Virtualbox - http://download.virtualbox.org/virtualbox

## How to use?

1. Clone https://github.com/spy86/vagrant-DockerSwarm
2. Run `vagrant up` from terminal and wait while virtual machine is installed.

# Customize

By default `vagrant up` spins up 3 machines: `manager`, `worker1`, `worker2`. You can adjust how many
workers you want in the `Vagrantfile`, by setting the `numworkers` variable. Manager, by default, has address "192.168.10.2", workers have consecutive ips. 

```ruby
numworkers = 2
```

If your provisioner is `Virtualbox`, you can modify the vm allocations for memory and cpu by changing these variables:

```ruby
vmmemory = 4096
```

```ruby
numcpu = 1
```


`/etc/hosts` on every machine is populated with an IP address and a name of every other machine, so that names are resolved within the cluster. This mechanism is not idempotent, reprovisioning will append the hosts again. 

# Play with this box

After starting swarm, you can use my testing Docker image to play with.

Go to the master node and start docker swarm:

```bash
(host)# vagrant ssh manager
(manager)# docker swarm init --advertise-addr 192.168.10.2

docker swarm join \
--token SWMTKN-1-59h28hcbb8gzs2xs24oyh7hjvc7fp8skjzvnpw9cksmp96m4y2-35er9ai3u1f1ae5esb7x8l1hx \
192.168.10.2:2377
```

Now join the swarm on the nodes with the command from the manager, do it on both nodes.

```bash
vagrant@manager:~$ docker node ls
ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
0rtuyz07e0wazmxvoed1llmx3 *  manager   Ready   Active        Leader
1iab1w9m5znyzmh7hpzyn7rrw    worker2   Ready   Active
4mmca5rxrxxhb0tb7s5du5hpe    worker1   Ready   Active
```
Now we create a web service, with 1 replica, on any of the nodes:

```bash
docker service create --name webservice --replicas 1 --publish 8080:8080 nginxdemos/hello
```

You can see the status od the service with `docker service ps webservice`. 

```
vagrant@manager:~$ docker service  ps web
ID                         NAME   IMAGE            NODE     DESIRED STATE  CURRENT STATE           ERROR
7m0s3ikyjyvps4xgpk655uv9k  webservice.1  nginxdemos/hello  manager  Running        Running 16 seconds ago
```

We can also scale up the service if we want:

```bash 
docker service scale webservice=4
```
