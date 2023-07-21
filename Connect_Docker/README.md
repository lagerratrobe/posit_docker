# Posit Connect

## Overview
The files and instructions in this directory will create a containerized instance of Posit Connect with the following characteristics:
1. Connect version 2023.05.0
2. R versions 3.6.2, 4.1.3, 4.2.0, 4.3.0
3. Python versions 3.8.10, 3.9.5
4. Host's `/data/connect_data/` dir mounted as `/data` in container
5. Password authentication and host's Linux users and groups mounted into the container
6. TinyTeX install fixed and pinned to v. 2023-04-08 of texlive 
7. Quarto version 1.3.340
8. R reticulate installed in R version 4.2.0
9. RPackageRepository set to "http://posit2:4242/cran/__linux__/bionic/latest" (local PM)
10. Small set of linux libraries installed to support geospatial R and Python packages


## Prerequisites
* Host machine with the docker CLI setup
* Github access
* A valid Connect license file

## Setup Steps

### 1. Update the Dockerfile to point at your lic file
Replace "Connect_License.lic" in the Dockerfile entry below with the name of your file.
```
# Copy our license file into the container
COPY Connect_License.lic /etc/rstudio-server/Connect_License.lic
```

### 2. Change the name of the host for Package Manager
Edit the Dockerfile and to point at the Package Manager instance you wish to use.

In the Dockerfile, replace "posit2:4242" with the name or IP of you PM instance in the line below.
```
RUN /opt/R/4.2.0/bin/Rscript -e 'install.packages("reticulate", repos = "http://posit2:4242/cran/__linux__/bionic/latest")'
```

In the rstudio-connect.gcfg, replace "posit2:4242" with the name or IP of your PM instance.
```
[RPackageRepository "CRAN"]
URL = http://posit2:4242/cran/__linux__/bionic/latest
```

### 3. Build
Using the docker CLI, build from within the same directory as the Dockerfile.
```
$ docker build -t rstudio/connect-docker .
```

### 4. Add rstudio-connect user and set groups
We need the Run As user set, and it's also handy to add other users to the same group.
```
$ sudo useradd rstudio-connect
$ sudo groupadd r-users
sudo usermod -a -G r-users rstudio
```

__Note:__If you don't do this, you will get the following error:
```
Cannot verify the Applications.RunAs user 'rstudio-connect': user: unknown user rstudio-connect"
```

### 5. Set up a service configuration for Connect
To have Connect start up automatically and shut down cleanly when the machine is turned off, you'll want to setup a service definition.  There is an included file in this directory that can be used as a starting point, `connect-docker.service`.  

* Edit the file to reflect where on the host machine you want Connect to persist data.  In the block below:
```
ExecStart=/usr/bin/docker run --rm \
    --privileged \
    --mount type=bind,source=/etc/passwd,target=/etc/passwd \
    --mount type=bind,source=/etc/shadow,target=/etc/shadow \
    --mount type=bind,source=/etc/group,target=/etc/group \
    -v /data/connect_data:/data \
    -v /tmp:/tmp \
    -p 3939:3939 \
    rstudio/connect-docker:latest
```
...replace the `/data/connect_data` entry to whatever file path is appropriate on your host. (Note that you should leave the `:/data` there and only modify what is to the left of the colon.)

* Move the edited `connect-docker.service` file into place, which on my Ubuntu 20.04 machine is `/etc/systemd/system/`.

* Enable, and then start the service
```
$ sudo systemctl enable connect-docker.service
$ sudo systemctl start connect-docker.service
```

