FROM jupyter/datascience-notebook:julia-1.5.2

# Install the AWS CLI:
RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" && \
    unzip awscliv2.zip
USER root
RUN ./aws/install
USER $NB_UID

# Install various Python utilities for SageMaker, and PyCall to enable using them from Julia:
RUN julia -e 'import Pkg; Pkg.update()' && \
    julia -e 'using Pkg; pkg"add PyCall"; pkg"precompile"' && \
    pip install boto3 sagemaker sagemaker-experiments sagemaker-studio-image-build smdebug
