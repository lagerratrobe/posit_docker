Running Posit Applications in Docker
=============================

## Overview and Purpose of this Document

Posit applications are intended to be installed on stand-alone servers running a Linux OS.  The applications run as services on the server and configuration of their runtime settings are done via edits of config files at the Linux filesystem level.  Users of the applications interact with them via Web front-ends provided by the server on specified ports (when running HTTP), or via a single secure port (typically 443 when using HTTPS).

Given how these applications run, it is relatively simple to implement them as containerized services inside of Docker and, in fact, several Posit installation configurations (such as Kubernetes integration) explicitly require the use of containers.  In that case, Helm charts determine the configuration of the deployed applications inside of Kubernetes.  However, it is also possible to deploy Posit applications as standalone containers running on a VM or other Linux host without the use of Helm charts.  When run in this fashion, a common starting point is to use the images which are available at  the [Docker Hub rstudio page](https://hub.docker.com/u/rstudio).

When using the images available on Docker Hub as a starting point, there is information on each application's homepage that explains how to spin up the application in a basic fashion.

- https://hub.docker.com/r/rstudio/rstudio-connect
- https://hub.docker.com/r/rstudio/rstudio-workbench
- https://hub.docker.com/r/rstudio/rstudio-package-manager

The intent of this document is to add some detail around how each image  can be customized to enhance or change their basic configuration.  These changes are focused on capturing basic tasks that a system administrator deploying these containers might wish to implement for the users that will ultimately be interacting with the application.


## Basic Docker Image Customization Concepts

There are different ways in which a Posit application running inside a Docker image can be updated or modified.  Several of these are discussed on the Docker Hub pages, but some additional context will be provided below.  Three of the most commonly used approaches are shown below.

### 1. Custom runtime parameters supplied when the image is started, or "run".
In the example below, the `-p`, `-v` and `-e` flags all provide optional runtime parameters that affect the operation of the `rstudio-connect` image.
```
docker run -it --privileged \
    -p 3939:3939 \
    -v $PWD/data/rsc:/data \
    -e RSC_LICENSE=$RSC_LICENSE \
    rstudio/rstudio-connect:ubuntu1804
```

### 2. The injection of custom config files into the image at runtime
The Posit Docker images are provided with minimal config files that control the behavior of each application.  It is possible to replace those config files with modified ones that contain additional or modified values to enable additional functionality or make changes to the default behavior of the application.

In the example below, the `-v` command specifies that the `rstudio-connect.gcfg` in the current local working directory will be brought into the container and mounted in the place of the default `/etc/rstudio-connect/rstudio-connect.gcfg` that is shipped with the image.

```
docker run -it --privileged \
    -p 3939:3939 \
    -v $PWD/rstudio-connect.gcfg:/etc/rstudio-connect/rstudio-connect.gcfg \
    rstudio/rstudio-connect:ubuntu1804
```

### 3. Use of a Dockerfile to build a custom image that contains modifications which are "built-in" to the image.
In this Dockerfile example, the Ubuntu Advanced Package Tool (apt) is told to update it's knowledge of available system packages and then install the `gdebi` tool.  The `RUN` command indicates that the command immediately following it should be executed inside the image as part of its "build".

```
# Simple Dockerfile
FROM rstudio/rstudio-connect:bionic-2023.05.0

RUN apt-get update
RUN apt-get install -y gdebi
```
When this Dockerfile is used in a build, typically with a command like this...
```
$ docker build -t rstudio/connect-docker .
```
...the resulting `rstudio/connect-docker` image will contain the `gdebi` tool.  In effect, the build process has taken the starting `rstudio/rstudio-connect` image and modified it by adding the `gdebi` system tool. A new image named `rstudio/connect-docker` will be created and anytime that new image is used, the `gdebi` tool will already be present in it.

### So what's the "right" way to do this?

"It depends".  Sometimes it is beneficial to use a combination of all three methods, but there is no real _"right"_ way to do it.  Some methods might be more beneficial in certain circumstances than in others, but the instructions shown here should result in functional changes that will run.

## Generic Overview of Deployment

Using Connect as an example, below is the typical workflow that is used to deploy a Posit application.  These instructions are intended to show a generic process for getting Connect running in a local Docker container that meets the  the following requirements:

- Picks up License key at runtime
- Has persistent storage for users and published work
- Respects options in Connect cfg file

There are specific Dockerfiles and additional info in each of the application level dirs of this repo to configure each app in very specific ways.

#### IT IS NOT RECOMMENDED THAT YOU DEPLOY Connect USING THE INSTRUCTIONS BELOW!!!!

### Basic Docker Hub Install

A good starting point are the [public facing docs](https://hub.docker.com/r/rstudio/rstudio-connect) on Docker Hub.  With these, you can get a local Docker instance of Connect running at http://localhost:3939.  As the page points out though, this lacks some functionality and is really just for testing.

__Toy Docker Connect Instance:__

```
$ docker pull rstudio/rstudio-connect

$ export RSC_LICENSE=XXX-XXX-XXX-XXX-XXX-XXX-XXX

$ docker run -it --privileged -p 3939:3939 -e RSC_LICENSE=$RSC_LICENSE rstudio/rstudio-connect:latest
```

However, it's actually quite easy to expand upon on that basic install to meet the requirements above.


### Create Dockerfile

Simple Dockerfile that pulls in the Docker Hub image and sets up a few things.

```
FROM rstudio/rstudio-connect

# 1. Copy our configuration over the default install configuration
COPY rstudio-connect.gcfg /etc/rstudio-connect/rstudio-connect.gcfg

# 2. Copy our license key into the container
COPY license.key /etc/rstudio-server/license.key

# 3. Activate the license
RUN /opt/rstudio-connect/bin/license-manager activate `cat /etc/rstudio-server/license.key`

# 4. Expose the configured listen port
EXPOSE 3939

# Launch Connect.
CMD ["--config", "/etc/rstudio-connect/rstudio-connect.gcfg"]
ENTRYPOINT ["/opt/rstudio-connect/bin/connect"]

```

Note: This relies on having a `license.key` file that contains a valid RStudio Connect key.

### Create Connect Config File

In addition to the Dockerfile, we need a `rstudio-connect.gcfg` file.  Below is a minimal one that works.

```
; /etc/rstudio-connect/rstudio-connect.gcfg

; RStudio Connect sample configuration
[Server]
SenderEmail = account@company.com
Address = http://0.0.0.0:3939

; The persistent data directory mounted into our container.
DataDir = /data

[Licensing]
LicenseType = local

; Use and configure local database.
[Database]
Provider = sqlite

[HTTP]
; RStudio Connect will listen on this network address for HTTP connections.
Listen = :3939

[Authentication]
; Specifies the type of user authentication.
Provider = password
```

### Tying it all together

When you build the image, you should see that your `rstudio-connect.gcfg` is being copied into the container and that the license manager has activated your lic key.

```
$ docker build -t rstudio/connect-docker .
```
Should see the following:

```
Step 2/6 : COPY rstudio-connect.gcfg /etc/rstudio-connect/rstudio-connect.gcfg
<snip>

Step 4/6 : RUN /opt/rstudio-connect/bin/license-manager activate XXX-XXX-XXX-XXX-XXX-XXX-XXX
 ---> Running in 7cc4530bd04a
{"status":"activated","product-key":"XXX-XXX-XXX-XXX-XXX-XXX-XXX","has-key":true,"has-trial":true,"users":"20","user-activity-days":"365","shiny-users":"20","allow-apis":"1","expiration":1717891200000.0,"days-left":709,"license-engine":"4.4.3.0","license-scope":"system","result":0,"connection-problem":false}

<snip>
Successfully built 1920e66b0da3
Successfully tagged rstudio/connect-docker:latest
```

Then, you can run the image.

```
$ docker run -d --rm --privileged -p 3939:3939 -v /data/connect_data/:/data rstudio/connect-docker:latest
```

The `docker run` command is where you can specify what local directory you want Docker to mount as `/data` by using the `-v` flag.  In my case, I'm using `/data/connect_data`.

### Verify Container is running as desired

It's easy enough to see that the container is running.

```
$ docker container ls
CONTAINER ID   IMAGE                           COMMAND                  CREATED          STATUS          PORTS                                       NAMES
24a365d28559   rstudio/connect-docker:latest   "/opt/rstudio-connec…"   11 minutes ago   Up 11 minutes   0.0.0.0:3939->3939/tcp, :::3939->3939/tcp   reverent_cray
```

What is a bit more useful though, is to login to the container itself via the `exec` command and examine the Connect logs, etc.

```
$ docker exec -it 24a365d28559 bash

root@24a365d28559:/# tail -f /var/log/rstudio/rstudio-connect/rstudio-connect.log
<snip>
time="2022-07-06T19:54:05.988Z" level=info msg="Using file /var/log/rstudio/rstudio-connect/rstudio-connect.access.log to store Access Logs."
time="2022-07-06T19:54:05.988Z" level=info msg="Startup complete (~854.334713ms)"
time="2022-07-06T19:54:05.988Z" level=info msg="Starting HTTP server on :3939"
time="2022-07-06T19:54:05.987Z" level=info msg="Sweeping ad-hoc variants"
time="2022-07-06T19:54:35.287Z" level=info msg="Creating the initial worker;
```

### Shutdown the Container

```
$ docker stop 24a365d28559
24a365d28559
```

### Adding Docker as a Service

To have the system automatically start and stop the Docker instance, you can create a service file in systemd.

```
$ sudo vi /etc/systemd/system/connect-docker.service

[Unit]
Description=RStudio Connect Service
After=docker.service
Requires=docker.service

[Service]
TimeoutStartSec=0
Restart=always
ExecStartPre=-/usr/bin/docker exec connect-docker stop
ExecStart=/usr/bin/docker run --rm --privileged \
    -v /data/connect_data/:/data \
    -p 3939:3939 \
    rstudio/connect-docker:latest

[Install]
WantedBy=default.target

```

Then you can enable and start/stop the service using `systemctl`.

```
$ systemctl enable connect-docker

$ sudo systemctl start connect-docker.service 
```

