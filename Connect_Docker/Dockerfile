
FROM rstudio/rstudio-connect:jammy-2023.12.0

# Make it possible to install system libs
RUN apt-get update
RUN apt-get install -y gdebi unixodbc unixodbc-dev

# Setup system libs for sf and raster (GDAL, etc)
RUN apt-get update
RUN apt-get install -y libgdal-dev libgeos-dev libproj-dev libudunits2-dev

# Fix the TinyTeX install
RUN chmod -R 777 /opt/TinyTeX
#RUN tlmgr option repository https://texlive.info/tlnet-archive/2023/11/19/tlnet/
RUN tlmgr update --self
RUN tlmgr install koma-script
RUN tlmgr install caption
RUN tlmgr install tcolorbox
RUN tlmgr install pgf
RUN tlmgr install environ
RUN tlmgr install oberdiek
RUN tlmgr install bookmark
RUN chmod 666 /opt/TinyTeX/tlpkg/texlive.tlpdb

# Install the R Reticulate packge in at least one version of R
RUN /opt/R/4.2.3/bin/Rscript -e 'install.packages("reticulate", repos = "http://posit2:4242/cran/__linux__/jammy/latest")'

# Install old version of R
#RUN wget https://cdn.rstudio.com/r/ubuntu-1804/pkgs/r-4.1.3_1_amd64.deb && gdebi -n r-4.1.3_1_amd64.deb

# Install current version of R
RUN wget https://cdn.rstudio.com/r/ubuntu-2204/pkgs/r-4.3.0_1_amd64.deb && gdebi -n r-4.3.0_1_amd64.deb

# Install Python 3.12.0 and set pip to 23.3.1
RUN wget https://cdn.rstudio.com/python/ubuntu-2204/pkgs/python-3.12.0_1_amd64.deb && gdebi -n python-3.12.0_1_amd64.deb
RUN /opt/python/3.12.0/bin/pip install pip==23.3.1 setuptools wheel

# Update pip in both of the pre-installed Python versions
RUN /opt/python/3.8.17/bin/python3.8 -m pip install --upgrade pip
RUN /opt/python/3.9.17/bin/python3.9 -m pip install --upgrade pip

# Install parquet file support
RUN /opt/python/3.8.17/bin/pip install pyarrow
RUN /opt/python/3.8.17/bin/pip install fastparquet

# Pro-Driver setup
RUN cd /tmp && curl -O https://cdn.rstudio.com/drivers/7C152C12/installer/rstudio-drivers_2023.05.0_amd64.deb && gdebi -n rstudio-drivers_2023.05.0_amd64.deb && cp /opt/rstudio-drivers/odbcinst.ini.sample /etc/odbcinst.ini

# Copy our configuration over the default install configuration
COPY rstudio-connect.gcfg /etc/rstudio-connect/rstudio-connect.gcfg

# Copy our license file into the container
COPY connect.lic /etc/rstudio-server/connect.lic

# Activate the license 
RUN /opt/rstudio-connect/bin/license-manager activate-file /etc/rstudio-server/connect.lic

# Update the timezone
ENV TZ=America/Los_Angeles
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# Install latest Quarto
RUN cd /tmp && curl -LO https://quarto.org/download/latest/quarto-linux-amd64.deb && gdebi -n quarto-linux-amd64.deb
