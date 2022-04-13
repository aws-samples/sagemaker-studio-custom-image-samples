# SageMaker Studio Custom Image Samples

## Overview

This repository contains examples of Docker images that are valid custom images for KernelGateway Apps in SageMaker Studio. These custom images enable you to bring your own packages, files, and kernels for use with notebooks, terminals, and interactive consoles within SageMaker Studio.

### Examples

- [conda-env-kernel-image](examples/conda-env-kernel-image) - This example creates a custom Conda environment in the Docker image and demonstrates using it as a custom kernel. 
- [echo-kernel-image](examples/echo-kernel-image) - This example uses the echo_kernel from Jupyter as a "Hello World" introduction into writing custom KernelGateway images.
- [jupyter-docker-stacks-julia-image](examples/jupyter-docker-stacks-julia-image) - This example leverages the Data Science image from Jupyter Docker Stacks to add a Julia kernel.
- [python-poetry-image](examples/python-poetry-image) - This example uses Poetry to manage the package dependencies in Python.
- [r-image](examples/r-image) - This example contains the `ir` kernel and a selection of R packages, along with the AWS Python SDK (boto3) and the SageMaker Python SDK which can be used from R using `reticulate`
- [rapids-image](examples/rapids-image) - This example uses the offical rapids.ai image from Dockerhub. Use with a GPU instance on Studio
- [scala-image](examples/scala-image) - This example adds a Scala kernel based on [Almond Scala Kernel](https://almond.sh/).
- [tf2.3-image](examples/tf23-image) - This examples uses the official TensorFlow 2.3 image from DockerHub and demonstrates bundling custom files along with the image.
- [spark-image](examples/spark-image) - This example gives the spark image to do python spark code developement in SageMaker studio.

#### One-time setup

All examples have a one-time setup to create an ECR repository

```bash
REGION=<aws-region>
aws --region ${REGION} ecr create-repository \
    --repository-name smstudio-custom
```

### Developing Custom Images

See [DEVELOPMENT.md](DEVELOPMENT.md)

### License

This sample code is licensed under the MIT-0 License. See the LICENSE file.
