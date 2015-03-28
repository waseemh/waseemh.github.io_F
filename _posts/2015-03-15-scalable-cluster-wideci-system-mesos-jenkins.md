---
layout: post
title: "Cluster-wide continuous integration system with Mesos"
comments: true
permalink: cluster-wide-ci-system-mesos
---
In this paper you will be introduced to establishing a scalable, fault-tolerant and highly available distributed continuous integration system with efficient resource utilization and isolation. The whole ecosystem is composed of five major components: Apache Mesos, Apache ZooKeeper, Marathon, Jenkins and Docker.

## Ecosystem Overview
 
Apache Mesos is an open source cluster management software which provides dynamic resource sharing and isolation in a cluster environment.
Mesos abstracts resource sharing by aggregating all available resources in a cluster into one big compute unit. Below diagram describes resource sharing in Mesos, compared to static resource allocation:
Resource isolation (containerization in Mesos terms) is empowered by leveraging Linux modern kernel features (cgroups) to provide isolation of CPU, memory, IO and filesystem. Mesos currently provides native containerization support by Linux Containers, and Docker as an alternative.
Combining these two capabilities (resource sharing and isolation) results in a highly efficient resource utilization across a cluster of machines. 
Mesos operates in a master-slave model, such that a Mesos master manages Mesos slaves and Mesos applications (frameworks). Mesos master is also responsible for managing resources sharing by offering resources to its registered frameworks. Each Mesos framework is composed of two major components: 1) Scheduler (master-side) which receives resource offers from master and decides which resources to use. 2) Executor (slave-side): which launches framework's tasks in slaves.

In terms of fault-tolerance and high availability, multiple Mesos masters can be on standby in case of fail-over. Apache ZooKeeper is used to coordinate between Mesos masters, slaves and schedulers. Initially, ZooKeeper elects a single Mesos master to be the active master (leader). Once an active master fails, slaves and schedulers connect to the next-elected master by ZooKeeper. Aside from handling master failures, Mesos reports node failures and executor crashes to frameworks’ schedulers. Frameworks can then react to these failures using the policies of their choice. We will see later how specific frameworks implements high-availability for production use.
For more specific details on how Mesos implements high-availability, you may refer to following link. 
An in-depth technical specifications about Mesos in general, can be found in its research paper.

In our specific scenario, there are two relevant frameworks to run on top of Mesos: Marathon and Jenkins-Mesos plugin.

Marathon is "A cluster-wide init and control system for services". Mesos-wise, Marathon is a Mesos framework for long-running services. Unlike other application-specific frameworks (such as Hadoop or Spark frameworks), Marathon can schedule and launch tasks for any application service. Marathon ensures high-available services by automatically responding to service failures and keeping services always running (You can think of it as an "init.d" across the entire cluster). Moreover, multiple Marathon instances can be used to provide framework-level high availability.

Mesos Jenkins plugin is a Mesos framework for launching on-demand Jenkins slaves, based on current builds' queue. Whenever a new build for a specific Jenkins job is started, Mesos-Jenkins plugin will dynamically launch a Jenkins slave on a cluster node and execute the required job. Note that unlike standard master-slave Jenkins setup, there are no predefined Jenkins slaves on a Mesos-Jenkins setup. Jenkins master is able to schedule jobs on dynamically-launched slaves.

## Putting it all together

Entire flow can be described according to below:

1- Launch a Jenkins master instance as a service on a Mesos slave using Marathon framework.

2- Install Mesos plugin on Jenkins master and register it as Mesos framework in Mesos master. Jenkins framework scheduler can now receive resource offers from Mesos master.

3- Configure Jenkins jobs to schedule build slaves from Mesos cluster.

4- Once Jenkins job is triggered, the framework scheduler will accept a resource offer and send its task information to Mesos master.

5- Mesos master will launch Jenkins task on a Mesos slave through framework executor. The task would basically be a Jenkins slave JNLP launch command execution.
