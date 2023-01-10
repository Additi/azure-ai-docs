---
title: "Quickstart: Create workspace resources"
titleSuffix: Azure Machine Learning
description: Create an Azure Machine Learning workspace and cloud resources that can be used to train machine learning models.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: quickstart
author: sdgilley
ms.author: sgilley
ms.reviewer: sgilley
ms.date: 08/26/2022
adobe-target: true
ms.custom: FY21Q4-aml-seo-hack, contperf-fy21q4, mode-other, ignite-2022
#Customer intent: As a data scientist, I want to create a workspace so that I can start to use Azure Machine Learning.
---

# Quickstart: Create workspace resources you need to get started with Azure Machine Learning

In this quickstart, you'll create:

* A *workspace*.  To use Azure Machine Learning, you'll first need a workspace.  The workspace is the central place to view and manage all the artifacts and resources you create. 
* A *compute instance*.  A compute instance is a pre-configured cloud-computing resource that you can use to train, automate, manage, and track machine learning models. A compute instance is the quickest way to start using the Azure Machine Learning SDKs and CLIs. You'll use it to run Jupyter notebooks and Python scripts in the rest of the tutorials.

## Prerequisites

- An Azure account with an active subscription. [Create an account for free](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## Create the workspace

The workspace is the top-level resource for your machine learning activities, providing a centralized place to view and manage the artifacts you create when you use Azure Machine Learning.

If you  already have a workspace, skip this section and continue to [Create a compute instance](#create-a-compute-instance).

If you don't yet have a workspace, create one now: 

1. Sign in to [Azure Machine Learning studio](https://ml.azure.com)
1. Select **Create workspace**
1. Provide the following information to configure your new workspace:

   Field|Description 
   ---|---
   Workspace name |Enter a unique name that identifies your workspace. Names must be unique across the resource group. Use a name that's easy to recall and to differentiate from workspaces created by others. The workspace name is case-insensitive.
   Subscription |Select the Azure subscription that you want to use.
   Resource group | Use an existing resource group in your subscription or enter a name to create a new resource group. A resource group holds related resources for an Azure solution. You need *contributor* or *owner* role to use an existing resource group.  For more information about access, see [Manage access to an Azure Machine Learning workspace](how-to-assign-roles.md).
   Region | Select the Azure region closest to your users and the data resources to create your workspace.
1. Select **Create** to create the workspace

> [!NOTE]
> This creates a workspace along with all required resources. If you would like to reuse resources, such as Storage Account, Azure Container Registry, Azure KeyVault, or Application Insights, use the [Azure portal](https://ms.portal.azure.com/#create/Microsoft.MachineLearningServices) instead.

## Create a compute instance

You'll use the *compute instance* to run Jupyter notebooks and Python scripts in the rest of the tutorials.

Create a compute instance now.  

1. On the left navigation, select **Notebooks**.
1. Select **Create compute** in the middle of the page.  (You'll only see this option if you don't yet have a compute instance in your workspace.)
1. Supply a name. Keep all the defaults on the first page.
1. Select **Next** to see **Advanced Settings**.
1. Select **Enable idle shutdown** so that the machine will shut down after a period of inactivity.  
1. Keep the default values for the rest of the page.
1. Select **Create**.

In about two minutes, you'll see the **State** of the compute instance change from **Creating** to **Running**. It's now ready to go.

## Quick tour of the studio

The studio is your web portal for Azure Machine Learning. This portal combines no-code and code-first experiences for an inclusive data science platform.

Review the parts of the studio on the left-hand navigation bar:

* The **Author** section of the studio contains multiple ways to get started in creating machine learning models.  

    * **Notebooks** section allows you to create Jupyter Notebooks, copy sample notebooks, and run notebooks and Python scripts.
    * **Automated ML** steps you through creating a machine learning model without writing code.
    * **Designer** gives you a drag-and-drop way to build models using prebuilt components.

* The **Assets** section of the studio helps you keep track of the assets you create as you run your jobs.  If you have a new workspace, there's nothing in any of these sections yet.

* The **Manage** section of the studio lets you create and manage compute and external services you link to your workspace. It's also where you can create and manage a **Data labeling** project.

:::image type="content" source="media/quickstart-create-resources/overview.png" alt-text="Screenshot of Azure ML studio." lightbox="media/quickstart-create-resources/overview.png":::


## Clean up resources

If you plan to continue now to other tutorials, skip to [Next steps](#next-steps).

### Stop compute instance

If you're not going to use it now, stop the compute instance:

1. In the studio, on the left, select **Compute**.
1. In the top tabs, select **Compute instances**
1. Select the compute instance in the list.
1. On the top toolbar, select **Stop**.

### Delete all resources

[!INCLUDE [aml-delete-resource-group](../../includes/aml-delete-resource-group.md)]

## Next steps

You now have an Azure Machine Learning workspace, which contains a compute instance to use for your development environment.

Continue on to learn how to use the compute instance to run notebooks and scripts in the Azure Machine Learning cloud.  

> [!div class="nextstepaction"]
> [Quickstart: Set up your Azure Machine Learning cloud workstation](quickstart-run-notebooks.md)

Use your compute instance with the following tutorials to train and deploy and deploy a model.

|Tutorial  |Description  |
|---------|---------|
| [Azure Machine Learning in a day](tutorial-azure-ml-in-a-day.md)     |  Basic end-to-end train and deploy a model      |
| [Access and explore your data]()     |  Store large data in the cloud and retrieve it from notebooks and scripts |
| [Train a model]()   |    Dive in to the details of training a model     |
| [Deploy a model]()  |   Dive in to the details of deploying a model      |