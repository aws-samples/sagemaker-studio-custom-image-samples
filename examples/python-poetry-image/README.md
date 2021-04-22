## Python Poetry Image

### Overview

This example creates a custom image in Amazon SageMaker Studio using [Poetry](https://python-poetry.org/) to manage the Python dependencies.

### Building the image

Build the Docker image and push to Amazon ECR.
```
# Modify these as required. The Docker registry endpoint can be tuned based on your current region from https://docs.aws.amazon.com/general/latest/gr/ecr.html#ecr-docker-endpoints
REGION=<aws-region>
ACCOUNT_ID=<account-id>

# Build the image
IMAGE_NAME=custom-poetry-kernel
aws --region ${REGION} ecr get-login-password | docker login --username AWS --password-stdin ${ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/smstudio-custom
docker build . -t ${IMAGE_NAME} -t ${ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/smstudio-custom:${IMAGE_NAME}
```

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
aws --region ${REGION} sagemaker update-domain --cli-input-json file://update-domain-input.json
```

### Notes

* Since SageMaker Studio overrides `ENTRYPOINT` and `CMD` instructions (see [documentation](https://docs.aws.amazon.com/sagemaker/latest/dg/studio-byoi-specs.html)), this sample disables the Poetry virtual environments as recommended in their [FAQ](https://python-poetry.org/docs/faq/#i-dont-want-poetry-to-manage-my-virtual-environments-can-i-disable-it). 
* Note that `ipykernel` must be installed on custom images for SageMaker Studio. If this package is not installed by default on the parent image, then it should be included in the `pyproject.toml` file. 