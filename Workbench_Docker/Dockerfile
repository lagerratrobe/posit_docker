FROM rstudio/rstudio-workbench:ubuntu1804

# Setup system libs
RUN apt update
RUN apt install -y libgdal-dev libgeos-dev libproj-dev libudunits2-dev unixodbc unixodbc-dev libfribidi0 wget gdebi vim libffi-dev libgdbm-dev libsqlite3-dev libssl-dev zlib1g-dev

# Copy the offline lic file into the image
COPY license.key /etc/rstudio/license.key

# Activate the license
RUN /usr/lib/rstudio-server/bin/license-manager activate-file /etc/rstudio/license.key

# Update the timezone
ENV TZ=America/Los_Angeles
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# Update the TinyTeX install (note that this pegs the texlive version)
RUN chmod -R 777 /opt/TinyTeX
RUN tlmgr option repository https://texlive.info/tlnet-archive/2023/04/08/tlnet/
RUN tlmgr install koma-script
RUN tlmgr install caption
RUN tlmgr install tcolorbox
RUN tlmgr install pgf
RUN tlmgr install environ
RUN tlmgr install oberdiek
RUN tlmgr install bookmark
RUN chmod 666 /opt/TinyTeX/tlpkg/texlive.tlpdb

# Add an installed Python to System Path
RUN printf "PATH=/opt/python/3.8.15/bin:$PATH \n" >> /etc/profile.d/python.sh

# Set RETICULATE_PYTHON
RUN printf "export RETICULATE_PYTHON=/opt/python/3.8.15/bin/python \n" >> /etc/rstudio/rsession-profile

# Install current version of R
RUN cd /opt && wget https://cdn.rstudio.com/r/ubuntu-1804/pkgs/r-4.3.0_1_amd64.deb && gdebi -n r-4.3.0_1_amd64.deb

# Setup r-versions
RUN printf "/opt/R/4.3.0 \n" >> /etc/rstudio/r-versions
RUN printf "/opt/R/4.2.3 \n" >> /etc/rstudio/r-versions
RUN printf "/opt/R/4.1.3 \n" >> /etc/rstudio/r-versions

# Install the R reticulate and odbc packges in at least one version of R (pointed at local pkg manager)
RUN /opt/R/4.1.3/bin/Rscript -e 'install.packages(c("reticulate","odbc"), repos = "http://posit2:4242/cran/__linux__/bionic/latest")'
RUN /opt/R/4.2.3/bin/Rscript -e 'install.packages(c("reticulate","odbc"), repos = "http://posit2:4242/cran/__linux__/bionic/latest")'
RUN /opt/R/4.3.0/bin/Rscript -e 'install.packages(c("reticulate","odbc"), repos = "http://posit2:4242/cran/__linux__/bionic/latest")'

# Point at Private Package Manager
COPY repos.conf /etc/rstudio/repos.conf

# Add an Rprofile.site to set repo at CLI
COPY Rprofile.site /opt/R/4.1.3/lib/R/etc/Rprofile.site
COPY Rprofile.site /opt/R/4.2.3/lib/R/etc/Rprofile.site
COPY Rprofile.site /opt/R/4.3.0/lib/R/etc/Rprofile.site

# Overwrite the default rserver.conf
RUN rm -rf /etc/rstudio/rserver.conf
COPY rserver.conf /etc/rstudio/rserver.conf

# Remove the existing Jupyter install
RUN rm -rf /usr/local/bin/jupyter
RUN rm -rf /opt/python/jupyter

# Install Jupyter lab, notebooks etc
RUN /opt/python/3.8.15/bin/pip install jupyter jupyterlab rsp_jupyter rsconnect_jupyter workbench_jupyterlab

# Install the Jupyter extensions
RUN /opt/python/3.8.15/bin/jupyter-nbextension install --sys-prefix --py rsp_jupyter
RUN /opt/python/3.8.15/bin/jupyter-nbextension enable --sys-prefix --py rsp_jupyter
RUN /opt/python/3.8.15/bin/jupyter-nbextension install --sys-prefix --py rsconnect_jupyter
RUN /opt/python/3.8.15/bin/jupyter-nbextension enable --sys-prefix --py rsconnect_jupyter
RUN /opt/python/3.8.15/bin/jupyter-serverextension enable --sys-prefix --py rsconnect_jupyter

# Overwrite the default jupyter.conf
RUN rm -rf /etc/rstudio/jupyter.conf
COPY jupyter.conf /etc/rstudio/jupyter.conf

# Set the default R version to one that we know is stable
RUN printf "rsession-which-r=/opt/R/4.2.3/bin/R \n" >> /etc/rstudio/rserver.conf

# Enable Project sharing
RUN printf "server-project-sharing=1 \n" >> /etc/rstudio/rserver.conf

# Set shared project folder location
RUN printf "server-project-sharing-root-dir=/home/SharedProjects \n" >> /etc/rstudio/rserver.conf

# Set default project and work dirs
RUN printf "session-default-working-dir=/data/RProjects \n" >> /etc/rstudio/rsession.conf
RUN printf "session-default-new-project-dir=/data/RProjects \n" >> /etc/rstudio/rsession.conf

# Add admin-super user
RUN printf "admin-superuser-group=r-users \n" >> /etc/rstudio/rserver.conf

# Install the MS SQL Server V. 17 ODBC Driver
RUN cd /tmp && curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add - && curl https://packages.microsoft.com/config/ubuntu/18.04/prod.list > /etc/apt/sources.list.d/mssql-release.list && apt update && ACCEPT_EULA=Y apt install -y msodbcsql17

# Get rid of buildkitsandbox error (should only be in bionic)
RUN rm -rf /var/lib/rstudio-launcher/Local/jobs/buildkitsandbox 

# 18. Set additional explicit python kernels in Jupyter
RUN /opt/python/3.9.14/bin/pip install ipykernel
RUN /opt/python/3.9.14/bin/python -m ipykernel install --name py3.9.14 --display-name "Python 3.9.14"
