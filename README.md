## SageMaker Studio Custom Image Samples

### Overview

This repository contains examples of Docker images that are valid custom images for KernelGateway Apps in SageMaker Studio. These custom images enable you to bring your own packages, files, and kernels for use with notebooks, terminals, and interactive consoles within SageMaker Studio.

You can find more information about using Custom Images in the [SageMaker Developer Guide](https://docs.aws.amazon.com/sagemaker/latest/dg/studio-byoi.html).

### Examples

- [echo-kernel-image](examples/echo-kernel-image) - Uses the echo_kernel from Jupyter as a "Hello World" introduction into writing custom KernelGateway images.
- [javascript-tf-image](examples/javascript-tf-image) - [tslab](https://www.npmjs.com/package/tslab)-based kernels for JavaScript or TypeScript, including [TensorFlow.js](https://www.tensorflow.org/js) and CUDA GPU libraries.
- [jupyter-docker-stacks-julia-image](examples/jupyter-docker-stacks-julia-image) - Leverages the Data Science image from Jupyter Docker Stacks to add a Julia kernel.
- [r-image](examples/r-image) - Contains the `ir` kernel and a selection of R packages, along with the AWS Python SDK (boto3) and the SageMaker Python SDK which can be used from R using `reticulate`
- [rapids-image](examples/rapids-image) - Uses the offical rapids.ai image from Dockerhub. Use with a GPU instance on Studio
- [scala-image](examples/scala-image) - Adds a Scala kernel based on [Almond Scala Kernel](https://almond.sh/).
- [tf2.3-image](examples/tf23-image) - Uses the official TensorFlow 2.3 image from DockerHub and demonstrates bundling custom files along with the image.

### One-time setup

All examples have a one-time setup to create an ECR repository

```
REGION=<aws-region>
aws --region ${REGION} ecr create-repository \
    --repository-name smstudio-custom
```

### Developing Custom Images

See [DEVELOPMENT.md](DEVELOPMENT.md)

### License

This sample code is licensed under the MIT-0 License. See the LICENSE file.
