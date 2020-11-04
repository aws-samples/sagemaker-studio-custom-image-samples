## RAPIDS.ai Image

### Overview

A RAPIDS kernel that is a compatible as a Bring-Your-Own Image in SageMaker Studio.

The image is based on the [RAPIDS image](https://hub.docker.com/r/rapidsai/rapidsai/).

### Building the image

Build the Docker image and push to Amazon ECR. 
```
# Modify these as required. The Docker registry endpoint can be tuned based on your current region from https://docs.aws.amazon.com/general/latest/gr/ecr.html#ecr-docker-endpoints
REGION=<aws-region>
ACCOUNT_ID=<account-id>


# Build the image
IMAGE_NAME=custom-rapids
aws --region ${REGION} ecr get-login-password | docker login --username AWS --password-stdin ${ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/smstudio-custom
docker build . -t ${IMAGE_NAME} -t ${ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/smstudio-custom:${IMAGE_NAME}
```

```
docker push ${ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/smstudio-custom:${IMAGE_NAME}
```

### Using with SageMaker Studio

```
pip install sagemaker-studio-image-build
```

then build and push using sm-docker build (make sure you are in this directory with the dockerfile)

```
sm-docker build .
```


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

Create a AppImageConfig for this image

```
aws --region ${REGION} sagemaker create-app-image-config --cli-input-json file://app-image-config-input.json

```

Create a Domain, providing the SageMaker Image and AppImageConfig in the Domain creation. Replace the placeholders for VPC ID, Subnet IDs, and Execution Role in `create-domain-input.json`

```
aws --region ${REGION} sagemaker create-domain --cli-input-json file://create-domain-input.json
```

If you have an existing Domain, you can also use the `update-domain`

```
aws --region ${REGION} sagemaker update-domain --cli-input-json file://update-domain-input.json
```
