---
layout: post
title: "Virtualized Web Automation Stack: Vagrant + Ubuntu + LXC + Headless FF/Chrome + Selenium Grid"
comments: true
permalink: virtualized-web-automation-stack
---

Creating a robust and scalable test automation process is a very essential phase. After automated tests were developed, they need to be executed frequently and rapidly. In automated tests for web, there is an additional need to cover different browsers and platforms. 
In this tutorial we are going to show, step by step, how to setup a fully functional web automation execution process in a virtualized environment.

LXC will be used to bring up multiple light-weight virtualizated machines for executing tests over different headless browsers. Selenium Grid will manage the execution of these tests. Vagrant will help us create a portable and self-contained execution environment, all stacked up in a single VM.

![Vagrant and LXC](/assets/vagrant_lxc.png)

## Setting up main station (Virtual Box + Vagrant with Ubuntu)

Vagrant is an open source tool for managing virtual machines. It supports different providers such as VirtualBox, VMWare, AWS and OpenStack.
It basically adds an additional layer to VM creation and management, making the whole process much easier and flexible.
Vagrant uses "boxes" to package a virtual machine along with its configuration. You can provide anyone with a Vagrant "box" so they can bring up an identical working environment, across different providers.

[Install VirtualBox](https://www.virtualbox.org/wiki/Downloads)
[Install Vagrant](http://www.vagrantup.com/downloads.html)

Once above are installed, we should initialize a new vagrant VM using following command:
	
	vagrant init
	vagrant up

Now you should have a new Ubuntu VM up and running. You can SSH to new VM using command:
	
	vagrant ssh 

Note: This step is optional. If you already have an Ubuntu machine, you can use it to bring up all necessary environment in next steps.However, using Vagrant has many benefits such as providing portable environments, source control of configurations and easy customizations of VMs.

## Installing LXC

LXC (Linux Containers) is a virtualization method at OS-level. It allows to run multiple Linux virtualized systems (containers) on a single parent machine (host). LXC is a light-weight alternative to hypervisors virtualization such as VMWare ESXi or KVM.

We can setup multiple isolated Linux instances inside a single LXC host. Each instance (container) will have its own IP address, local file system, services, and memory/CPU allocation.

Let's start by installing LXC package in our Ubuntu VM. This package will install all LXC related files and configurations. 
	
	sudo apt-get install lxc

## Creating containers

After all required packages for LXC support are installed, we can start creating containers. Containers are created using "lxc-create" command.
Following call will install a basic Ubuntu container named "c1" inside VM.
#lxc-create -n c1 -t ubuntu

You can see the current status of installed containers using "lxc-ls --fancy"  command.
 
NAME  STATE    IPV4  IPV6  AUTOSTART
------------------------------------
c1    STOPPED  -     -     NO

Notice that initial status of container is to be stopped. We can use "lxc-start" command to start container:
	lxc-start -n c1 -d

This command will bring up our "c1" container in the background. Let's take a look again at status of containers:
	lxc-ls --fancy
	
NAME  STATE    IPV4       IPV6  AUTOSTART
-----------------------------------------
c1    RUNNING  10.0.3.31  -     NO

Notice how container has been started and assigned a new IP address.

We can repeat above procedure to install more containers. 
 
Each container comes with SSH access. You can SSH from LXC host to any container using lxc-console command or using an SSH client inside VM.

## Installing headless browser (Firefox or Chrome)

Since containers don't have a display (SSH access only), we will launch browsers in headless mode during automated tests. It means that browsers will run on a fake display, using xfvb as a display server. No worries, we are not testing any fake browsers. We are only using an X-Server which doesn't produce any output.
 
Below installations should be done on all containers, since they will launch the browsers. 

Install xfvb server:
	sudo apt-get install xvfb
 
Install Firefox:
	sudo apt-get update 
	sudo apt-get install firefox
 
Install Chrome: 
	wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | sudo apt-key add -
	sudo sh -c 'echo "deb http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google.list' 
	sudo apt-get update
	sudo apt-get install google-chrome-stable
 
Start xfvb server with aribtary display number (here used 8)
	Xvfb :8 -ac 

Set the DISPLAY environment variable with the display number we used to start xfvb server:
	export DISPLAY=:8

Launch Firefox browser headlessly:
	firefox

If you didn't get any error during launch, then Firefox will be running in a headless mode.

## Installing Selenium Grid

Selenium Grid is used to run Selenium/WebDriver based tests on different machines, browsers and operating systems in parallel. It makes the whole process of automation tests execution more robust and scalable.
Selenium Grid operates in a hub-node mode. A hub is the main station which runs the tests and distribute their execution over the nodes. Each node may have different browsers and run on different platforms.

In our environment, the LXC host will act as a hub and the containers will act as nodes. Each container node will use a different browser when executing tests from hub.

IMAGE

First, we need to download Selenium Server and install it on both hub and nodes. Since Selenium Server is a Java application, we also need to install JDK in all machines (you will also need it to build your Java tests and run them from hub).

Install latest JDK version:
	sudo apt-get install default-jdk

Selenium Server can be downloaded from this link to any preferred location in LXC host and containers.
	wget http://selenium-release.storage.googleapis.com/2.43/selenium-server-standalone-2.43.0.jar
 
After we are done installing JDK and Selenium Grid, we can start bringing up the hubs and nodes.

Inside the LXC host we launch a hub:
	java -jar selenium-server-standalone-2.43.0.jar -role hub
root@vagrant-ubuntu-trusty-32:/home/vagrant# java -jar selenium-server-standalone-2.43.0.jar -role hub
20:54:05.800 INFO - Launching a selenium grid server
 

If everything went fine, you should be able to access following URL (port 4444 is the default port bind for Selenium Grid server) 
http://LXC-HOST-IP:4444/grid/console

IMAGE

After our hub is ready, we should go over containers and launch the nodes:
#java -jar selenium-server-standalone-2.30.0.jar -role node -hub http://LXC-HOST-IP:4444/grid/register 
ubuntu@c1:~$ java -jar selenium-server-standalone-2.43.0.jar -role node -hub http://10.0.2.15:4444/grid/register
21:02:35.042 INFO - Launching a selenium grid node
 

When we launch the nodes on all containers, we should see them in Grid's web interface

IMAGE

## Running tests with Grid

Now that all nodes are ready, we can start running the tests on hub.

A minor modification of WebDriver initialization is required to run the tests over grid. Instead of creating a specific WebDriver browser instance such as FirefoxWebDriver or ChromeWebDriver, we should use RemoteWebDriver and DesiredCapabilites instead.

RemoteWebDriver -  used to execute the tests remotely on nodes.
DesiredCapabilities - used to define the browser, version and platform that node should match.

RemoteWebDriver instance is initialized with URL of hub and the DesiredCapability object. For example:
	DesiredCapabilities capability = DesiredCapabilities.firefox();
	WebDriver driver = new RemoteWebDriver(new URL("http://LXC-HOST-IP:4444/wd/hub"), capability);

A node that matches the criteria in DesiredCapabilities will be chosen by RemoteWebDriver to run the tests.

Below is a complete example of a simple WebDriver test that uses Selenium Grid. 

EXAMPLE
 
More to the stack

Tests can be built and run using different test methods. For example: combination of Maven as a build tool and JUnit/TestNG as a testing framework. Maven is a powerful tool for building Java projects and dependency management.
A configuration management tool such as Puppet or Chef can be used to install and manage all needed packages for setting up the environment. Vagrant even has built-in provisioning support for such tools, which makes it easier to configure Vagrant boxes using Chef recipes or Puppet. 
Additionally, you may setup a Jenkins build server on LXC host (hub) to run tests in an orderly manner using scheduled jobs for Continuous Integration.

Now you should have a fully functional environment for running automated web tests over different machines and browsers in parallel. All these features are stacked inside a single portable VM which can be scaled up according to execution needs.

