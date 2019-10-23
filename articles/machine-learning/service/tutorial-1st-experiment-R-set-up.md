---
title: "Tutorial: Your first ML experiment with R: Setup"
titleSuffix: Azure Machine Learning
description: In this tutorial series, you complete the end-to-end steps to get started with the Azure Machine Learning R SDK.  Part one covers creating a cloud notebook server environment as well as creating a workspace to manage your experiments and machine learning models.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
author: sdgilley
ms.author: sgilley
ms.date: 10/14/2019
---

# Tutorial: Get started with Azure Machine Learning and its R SDK
[!INCLUDE [applies-to-skus](../../../includes/aml-applies-to-basic-enterprise-sku.md)]

In this tutorial, you complete the end-to-end steps to get started with the Azure Machine Learning SDK for R. This tutorial is **part one of a two-part tutorial series**, and covers R environment setup and configuration, as well as creating a workspace to manage your experiments and machine learning models. [**Part two**](tutorial-1st-experiment-sdk-train.md) builds on this to train multiple machine learning models and introduce the model management process using both the Azure Machine Learning studio and the SDK.

In this tutorial, you:

> [!div class="checklist"]
> * Create an [Azure Machine Learning Workspace](concept-workspace.md) to use in the next tutorial.
> * Create a cloud-based compute instance, which has the Azure Machine Learning SDK for R installed and pre-configured.
> * Launch RStudio on the compute instance.

If you don’t have an Azure subscription, create a free account before you begin. Try the [free or paid version of Azure Machine Learning](https://aka.ms/AMLFree) today.

## Create a workspace

An Azure Machine Learning workspace is a foundational resource in the cloud that you use to experiment, train, and deploy machine learning models. It ties your Azure subscription and resource group to an easily consumed object in the SDK. If you already have an Azure Machine Learning workspace, [skip to the next section](#clone). Otherwise, create one now.

[!INCLUDE [aml-create-portal](../../../includes/aml-create-in-portal.md)]

## <a name="clone"></a>Clone the vignettes folder

This example uses the cloud notebook server in your workspace for an install-free and pre-configured experience. Use [your own environment](https://azure.github.io/azureml-sdk-for-r/articles/installation.html) if you prefer to have control over your environment, packages and dependencies.

You complete the following experiment set-up and run steps in Azure Machine Learning studio, a consolidated interface that includes machine learning tools to perform data science scenarios for data science practitioners of all skill levels.

1. Sign in to [Azure Machine Learning studio](https://ml.azure.com/).

1. Select your subscription and the workspace you created.

1. Select **Notebooks and Files** on the left.

1. Select the **"..."** at the right of the **vignettes** folder and then select **Clone**.

    ![Clone folder](media/tutorial-1st-experiment-sdk-setup/clone-tutorials.png)

1. There is a folder displayed for each user who accesses the workspace.  Select your folder to clone the **vignettes**  folder there.

## <a name="compute-instance"></a>Create a compute instance

From your workspace, create a cloud resource to get started using R Studio. This resource is a cloud-based Linux virtual machine pre-configured with everything you need to run Azure Machine Learning with R.

1. On your workspace page in the Azure Machine Learning studio, select **Compute** on the left.

1. Select **+ Add Compute** to create a new Azure Machine Learning compute instance.

1. Provide a name for your compute instance. 
   + The name must be between 2 to 16 characters. Valid characters are letters, digits, and the - character.  
   + The name must also be unique across your Azure subscription.

1. Then select **Create**. It can take a moment to set up your machine.

1. Wait until the status changes to **Running**.
   After your VM is running, use the **Compute** section to launch the RStudio web interface.

1. Select **RStudio** in the **URI** column for your VM.

   The link starts the RStudio server and opens RStudio in a new browser tab.  


## Next steps

In this tutorial, you completed these tasks:

* Created an Azure Machine Learning workspace.
* Created and configured a compute instance in your workspace.
* Started RStudio on the compute instance.

In part two of the tutorial, you run the code in `tutorial-1st-experiment-sdk-train.Rmd` to train a machine learning model. 

> [!div class="nextstepaction"]
> [Tutorial: Train your first model](tutorial-1st-experiment-r-train-model.md)

> [!IMPORTANT]
> If you do not plan on following part 2 of this tutorial or any other tutorials, you should [stop the compute instance](tutorial-1st-experiment-sdk-train.md#clean-up-resources) when you are not using it to reduce cost.
