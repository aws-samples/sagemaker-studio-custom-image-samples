## JavaScript / TypeScript Image with TensorFlow.js

### Overview

> NOTE: This Dockerfile installs dependencies that may be licensed under copyleft licenses such as GPLv3. You should review the license terms and make sure they are acceptable for your use case before proceeding and downloading this image.

A SageMaker Studio-compatible notebook kernel image for JavaScript or TypeScript, based on [tslab](https://www.npmjs.com/package/tslab).

This example:

- Derives from [nvidia/cuda](https://hub.docker.com/r/nvidia/cuda) images (as some of the [AWS Deep Learning Containers](https://github.com/aws/deep-learning-containers) do), for GPU driver support
- Includes [TensorFlow.js](https://www.tensorflow.org/js) and the [AWS SDK for JavaScript](https://aws.amazon.com/sdk-for-javascript/)
- Packages some additional Python-oriented AWS and SageMaker utilities including the [AWS CLI](https://aws.amazon.com/cli/), and [SageMaker SDK for Python](https://sagemaker.readthedocs.io/en/stable/)


### Building the image

Build the Docker image and push to Amazon ECR.

> ⏰ **Note:** This image can take several minutes to build Python components from source, and typically reaches several GB in size.

```bash
# Modify these as required. The Docker registry endpoint can be tuned based on your current region from https://docs.aws.amazon.com/general/latest/gr/ecr.html#ecr-docker-endpoints
REGION=<aws-region>
ACCOUNT_ID=<account-id>
IMAGE_NAME=custom-jsts

aws --region ${REGION} ecr get-login-password | docker login --username AWS --password-stdin ${ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/smstudio-custom
docker build . -t ${IMAGE_NAME} -t ${ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/smstudio-custom:${IMAGE_NAME}
```

```bash
docker push ${ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/smstudio-custom:${IMAGE_NAME}
```


### Using with SageMaker Studio

Once the image is pushed to Amazon ECR, you can attach it to your SageMaker Studio domain via either the console UI or the CLI. See the [SageMaker Developer Guide](https://docs.aws.amazon.com/sagemaker/latest/dg/studio-byoi-create.html) for more details.

The tslab installation in this image exports **two Jupyter kernels** you can use:

- `tslab`: For working in [TypeScript](https://www.typescriptlang.org/)
- `jslab`: For standard JavaScript

Both kernel options see the same globally-pre-installed npm packages, and use the same SageMaker Studio filesystem configuration (user, group, folder) as shown in [app-image-config-input.json](app-image-config-input.json).

At the time of writing SageMaker Studio does not (yet?) fully support multi-kernel images, so you'll need to create **two** ["SageMaker Images"](https://console.aws.amazon.com/sagemaker/home?#/images) (referencing the same [ECR](https://console.aws.amazon.com/ecr/repositories/private/) URI)) - if you want to expose both JS and TS options to users in your domain.

For example, to create the SageMaker Image via AWS CLI:

```bash
# IAM Role in your account to be used for the SageMaker Image setup process:
ROLE_ARN=<role-arn>

# May want to use an alternative SM_IMAGE_NAME if registering two SageMaker images to same ECR container:
SM_IMAGE_NAME=${IMAGE_NAME}

aws --region ${REGION} sagemaker create-image \
    --image-name ${SM_IMAGE_NAME} \
    --role-arn ${ROLE_ARN}

aws --region ${REGION} sagemaker create-image-version \
    --image-name ${IMAGE_NAME} \
    --base-image "${ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/smstudio-custom:${IMAGE_NAME}"

# Verify the image-version is created successfully.
# Do NOT proceed if image-version is in CREATE_FAILED state or in any other state apart from CREATED.
aws --region ${REGION} sagemaker describe-image-version --image-name ${IMAGE_NAME}
```

...and then configure the container runtime settings in SageMaker:

```bash
# TODO: Edit the JSON to point to whichever of 'tslab' or 'jslab' kernel you intend to use
aws --region ${REGION} sagemaker create-app-image-config --cli-input-json file://app-image-config-input.json
```

The final step is to attach your SageMaker Custom Image(s) to your SageMaker Studio domain. You can do this from the [AWS Console for SageMaker Studio](https://console.aws.amazon.com/sagemaker/home?#/studio/d-doedz9htjn38), or using the `aws sagemaker update-domain` [command](https://docs.aws.amazon.com/cli/latest/reference/sagemaker/update-domain.html) in the AWS CLI.


### Further information

#### Magics and package installations

SageMaker Python users may be used to installing packages inline on kernels using `!pip install ...` commands.

[Magics](https://ipython.readthedocs.io/en/stable/interactive/magics.html) (like `%%bash` or `!` for running shell scripts) are generally specific to the IPython (Python) kernel and may not be available in other implementations (like `tslab` used here).

See [this discussion](https://github.com/yunabe/tslab/issues/35) for alternatives and roadmap for running shell commands like `npm install` in tslab.

You could, for example, run a cell like:

```js
const { execSync } = require("child_process");
execSync("npm install lodash", { encoding: "utf-8" });
```

#### A note on GPU-accelerated notebooks

This kernel includes CUDA libraries in case you want to experiment with GPU-accelerated instance types like `ml.g4dn.xlarge` in notebook.

**However**, remember that a general best practice is to **package your high-resource code as SageMaker Jobs** (such as Processing, Training, or Batch Transform jobs) and keep your notebook environment resources modest (such as an `ml.t3.medium`). Working with SageMaker Jobs early in the build process can help:

- Optimize infrastructure costs (since these jobs spin up and release their infrastructure on-demand)
- Improve experiment tracking (since the SageMaker APIs automatically store history of training job inputs, parameters, metrics, and so on)
- Accelerate the path to production (by e.g. attempting to train models in container environments that are already set up for inference deployment)
