
FROM rstudio/rstudio-package-manager:jammy-2023.08.4

USER root
RUN apt update && apt install locales
# Set the locale
ENV LC_ALL=C
RUN locale-gen en_US.UTF-8
USER rstudio-pm

# Copy our configuration over the default install configuration
COPY rstudio-pm.gcfg /etc/rstudio-pm/rstudio-pm.gcfg

# Copy our license key into the container (note that with a key, follow docker hub instructions)
COPY package_manager.lic /etc/rstudio-pm/package_manager.lic

# Activate the license 
RUN /opt/rstudio-pm/bin/license-manager activate-file /etc/rstudio-pm/package_manager.lic

# Update the timezone
USER root
ENV TZ=America/Los_Angeles
RUN sudo ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
USER rstudio-pm
