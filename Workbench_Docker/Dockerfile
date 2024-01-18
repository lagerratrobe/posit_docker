FROM rstudio/rstudio-workbench:jammy-2023.12.0

# Setup system libs
RUN apt update
RUN apt install -y libgdal-dev libgeos-dev libproj-dev libudunits2-dev unixodbc unixodbc-dev libfribidi0 wget gdebi vim libffi-dev libgdbm-dev libsqlite3-dev libssl-dev zlib1g-dev libzmq3-dev

# Copy the offline lic file into the image
COPY workbench.lic /etc/rstudio/workbench.lic

# Copy /etc/rstudio/database.conf into place
COPY database.conf /etc/rstudio/database.conf

# Activate the license
RUN /usr/lib/rstudio-server/bin/license-manager activate-file /etc/rstudio/workbench.lic

# Update the timezone
ENV TZ=America/Los_Angeles
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# Update the TinyTeX install (note that this pegs the texlive version)
RUN chmod -R 777 /opt/TinyTeX
#RUN tlmgr option repository https://texlive.info/tlnet-archive/2023/04/08/tlnet/
RUN tlmgr install koma-script
RUN tlmgr install caption
RUN tlmgr install tcolorbox
RUN tlmgr install pgf
RUN tlmgr install environ
RUN tlmgr install oberdiek
RUN tlmgr install bookmark
RUN chmod 666 /opt/TinyTeX/tlpkg/texlive.tlpdb

# Add an installed Python to System Path
RUN printf "PATH=/opt/python/3.8.17/bin:$PATH \n" >> /etc/profile.d/python.sh

# Set RETICULATE_PYTHON
RUN printf "export RETICULATE_PYTHON=/opt/python/3.8.17/bin/python \n" >> /etc/rstudio/rsession-profile

# Install current version of R
RUN cd /opt && wget https://cdn.rstudio.com/r/ubuntu-2204/pkgs/r-4.3.0_1_amd64.deb && gdebi -n r-4.3.0_1_amd64.deb

# Setup r-versions
RUN printf "Path: /opt/R/4.3.0\n" >> /etc/rstudio/r-versions
RUN printf "Label: R 4.3.0\n" >> /etc/rstudio/r-versions
RUN printf "\n" >> /etc/rstudio/r-versions
RUN printf "Path: /opt/R/4.2.3\n" >> /etc/rstudio/r-versions
RUN printf "Label: R 4.2.3\n" >> /etc/rstudio/r-versions
RUN printf "\n" >> /etc/rstudio/r-versions
RUN printf "Path: /opt/R/4.1.3\n" >> /etc/rstudio/r-versions
RUN printf "Label: R 4.1.3\n" >> /etc/rstudio/r-versions
RUN printf "\n" >> /etc/rstudio/r-versions

# Install the R reticulate, odbc and IRkernel packages
RUN /opt/R/4.1.3/bin/Rscript -e 'install.packages(c("reticulate","odbc", "IRkernel"), repos = "http://posit2:4242/cran/__linux__/jammy/latest")'
RUN /opt/R/4.2.3/bin/Rscript -e 'install.packages(c("reticulate","odbc", "IRkernel"), repos = "http://posit2:4242/cran/__linux__/jammy/latest")'
RUN /opt/R/4.3.0/bin/Rscript -e 'install.packages(c("reticulate","odbc", "IRkernel"), repos = "http://posit2:4242/cran/__linux__/jammy/latest")'

# Point at Private Package Manager
#COPY repos.conf /etc/rstudio/repos.conf

# Add an Rprofile.site to set repo at CLI
#COPY Rprofile.site /opt/R/4.1.3/lib/R/etc/Rprofile.site
#COPY Rprofile.site /opt/R/4.2.3/lib/R/etc/Rprofile.site
#COPY Rprofile.site /opt/R/4.3.0/lib/R/etc/Rprofile.site

# Overwrite the default rserver.conf
RUN rm -rf /etc/rstudio/rserver.conf
COPY rserver.conf /etc/rstudio/rserver.conf

# Remove the existing Jupyter install
RUN rm -rf /usr/local/bin/jupyter
RUN rm -rf /opt/python/jupyter

# Remove existing Python 3.9.14 and 3.8.15
RUN rm -rf /opt/python/3.9.14
RUN rm -rf /opt/python/3.8.15
RUN rm -rf /opt/python/default

# Install Python versions that match my Connect server
RUN wget https://cdn.rstudio.com/python/ubuntu-2204/pkgs/python-3.8.17_1_amd64.deb && gdebi -n python-3.8.17_1_amd64.deb
RUN /opt/python/3.8.17/bin/pip install --upgrade pip
RUN /opt/python/3.8.17/bin/pip install pyarrow
RUN /opt/python/3.8.17/bin/pip install fastparquet


RUN wget https://cdn.rstudio.com/python/ubuntu-2204/pkgs/python-3.12.0_1_amd64.deb && gdebi -n python-3.12.0_1_amd64.deb
RUN /opt/python/3.12.0/bin/pip install --upgrade pip
RUN /opt/python/3.12.0/bin/pip install pyarrow
RUN /opt/python/3.12.0/bin/pip install fastparquet

# Install Jupyter lab, notebooks etc
RUN echo `ls -l /opt/python`
RUN /opt/python/3.8.17/bin/pip install jupyter jupyterlab rsp_jupyter rsconnect_jupyter workbench_jupyterlab

# Set jupyter on the PATH
RUN ln -s /opt/python/3.8.17/bin/jupyter /usr/local/bin/jupyter

# Install the Jupyter extensions
RUN /opt/python/3.8.17/bin/jupyter-nbextension install --sys-prefix --py rsp_jupyter
RUN /opt/python/3.8.17/bin/jupyter-nbextension enable --sys-prefix --py rsp_jupyter
RUN /opt/python/3.8.17/bin/jupyter-nbextension install --sys-prefix --py rsconnect_jupyter
RUN /opt/python/3.8.17/bin/jupyter-nbextension enable --sys-prefix --py rsconnect_jupyter
RUN /opt/python/3.8.17/bin/jupyter-serverextension enable --sys-prefix --py rsconnect_jupyter

# Overwrite the default jupyter.conf
RUN rm -rf /etc/rstudio/jupyter.conf
COPY jupyter.conf /etc/rstudio/jupyter.conf

# Set the default R version to one that we know is stable
RUN printf "rsession-which-r=/opt/R/4.2.3/bin/R \n" >> /etc/rstudio/rserver.conf

# Enable Project sharing
RUN printf "server-project-sharing=1 \n" >> /etc/rstudio/rserver.conf

# Set shared project folder location
#RUN printf "server-project-sharing-root-dir=/home/SharedProjects \n" >> /etc/rstudio/rserver.conf

# Set default project and work dirs
RUN printf "session-default-working-dir=/data/RProjects \n" >> /etc/rstudio/rsession.conf
RUN printf "session-default-new-project-dir=/data/RProjects \n" >> /etc/rstudio/rsession.conf

RUN printf "r-cran-repos-url=http://posit2:4242/__api__/repos \n" >> /etc/rstudio/rsession.conf
RUN printf "allow-r-cran-repos-edit=1 \n" >> /etc/rstudio/rsession.conf

# Add admin-super user
RUN printf "admin-superuser-group=r-users \n" >> /etc/rstudio/rserver.conf

# Install the MS SQL Server V. 17 ODBC Driver
RUN cd /tmp && curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add - && curl https://packages.microsoft.com/config/ubuntu/22.04/prod.list > /etc/apt/sources.list.d/mssql-release.list && apt update && ACCEPT_EULA=Y apt install -y msodbcsql17

# Get rid of buildkitsandbox error (should only be in bionic)
#RUN rm -rf /var/lib/rstudio-launcher/Local/jobs/buildkitsandbox 

# 18. Set additional explicit kernels in Jupyter
RUN /opt/python/3.12.0/bin/pip install ipykernel
RUN /opt/python/3.12.0/bin/python -m ipykernel install --name py3.12.0 --display-name "Python 3.12.0"

# R
RUN /opt/R/4.1.3/bin/Rscript -e 'IRkernel::installspec(user = FALSE, name="R4.1.3", displayname="R 4.1.3")'
RUN /opt/R/4.2.3/bin/Rscript -e 'IRkernel::installspec(user = FALSE, name="R4.2.3", displayname="R 4.2.3")'
RUN /opt/R/4.3.0/bin/Rscript -e 'IRkernel::installspec(user = FALSE, name="R4.3.0", displayname="R 4.3.0")'

