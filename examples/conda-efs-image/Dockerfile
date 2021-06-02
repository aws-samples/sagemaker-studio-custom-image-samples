FROM public.ecr.aws/ubuntu/ubuntu:20.04

ARG NB_USER="sagemaker-user"
ARG NB_UID="1000"
ARG NB_GID="100"
ARG NB_ENV="custom"

######################
# OVERVIEW
# 1. Creates the `sagemaker-user` user with UID/GID 1000/100.
# 2. Ensures this user can `sudo` by default.
# 5. Make the default shell `bash`. This enhances the experience inside a Jupyter terminal as otherwise Jupyter defaults to `sh`
######################

# Setup the "sagemaker-user" user with root privileges.
RUN \
    apt-get update && \
    apt-get install -y sudo wget nano && \
    useradd -m -s /bin/bash -N -u $NB_UID $NB_USER && \
    chmod g+w /etc/passwd && \
    echo "${NB_USER}    ALL=(ALL)    NOPASSWD:    ALL" >> /etc/sudoers && \
    # Prevent apt-get cache from being persisted to this layer.
    rm -rf /var/lib/apt/lists/*

# installing miniconda
RUN wget -q https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
RUN bash Miniconda3-latest-Linux-x86_64.sh -b -p /miniconda
ENV PATH=$PATH:/miniconda/condabin:/miniconda/bin

# install libraries for the base environment
RUN conda install ipykernel

# update kernelspec to load our custom environment strored on EFS
RUN rm -rf /miniconda/share/jupyter/kernels/python3
# write new kernel which initialises uses conda run for custom env
# see: https://github.com/ipython/ipykernel/issues/416
COPY custom_kernel_spec/ /miniconda/share/jupyter/kernels/python3

# Make the default shell bash (vs "sh") for a better Jupyter terminal UX
ENV SHELL=/bin/bash \
    NB_USER=$NB_USER \
    NB_ENV=$NB_ENV \
    NB_UID=$NB_UID \
    NB_GID=$NB_GID \
    HOME=/home/$NB_USER

# Set the conda envs path to map to EFS (without of requiring .condarc)
ENV CONDA_AUTO_ACTIVATE_BASE=false \
    CONDA_ENVS_PATH=/home/$NB_USER/.conda/envs

WORKDIR $HOME
USER $NB_UID