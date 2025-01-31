---
description: MikoPBX Installation Guide using Docker container
---

# Running MikoPBX in a container

To work with MikoPBX in a container, you need to install Docker and Docker Compose, as well as create a user and directories for storing configuration settings and call recordings according to the instructions

{% content-ref url="docker-installation-and-creating-a-user-and-directories.md" %}
[docker-installation-and-creating-a-user-and-directories.md](docker-installation-and-creating-a-user-and-directories.md)
{% endcontent-ref %}

{% embed url="https://youtu.be/3w7odcxcNvk" %}

### Launching the Docker container

To launch the container with your application, use the following commands:

```bash
# Pulling the container image
sudo docker pull ghcr.io/mikopbx/mikopbx-x86-64

# Running the container in unprivileged mode
sudo docker run --cap-add=NET_ADMIN --net=host --name mikopbx --hostname mikopbx \
           -v /var/spool/mikopbx/cf:/cf \
           -v /var/spool/mikopbx/storage:/storage \
           -e SSH_PORT=23 \
           -e ID_WWW_USER="$(id -u www-user)" \
           -e ID_WWW_GROUP="$(id -g www-user)" \
           -it -d --restart always ghcr.io/mikopbx/mikopbx-x86-64
```

### Testing the functionality

To ensure that your MikoPBX application is posted and working in the Docker container, you can follow these steps after launching it. These steps will help you verify the container's status and view its logs.

#### Step 1: Check container status

First, ensure that the container is successfully launched and running. To do this, use the command `docker ps`, which will show a list of running containers and their statuses.

```bash
sudo docker ps
```

This command will display information about all active containers. Make sure that the `mikopbx` container is present in the list and its status indicates that it is running (e.g., status **up**).

#### Step 2: View container logs

After confirming that the container is running, the next step is to view the logs to ensure that the application has loaded without errors and is functioning properly. The docker logs command will allow you to see the output generated by your application.

```bash
sudo docker logs mikopbx
```

Check the command output for a message similar to the one below. This message indicates that MikoPBX is successfully loaded and ready for use:

```
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
|               All services are fully loaded welcome                |
|                       MikoPBX 2024.1.60.                           |
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
|                        Web Interface Access                        |
|                                                                    |
| Local Network Address:                                             |
| https://10.0.0.4                                                   |
|                                                                    |
| Web credentials:                                                   |
|    Login: admin                                                    |
|    Password: admin                                                 |
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
| SSH access disabled!                                               |
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
```

If you see the MikoPBX startup process, wait a moment and re-run the command `sudo docker logs mikopbx`

#### Step 3: Check access to the web Interface

When the container starts, it lacks information about the host system's address, so you need to open the external address of the host system, in this case, Ubuntu, in a web browser. https://\<host machine IP>

Log into the web interface using the `admin` login and the `admin` password to make sure that the web interface is accessible and functioning correctly.

<figure><img src="../../.gitbook/assets/MikoPBXProxmoxInstallation_17 (2).png" alt=""><figcaption></figcaption></figure>

### Features of containerized MikoPBX

* The **NET\_ADMIN** flag is required for the proactive protection system **fail2ban** and the firewall **iptables** to function inside the container. When an access block is triggered, for example, by entering an incorrect password, access from the IP address of the attacker will be blocked.
* If you need to use the "[Backup Module](../../manual/maintenance/backup.md)", the container should be run with the **–privileged** flag. When MikoPBX is run in a container, backups can also be performed by manually archiving the **cf** and **storage** directories. In this case, the privileged mode is not necessary, but the container must be stopped during copying.
* The **–net=host** flag indicates that NAT between the host and container will not be used. MikoPBX will be directly connected to the host machine's network. All ports that the container needs to occupy will also be occupied on the host machine. If any port on the host machine is unavailable, errors will occur when loading MikoPBX. More details in the [Docker documentation...](https://docs.docker.com/network/host/)
* If necessary, you can adjust the standard set of ports used by MikoPBX. This can be done by declaring environment variables when launching the container.

### Creating a container from a tar archive

In addition to using our official registry, you might need to create a container from an image, for example, for a beta version. Our published releases and pre-releases include a tar archive, which we use to create a container.

Here is an example code for its use:

```bash
# Create a container from a tar archive
sudo docker import \
  --change 'ENTRYPOINT ["/bin/sh", "/sbin/docker-entrypoint"]' \
  mikopbx-2024.1.114-x86_64.tar \
  "mikopbx:2024.1.114"

# Launch the created container
sudo docker run --cap-add=NET_ADMIN --net=host --name mikopbx --hostname mikopbx \
	 -v mikopbx_cf:/cf \
	 -v mikopbx_storage:/storage \
	 -e SSH_PORT=23 \
	 -e ID_WWW_USER="$(id -u www-user)" \
	 -e ID_WWW_GROUP="$(id -g www-user)" \
	 -it mikopbx:2024.1.114
```

### Environment variables for configuring MikoPBX

Below are some of the environment variables that will allow you to adjust the MikoPBX ports and settings used.

* **SSH\_PORT** - port for SSH (**22**)
* **WEB\_PORT** - port for the web interface via HTTP protocol (**80**)
* **WEB\_HTTPS\_PORT** - port for the web interface via HTTPS protocol (**443**)
* **SIP\_PORT** - port for connecting a SIP client (**5060**)
* **TLS\_PORT** - port for connecting a SIP client with encryption (**5061**)
* **RTP\_PORT\_FROM** - beginning of the RTP port range, voice transmission (**10000**)
* **RTP\_PORT\_TO** - end of the RTP port range, voice transmission (**10800**)
* **IAX\_PORT** - port for connecting IAX clients (**4569**)
* **AMI\_PORT** - AMI port (**5038**)
* **AJAM\_PORT** - AJAM port used for connecting the telephony panel for 1C (**8088**)
* **AJAM\_PORT\_TLS** - AJAM port used for connecting the telephony panel for 1C (**8089**)
* **BEANSTALK\_PORT** - port for the Beanstalkd queue server (**4229**)
* **REDIS\_PORT** - port for the Redis server (**6379**)
* **GNATS\_PORT** - port for the gnatsd server (**4223**)
* **ID\_WWW\_USER** - identifier for www-user (can be set with the expression\
  `$(id -u www-user)`, where **www-user** is **NOT a root** user)
* **ID\_WWW\_GROUP** - group identifier for www-user (can be set with the expression\
  `$(id -g www-user)`, where **www-user** is **NOT a root** group)
* **WEB\_ADMIN\_LOGIN** - login for Web interface access
* **WEB\_ADMIN\_PASSWORD** - password for Web interface access

A full list of all possible setting parameters is available in the source code [here](https://github.com/mikopbx/Core/blob/develop/src/Common/Models/PbxSettingsConstants.php).
