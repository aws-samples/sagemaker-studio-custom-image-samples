## Jupyter Docker Stacks Data Science


### Overview

This custom image uses the [Jupyter Docker Stacks Data Science image](https://github.com/jupyter/docker-stacks/tree/master/datascience-notebook) in SageMaker Studio.

### Building the image

Build the Docker image and push to Amazon ECR. 
```
# Modify these as required
REGION=<aws-region>
ACCOUNT_ID=<aws-account-id>


# Build the image
IMAGE_NAME=julia-datascience
$(aws --region ${REGION} ecr get-login --no-include-email)
docker build . -t ${IMAGE_NAME} -t ${ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/smstudio-custom:${IMAGE_NAME}
```

```
docker push ${ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/smstudio-custom:${IMAGE_NAME}
```

### Using with SageMaker Studio

Create a SageMaker Image (SMI) with the image in ECR. 

```
# Role in your account to be used for SMI. Modify as required.

ROLE_ARN=arn:aws:iam::421258792169:role/Admin
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

NOTE: This image contains multiple kernels (R, Julia, Python). However, we can only specify one image in the AppImageConfig API. Only this kernel will be shown to users *before* the image has started. Once the image is running, all the kernels will be visible in JupyterLab. 

Create a Domain, providing the SageMaker Image and AppImageConfig in the Domain creation. Replace the placeholders for VPC ID, Subnet IDs, and Execution Role in `create-domain-input.json`

```
aws --region ${REGION} sagemaker create-domain --cli-input-json file://create-domain-input.json
```

If you have an existing Domain, you can also use the `update-domain`

```
aws --region ${REGION} sagemaker update-domain --cli-input-json file://update-domain-input.json
```

Now create a user, and start a Notebook using the SageMaker Studio launcher 