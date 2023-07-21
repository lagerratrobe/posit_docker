# Posit Package Manager

## Overview
The files and instructions in this directory will create a containerized instance of Posit Package Manager with the following characteristics:
1. Package Manager version 2023.04.0-6
2. R versions 3.6.2 and 4.2.0
3. Host's `/data/rspm_data` dir mounted as `/data` in container
4. CRAN repo setup as "cran" on PM
5. PyPi repo setup as "pypi" on PM
6. Curated CRAN repo setup as "Restricted_CRAN" on PM
7. Gitbuilder R repo pulling from Github and setup as "git-hello-world on PM
8. Combined repo with Curated Cran and Gitbuilder sources as "NSW_Internal" on PM

## Prerequisites
* Host machine with the docker CLI setup
    - Use instructions here https://docs.docker.com/engine/install/ubuntu/ to install Docker engine
    - Use https://docs.docker.com/engine/install/linux-postinstall/ to be able to run docker commands as non-root user
* Github access
* A valid Package Manager license file

## Setup Steps

### 1. Update the Dockerfile to point at your lic file
Replace "RSPM_2023-06-14.lic" in the Dockerfile entry below with the name of your file.
```
# Copy our license key into the container
COPY RSPM_2023-06-14.lic /etc/rstudio-pm/license.lic
```

### 2. Change the name of the host to match your machine
Edit the rstudio-pm.gcfg (included in this repo) and replace "posit2" in the line below with that name or IP of your host.
```
Address = http://posit2:4242
```

### 3. Build
Using the docker CLI, build from within the same directory as the Dockerfile.
```
$ docker build -t rstudio/package_manager-docker .
```

### 5. Set up a service configuration for Package Manager
To have Package Manager start up automatically and shut down cleanly when the machine is turned off, you'll want to setup a service definition.  There is an included file in this directory that can be used as a starting point, `package_manager-docker.service`.  

* Edit the file to reflect where on the host machine you want Package Manager to persist data.  In the block below:
```
ExecStart=/usr/bin/docker run --rm \
    --privileged \
    -v /data/rspm_data:/data \
    -p 4242:4242 \
    rstudio/package_manager-docker:latest
```
...replace the `/data/rspm_data` entry to whatever file path is appropriate on your host. (Note that you should leave the `:/data` there and only modify what is to the left of the colon.)

* Move the edited `package_manager-docker.service` file into place, which on my Ubuntu 20.04 machine is `/etc/systemd/system/`.

* Enable, and then start the service
```
$ sudo systemctl enable package_manager-docker.service
$ sudo systemctl start package_manager-docker.service
```

### 6. Exec into the running Package Manager container to configure repos
Find the container that is running Package Manager and `exec` into it.  This will allow you to then run the `rspm` CLI tool to add some repos. (Note that the `docker ps` command will output more than the 2 columns shown here.  I have truncated the output to just "CONTAINER ID" and "IMAGE" for clarity.) 
```
$ docker ps
CONTAINER ID IMAGE
dc2007db0e93 rstudio/package_manager-docker:latest

$ docker exec -it dc2007db0e93 bash
```
Once you are logged into the container, you can start adding repos.

### 7. Add a CRAN repo
Note that we are naming this PM repo "cran".
```
rstudio-pm$ rspm create repo --name=cran --description='Access CRAN packages'
rstudio-pm$ rspm subscribe --repo=cran --source=cran
rstudio-pm$ rspm sync --type=cran
```

### 8. Add a PyPi repo
Note that we are naming this PM repo "pypi".
```
rstudio-pm$ rspm create repo --name=pypi --type=python --description='Access PyPI packages'
rstudio-pm$ rspm subscribe --repo=pypi --source=pypi
rstudio-pm$ rspm sync --type=pypi
```

### 9. Add a Gitbuilder R repo
Note that we are naming this PM repo "git-hello-world".
```
rstudio-pm$ rspm create source \
--type=git \
--name=git-hello-world

rstudio-pm$ rspm create git-builder \
--url=https://github.com/lagerratrobe/R_Pkg_Hello_World.git \
--source=git-hello-world \
--build-trigger=commits

rstudio-pm$ rspm create repo \
--name=git-hello-world \
--description='Git Sourced Private R Package'

rstudio-pm$ rspm subscribe \
--source=git-hello-world 
--repo=git-hello-world
```

### 10. Add a Curated CRAN repo
This will contain a subset of packages to show how whitelisting works.  (Note that the "snapshot" date can be changed to whatever you want.)
```
$ rspm create source --name=restricted_cran --type=curated-cran
$ rspm add --packages=ggplot2,dplyr,stringr,sf,terra --source=restricted_cran
$ rspm add --packages=ggplot2,dplyr,stringr,sf,terra --source=restricted_cran --commit --snapshot=2023-07-06
$ rspm create repo --name=Restricted_CRAN --description='Restricted CRAN'
$ rspm subscribe --repo=Restricted_CRAN --source=restricted_cran
```

### 11. Add a repo with multiple sources
So far, we have created repos that contain a single source in them.  However it is useful to show that a Package Manager repo can contain multiple sources.
```
$ rspm create repo --name=NSW_Internal --description='Restricted CRAN & Private R'
Repository: NSW_Internal - Restricted CRAN & Private R - R

$ rspm subscribe --repo=NSW_Internal --source=restricted_cran
$ rspm subscribe --repo=NSW_Internal --source=git-hello-world
```
