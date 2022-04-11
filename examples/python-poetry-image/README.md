## Python Poetry Image

### Overview

This example creates a custom image in Amazon SageMaker Studio using [Poetry](https://python-poetry.org/) to manage the Python dependencies.

### Building the image

Build the Docker image and push to Amazon ECR.
```
# Modify these as required. The Docker registry endpoint can be tuned based on your current region from https://docs.aws.amazon.com/general/latest/gr/ecr.html#ecr-docker-endpoints
REGION=<aws-region>
ACCOUNT_ID=<account-id>
IMAGE_NAME=custom-poetry-kernel

# Create ECR Repository. Ignore if it exists. For simplcity, all examples in the repo
# use same ECR repo with different image tags
aws --region ${REGION} ecr create-repository --repository-name smstudio-custom

# Build the image
aws --region ${REGION} ecr get-login-password | docker login --username AWS --password-stdin ${ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/smstudio-custom
docker build . -t ${IMAGE_NAME} -t ${ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/smstudio-custom:${IMAGE_NAME}

docker push ${ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/smstudio-custom:${IMAGE_NAME}
```

### Using it with SageMaker Studio

Create a SageMaker Image (SMI) with the image in ECR. Request parameter RoleArn value is used to get
ECR image information when and Image version is created. After creating Image, create an Image Version during which 
SageMaker stores image metadata like SHA etc. Everytime an image is updated in ECR, a new image version should be created.
See [Update Image](#updating-image-with-sageMaker-studio)

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

### Update Image with SageMaker Studio
If you found an issue with your image or want to update Image with new features, Use following steps

Re-Build and push the image to ECR

```
# Build and push the image
aws --region ${REGION} ecr get-login-password | docker login --username AWS --password-stdin ${ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/smstudio-custom
docker build . -t ${IMAGE_NAME} -t ${ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/smstudio-custom:${IMAGE_NAME}
docker push ${ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/smstudio-custom:${IMAGE_NAME}
```


Create new App Image Version.
```
aws --region ${REGION} sagemaker create-image-version \
    --image-name ${IMAGE_NAME} \
    --base-image "${ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/smstudio-custom:${IMAGE_NAME}"

# Verify the image-version is created successfully. Do NOT proceed if image-version is in CREATE_FAILED state or in any other state apart from CREATED.
aws --region ${REGION} sagemaker describe-image-version --image-name ${IMAGE_NAME}
```


Re-Create App in SageMaker studio. 

### Notes

* Since SageMaker Studio overrides `ENTRYPOINT` and `CMD` instructions (see [documentation](https://docs.aws.amazon.com/sagemaker/latest/dg/studio-byoi-specs.html)), this sample disables the Poetry virtual environments as recommended in their [FAQ](https://python-poetry.org/docs/faq/#i-dont-want-poetry-to-manage-my-virtual-environments-can-i-disable-it). 
* Note that `ipykernel` must be installed on custom images for SageMaker Studio. If this package is not installed by default on the parent image, then it should be included in the `pyproject.toml` file. 
