# Install Using Docker-Compose

Harness and all services it depends on run in Docker Containers. This makes it fairly easy to install on a single machine for experiments or when only vertical scaling is required.

The Docker maps certain directories inside containers into the host filesystem so you can configure and manage persistence for the databases, and see logs.

There are 2 forms of the Docker Compose, one with all required services including the Harness-Auth Server and Vowpal Wabbit for the Contextual Bandit Engine.

## Installation

 - Install [Docker](https://docs.docker.com/install/) for your host. Get the latest stable version of the "Docker Engine" since your host will be running a composed container network.
 - Get this repository using Git: `git clone https://github.com/actionml/harness-docker-compose.git`
 - There are several docker-compose yaml files for different versions of Harness with optional components. They all use some form of `docker-compose ... up` as the only thing needed to launch your container network.

## Configuration

To configure the docker-compose network, see the section for the flavor of Harness that you are launching in this doc.

## Deployment

For any docker-compose you will launch a first time while building the containers, then you can rebuild or relaunch or login to containers as desired. The network of containers is set to reboot when the host reboots and to re-launch if a container crashes.

 - `cd harness-docker-compose`
 - `docker-compose up -d --build` for first time setup
 - `docker-compose up -d` if you want to re-launch the containers after `build` has been done at least once.
 - `git pull && docker-compose down && docker-compose up -d --build --force-recreate` to update harness and takedown old containers and create new containers with new harness version

To control a container network it is best to read the [Docker](https://docs.docker.com/) and [Docker-compose](https://docs.docker.com/compose/overview/) documents. They contain many helpful and powerful techniques. 

**NOTE**: Add `-f docker-compose-cb.yml` to launch the full-stack version of Harness, this uses a slightly different config file.

## Operations

Once installed the containers work like a cluster of machines all running on a single host. You can login to them, examine logs, and start and stop them.

### Logs

For instance logs can be view by using:

    docker-compose logs <some-container-id>
    
This will show a screen full of logs, use

    docker-compose logs -f <some-container-id>
    
in place of `tail`ing to stream incoming logs to the console continuously.

If you want to examine historical logs look into the persistent files stored in `harness-docker-compose/framework/...` for the log files of the different services running in containers. 

### Monitoring
   
Simple monitoring can be done for that host looking at memory, disk and CPU usage since all containers are running on the host.

To get more granular several tools allow monitoring individual containers.

## Persistence

The way Docker supports persistence uses a mapping of container internal file system to the host's file system. Using git to pull the repo will prepare directories on the host for persistence. Look in `harness-docker-compose/framework/...` for the data, conf, and logs of the containers. They are written or read in realtime.

# Simple Docker-Compose: UR only, no TLS, no Auth, no VW

![](https://docs.google.com/drawings/d/e/2PACX-1vRja3fTemDMe_0AA8DMMX5fkU-TrI9uTKXJYQJY2-WMyspTjdRVdGGwtcD_wpgvCmh4snFblZC7dhdr/pub?w=1193&h=758)

Make sure you have a running Docker Engine daemon with CLI installed. These instructions assume you are using Ubuntu. **Note**: Docker for macOS and Windows runs in a VM and so has subtle differences that may cause some problems. Please report these in the repo issues list.

## Configure

The compose will map container directories into the host filesystem for all of the composed containers. These are prepared by cloning the `harness-docker-compose` repo [here](https://github.com/actionml/harness-docker-compose). For a simple localhost installation no further config is required.


# Full Stack Docker-Compose

To use any Engine with TLS and Authentication, you will need this version of docker compose.

![](https://docs.google.com/drawings/d/e/2PACX-1vSqEwan6xpPT5UKwv4f2HXGf19IpcP3kU8c2JARsl3GY6X0HJ5-3g1YshKUPfEnmt6msVoB-rZ5lUT9/pub?w=1193&h=758)

## Configure

Before you deploy the "full-stack" docker-compose network you will need to setup configuration for TLS. This means you will need to create a harness.jks file that has the TLS cert for your host. This must be put into some place on the host and the path to the jks file must be passed in to docker-compose. This way the key/cert file is outside of any git repo. 

**NOTE: If the key/cert file and the path passed to docker-compose harness will fail to launch.**

The compose will map container directories into the host filesystem for all of the composed containers. These are prepared by cloning the `harness-docker-compose` repo [here](https://github.com/actionml/harness-docker-compose). For a simple localhost installation no further config is required.

## Deployment

This is the same as the simple-docker-compose above but you pass a different Yaml file to docker-compose:

 - `cd harness-docker-compose`
 - `docker-compose up -f docker-compose-cb.yml -d --build` for first time setup
 - `docker-compose up -f docker-compose-cb.yml -d` if you want to re-launch the containers after `build` has been done at least once.
 - `git pull && docker-compose -f docker-compose-cb.yml down && docker-compose -f docker-compose-cb.yml up -d --build --force-recreate` to update harness and takedown old containers and create new containers with new harness version

# Using the Harness CLI

Harness must be told which Engines to use and how to configure them. To do this we use the Harness CLI. It communicates with a Harness Server via its REST API. Install following instructions in the README.md for the [repo here](https://github.com/actionml/harness-cli)

The default config points to Harness on `localhost:9090` which should be fine for the simple docker-compose method of deployment. 


Some further notes may be useful in [Alfonso-notes.md](Alfonso-notes.md)
