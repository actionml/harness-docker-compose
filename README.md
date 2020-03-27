# Harness with Docker-compose
**Version 1.1-SNAPSHOT**

With docker-compose Harness and all services it depends on run in Docker Containers, even the harness-cli is installed in its own container. This makes it fairly easy to install on a single machine for experiments or when only vertical scaling is required.

# Harness + UR

![](https://docs.google.com/drawings/d/e/2PACX-1vQocPhRrFn1TJAeWdcr2v9oD7_T9C261DLTwqueoESuubOcQtk1Iv7KZt6M7sSdvqocl8fSvtU6bf_K/pub?w=1193&amp;h=758)


# Prerequisites

 1. Install the Docker components for managing containers. This should be done as a regular user on the host -- a non-root user with passwordless sudoer permissions. See instructions [like these](https://docs.docker.com/install/) or as appropriate for your "host" OS. You'll need from Docker:
     - Docker itself, including whtever is needed to host running containers.
     - Docker-compose, some extensions that allow a network of containers to be run on a single host. 
 2. Although this project MAY work on Windows it has not been tested and the examples commands below assume a 'nix style command shell like bash.

# Configure

Map container directories into the host filesystem for all of the composed containers.

 - `cd harness-docker-compose`
 - `cp .env.sample .env`
 - edit the `.env` file if the defaults are not adequate. 

One important thing to note is that in order to import files using `harness-cli import <engine-id> some/path/to/json` the path to the json must be resolved in the harness container AND the harness-cli container will need a place to persist engine json files. This is solved by mapping a host directory into both containers (for convenience) like this:

![](https://docs.google.com/drawings/d/e/2PACX-1vQFb4EfmP6Ocy1UxjqBd8bVPFVumIIY_vrgDO8i5zvmrvwporCpG2O3L9ZKhsiZl3N0zO_SWKuFZ4Nt/pub?w=1123&h=271)

# Deployment

With the docker daemon running:

 - `docker-compose up -d --build` for first time setup

Once deployed one or more containers in the collection can be updated. It is best to explore the docker-compose cli and options as well as docker commands. Some useful commands for updates are:
 
 - `docker-compose down` stops all container in the local yaml file. Do this before any other docker-compose updates.
 - `git pull origin <branch>` for this repo the lastest vesion under test is in branch `develop`, the last stable release is in `master`. The `git` repo contains the latest project structure and `docker-compose.yml`.
 - `docker-compose up -d --build --force-recreate` to bring up all updated containers by recreating all images.
 - `docker-compose pull` is a very important command that will get the latest image version from the ActionML automated CI/CD pipeline. **Note**: this project uses a possibly unstable develop/SNAPSHOT version of Harness. To change this, edit docker-compose.yml and change the versions to `harness:latest` and `harness-cli:latest`, which will get stable released versions.

# Operations

Once installed the containers work somewhat like a cluster of virtual machines all running on a single host. You can login to them, examine logs, and start and stop them.

## Upgrades Experimental

**Note:** This project uses [watchtower](https://containrrr.github.io/watchtower/) to monitor the image tagged actionml/harness:develop When it is updated the new image will be automatically pulled and deployed. This may not fit your use case and since the "develop" image is targeted this will pull **unreleased** code! To change this, fork the project and edit docker-compose.yml to target any supported image tag, like actionml/harness/latest to get the latest release. 

This is not a thorough upgrade mechanism since some migration of data may be required during an upgrade so beware anything but experimental use of this feature based on watchtower.

## Upgrades Stable

To use this docker-comnpose project reliably is is best to target a named version of Harness tagged with 0.5.1 (due for release Feb 28 2020) or later and only upgrade manually by updating the image tag you have in docker-compose.yml and following Docker instructions for things like:

 - `docker-compose pull`
 - `docker-compose down`
 - `docker-compose up -d --build --force-recreate`

 Be aware the this may be dangerous if Harness schemas have changed, consult release notes for your version and the version you wish to use

## Logs

Harness logs are in the `docker-persistence/harness/...` directory and can be `tail`ed on the host as any local log file. Other containers may have logs available by using:

    docker-compose logs <some-container-id>

## Monitoring
   
Simple monitoring can be done by looking at memory, disk and CPU usage since all containers are running on the host.

To get more granular several tools allow monitoring individual containers.

# Persistence

The way Docker supports persistence uses a mapping of container internal file system to the host's file system. By default they will appear in `harness-docker-compose/docker-persistence/...` Be careful with these files, they will contain data for the database and elasticsearch. 

# Harness CLI

The Harness-CLI is also started in a container. To use it, log-in.

 - `docker-compose exec harness-cli bash`

    This starts a `bash` shell in the container, configured to communicate with the Harness container
    
 - `docker-compose exec harness-cli bash -c 'harness-cli status'`

    this will return the status of Harness

The [harness-cli](https://github.com/actionml/harness-cli) can also be installed on the host OS as desired. It uses the REST API to control Harness and so can be on any host that can connect.

# Versions

 - **1.1-SNAPSHOT**: Upgrades to Mongo 4.2

 - **1.0**: containers: 
     - harness develop
     - harness-cli develop
     - mongo:3.2
     - elasticsearch:7.6 