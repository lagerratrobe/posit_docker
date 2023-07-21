
FROM rstudio/rstudio-connect:bionic-2023.05.0

# Make it possible to install system libs
RUN apt-get update
RUN apt-get install -y gdebi

# Setup system libs for sf and raster (GDAL, etc)
RUN apt-get update
RUN apt-get install -y libgdal-dev libgeos-dev libproj-dev libudunits2-dev

# Fix the TinyTeX install
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

# Install the R Reticulate packge in at least one version of R
RUN /opt/R/4.2.0/bin/Rscript -e 'install.packages("reticulate", repos = "http://posit2:4242/cran/__linux__/bionic/latest")'

# Install old version of R
RUN wget https://cdn.rstudio.com/r/ubuntu-1804/pkgs/r-4.1.3_1_amd64.deb && gdebi -n r-4.1.3_1_amd64.deb

# Install current version of R
RUN wget https://cdn.rstudio.com/r/ubuntu-1804/pkgs/r-4.3.0_1_amd64.deb && gdebi -n r-4.3.0_1_amd64.deb

# Copy our configuration over the default install configuration
COPY rstudio-connect.gcfg /etc/rstudio-connect/rstudio-connect.gcfg

# Copy our license file into the container
COPY Connect_License.lic /etc/rstudio-server/Connect_License.lic

# Activate the license 
RUN /opt/rstudio-connect/bin/license-manager activate-file /etc/rstudio-server/Connect_License.lic

# Update the timezone
ENV TZ=America/Los_Angeles
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

