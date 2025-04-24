
ARG BUILD_FROM=ghcr.io/hassio-addons/ubuntu-base:9.0.5
# hadolint ignore=DL3006
FROM $BUILD_FROM AS base

# Set shell
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# CodeProject.AI Server 3.0+
#
# Docker file for x86_64 CPU enabled image
#
#
# ASSUMPTION: This is being built from the root directory of the solution. This
# is set in the build_docker script as the last param to the docker buildx command,
# namely the "../..". This takes us from /Installers/Docker to /. and then this
# image file proceeds


# INITIAL SETUP ===============================================================

# It's important to pull the correct base for the target architecture, otherwise
# building this image on, say, an Arm64 machine will result in the base image
# being Arm64, not x86_64 as assumed.

ARG BUILD_ARCH

RUN echo "BUILD_ARCH: ${BUILD_ARCH}" 

# Environment vars ------------------------------------------------------------

#ENV ASPNETCORE_URLS=http://+:32168;http://+:5000

# The simple log format is easier on my brain than json
ENV LOGGING__CONSOLE__FORMATTERNAME=simple

# Magic that was being done by the Microsoft ASP.NET base image that we aren't using anymore
ENV DOTNET_RUNNING_IN_CONTAINER=true

# noninteractive frontend means no prompts or questions are asked and whenever a call requires an
# answer, the default will be used. Installs will be non-interrupted and so won't hang. The Python
# installs, for instance, require this in this current environment
ENV DEBIAN_FRONTEND=noninteractive

ENV LANGUAGE=C
ENV LC_ALL=C
ENV LANG=C
ENV LC_CTYPE=C

# hadolint ignore=DL3008,DL3015,DL3009
RUN     apt-get update  -y && \
        apt-get upgrade -y --no-install-recommends apt-utils && \
        apt-get install -y --no-install-recommends \
                libfontconfig1  \
                libgdiplus      \
                libjpeg-dev     \
                zlib1g-dev      \
                                \
                ffmpeg          \
                libc6-dev       \
                libsm6          \
                libxext6        \
                                \
                mesa-utils      \
                                \
                gcc             \
                python3-dev     \
                                \
                ca-certificates \
                curl            \
                jq              \
                rsync           \
                unzip           \
                wget            \
                                \
                psmisc          \
                                \
                patch           \
                                \
                kmod            > /dev/null && \
        apt-get install software-properties-common -y
# Install .NET 9.0 runtime
# hadolint ignore=DL3008,DL3009,DL3015
RUN     add-apt-repository ppa:dotnet/backports -y && \  
        apt-get update && \
        apt-get install -y aspnetcore-runtime-9.0 

# Setup Python ----------------------------------------------------------------

# *** TODO: *** all of this is in /app/SDK/Scripts/utils.sh under "setupPython".
# We should use that method rather than having the same commands copy and pasted
# here. BEWARE that the list of apt installed here differs a little from what is
# installed via setupPython. installRequiredPythonPackages also installs some
# tools from the list below

# Combining all these together into a single layer
# 1. Add deadsnakes repo
# 2. Install pip for python3,  python3-apt and python3-setuptools
#    Note that this (currently) installs python3.8 as a dependency. It could be
#    in the future that a different version of python is installed instead, in
#    which case we may need to pin the installed python version.
# 3. Install Python 3.8 as well as dev and distutils so packages can be built
#    if needed, and venv for creating virtual environments
# 4. Using the now installed Python 3.8, upgrade the python setuptools and dev
#    packages to the latest
# 5. Install virtualenv virtualenvwrapper for python 3.8

# hadolint ignore=DL3008,DL3059,DL3013,DL3042,DL3015
RUN     apt-get install software-properties-common -y && \
        add-apt-repository ppa:deadsnakes/ppa -y && \
        apt-get install -y --no-install-recommends            \
                python3-pip                                       \
                python3-apt                                       \
                python3-setuptools && \
        apt-get install -y --no-install-recommends            \
                python3.8&& \
        apt-get install -y --no-install-recommends            \
                python3.8-distutils                               \
                python3.8-dev                                     \
                python3.8-venv && \
        python3.8 -m pip install --upgrade setuptools && \
        python3.8 -m pip install --upgrade pip && \
        python3.8 -m pip install virtualenv virtualenvwrapper > /dev/null

# Python 3.9: Combining all these together into a single layer, but leveraging
# the work done in Python 3.8:
# 1. Install Python 3.9 as well as dev and distutils so packages can be built
#    if needed, and venv for creating virtual environments
# 4. Using the now installed Python 3.8, upgrade the python setuptools and dev
#    packages to the latest
# 5. Install virtualenv virtualenvwrapper for python 3.8


# hadolint ignore=DL3008,DL3059,DL3013,DL3042,DL3015
RUN     apt-get install -y --no-install-recommends            \
                python3.9 && \
        apt-get install -y --no-install-recommends            \
                python3.9-distutils                               \
                python3.9-dev                                     \
                python3.9-venv && \
        python3.9 -m pip install --upgrade setuptools && \
        python3.9 -m pip install --upgrade pip && \
        python3.9 -m pip install virtualenv virtualenvwrapper > /dev/null

# Install Python3.11, distutils and dev tools (so packages can be built if needed)
# and pip install so we can install python packages. We don't need this for the
# built in modules, but we know we have downloadable modules that could use Python3.11
# so we preempt the need.

# hadolint ignore=DL3008,DL3059,DL3013,DL3042,DL3015
RUN     apt-get install -y --no-install-recommends             \
        python3.11 && \
        apt-get install -y --no-install-recommends             \
                python3.11-distutils                               \
                python3.11-dev                                     \
                python3.11-venv && \
        python3.11 -m pip install --upgrade setuptools && \
        python3.11 -m pip install --upgrade pip && \
        python3.11 -m pip install virtualenv virtualenvwrapper > /dev/null

# Now we can install the CodeProject.AI SDK
RUN mkdir -p /install
WORKDIR /install

ARG BUILD_ARCH
ARG CPAI_VERSION_AARCH64="2.9.7"
ARG CPAI_VERSION_AMD64="2.9.5"
ARG CPAI_FILE_NAME_AARCH64="codeproject.ai-server_2.9.7_Ubuntu_arm64"
ARG CPAI_FILE_NAME_AMD64="codeproject.ai-server_2.9.5_Ubuntu_x64"
ARG CPAI_INSTALLER_URL_AMD64="https://codeproject-ai-bunny.b-cdn.net/server/installers/linux/codeproject.ai-server_2.9.5_Ubuntu_x64.zip"
ARG CPAI_INSTALLER_URL_AARCH64="https://codeproject-ai-bunny.b-cdn.net/server/installers/linux/codeproject.ai-server_2.9.7_Ubuntu_arm64.zip"

# Download the installer
RUN \
        set -x \
        && if [ "${BUILD_ARCH}" = "aarch64" ]; then \
                export INSTALLER_URL="${CPAI_INSTALLER_URL_AARCH64}"; \
        elif [ "${BUILD_ARCH}" = "amd64" ]; then \
                export INSTALLER_URL="${CPAI_INSTALLER_URL_AMD64}"; \
        fi \
        \
        && wget --progress=dot:giga "${INSTALLER_URL}"
# Unzip the installer
RUN \
        set -x \
        && if [ "${BUILD_ARCH}" = "aarch64" ]; then \
                export FILE_NAME="${CPAI_FILE_NAME_AARCH64}"; \
        elif [ "${BUILD_ARCH}" = "amd64" ]; then \
                export FILE_NAME="${CPAI_FILE_NAME_AMD64}"; \
        fi \
        \ 
        && unzip "${FILE_NAME}.zip"
# Install the package
RUN \
        set -x \
        && if [ "${BUILD_ARCH}" = "aarch64" ]; then \
                export FILE_NAME=${CPAI_FILE_NAME_AARCH64}; \
        elif [ "${BUILD_ARCH}" = "amd64" ]; then \
                export FILE_NAME=${CPAI_FILE_NAME_AMD64}; \
        fi \
        \ 
        && dpkg -i "${FILE_NAME}.deb"
# Clean up
RUN rm -rf /install

WORKDIR /

# Create symlink to /app
RUN \
        set -x \
        && if [ "${BUILD_ARCH}" = "aarch64" ]; then \
                export CPAI_VERSION="${CPAI_VERSION_AARCH64}"; \
        elif [ "${BUILD_ARCH}" = "amd64" ]; then \
                export CPAI_VERSION="${CPAI_VERSION_AMD64}"; \
        fi \
        && \
        ln -s "/usr/bin/codeproject.ai-server-${CPAI_VERSION}" /app 

RUN  cat /app/server/appsettings.json  | \
        sed -e 's/\/\/ .*$//g' | jq ".ServerOptions.DisableLegacyPort=true" > /app/server/appsettings.json.new \
        && mv /app/server/appsettings.json.new /app/server/appsettings.json        

# Copy root filesystem
COPY rootfs /

# Health check
HEALTHCHECK --start-period=5m \
    CMD curl --insecure --fail http://localhost:32168 || exit 1

# Build arguments
ARG BUILD_ARCH
ARG BUILD_DATE
ARG BUILD_DESCRIPTION
ARG BUILD_NAME
ARG BUILD_REF
ARG BUILD_REPOSITORY
ARG BUILD_VERSION

# Labels
LABEL \
    io.hass.name="${BUILD_NAME}" \
    io.hass.description="${BUILD_DESCRIPTION}" \
    io.hass.arch="${BUILD_ARCH}" \
    io.hass.type="addon" \
    io.hass.version=${BUILD_VERSION} \
    maintainer="StevusPrimus <stevusprimus@users.noreply.github.com>" \
    org.opencontainers.image.title="${BUILD_NAME}" \
    org.opencontainers.image.description="${BUILD_DESCRIPTION}" \
    org.opencontainers.image.vendor="Home Assistant 3rd Party Add-ons" \
    org.opencontainers.image.authors="StevusPrimus <stevusprimus@users.noreply.github.com>" \
    org.opencontainers.image.licenses="MIT" \
    #org.opencontainers.image.url="https://addons.community" \
    org.opencontainers.image.source="https://github.com/${BUILD_REPOSITORY}" \
    org.opencontainers.image.documentation="https://github.com/${BUILD_REPOSITORY}/blob/main/README.md" \
    org.opencontainers.image.created=${BUILD_DATE} \
    org.opencontainers.image.revision=${BUILD_REF} \
    org.opencontainers.image.version=${BUILD_VERSION}
