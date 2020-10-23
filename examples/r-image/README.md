## R Image

### Overview

> NOTE: This Dockerfile installs dependencies that may be licensed under copyleft licenses such as GPLv3. You should review the license terms and make sure they are acceptable for your use case before proceeding and downloading this image.

A R kernel that is a compatible as a Bring-Your-Own Image in SageMaker Studio.

The image contains a selection of R packages, along with the AWS Python SDK (boto3) and the SageMaker Python SDK which can be used from R using `reticulate`.

### Building the image

Build the Docker image and push to Amazon ECR. 
```
# Modify these as required
REGION=<aws-region>
ACCOUNT_ID=<account-id>


# Build the image
IMAGE_NAME=custom-r
$(aws --region ${REGION} ecr get-login --no-include-email)
docker build . -t ${IMAGE_NAME} -t ${ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/smstudio-custom:${IMAGE_NAME}
```

```
docker push ${ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/smstudio-custom:${IMAGE_NAME}
```

### Using with SageMaker Studio

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