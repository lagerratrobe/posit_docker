
FROM rstudio/rstudio-package-manager:bionic-2023.04.0

# Copy our configuration over the default install configuration
COPY rstudio-pm.gcfg /etc/rstudio-pm/rstudio-pm.gcfg

# Copy our license key into the container (note that with a key, follow docker hub instructions)
COPY RSPM_2023-06-14.lic /etc/rstudio-pm/license.lic

# Activate the license 
RUN /opt/rstudio-pm/bin/license-manager activate-file /etc/rstudio-pm/license.lic

# Update the timezone
USER root
ENV TZ=America/Los_Angeles
RUN sudo ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
USER rstudio-pm
