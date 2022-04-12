## Conda Environments as Kernels


### Overview

This custom image sample demonstrates how to create a custom Conda environment in a Docker image and use it as a custom kernel in SageMaker Studio. 

The Conda environment must have the appropriate kernel package installed, for e.g., `ipykernel` for a Python kernel. This example creates a Conda environment called `myenv` with a few Python packages (see [environment.yml](environment.yml)) and the `ipykernel`. SageMaker Studio will automatically recognize this Conda environment as a kernel named `conda-env-myenv-py` (See  [app-image-config-input.json](app-image-config-input.json))

### Building the image

Build the Docker image and push to Amazon ECR. 
```
# Modify these as required. The Docker registry endpoint can be tuned based on your current region from https://docs.aws.amazon.com/general/latest/gr/ecr.html#ecr-docker-endpoints
REGION=<aws-region>
ACCOUNT_ID=<aws-account-id>
IMAGE_NAME=conda-env-kernel

# Create ECR Repository. Ignore if it exists. For simplcity, all examples in the repo
# use same ECR repo with different image tags
aws --region ${REGION} ecr create-repository --repository-name smstudio-custom

# Build and push the image
aws --region ${REGION} ecr get-login-password | docker login --username AWS --password-stdin ${ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/smstudio-custom
docker build . -t ${IMAGE_NAME} -t ${ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/smstudio-custom:${IMAGE_NAME}
docker push ${ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/smstudio-custom:${IMAGE_NAME}
```

### Using with SageMaker Studio
Create a SageMaker Image (SMI) with the image in ECR. Request parameter RoleArn value is used to get
ECR image information when and Image version is created. After creating Image, create an Image Version during which 
SageMaker stores image metadata like SHA etc. Everytime an image is updated in ECR, a new image version should be created.
See [Update Image](#updating-image-with-sageMaker-studio)

```bash
# Role in your account to be used for SageMakerImage. Modify as required.

ROLE_ARN=arn:aws:iam::${ACCOUNT_ID}:role/RoleName
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

```bash
aws --region ${REGION} sagemaker create-app-image-config --cli-input-json file://app-image-config-input.json

```

Create a Domain, providing the SageMaker Image and AppImageConfig in the Domain input. Replace the placeholders for VPC ID, Subnet IDs, and Execution Role in `create-domain-input.json`

```bash
aws --region ${REGION} sagemaker create-domain --cli-input-json file://create-domain-input.json
```

If you have an existing Domain, you can also use the `update-domain`

```bash
aws --region ${REGION} sagemaker update-domain --cli-input-json file://update-domain-input.json
```

Create a User Profile, and start a Notebook using the SageMaker Studio launcher.

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
