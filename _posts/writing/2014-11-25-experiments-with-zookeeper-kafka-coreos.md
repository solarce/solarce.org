---
layout: post
title: "Experimenting with Exhibitor/Zookeeper and Kafka on CoreOS"
excerpt: ""
categories: writing
tags: [zookeeper, exhibitor, kafka, coreos, docker]
comments: true
share: true
---

# Experimenting with CoreOS, Zookeeper, Exhibitor, and Kafka for Hacksgiving 2014

As part of [Hacksgiving 2014](http://hacksgiving.com/2012/11/what-is-hacksgiving/) at [Lookout](https://www.lookout.com) I experimented with seeing how hard it would be to get __Zookeeper__ and __Kafka__ working directly on [CoreOS](https://coreos.com).

## Deploying CoreOS

Deploying CoreOS was pretty easy. I opted to deploy the latest stable release and use their [CloudFormation](https://s3.amazonaws.com/coreos.com/dist/aws/coreos-stable-hvm.template) template as a starting place.

I customized it a bit to support being inside our VPC and just use our default security groups.

First I got a new [discovery token for etcd](https://coreos.com/docs/cluster-management/setup/cluster-discovery/), with `curl https://discovery.etcd.io/new` and then I filled in the other parameters in the CloudFormation template I made. I've put [an example template on Github](https://github.com/solarce/coreos_zk_kafka_notes/blob/master/coreos_cfn_template.json).

Then I just used the [CloudFormation GUI](https://console.aws.amazon.com/cloudformation/home?region=us-west-2#/stacks/new) in the AWS Console to launch a stack with my template. This built an [Auto-Scaling Group](http://docs.aws.amazon.com/AutoScaling/latest/DeveloperGuide/WhatIsAutoScaling.html) and then launched three EC2 instances for me.

Once all the instances were launched and I confirmed I could SSH into them, I added DNS records for them.

## Getting Zookeeper working

Launching services on CoreOS is pretty easy to get going, as it uses [Docker](https://docker.io/) containers to _package_ the services and [unit files](https://coreos.com/docs/launching-containers/launching/fleet-unit-files/) to define how a service should be run. But because CoreOS is designed to run more ephemeral workloads, it's kind of a challenge to run something like [Zookeeper](https://zookeeper.apache.org) by itself. Luckily the folks at Netflix released [Exhibitor](https://github.com/Netflix/exhibitor/wiki/Running-Exhibitor) to tackle this kind of challenge and uses S3 to share a config file for the Zookeeper cluster. So I made an S3 bucket that the IAM credentials I was going to use could write to.

Then I found a docker image that [packaged exhibitor and zookeeper together](https://github.com/thefactory/docker-zk-exhibitor) and I wrote a unit file to launch it. I then launched this with [fleetctl](https://coreos.com/docs/launching-containers/launching/launching-containers-fleet/), which is the tool CoreOS built on top of [systemd](http://www.freedesktop.org/software/systemd/) to manage launching services across a CoreOS cluster.

The unit file looked like:

~~~
[Unit]
Description=ex-zk1
After=docker.service
Requires=docker.service

[Service]
Type=Simple
TimeoutStartSec=0
EnvironmentFile=/etc/environment
ExecStartPre=-/usr/bin/docker kill ex-zk1
ExecStartPre=-/usr/bin/docker rm ex-zk1
ExecStartPre=/usr/bin/docker pull thefactory/zookeeper-exhibitor:latest
ExecStart=/usr/bin/docker run --name ez-zk1 \
                          -p 8181:8181 \
                          -p 2181:2181 \
                          -p 2888:2888 \
                          -p 3888:3888 \
                          -e S3_BUCKET=<BUCKET_NAME> \
                          -e S3_PREFIX=<KEY_NAME> \
                          -e AWS_ACCESS_KEY_ID=<ACCESS_KEY> \
                          -e AWS_SECRET_ACCESS_KEY=<SECRET_KEY> \
                          -e HOSTNAME=ez-zk1 \
                          thefactory/zookeeper-exhibitor:latest
ExecStop=/usr/bin/docker stop ex-zk1
~~~

### A single zookeeper!

I got a single node up and running pretty easily with the following commands:

1. `fleetctl submit ex-zk1.service`
2. `fleetctl start ex-zk1`
3. `fleetctl status ex-zk1`

I was then able to view the Exhibitor UI.

### A cluster of zookeepers!

A single zookeeper was easy, but when I launched three of them with the first iteration of my unit file, I found that each zookeeper instance would stick in standalone mode, despite them all being able to write to my S3 bucket.

After some troubleshooting I realized this was because the hostname I was giving to each container was not resolvable by the other containers, so the zookeeper services couldn't actually communicate and perform leader/follower elections. Since docker is binding the ports of each container to the IP of the CoreOS hosts, I found that if I told each container it had the hostname of it's CoreOS host, then election occured and Exhibitor was able to get all three members in sync and happy.

So the unit files then looked like:

~~~
[Unit]
Description=ex-zk1
After=docker.service
Requires=docker.service

[Service]
Type=Simple
TimeoutStartSec=0
EnvironmentFile=/etc/environment
ExecStartPre=-/usr/bin/docker kill ex-zk1
ExecStartPre=-/usr/bin/docker rm ex-zk1
ExecStartPre=/usr/bin/docker pull thefactory/zookeeper-exhibitor:latest
ExecStart=/usr/bin/docker run --name ez-zk1 \
                          -p 8181:8181 \
                          -p 2181:2181 \
                          -p 2888:2888 \
                          -p 3888:3888 \
                          -e S3_BUCKET=<BUCKET_NAME> \
                          -e S3_PREFIX=<KEY_NAME> \
                          -e AWS_ACCESS_KEY_ID=<ACCESS_KEY> \
                          -e AWS_SECRET_ACCESS_KEY=<SECRET_KEY> \
                          -e HOSTNAME=<DNS_NAME_OF_COREOS_HOST> \
                          thefactory/zookeeper-exhibitor:latest
ExecStop=/usr/bin/docker stop ex-zk1
~~~

To deploy two more nodes I just copied the `ex-zk1.service` file and tweaked the names and `HOSTNAME` to match where fleetctl would distribute the containers. I then launched `ex-zk2` and `ex-zk3` and had a three node cluster running.

Obviously this isn't ideal and support for more complex networking on CoreOS would be needed for a more production-like deployment, but for the purposes of this Hacksgiving it got what I wanted.

## Getting Kafka working

The end goal of my experimenting was a working three node Kafka cluster, but most of the work was getting Zookeeper working, because Kafka uses Zookeeper for its cluster discovery and leader election. So like with zookeeper, I found a docker image that looked good and made a unit file for it, like the one shown below:

~~~
[Unit]
Description=kafka1
After=docker.service
Requires=docker.service

[Service]
Type=Simple
TimeoutStartSec=0
EnvironmentFile=/etc/environment
ExecStartPre=-/usr/bin/docker kill kafka1
ExecStartPre=-/usr/bin/docker rm kafka1
ExecStartPre=/usr/bin/docker pull wurstmeister/kafka:0.8.1.1-1
ExecStart=/usr/bin/docker run --name kafka1 \
                          -p 6667:6667 \
                          -e KAFKA_ADVERTISED_PORT=6667 \
                          -e KAFKA_ZOOKEEPER_CONNECT=10.1.1.10:2181,10.1.1.12:2181,10.2.1.15:2181 \
                          -e KAFKA_BROKER_ID=100 \
                          wurstmeister/kafka:0.8.1.1-1
ExecStop=/usr/bin/docker stop kafka1
~~~

The main downside to the above unit file is that I'm hardcoding my zookeeper configs based on the IPs of the CoreOS hosts, ideally we'd tap into [etcd](https://coreos.com/docs/distributed-configuration/getting-started-with-etcd/) or something to manage this more dynamically.

Once I started one instance and confirmed it was up with `fleetctl` I launched two more and confirmed they all joined the cluster by checking the contents of `/brokers/ids` in zookeeper. Then I tested making a topic with three [topic partitions](https://confluence.corp.lookout.com/display/eng/Kafka#Kafka-Partitions) and a [replication factor](https://confluence.corp.lookout.com/display/eng/Kafka#Kafka-ReplicationFactor) of three. This would confirm that the cluster was truly working as expected.

~~~
bburton@althalus ~ # kafka-topics.sh --create --zookeeper coreos-1 --replication-factor 3 --partitions 3 --topic bburton.test3
Created topic "bburton.test3".
bburton@althalus ~ # kafka-topics.sh --describe --zookeeper lookout-coreos-hacksgiving1.flexilis.org
Topic:bburton.test3     PartitionCount:3        ReplicationFactor:3     Configs:
        Topic: bburton.test3    Partition: 0    Leader: 300     Replicas: 300,200,100   Isr: 300,200,100
        Topic: bburton.test3    Partition: 1    Leader: 100     Replicas: 100,300,200   Isr: 100,300,200
        Topic: bburton.test3    Partition: 2    Leader: 200     Replicas: 200,100,300   Isr: 200,100,300
~~~

## Conclusions

That's pretty much it for what I did during the hackathon!

I put up [samples of my unit files and the cloudformation template on Github as](https://github.com/solarce/coreos_zk_kafka_notes) well.

## PS. Taking it further

Some thoughts on how we could take it further. We'd need to figure out things like:

__Durable Storage__

- The [Launch Configuration](http://docs.aws.amazon.com/AutoScaling/latest/DeveloperGuide/WorkingWithLaunchConfig.html) I used didn't mount the ephemeral drives of each instance, nor did it add any EBS volumes. Additionally, the unit files I made don't use any [docker volume mounts](https://docs.docker.com/userguide/dockervolumes/) to let container data persist when instances are recreated.

__Complex Networking__

- Because of how the unit files are wrote are doing [docker port binding](https://docs.docker.com/userguide/dockerlinks/) we can only support one kind of service port CoreOS host. Figuring out more complex networking would allow us to support multiple services in different in clusters on the same CoreOS hosts.

__Unit File Management__

- More research into best practices for versioning and deploying unit files is needed.

__Thinking in a docker world__

- Using CoreOS brings docker and many things have to be rethought when in a docker world, such as
  - SSH
  - Logging
  - Storage Persistance
  - Etc

__Docker Image Hosting__

- Would we want to run a private registry to build and host our own docker containers?

__Monitoring of CoreOS and Containers__

- How do we best monitor CoreOS and the containers we run on it?

__Where does Chef fit in?__

- Building containers from Chef (and something like [Packer](https://packer.io)?)
- Deploying CoreOS with something like [chef-provisioning](https://github.com/opscode/chef-provisioning)?
