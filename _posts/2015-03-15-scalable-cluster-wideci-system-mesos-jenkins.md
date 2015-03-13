
In this paper you will be introduced establishing a scalable, fault-tolerant and highly available distributed continuous integration system with efficient resource utilization and isolation. The whole ecosystem is composed of five major components: Apache Mesos, Apache ZooKeeper, Marathon, Jenkins and Docker.

Ecosystem Overview
 
Apache Mesos
An open source cluster management software which provides dynamic resource sharing and isolation in a cluster environment. 
Resource sharing is  available resources in a cluster as a one big compute unit and enables resource consumption. Below diagram describes resource sharing in Mesos:
Resource isolation (containerization in Mesos terms) is empowered by leveraging Linux modern kernel features (cgroups) to provide isolation of CPU, memory, IO and filesystem. Mesos currently provides native containerization support by Linux Containers, and Docker as an alternative.
Combining these two capabilities results in a highly efficient resource utilization across a cluster of machines. 
For an in-depth technical specifications, you may refer to this paper.
