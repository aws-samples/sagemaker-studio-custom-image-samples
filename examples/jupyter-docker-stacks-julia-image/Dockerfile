FROM jupyter/datascience-notebook:julia-1.8.5

# Install the AWS CLI:
RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" && \
    unzip awscliv2.zip
USER root
RUN ./aws/install
USER $NB_UID

# Install various Python utilities for SageMaker, and PyCall to enable using them from Julia:
# (Pinned to last tested major version for repeatability)
RUN julia -e 'import Pkg; Pkg.update()' && \
    julia -e 'using Pkg; pkg"add PyCall@1"; pkg"precompile"' && \
    pip install \
        'boto3>=1,<2' \
        'sagemaker>=2,<3' \
        'sagemaker-experiments>=0.1,<0.2' \
        'sagemaker-studio-image-build>=0.4,<0.5' \
        'smdebug>=0.9,<0.10'
