# A CUDA-capable, tslab-based JS/TS kernel container for SageMaker Studio, including TensorFlow.js.
#
# Python & CUDA configuration with inspiration from the AWS TensorFlow Deep Learning containers, e.g:
# https://github.com/aws/deep-learning-containers/blob/master/tensorflow/training/docker/2.4/py3/cu110/Dockerfile.gpu
#
# Use with Jupyter kernel 'jslab' or 'tslab'; user config as per NB_UID/NB_GID; home folder /home/sagemaker-user
FROM nvidia/cuda:11.0-base-ubuntu18.04

ARG NB_USER="sagemaker-user"
ARG NB_UID="1000"
ARG NB_GID="100"

ARG NODEJS_VERSION=14.x
ARG PYTHON_VERSION=3.9.4
ARG PYTHON=python3.9
ARG PYTHON_PIP=python3-pip
ARG PIP=pip3

# Prevent setup prompts hanging on user input in our non-interactive environment:
ENV DEBIAN_FRONTEND=noninteractive
ENV DEBCONF_NONINTERACTIVE_SEEN=true

# Python config for logging, IO, etc:
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1
ENV PYTHONIOENCODING=UTF-8
ENV LANG=C.UTF-8
ENV LC_ALL=C.UTF-8

# Optimizing TF for Intel/MKL, as per:
# https://software.intel.com/content/www/us/en/develop/articles/maximize-tensorflow-performance-on-cpu-considerations-and-recommendations-for-inference.html
# (May not be relevant for TensorFlow.js?)
ENV KMP_AFFINITY=granularity=fine,compact,1,0
ENV KMP_BLOCKTIME=1
ENV KMP_SETTINGS=0
ENV MANUAL_BUILD=0

USER root
WORKDIR /root

# Setup the NB user with root privileges.
RUN apt-get update && \
  apt-get install -y sudo && \
  useradd -m -s /bin/bash -N -u $NB_UID $NB_USER && \
  chmod g+w /etc/passwd && \
  echo "${NB_USER}    ALL=(ALL)    NOPASSWD:    ALL" >> /etc/sudoers && \
  # Prevent apt-get cache from being persisted to this layer.
  rm -rf /var/lib/apt/lists/*

RUN apt-get update \
  && apt-get install -y --no-install-recommends --allow-unauthenticated \
    ca-certificates \
    cuda-command-line-tools-11-0 \
    cuda-cudart-dev-11-0 \
    libcufft-dev-11-0 \
    libcurand-dev-11-0 \
    libcusolver-dev-11-0 \
    libcusparse-dev-11-0 \
    curl \
    libcudnn8=8.0.5.39-1+cuda11.0 \
    # TensorFlow doesn't require libnccl anymore but Open MPI still depends on it
    libnccl2=2.7.8-1+cuda11.0 \
    libgomp1 \
    libnccl-dev=2.7.8-1+cuda11.0 \
    libfreetype6-dev \
    libhdf5-serial-dev \
    liblzma-dev \
    libtemplate-perl \
    libzmq3-dev \
    git \
    unzip \
    wget \
    libtool \
    libssl1.1 \
    openssl \
    build-essential \
    zlib1g-dev \
  && apt-get update \
  && apt-get install -y --no-install-recommends --allow-unauthenticated \
    libcublas-11-0=11.2.0.252-1 \
    libcublas-dev-11-0=11.2.0.252-1 \
    # The 'apt-get install' of nvinfer-runtime-trt-repo-ubuntu1804-5.0.2-ga-cuda10.0
    # adds a new list which contains libnvinfer library, so it needs another
    # 'apt-get update' to retrieve that list before it can actually install the
    # library.
    # We don't install libnvinfer-dev since we don't need to build against TensorRT,
    # and libnvinfer4 doesn't contain libnvinfer.a static library.
    # nvinfer-runtime-trt-repo doesn't have a 1804-cuda10.1 version yet. see:
    # https://developer.download.nvidia.cn/compute/machine-learning/repos/ubuntu1804/x86_64/
  && apt-get update && apt-get install -y --no-install-recommends --allow-unauthenticated  \
    nvinfer-runtime-trt-repo-ubuntu1804-5.0.2-ga-cuda10.0 \
  && apt-get update && apt-get install -y --no-install-recommends --allow-unauthenticated  \
    libnvinfer7=7.1.3-1+cuda11.0 \
  && rm -rf /var/lib/apt/lists/*

# Set default NCCL parameters
RUN echo NCCL_DEBUG=INFO >> /etc/nccl.conf

# /usr/local/lib/libpython* needs to be accessible for dynamic linking
ENV LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH

RUN apt-get update \
  && apt-get install -y --no-install-recommends \
    libbz2-dev \
    libc6-dev \
    libffi-dev \
    libgdbm-dev \
    libncursesw5-dev \
    libreadline-gplv2-dev \
    libsqlite3-dev \
    libssl-dev \
    tk-dev \
  && rm -rf /var/lib/apt/lists/* \
  && apt-get clean

# Install specific Python version:
RUN wget https://www.python.org/ftp/python/$PYTHON_VERSION/Python-$PYTHON_VERSION.tgz \
  && tar -xvf Python-$PYTHON_VERSION.tgz \
  && cd Python-$PYTHON_VERSION \
  && ./configure --enable-shared && make && make install \
  && rm -rf ../Python-$PYTHON_VERSION*

RUN ${PIP} --no-cache-dir install --upgrade \
    pip \
    setuptools

# Provide a "python" binary for any tools that need it:
RUN ln -s $(which ${PYTHON}) /usr/local/bin/python \
  && ln -s $(which ${PIP}) /usr/bin/pip

# Install specific NodeJS version:
RUN curl --silent --location https://deb.nodesource.com/setup_$NODEJS_VERSION | bash -
RUN apt-get install --yes nodejs

# Optional Python packages for AWS/SageMaker/DataScience/C++ in case they're helpful:
RUN ${PIP} install --no-cache-dir \
    pybind11 \
    cmake==3.18.2.post1 \
    # (Numpy & Pandas needed for SageMaker SDK)
    numpy==1.20.0 \
    pandas==1.2.4 \
    # python-dateutil==2.8.1 to satisfy botocore associated with latest awscli
    python-dateutil==2.8.1 \
    # install PyYAML>=5.4 to avoid conflict with latest awscli
    "pyYAML>=5.4,<5.5" \
    requests==2.25.1 \
    "awscli<2" \
    "sagemaker>=2,<3" \
    sagemaker-experiments==0.* \
    smclarify \
    smdebug==1.0.8

# More setup of optional Python packages:
ENV CPATH="/usr/local/lib/python3.9/dist-packages/pybind11/include/"
RUN apt-get update && apt-get -y install cmake protobuf-compiler

# Required Python packages:
RUN ${PIP} install --no-cache-dir \
    # tslab brings a separate kernel, but uses Python during setup so needs ipykernel to be present:
    "ipykernel>=5,<6" \
  && ${PYTHON} -m ipykernel install --sys-prefix

# Install NodeJS libraries:
# `npm install -g` pushes global installs to /usr/lib/node_modules in this case, which tslab doesn't seem
# able to resolve per the issue below - even if we `ENV NODE_PATH=/usr/lib/node_modules`... So instead,
# we'll central/kernel-provided libs non-"globally" at the filesystem root:
WORKDIR /
RUN npm install \
    aws-sdk@2 \
    # (For performance, use -node-gpu where you can, else -node, else tfjs)
    @tensorflow/tfjs@3.5 \
    @tensorflow/tfjs-node@3.5 \
    @tensorflow/tfjs-node-gpu@3.5 \
    # tslab supports both TypeScript and JavaScript:
    typescript@1.4

# The `tslab` kernel provider should be installed globally, and then hooked in to Jupyter:
RUN npm install -g tslab@1.0 \
  && tslab install

# Now final user setup:
USER $NB_UID

# Set up user env vars:
# (Bash default shell gives a better Jupyter terminal UX than `sh`)
ENV SHELL=/bin/bash \
  NB_USER=$NB_USER \
  NB_UID=$NB_UID \
  NB_GID=$NB_GID \
  HOME=/home/$NB_USER

WORKDIR $HOME

# SageMaker will override the entrypoint when running in context - so just set bash for debugging:
CMD ["/bin/bash"]
