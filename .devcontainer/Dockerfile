FROM debian:bullseye

# Set non-built-time labels
LABEL org.opencontainers.image.authors="Ayaz BADOURALY"
LABEL org.opencontainers.image.description="Docker image for remote development in R with vscode"

# This Dockerfile adds a non-root user with sudo access. Use the "remoteUser"
# property in devcontainer.json to use it. On Linux, the container user's GID/UIDs
# will be updated to match your local UID/GID (when using the dockerFile property).
# See https://aka.ms/vscode-remote/containers/non-root-user for details.
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Set to false to skip installing zsh and Oh My ZSH!
ARG INSTALL_ZSH="false"

# Location and expected SHA for common setup script - SHA generated on release
ARG COMMON_SCRIPT_SOURCE="https://raw.githubusercontent.com/microsoft/vscode-dev-containers/master/script-library/common-debian.sh"
ARG COMMON_SCRIPT_SHA="dev-mode"

# Avoid warnings by switching to noninteractive
ENV DEBIAN_FRONTEND=noninteractive

# Configure apt and install packages
RUN apt-get update && \
    apt-get install --assume-yes --no-install-recommends apt-utils ca-certificates dialog wget && \
    #
    # Verify git, common tools / libs installed, add/modify non-root user, optionally install zsh
    wget -q -O /tmp/common-setup.sh $COMMON_SCRIPT_SOURCE && \
    if [ "$COMMON_SCRIPT_SHA" != "dev-mode" ]; then echo "$COMMON_SCRIPT_SHA /tmp/common-setup.sh" | sha256sum -c - ; fi && \
    /bin/bash /tmp/common-setup.sh "$INSTALL_ZSH" "$USERNAME" "$USER_UID" "$USER_GID" && \
    rm /tmp/common-setup.sh && \
    #
    # Install r-base, r-languageserver and radian dependencies, along with tini
    apt-get install --assume-yes --no-install-recommends \
            build-essential \
            gnupg \
            libcurl4-openssl-dev \
            libssl-dev \
            libxml2-dev \
            libz-dev \
            python3 \
            python3-pip \
            tini \
            && \
    #
    # Clean up
    rm -rf /var/cache/apt/* var/lib/apt/lists/* && \
    rm -rf /tmp/* /var/tmp/*

# Set R_VARIANT build arg to override the R version to be used
# 40 for R 4.0 (default), 35 for R 3.6
ARG R_VARIANT=40

# Split installation in different layers to leverage caches
RUN APT_KEY_DONT_WARN_ON_DANGEROUS_USAGE=true apt-key adv --keyserver keyserver.ubuntu.com --recv-key "95C0FAF38DB3CCAD0C080A7BDC78B2DDEABC47B7" && \
    #
    # Install r-base from https://cloud.r-project.org/bin/linux/debian/
    echo "deb http://cloud.r-project.org/bin/linux/debian bullseye-cran$R_VARIANT/" > /etc/apt/sources.list.d/r-project.list && \
    apt-get update && \
    apt-get install --assume-yes --target-release bullseye-cran$R_VARIANT r-base && \
    #
    # Install r-languageserver
    Rscript -e 'install.packages("languageserver", repos="https://cloud.r-project.org/")' && \
    #
    # Install radian
    python3 -m pip install --no-cache-dir --upgrade setuptools wheel && \
    python3 -m pip install --no-cache-dir --upgrade radian && \
    #
    # Clean up
    rm -rf /var/cache/apt/* var/lib/apt/lists/* && \
    rm -rf /tmp/* /var/tmp/*

# Switch back to dialog for any ad-hoc use of apt-get
ENV DEBIAN_FRONTEND=dialog

# Setup entrypoint and cmd
ENTRYPOINT [ "tini" ]
CMD [ "sleep", "inf" ]
