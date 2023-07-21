# Posit Workbench

## Overview
The files and instructions in this directory will create a containerized instance of Posit Workbench with the following characteristics:
1. Workbench version 2023.03.2+454.pro2
2. R versions 4.1.3, 4.2.3, 4.3.0
3. Python versions 3.8.15, 3.9.14
4. Host's `/data/RProjects` dir mounted as `/home` in container
5. Host's Linux users and groups mounted into the container
6. TinyTeX install fixed and pinned to v.2023-04-08 of texlive 
7. Quarto version 1.1.251 (symbolically linked to system PATH in /usr/local/bin)
8. R reticulate installed in R versions 4.1.3, 4.2.3, 4.3.0
9. System Python path set to python 3.8.15 via `/etc/profile.d/python.sh`
10. RETICULATE_PYTHON set to python 3.8.15
11. repos.conf pointed to local PM
12. Small set of linux libraries installed to support geospatial R and Python packages
13. repos option set in Rprofile.site set for R versions 4.1.3, 4.2.3, 4.3.0
14. Admin dashboard enabled
15. Project Sharing enabled
16. Admin super-user enabled
17. MS SQL Server v 17 ODBC driver installed
18. Python kernel for Python 3.9.14 set

## Prerequisites
* Host machine with the docker CLI setup
* Github access
* A valid Connect license file

## Setup Steps

### 1. Update the Dockerfile to point at your lic file
Replace "workbench.lic" in the Dockerfile entry below with the name of your file.
```
# Copy our license file into the container
COPY workbench.lic /etc/rstudio-server/workbench.lic
```

### 2. Change the name of the host for Package Manager in all files
Change all instances of "posit2" in the following files...
- Dockerfile
- Rprofile.site
- repos.conf
...to the IP address or hostname of your server.
  

ie. in the Dockerfile, replace "posit2" with the name or IP of you PM instance in the line below.
```
RUN /opt/R/4.2.0/bin/Rscript -e 'install.packages("reticulate", repos = "http://posit2:4242/cran/__linux__/bionic/latest")'
```

In the rstudio-connect.gcfg, replace "posit2" with the name or IP of your PM instance.
```
[RPackageRepository "CRAN"]
URL = http://posit2:4242/cran/__linux__/bionic/latest
```

### 3. Build
Using the docker CLI, build from within the same directory as the Dockerfile.
```
$ docker build -t rstudio/workbench-docker .
```

Note: You may want to build using this command instead:
```
DOCKER_BUILDKIT=0 docker build -t rstudio/workbench-docker .
```

### 5. Set up a service configuration for Workbench
To have Workbench start up automatically and shut down cleanly when the machine is turned off, you'll want to setup a service definition.  There is an included file in this directory that can be used as a starting point, `workbench-docker.service`.  

* Edit the file to reflect where on the host machine you want Connect to persist data.  In the block below:
```
[Unit]
Description=RStudio Workbench Service
After=docker.service
Requires=docker.service

[Service]
TimeoutStartSec=0
Restart=always
ExecStartPre=-/usr/bin/docker exec workbench-docker stop
ExecStart=/usr/bin/docker run --rm --privileged \
    --mount type=bind,source=/etc/passwd,target=/etc/passwd \
    --mount type=bind,source=/etc/shadow,target=/etc/shadow \
    --mount type=bind,source=/etc/group,target=/etc/group \
    -v /data/RProjects:/home \
    -e RSW_TESTUSER="" \
    -p 8787:8787 \
    -p 5559:5559 \
    rstudio/workbench-docker:latest

[Install]
WantedBy=default.target

```
...replace the `/data/RProjects` entry to whatever file path is appropriate on your host. (Note that you should leave the `:/home` there and only modify what is to the left of the colon.)

* Move the edited `workbench-docker.service` file into place, which on my Ubuntu 20.04 machine is `/etc/systemd/system/`.

* Enable, and then start the service
```
$ sudo systemctl enable workbench-docker.service
$ sudo systemctl start workbench-docker.service
```



