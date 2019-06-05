---
title: How to deploy a model using a custom Docker image 
titleSuffix: Azure Machine Learning service
description: 'Learn how to use a custom Docker image when deploying your Azure Machine Learning service models. When deploying a trained model, a Docker image is created to host the image, web server, and other components needed to run the service. While Azure Machine Learning service provides a default image for you, you can also use your own image.'
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: jordane
author: jpe316
ms.reviewer: larryfr
ms.date: 06/05/2019
---

# Deploy a model using a custom Docker image

Learn how to use a custom Docker image when deploying trained models with the Azure Machine Learning service.

When you deploy a trained model to a web service or IoT Edge device, a Docker image is created. This image contains the model, conda environment, and assets needed to use the model. It also contains a web server to handle incoming requests when deployed as a web service, and components needed to work with Azure IoT Hub.

Azure Machine Learning service provides a default Docker image so you don't have to worry about creating on. You can also use a custom image that you create as a _base image_. A base image is used as the starting point when an image is created for a deployment. It provides the underlying operating system and components. The deployment process then adds additional components, such as your model, conda environment, and other assets, to the image before deploying it.

> [!IMPORTANT]
> When deploying a model, you cannot override core components such as the web server or IoT Edge components. The components used provide a known working environment that is tested and supported by Microsoft.

Microsoft also provides several base images that use the ONNX Runtime. These images are useful when deploying ONNX models.

## Prerequisites

* An Azure Machine Learning service workgroup
* The Azure CLI
* An Azure Container Registry or other Docker registry that is accessible on the internet. The steps in this document use an Azure Container Registry.

## Image requirements

Azure Machine Learning service supports Docker images that provide the following software:

* Ubuntu 16.04 or greater.
* Conda 4.5.# or greater.
* Python 3.5.# or 3.6.#.

For example, the following Dockerfile uses Ubuntu 16.04, the latest miniconda, and finally uses conda to install Python 3.6:

```text
FROM ubuntu:16.04
# update and install wget and bzip2
RUN apt-get update -y && yes|apt-get upgrade
RUN apt-get install -y wget bzip2
# add a user named 'ubuntu'
RUN apt-get -y install sudo
RUN adduser --disabled-password --gecos '' ubuntu
RUN adduser ubuntu sudo
RUN echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers
# switch to 'ubuntu' user and set up home directory
USER ubuntu
WORKDIR /home/ubuntu/
RUN chmod a+rwx /home/ubuntu
# Install miniconda
RUN wget https://repo.anaconda.com/miniconda/Miniconda2-latest-Linux-x86_64.sh
RUN bash Miniconda2-latest-Linux-x86_64.sh -b
RUN rm Miniconda2-latest-Linux-x86_64.sh
# Add to user path
ENV PATH /home/ubuntu/miniconda2/bin:$PATH
# Update conda packages
RUN conda update conda
# Set default python to 3.6
RUN conda install python=3.6
```

## Get container registry address

The steps in this document assume that you are using an Azure Container Registry. A container registry is created when you train or deploy a model in your Azure Machine Learning service workspace, or you can use a container registry that you have manually created.

The difference between the container registry for your workspace and a manually created one is that Azure Machine Learning service can automatically authenticate to the one for the workspace. For a separate container registry, you may need to enable admin access and provide the user name and password in your Python code.

### Use the container registry in your workspace

If you've already trained or deployed models using the Azure Machine Learning service, a container registry was created for your workspace. To find the name of this container registry, use the following steps:

1. Open a new shell or command-prompt and use the following command to authenticate to your Azure subscription:

    ```
    az login
    ```

    Follow the prompts to authenticate to the subscription.

2. Use the following command to list the container registry for the workspace. Replace `<myworkspace>` with your Azure Machine Learning service workspace name. Replace `<resourcegroup>` with the Azure resource group that contains your workspace:

    ```
    az ml workspace show -w <myworkspace> -g <resourcegroup> --query containerRegistry
    ```

    The information returned is similar to the following text:

    ```text
    /subscriptions/<subscription_id>/resourceGroups/<resource_group>/providers/Microsoft.ContainerRegistry/registries/<registry_name>
    ```

    Save the `<registry_name>` value. It will be used in later steps.

#### Use a container registry outside your workspace

TBD

## Build a custom image

1. Create a new text file named `Dockerfile`, and use the following text as the contents:

    ```text
    FROM ubuntu:16.04
    # update and install wget and bzip2
    RUN apt-get update -y && yes|apt-get upgrade
    RUN apt-get install -y wget bzip2
    # add a user named 'ubuntu'
    RUN apt-get -y install sudo
    RUN adduser --disabled-password --gecos '' ubuntu
    RUN adduser ubuntu sudo
    RUN echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers
    # switch to 'ubuntu' user and set up home directory
    USER ubuntu
    WORKDIR /home/ubuntu/
    RUN chmod a+rwx /home/ubuntu
    # Install miniconda
    RUN wget https://repo.anaconda.com/miniconda/Miniconda2-latest-Linux-x86_64.sh
    RUN bash Miniconda2-latest-Linux-x86_64.sh -b
    RUN rm Miniconda2-latest-Linux-x86_64.sh
    # Add to user path
    ENV PATH /home/ubuntu/miniconda2/bin:$PATH
    # Update conda packages
    RUN conda update conda
    # Set default python to 3.6
    RUN conda install python=3.6
    ```

2. From a shell or command-prompt, use the following to authenticate to the Azure Container Registry. Replace the `<registry_name>` with the one retrieved earlier:

    ```
    az acr login --name <registry_name>
    ```

3. To upload the Dockerfile, and build it, use the following command. Replace `<registry_name>` with the one retrieved earlier:

    ```
    az acr build --image myimage:v1 --registry <registry_name> --file Dockerfile .
    ```

    During the build process, information is streamed to back to the command line. If the build is successful, you receive a message similar to the following text:

    ```text
    Run ID: cda was successful after 2m56s
    ```

For more information on building images with an Azure Container Registry, see [Build and run a container image using Azure Container Registry Tasks](/docs.microsoft.com/azure/container-registry/container-registry-quickstart-task-cli.md)

For more information on uploading existing images to an Azure Container Registry, see [Push your first image to a private Docker container registry](/azure/container-registry/container-registry-get-started-docker-cli.md).

## Use the custom base image

### From the Azure Machine Learning SDK

To use a custom image, set the `base_image` property of the [inference configuration object](https://docs.microsoft.com/python/api/azureml-core/azureml.core.model.inferenceconfig?view=azure-ml-py) to the address of the image. The following example demonstrates how to use an image from the __container registry for your workspace__. Replace `<registry_name>` with the one retrieved earlier for your workspace:

```python
# use an image from the registry in your workspace without authentication
inference_config.base_image = "<registry_name>/myimage:v1"
```

You can use the same approach to use an image in a __public container registry__. The following exmaple uses a default image provided by Microsoft:

```python
# use an image available in public Container Registry without authentication
inference_config.base_image = "mcr.microsoft.com/azureml/o16n-sample-user-base/ubuntu-miniconda"
```

To use an image from a __private container registry__ that is not in your workspace, you must specify the address of the repository and a user name and password:

```python
# Use an image available in a private Container Registry
inference_config.base_image = "myregistry.azurecr.io/mycustomimage:1.0"
inference_config.base_image_registry.address = "myregistry.azurecr.io"
inference_config.base_image_registry.username = "username"
inference_config.base_image_registry.password = "password"
```

### Publicly available images

The following image URIs are for images provided by Microsoft, and can be used without providing a user name or password value:

* `mcr.microsoft.com/azureml/o16n-sample-user-base/ubuntu-miniconda`
* `mcr.microsoft.com/azureml/onnxruntime:v0.4.0`
* `mcr.microsoft.com/azureml/onnxruntime:v0.4.0-cuda10.0-cudnn7`
* `mcr.microsoft.com/azureml/onnxruntime:v0.4.0-tensorrt19.03`

To use these images, set the `base_image` to the URI from the list above.

> [!IMPORTANT]
> Microsoft images that use CUDA or TensorRT must be used on Microsoft Azure Services only.

> [!TIP]
>__If your model is trained on Azure Machine Learning Compute__, using __version 1.0.22 or greater__ of the Azure Machine Learning SDK, an image is created during training. The following example demonstrates how to use this image:
>
> ```python
> # Use an image built during training with SDK 1.0.22 or greater
> image_config.base_image = run.properties["AzureML.DerivedImageName"]
> ```