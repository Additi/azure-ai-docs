---
title: "Tutorial: Your first ML experiment: Setup"
titleSuffix: Azure Machine Learning
description: In this tutorial series, you complete the end-to-end steps to get started with the Azure Machine Learning R SDK running in Jupyter notebooks.  Part one covers creating a cloud notebook server environment as well as creating a workspace to manage your experiments and machine learning models.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
author: j-martens
ms.author: jmartens
ms.date: 09/20/2019
---

# Tutorial: Get started with Azure Machine Learning and its R SDK

In this tutorial, you complete the end-to-end steps to get started with the Azure Machine Learning Python SDK running in Jupyter notebooks. This tutorial is **part one of a two-part tutorial series**, and covers Python environment setup and configuration, as well as creating a workspace to manage your experiments and machine learning models. [**Part two**](tutorial-1st-experiment-sdk-train.md) builds on this to train multiple machine learning models and introduce the model management process using both the Azure Machine Learning studio and the SDK.

In this tutorial, you:

> [!div class="checklist"]
> * Create an [Azure Machine Learning Workspace](concept-workspace.md) to use in the next tutorial.
> * Create a cloud-based Jupyter notebook VM with Azure Machine Learning Python SDK installed and pre-configured.

If you don’t have an Azure subscription, create a free account before you begin. Try the [free or paid version of Azure Machine Learning](https://aka.ms/AMLFree) today.

## Create a workspace

An Azure Machine Learning workspace is a foundational resource in the cloud that you use to experiment, train, and deploy machine learning models. It ties your Azure subscription and resource group to an easily consumed object in the SDK. If you already have an Azure Machine Learning workspace, skip to the [next section](#azure). Otherwise, create one now.

[!INCLUDE [aml-create-portal](../../../includes/aml-create-in-portal.md)]

## <a name="azure"></a>Create a cloud notebook server

This example uses the cloud notebook server in your workspace for an install-free and pre-configured experience. Use [your own environment](how-to-configure-environment.md#local) if you prefer to have control over your environment, packages, and dependencies.

From your workspace, you create a cloud resource to get started using Jupyter notebooks. This resource is a cloud-based Linux virtual machine pre-configured with everything you need to run Azure Machine Learning.

1. Open your workspace in the [Azure Machine Learning studio](https://ml.azure.com/).  

1. On your workspace page in the Azure Machine Learning studio, select **Notebook VMs** on the left.

1. Select **+New** to create a notebook VM.

     ![Select New VM](./media/tutorial-1st-experiment-sdk-setup/add-workstation.png)

1. Provide a name for your VM. 
   + Your Notebook VM name must be between 2 to 16 characters. Valid characters are letters, digits, and the - character.  
   + The name must also be unique across your Azure subscription.

1. Then select **Create**. It can take a moment to set up your VM.

1. Wait until the status changes to **Running**.
   After your VM is running, use the **Notebook VMs** section to launch the Jupyter web interface.

1. Select **Jupyter** in the **URI** column for your VM.

    ![Start the Jupyter notebook server](./media/tutorial-1st-experiment-sdk-setup/start-server.png)

   The link starts your notebook server and opens the Jupyter notebook webpage in a new browser tab.  This link will only work for the person who creates the VM. Each user of the workspace must create their own VM.


## Next steps

In this tutorial, you completed these tasks:

* Created an Azure Machine Learning workspace.
* Created and configured a cloud notebook server in your workspace.

In part two of the tutorial, you run the code in `tutorial-1st-experiment-sdk-train.ipynb` to train a machine learning model. 

> [!div class="nextstepaction"]
> [Tutorial: Train your first model](tutorial-1st-experiment-r-train-model.md)

> [!IMPORTANT]
> If you do not plan on following part 2 of this tutorial or any other tutorials, you should [stop the cloud notebook server VM](tutorial-1st-experiment-sdk-train.md#clean-up-resources) when you are not using it to reduce cost.


