FROM python:3.7

ARG NB_USER="sagemaker-user"
ARG NB_UID="1000"
ARG NB_GID="100"

######################
# OVERVIEW
# 1. Creates the `sagemaker-user` user with UID/GID 1000/100.
# 2. Ensures this user can `sudo` by default. 
# 3. Installs and configures Poetry, then installs the environment defined in pyproject.toml
# 4. Configures the kernel (ipykernel should be installed on the parent image or defined in pyproject.toml)
# 5. Make the default shell `bash`. This enhances the experience inside a Jupyter terminal as otherwise Jupyter defaults to `sh`
######################

# Setup the "sagemaker-user" user with root privileges.
RUN \
    apt-get update && \
    apt-get install -y sudo && \
    useradd -m -s /bin/bash -N -u $NB_UID $NB_USER && \
    chmod g+w /etc/passwd && \
    echo "${NB_USER}    ALL=(ALL)    NOPASSWD:    ALL" >> /etc/sudoers && \
    # Prevent apt-get cache from being persisted to this layer.
    rm -rf /var/lib/apt/lists/*

# Install Poetry
RUN pip install poetry
# Disable virtual environments (see notes in README.md)
RUN poetry config virtualenvs.create false --local
# Copy the environment definition file and install the environment
COPY pyproject.toml /
RUN poetry install

# Configure the kernel
RUN python -m ipykernel install --sys-prefix

# Make the default shell bash (vs "sh") for a better Jupyter terminal UX
ENV SHELL=/bin/bash

USER $NB_UID