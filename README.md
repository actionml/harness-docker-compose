# Simple Harness with Docker-compose

This document describes using Simple Harness **without TLS and Authentication**.

There are 2 types of Harness docker-compose installations:

 - **Simple Harness**: Harness + The Universal Recommender **without TLS or Authentication**. Actually all engines are installed but the CBEngine will not operate correctly since Vowpal Wabbit is not included. This type of installation is described in [README.md](https://github.com/actionml/harness-docker-compose/blob/master/README.md) for this repo.
 - **Full-stack Harness**: Harness + all engine types + VW + TLS + Authentication. This installation is simple but configuration is considerably more complicated due to TLS + Authentication. This is due to creating certificates, keystores, and different user types. This type of installation is described in [Full Stack Harness with Docker-compose](https://github.com/actionml/harness-full-docker-compose/blob/master/README.md)

With docker-compose Harness and all services it depends on run in Docker Containers. even the harness-cli is installed in its own container. This makes it fairly easy to install on a single machine for experiments or when only vertical scaling is required.

# Simple Docker-Compose: UR only, no TLS, no Auth, no VW

![](https://docs.google.com/drawings/d/e/2PACX-1vRja3fTemDMe_0AA8DMMX5fkU-TrI9uTKXJYQJY2-WMyspTjdRVdGGwtcD_wpgvCmh4snFblZC7dhdr/pub?w=1193&h=758)

Make sure you have a running Docker Engine daemon with CLI installed. See instructions [here](https://docs.docker.com/install/).

## Configure

We need to map container directories into the host filesystem for all of the composed containers. These are configured by copying `.env.sample` to `.env`. This will set all persistence to the included `./docker-persistence` directory in the host file system. To change this edit `.env` and point stored files elsewhere.

## Deployment

To re-deploy over a previous version see **Updating**.

 - `cd harness-docker-compose`
 - configure the `.env` file **Not ready yet!**
 - `docker-compose up -d --build` for first time setup
 - `docker-compose up -d` if you want to re-launch the containers after `build` has been done at least once.

**Updating**: Once deployed one or more container in the collection can be updated. It is best to explore the docker-compose cli and options as well as docker commands. Some useful commands for updates are:
 
 - `docker-compose down` stops all container in the local yaml file. Do this before any other docker-compose updates.
 - `git origin <branch>` for this repo the lastest vesion under test is in branch `develop`, the last stable release is in `master`
 - `docker-compose pull` this will get all updated containers that are available. This is usually safe since these conatianers are tested together and so are in sync.
 - `docker-compose up -d --build --force-recreate` to bring up all updated containers by recreating all images.

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

The way Docker supports persistence uses a mapping of container internal file system to the host's file system. Using git to pull the repo will prepare directories on the host for persistence. Look in `harness-docker-compose/docker-persistence/...` for the data and logs of the containers.

## Harness CLI

The Harness-CLI is also started in a container. To use it, log-in.

 - `docker-compose exec harness-cli bash`

    This starts a `bash` shell in the container, configured to communicate with the Harness container
    
 - `docker-compose exec harness-cli bash -c 'harness status'`

    this will return the status of Harness

The [harness-cli](https://github.com/actionml/harness-cli) can also be installed on the host OS as desired. It uses the REST API to control Harness and so can be on any host that can connect to Harness even if it runs in a container.
