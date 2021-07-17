# Spark Image

## Overview

This image is Spark kernel as a Custom Image in SageMaker Studio. This custom image can be used to do interactive spark development in Python. This allow to read and write data from Amazon S3 buckets.

The image is based on Spark version 2.4.0, Hadoop version 2.7 and openjdk 8. This image also have latest version of sagemaker_pyspark (1.4.2), to have aws-hadoop and other dependents jar, to work with AWS services like Amazon simple storage service (s3).

Example notebook (pyspark-kernel-file-read) explains on how to use different credential provider to make calls to S3.

### Building the image

Build the Docker image and push to Amazon ECR.

```bash
# Modify these as required. The Docker registry endpoint can be tuned based on your current region from https://docs.aws.amazon.com/general/latest/gr/ecr.html#ecr-docker-endpoints
REGION=<aws-region>
ACCOUNT_ID=<account-id>


# Build the image
IMAGE_NAME=spark-kernel
aws --region ${REGION} ecr get-login-password | docker login --username AWS --password-stdin ${ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/smstudio-custom
docker build . -t ${IMAGE_NAME} -t ${ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/smstudio-custom:${IMAGE_NAME}
```

```bash
docker push ${ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/smstudio-custom:${IMAGE_NAME}
```

### Using with SageMaker Studio

Create a SageMaker Image with the image in ECR.

```bash
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

```bash
aws --region ${REGION} sagemaker create-app-image-config --cli-input-json file://app-image-config-input.json
```

Create a Domain, providing the SageMaker Image and AppImageConfig in the Domain creation. Replace the placeholders for VPC ID, Subnet IDs, and Execution Role in `create-domain-input.json`

```bash
aws --region ${REGION} sagemaker create-domain --cli-input-json file://create-domain-input.json
```

If you have an existing Domain, you can also use the `update-domain`

```bash
aws --region ${REGION} sagemaker update-domain --cli-input-json file://update-domain-input.json
```
