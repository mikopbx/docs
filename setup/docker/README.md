---
description: MikoPBX Installation Guide using Docker container
---

# Docker container

{% hint style="danger" %}
The "**Host system**" must run on Linux 5+. Tested on Debian 11, Ubuntu 21.04, and Ubuntu Server 22.04 LTS.
{% endhint %}

MikoPBX can be run in Docker using two main methods. The first method involves running the container directly using a Docker command with the necessary parameters. The second method involves using Docker Compose, which simplifies managing multi-container applications and allows the entire configuration to be described in a yaml file, making the deployment and maintenance of the system more convenient.

<table data-view="cards"><thead><tr><th></th><th></th><th data-hidden data-card-cover data-type="files"></th><th data-hidden data-card-target data-type="content-ref"></th></tr></thead><tbody><tr><td><strong>Docker installation and creating a user and directories</strong></td><td>Commands to install Docker and Docker Compose and configuration before creating the container</td><td><a href="../../.gitbook/assets/install.png">install.png</a></td><td><a href="docker-installation-and-creating-a-user-and-directories.md">docker-installation-and-creating-a-user-and-directories.md</a></td></tr><tr><td><strong>Running MikoPBX in a container</strong></td><td>Instructions for running a ready-made MikoPBX container, creating a container from a custom image, and running it</td><td><a href="../../.gitbook/assets/docker.png">docker.png</a></td><td><a href="running-mikopbx-in-a-container.md">running-mikopbx-in-a-container.md</a></td></tr><tr><td><strong>Running MikoPBX using docker compose</strong></td><td>Instructions for running multiple MikoPBX instances on a single host using Docker Compose</td><td><a href="../../.gitbook/assets/compose.png">compose.png</a></td><td><a href="running-mikopbx-using-docker-compose.md">running-mikopbx-using-docker-compose.md</a></td></tr></tbody></table>
