---
layout: post
title: "Containerized Mobile Test Environment"
comments: true
permalink: containerized-mobile-test-environment
category: Mobile Testing
intro: "Creating an isolated test environment for mobile automation using Docker"
---

Setting up a stable test envirnoment for mobile automation is a hassle. There are many dependent components, trials and errors, and configuration to take care of, until you have a clock-work test environment. execute automated mobile tests continously without facing environment issues.

What if we can isolate the whole environment inside a package and use this package to create multiple instances of the same exact environment on-the-fly? Even better, what if environment can be wiped out once tests are finished, thus ensuring a clean test environment on each execution?


In this article, I'm going to explain how we use **Docker** to achieve this. In a nutshell, Docker software allows you to package all your applicative setup inside isolated units, called containers. For more information and installation instructions, refer to Docker website.

Since we use Selenium WebDriver automation library, a typical mobile test environment will be composed of an Android device/emulator and an Appium server. Selenium WebDriver client communicates with Appium server, which in turn interacts with Android emulator or device. 
    
### Creating Docker Container
A Docker container must be created from an image. A Docker image is a binary template which includes all required configuration to run a container. You can think of it as our package.

I've already created a Docker image which includes Appium Server and Android SDK Emulator. I also added a VNC server installation to view the emulator graphical interface once its started. Assuming docker is already installed, you can run a container based on this image:

```sh
$ docker run -d -p 4723:4723 -p 5900:5900 waseemh/docker-mobile-appium:v0.1
```

The -p flag indicates the ports we want to expose from the container outside to the host machine (the machine running the Docker software). Since Appium Server listens on a specific port inside the container (4723 by default), we should expose this port if we need to access it from outside the container. Same applies for VNC server (port 5900). For example, if Docker host machine is DOCKER_HOST then I will be able to access Appium Server running inside the container by address: DOCKER_HOST:4723. 

In case mutiple containers will be started at once, each container should expose a different port to the docker host machine. For example:

```sh
$ docker run -d -p 4723:4723 -p 5900:5900 waseemh/docker-mobile-appium:v0.1
$ docker run -d -p 4724:4723 -p 5901:5900 waseemh/docker-mobile-appium:v0.1
```

### Building Your Own Image


### Running Mobile Tests

Once container is started, Appium server and emulator will be running inside the container. Appium server will automtically detect and the running emulator and connects to it. 

Now you are able to run your Selenium WebDriver mobile tests by initalizing a RemoteWebDriver instance with the Appium server address. Since we exposed port 4723, you can initalize 
