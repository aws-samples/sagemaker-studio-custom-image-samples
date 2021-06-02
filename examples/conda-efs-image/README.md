## Conda EFS Image

### Overview

This example creates a custom image in Amazon SageMaker Studio which will load a `custom` conda environment from a users [Elastic File System](https://docs.aws.amazon.com/sagemaker/latest/dg/studio-tasks-manage-storage.html) (EFS) home folder.

The Dockerfile is built from `ubuntu:20.04` and installs the latest [Minconda3](https://docs.conda.io/en/latest/miniconda.html) distribution.

### Building the image

Build the Docker image and push to Amazon ECR.
```
# Modify these as required. The Docker registry endpoint can be tuned based on your current region from https://docs.aws.amazon.com/general/latest/gr/ecr.html#ecr-docker-endpoints
REGION=<aws-region>
ACCOUNT_ID=<account-id>

# Build the image
IMAGE_NAME=custom-conda-efs
aws --region ${REGION} ecr get-login-password | docker login --username AWS --password-stdin ${ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/smstudio-custom
docker build . -t ${IMAGE_NAME} -t ${ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/smstudio-custom:${IMAGE_NAME} --build-arg NB_ENV=custom
```

NOTE: The `NB_ENV` maps to the conda environment which will need to be created on EFS (see below)

```
docker push ${ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/smstudio-custom:${IMAGE_NAME}
```

### Using it with SageMaker Studio

Create a SageMaker Image with the image in ECR. 

```
# Role in your account to be used for the SageMaker Image
ROLE_ARN=<role-arn>

aws --region ${REGION} sagemaker create-image \
    --image-name ${IMAGE_NAME} \
    --display-name "Conda EFS env" \
    --role-arn ${ROLE_ARN}

aws --region ${REGION} sagemaker create-image-version \
    --image-name ${IMAGE_NAME} \
    --base-image "${ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/smstudio-custom:${IMAGE_NAME}"

# Verify the image-version is created successfully. Do NOT proceed if image-version is in CREATE_FAILED state or in any other state apart from CREATED.
aws --region ${REGION} sagemaker describe-image-version --image-name ${IMAGE_NAME}
```

Create an AppImageConfig for this image.

```
aws --region ${REGION} sagemaker create-app-image-config --cli-input-json file://app-image-config-input.json
```

Create a Domain, providing the SageMaker Image and AppImageConfig in the Domain creation. Replace the placeholders for VPC ID, Subnet IDs, and Execution Role in `create-domain-input.json`.

```
aws --region ${REGION} sagemaker create-domain --cli-input-json file://create-domain-input.json
```

If you have an existing Domain, you can use the `update-domain` command.

```
DOMAIN_ID=<domain-id>
aws --region ${REGION} sagemaker update-domain --domain-id ${DOMAIN_ID} --cli-input-json file://update-domain-input.json
```

### Create your custom conda environment on EFS

Use the [Amazon SageMaker Studio Launcher](https://docs.aws.amazon.com/sagemaker/latest/dg/studio-launcher.html) to open a **System terminal** and run the following commands to setup your custom environment:

```bash
mkdir -p ~/.conda/envs
conda create -p ~/.conda/envs/custom
conda activate custom # or ". activate ~/.conda/envs/custom"
conda install ipykernel
```

### Verifying your installation

Use the [Amazon SageMaker Studio Launcher](https://docs.aws.amazon.com/sagemaker/latest/dg/studio-launcher.html) to launch a **Notebook** for your *Conda EFS env* SageMaker Image and run the following command in the first cell to verify you are now using a `custom` conda environment:

```
!conda info
```

### Installing additional libraries

Use the [Amazon SageMaker Studio Launcher](https://docs.aws.amazon.com/sagemaker/latest/dg/studio-launcher.html) to launch an **Image terminal** and run the following commands to install new libraries in your custom environment such as `numpy`:

```bash
. activate /home/$NB_USER/.conda/envs/$NB_ENV
conda install numpy
```