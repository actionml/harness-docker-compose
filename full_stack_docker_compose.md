# Full-stack Harness with Docker-compose

This document describes using the Full-stack Harness **with TLS and Authentication**.

There are 2 types of Harness docker-compose installations:

 - **Simple Harness**: Harness + The Universal Recommender **without TLS or Authentication**. Actually all engines are installed but the CBEngine will not operate correctly since Vowpal Wabbit is not included. This type of installation is described in [README.md](README.md) for this repo.
 - **Full-stack Harness**: Harness + all engine types + VW + TLS + Authentication. This installation is simple but configuration is considerably more complicated due to TLS + Authentication. This is due to creating certificates, keystores, and different user types. This type of installation is described in [The Guide to Full Stack Harness](full_stack_docker_compose.md)

We strongly suggest that you try the "simple" installation first!

Harness and all services it depends on run in Docker Containers. This makes it fairly easy to install on a single machine for experiments or when only vertical scaling is required.

To use any Engine with TLS and Authentication, you will need this version of docker compose.

![](https://docs.google.com/drawings/d/e/2PACX-1vSqEwan6xpPT5UKwv4f2HXGf19IpcP3kU8c2JARsl3GY6X0HJ5-3g1YshKUPfEnmt6msVoB-rZ5lUT9/pub?w=1193&h=758)

## Harness CLI

The Harness-CLI is also started in a container. To use it, log-in.

 - `docker-compose exec harness-cli bash`

    This starts a `bash` shell in the container, configured to communicate with the Harness container
    
 - `docker-compose exec harness-cli bash -c 'harness status'`

    this will return the status of Harness

The [harness-cli](https://github.com/actionml/harness-cli) can also be installed on the host OS as desired. It uses the REST API to control Harness and so can be on any host that can connect to Harness even if it runs in a container.

## Configure

Before you deploy the "full-stack" docker-compose network you will need to setup configuration for TLS. This means you will need to create a harness.jks file that has the TLS cert for your host. This must be put into some place on the host and the path to the jks file must be passed in to docker-compose. This way the key/cert file is outside of any git repo. 

**NOTE: If the key/cert file and the path passed to docker-compose harness will fail to launch. This is not yet defined for docker-compose**

### TLS/SSL

 - Get or create a certificate in the form of a `<some-cert>.pem` file
 - Create a Java KeyStore in `<some-keystore>.jks` with your certificate. This will require a password so make note of it.
 - put the path to these files in `.env` file of the repo
 - generating `<some-keystore>.jks` will require a password, add it to the `.env` file.

With the `.env` pointing to .jks and containing the password for the .jks Harness will start configured to use TLS.

### Authentication

The Harness Auth-Server is started with `docker-compose-cb.yml`. The Harness REST API groups all Engine Instance resources (Events, Queries, Jobs, etc) and enforces ALL access by the Roles and Permissions of the User credentials.

In order to use Authentication you must create an Admin User for the CLI. In addition this user's credentials have access to any Engine Instance and its sub-resources (Events, Queries, Jobs, etc). Access Permissions are defined for each Engine Instance.

Typically, once an Admin User is created it is desirable to create a Client User who has access to one or more Engine Instances. See [Security](https://github.com/actionml/harness/blob/develop/security.md)

The Auth-Server is a micro-service controlled by Harness so use the harness-cli to create users with permissions as described in [Commands](https://github.com/actionml/harness/blob/develop/commands.md) and [Security](https://github.com/actionml/harness/blob/develop/security.md)

## Deployment

This is the same as the "Simple" deployment but you use a different Yaml file with docker-compose:

 - `cd harness-docker-compose`
 - `docker-compose up -f docker-compose-cb.yml -d --build` for first time setup
 - `docker-compose up -f docker-compose-cb.yml -d` if you want to re-launch the containers after `build` has been done at least once.
 - `git pull && docker-compose -f docker-compose-cb.yml down && docker-compose -f docker-compose-cb.yml up -d --build --force-recreate` to update harness and takedown old containers and create new containers with new harness version
