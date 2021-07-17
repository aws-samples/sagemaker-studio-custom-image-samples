FROM ubuntu:18.04

ARG NB_USER="sagemaker-user"
ARG NB_UID="1000"
ARG NB_GID="100"

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Setup the "sagemaker-user" user with root privileges.
RUN apt-get update && \
    apt-get install -y sudo && \
    useradd -m -s /bin/bash -N -u $NB_UID $NB_USER && \
    chmod g+w /etc/passwd && \
    echo "${NB_USER}    ALL=(ALL)    NOPASSWD:    ALL" >> /etc/sudoers && \
    # Prevent apt-get cache from being persisted to this layer.
    apt-get clean && rm -rf /var/lib/apt/lists/* && \
    echo "en_US.UTF-8 UTF-8" > /etc/locale.gen

USER $NB_UID

# Make the default shell bash (vs "sh") for a better Jupyter terminal UX
ENV SHELL=/bin/bash \
    NB_USER=$NB_USER \
    NB_UID=$NB_UID \
    NB_GID=$NB_GID \
    LC_ALL=en_US.UTF-8 \
    LANG=en_US.UTF-8 \
    LANGUAGE=en_US.UTF-8 \
    HOME=/home/$NB_USER \
    MINICONDA_VERSION=4.6.14 \
    CONDA_VERSION=4.6.14 \
    MINICONDA_MD5=718259965f234088d785cad1fbd7de03 \
    CONDA_DIR=/opt/conda \
    PATH=$CONDA_DIR/bin:${PATH}


USER root
RUN apt-get update --yes
# COPY ./apt-packages.txt /root/apt-packages.txt
# RUN xargs -a /root/apt-packages.txt apt-get install -y --no-install-recommends

RUN apt-get update --yes && \
    apt-get install --yes --no-install-recommends \
    wget \
    curl \
    ca-certificates \
    sudo \
    locales \
    fonts-liberation \
    run-one && \
    apt-get clean && rm -rf /var/lib/apt/lists/* && \
    echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && \
    locale-gen

RUN mkdir -p $CONDA_DIR && \
    chown -R $NB_USER:$NB_GID $CONDA_DIR && \
    # Fix for devtools https://github.com/conda-forge/r-devtools-feedstock/issues/4
    ln -s /bin/tar /bin/gtar

# Copy a script that we will use to correct permissions after running certain commands
COPY fix-permissions /usr/local/bin/fix-permissions
RUN chmod a+rx /usr/local/bin/fix-permissions
COPY fix-host-settings /usr/local/bin/fix-host-settings
RUN chmod a+rx /usr/local/bin/fix-host-settings

RUN echo "auth requisite pam_deny.so" >> /etc/pam.d/su && \
    sed -i.bak -e 's/^%admin/#%admin/' /etc/sudoers && \
    sed -i.bak -e 's/^%sudo/#%sudo/' /etc/sudoers && \
    usermod -G root ${NB_USER} && \
    mkdir -p "${CONDA_DIR}" && \
    chown "${NB_USER}:${NB_GID}" "${CONDA_DIR}" && \
    chmod g+w /etc/passwd && \
    fix-permissions "${HOME}" && \
    fix-permissions "${CONDA_DIR}"

USER ${NB_UID}
ARG PYTHON_VERSION=3.6.14
ENV PATH=$CONDA_DIR/bin:${PATH}
WORKDIR /tmp

# Install conda via Miniconda
RUN curl --silent --show-error --output miniconda-installer.sh https://repo.anaconda.com/miniconda/Miniconda3-${MINICONDA_VERSION}-Linux-x86_64.sh && \
    echo "${MINICONDA_MD5} *miniconda-installer.sh" | md5sum -c - && \
    /bin/bash miniconda-installer.sh -f -b -p $CONDA_DIR && \
    rm miniconda-installer.sh && \
    conda config --system --prepend channels conda-forge && \
    conda config --system --set auto_update_conda false && \
    conda config --system --set show_channel_urls true && \
    conda config --system --set pip_interop_enabled true && \
    conda install --quiet --yes conda="${CONDA_VERSION%.*}.*" && \
    conda update --all --quiet --yes && \
    conda clean --all -f -y && \
    rm -rf /home/$NB_USER/.cache/yarn

RUN conda install --quiet --yes \
    tini \
    boto3 \
    'awscli>=1.18' \
    sagemaker_pyspark \
    'pyspark==2.4.0' \
    'notebook=6.4.0' \
    'jupyterhub=1.4.1' \
    'jupyterlab=3.0.16' && \
    conda clean --all -f -y && \
    npm cache clean --force && \
    jupyter notebook --generate-config && \
    jupyter lab clean && \
    rm -rf "/home/${NB_USER}/.cache/yarn" && \
    fix-permissions "${CONDA_DIR}" && \
    fix-permissions "/home/${NB_USER}"

EXPOSE 8888

# Copy local files as late as possible to avoid cache busting
COPY start.sh start-notebook.sh start-singleuser.sh /usr/local/bin/
# Currently need to have both jupyter_notebook_config and jupyter_server_config to support classic and lab
COPY jupyter_notebook_config.py /etc/jupyter/

# Fix permissions on /etc/jupyter as root
USER root

# Prepare upgrade to JupyterLab V3.0 #1205
RUN sed -re "s/c.NotebookApp/c.ServerApp/g" \
    /etc/jupyter/jupyter_notebook_config.py > /etc/jupyter/jupyter_server_config.py && \
    fix-permissions /etc/jupyter/

# Spark dependencies
# Default values can be overridden at build time
# (ARGS are in lower case to distinguish them from ENV)
ARG spark_version="2.4.0"
ARG hadoop_version="2.7"
ARG spark_checksum="5F4184E0FE7E5C8AE67F5E6BC5DEEE881051CC712E9FF8AEDDF3529724C00E402C94BB75561DD9517A372F06C1FCB78DC7AE65DCBD4C156B3BA4D8E267EC2936"
ARG openjdk_version="8"

ENV APACHE_SPARK_VERSION="${spark_version}" \
    HADOOP_VERSION="${hadoop_version}"

RUN apt-get update --yes && \
    apt-get install --yes --no-install-recommends \
    "openjdk-${openjdk_version}-jre-headless" \
    ca-certificates-java && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

# Spark installation
WORKDIR /tmp
RUN wget -q "https://archive.apache.org/dist/spark/spark-${APACHE_SPARK_VERSION}/spark-${APACHE_SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz" && \
    echo "${spark_checksum} *spark-${APACHE_SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz" | sha512sum -c - && \
    tar xzf "spark-${APACHE_SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz" -C /usr/local --owner root --group root --no-same-owner && \
    rm "spark-${APACHE_SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz"

WORKDIR /usr/local

# Configure Spark
ENV SPARK_HOME=/usr/local/spark
ENV SPARK_OPTS="--driver-java-options=-Xms1024M --driver-java-options=-Xmx4096M --driver-java-options=-Dlog4j.logLevel=info" \
    PATH="${PATH}:${SPARK_HOME}/bin"

RUN ln -s "spark-${APACHE_SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}" spark && \
    # Add a link in the before_notebook hook in order to source automatically PYTHONPATH
    mkdir -p /usr/local/bin/before-notebook.d && \
    ln -s "${SPARK_HOME}/sbin/spark-config.sh" /usr/local/bin/before-notebook.d/spark-config.sh

USER ${NB_UID}

WORKDIR $HOME
