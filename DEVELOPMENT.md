## Overview

This document describes the steps to build, test, and debug custom images for KernelGateway Apps in SageMaker Studio.

The */examples* directory has end-to-end working examples that can be used as a starting point. The contents of this document are meant to supplement the provided examples with instructions to test and debug locally before using the image in SageMaker Studio.

## Local testing

Run the image locally to verify that the kernels in the image are visible to a Kernel Gateway.

```
IMAGE_NAME=tensorflow/tensorflow:2.3.0
docker run -it "$IMAGE_NAME" bash
```

Within the running container, attempt to list the available kernelspecs.

```
jupyter-kernelspec list
```

Verify you can see an available kernel and keep note of the kernel name, e.g., "python3". If no kernel is present or `jupyter-kernelspec` isn't present, you may have to install a Jupyter kernel. An example for the IPython kernel is provided below.

```
pip install ipykernel
python -m ipykernel install --sys-prefix
```

Run the container with a KernelGateway to validate that the kernels are visible from the REST endpoint exposed to the host.

```
docker run -it -p 8888:8888 "$IMAGE_NAME" bash -c 'pip install jupyter_kernel_gateway  && jupyter-kernelgateway --ip 0.0.0.0 --debug --port 8888'
```

Verify the Kernel Gateway is started successfully (e.g., *[KernelGatewayApp] Jupyter Kernel Gateway at http://0.0.0.0:8888* in the Docker logs) and validate that you can list the kernelspecs in the the running container

```
curl http://0.0.0.0:8888/api/kernelspecs
```

## Registering the custom image

Once you have built the Docker image, run it locally to extract the kernelspecs and the user information (UID/GID)

Within the running container, get the UID and GID of the default user of the image

```
id -u
id -g
```

Within the running container, install `ipykernel` and list the available kernelspec

```
jupyter-kernelspec list
```

Use the UID/GID and kernelspec information to create an `AppImageConfig` using the `create-app-image-config` API.

For complete examples, including creating a SageMaker Image and associating the Image and its AppImageConfig with a SageMaker Studio Domain, see the READMEs of the sample images in the */examples* directory.

Once the images is registered with SageMaker Studio, look out for any error messages while creating a Domain or an App.

## Testing in SageMaker Studio

During the Domain creation process, look for any errors related to misconfiguration the `Image` and `AppImageConfig` entities if the Domain goes into a *FAILED* status by calling `DescribeDomain`. 

During the App creation process, look for any errors by calling `DescribeApp` if the App goes into a *FAILED* status. 

### CloudWatch Logs

SageMaker Studio also publishes logs into Cloudwatch Logs in your account. 

Look at the logs in the Log Group `/aws/sagemaker/studio` and the Log Stream `$domainID/$userProfileName/KernelGateway/$appName`. These logs will contain any errors that occur during the startup of the KernelGateway App or the kernel process within the KernelGateway App. 

## Common Issues

### Missing ECR permissions

Ensure your SageMaker Studio execution role has permissions to pull the image from ECR. See the [ECR documentation](https://docs.aws.amazon.com/AmazonECR/latest/userguide/security_iam_id-based-policy-examples.html) for IAM permissions required to pull an image.

### Files With Unsupported UIDs/GIDs

SageMaker only supports UIDs/GIDs less than 65535. You should ensure that your container does not have any layers containing a file with a GID/UID higher than 65535 - https://docs.aws.amazon.com/sagemaker/latest/APIReference/API_FileSystemConfig.html.

You can check if you have any files in your Docker image that exceed 65535 by adding the following RUN commands after each RUN command in your Dockerfile. We need to do this after each RUN command instead of once at the end since if files are created in one layer and deleted in another layer, it won't appear in the final Docker image, but will still cause issues when using SageMaker.
```
RUN $YOUR_RUN_COMMAND_HERE
RUN echo $(sudo find / -gid +65535 -ls 2>/dev/null)
RUN echo $(sudo find / -uid +65535 -ls 2>/dev/null)
```
or run the following commands in a terminal inside each intermediate container
```
$ echo $(sudo find / -gid +65535 -ls 2>/dev/null)
$ echo $(sudo find / -uid +65535 -ls 2>/dev/null)
```
If you find any output from the above commands, you have 2 options:

1. For the step in your Dockerfile that produces output from the previous commands, append the following to that step in your Dockerfile as shown:
  ```
  RUN YOUR_COMMANDS_HERE && echo $(sudo find / -uid +65535 -ls -delete | grep ".") && echo $(sudo find / -gid +65535 -ls -delete | grep ".")
  ```

2. Use a tool like [docker-squash](https://github.com/jwilder/docker-squash) to remove any unecessary intermediate layers from your Docker build. Using this option, you can simply run the following at the end of your Dockerfile.

  ```
  RUN echo $(sudo find / -gid +65535 -ls -delete | grep ".") && echo $(sudo find / -uuid +65535 -ls -delete | grep ".")
  ```

### UID/GID mismatch between AppImageConfig and Dockerfile

SageMaker will throw an error if the UID/GID associated with the default user in the container image does not match what is provided in the `AppImageConfig` via the CLI or the Admin console. This error message will be similar to *SageMaker is unable to launch the app using the image [...]. Ensure that the UID/GID provided in the AppImageConfig matches the default UID/GID defined in the image.*


To fix this mistmatch, run the container image locally.
```
IMAGE_NAME=<your-image-name>
docker run -it "$IMAGE_NAME" bash
```

Then, get the UID/GID of the default user in the image. Ensure these are the values that are provided in `AppImageConfig` 
```
# UID
id -u
# GID
id -g
```

NOTE: As of 2021-11-03, SageMaker only supports 1000/100 or 0/0 as the possible UID/GID values. Ensure that your container image has one of these possible values. For an example, see [this sample image](https://github.com/aws-samples/sagemaker-studio-custom-image-samples/blob/main/examples/r-image/Dockerfile#L19-L33).


