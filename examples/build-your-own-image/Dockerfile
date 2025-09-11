########################################################################################################################################################################################
# Setup Dependencies
########################################################################################################################################################################################

# Base image for this is ubuntu 22.04
FROM ubuntu:22.04

# Enable CICD Features
ENV EnableCICDFeaturesForLabVIEW=TRUE

# Set USER as root
ENV USER=root

# Install absolutely needed dependencies
RUN apt-get update && \
    apt-get install -y \
    ca-certificates \
    desktop-file-utils \
    gtk-update-icon-cache \
    libglu1-mesa \
    libx11-6 \
    libxinerama1 \
    xvfb\
    locales\
    vim \
    && rm -rf /var/lib/apt/lists/*
    
# Set locale to UTF-8
RUN locale-gen en_US.UTF-8 
#     update-locale LANG=en_US.UTF-8
# ENV LANG en_US.UTF-8
# ENV LANGUAGE en_US:en   
# ENV LC_ALL en_US.UTF-8


# Fix permission issues for X11
RUN mkdir -p /tmp/.X11-unix && chmod 1777 /tmp/.X11-unix

# Make directory for temporarily storing the LabVIEW installer
RUN mkdir -p /home/execs/

# Copy LabVIEW Installer and CEIP Default settings.json
COPY Installer/* /home/execs/
    
########################################################################################################################################################################################    
# Install and Setup LabVIEW
########################################################################################################################################################################################

# -------------------------------
# Handle LabVIEW year properly
# -------------------------------
# ARG is for build-time (can be overridden with --build-arg)
# ENV makes it available inside RUN and in the container
ARG LV_YEAR=2025
ENV LV_YEAR=${LV_YEAR}


# Install LabVIEW Pro, LabVIEWCLI, CEIP and VI Analyzer Toolkit, NI Vivado 2021.1 FPGA compile worker
RUN apt-get install -y /home/execs/ni-labview-2025-pro_25.3.2.49171-0+f19-ubuntu2204_all.deb && \
    apt-get install -y /home/execs/ni-vivado-2021.1-cg_25.1.0.49343-0+f191-ubuntu2004_all.deb && \
    apt-get update && \
    apt-get install -y \
    ni-labview-${LV_YEAR}-pro \
    ni-labview-command-line-interface \
    ni-ceip \
    ni-vialinux-labview-support \
    ni-vivado-2021.1-cg \
    && rm -rf /var/lib/apt/lists/*
    
# Set necessary INI Tokens to support LabVIEWCLI
# Enable VI Server
RUN mkdir -p /root/natinst/.config/LabVIEW-${LV_YEAR} && \
    echo "server.tcp.enabled=True" >> /root/natinst/.config/LabVIEW-${LV_YEAR}/labviewprofull.conf && \
    echo "server.tcp.enabled=True" >> /root/natinst/.config/LabVIEW-${LV_YEAR}/labview.conf && \
    echo "server.tcp.enabled=True" >> /root/natinst/.config/LabVIEW-${LV_YEAR}/labview64.conf

# Set unattended token
RUN echo "unattended=True" >> /root/natinst/.config/LabVIEW-${LV_YEAR}/labviewprofull.conf && \
    echo "unattended=True" >> /root/natinst/.config/LabVIEW-${LV_YEAR}/labview.conf && \
    echo "unattended=True" >> /root/natinst/.config/LabVIEW-${LV_YEAR}/labview64.conf
    
# Enable CEIP Logging and Sending
RUN mv -f /home/execs/settings.json /var/local/natinst/udc && \
    chmod 666 /var/local/natinst/udc/settings.json

# Setup Vivado 2021.1 compile worker to connect to host server, with username admin and password <blank>, or change NIFarmInfo.xml as needed
COPY NIFarmInfo.xml /usr/local/natinst/nifpgacompileworker/configuration/NIFarmInfo.xml
########################################################################################################################################################################################
# Cleanup
########################################################################################################################################################################################

# Remove installer and settings.json
RUN rm -rf /home/execs/


ENV TZ=America/New_York

CMD ["/bin/bash", "/usr/local/natinst/nifpgacompileworker/cw_wrapper.sh", "mono", "/usr/local/natinst/nifpgacompileworker/CompileWorker.exe"]