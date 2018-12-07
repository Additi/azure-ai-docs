---
title: What's happening to Azure Machine Learning Workbench? | Microsoft Docs
description: Learn about what's happening to the Workbench application, what changed in Azure Machine Learning, and what the support timeline is.
services: machine-learning
ms.service: machine-learning
ms.component: core
ms.topic: overview

ms.reviewer: jmartens
author: j-martens
ms.author: jmartens
ms.date: 12/04/2018
---
# What's happening to Azure Machine Learning Workbench?

The Machine Learning Workbench application and some other early features were deprecated and replaced in the September 2018 release to make way for an improved [architecture](concept-azure-machine-learning-architecture.md). To improve your experience, the release contains many significant updates prompted by customer feedback. The core functionality from experiment runs to model deployment hasn't changed. But now, you can use the robust <a href="https://aka.ms/aml-sdk" target="_blank">SDK</a> and [CLI](reference-azure-machine-learning-cli.md) to accomplish your machine learning tasks and pipelines.  

In this article, you learn about what changed and how it affects your preexisting work with the Azure Machine Learning Workbench and its APIs.

## What changed?

The latest release of the Azure Machine Learning service includes the following features:
+ A [simplified Azure resources model](concept-azure-machine-learning-architecture.md).
+ A [new portal UI](how-to-track-experiments.md) to manage your experiments and compute targets.
+ A new, more comprehensive Python <a href="https://aka.ms/aml-sdk" target="_blank">SDK</a>.
+ The new expanded [Azure CLI extension](reference-azure-machine-learning-cli.md) for machine learning.

The [architecture](concept-azure-machine-learning-architecture.md) was redesigned for ease of use. Instead of multiple Azure resources and accounts, you only need an [Azure Machine Learning service workspace](concept-azure-machine-learning-architecture.md#workspace). You can create workspaces quickly in the [Azure portal](quickstart-get-started.md). A workspace can be used by multiple users to store training and deployment compute targets, model experiments, Docker images, deployed models, and so on.

Although there are new improved CLI and SDK clients in the current release, the desktop Machine Learning Workbench application itself is deprecated. Now you can monitor your experiments in the [workspace dashboard in the Azure portal](how-to-track-experiments.md#view-the-experiment-in-the-azure-portal). Use the dashboard to get your experiment history, manage the compute targets attached to your workspace, manage your models and Docker images, and even deploy web services.

## How do I migrate?

Most of the artifacts created in the earlier version of the Azure Machine Learning service are stored in your own local or cloud storage. These artifacts won't ever disappear. To migrate, you need to register the artifacts again with the updated Azure Machine Learning service. Learn what you can migrate and how in this [migration article](how-to-migrate.md).

<a name="timeline"></a>

## Support timeline

You can continue to use your Machine Learning Experimentation and Model Management accounts as well as the Machine Learning Workbench application for a while longer after September 2018. Support for the following resources will be removed progressively in the three to four months after that release. You can still find the documentation for the old features in the [Resources section](../desktop-workbench/tutorial-classifying-iris-part-1.md) at the bottom of the table of contents.

|Retirement&nbsp;phase|Support details for earlier features|
|:---:|----------------|
|December 4, 2018|The ability to create Azure Machine Learning Experimentation and Model Management accounts in the Azure portal and from the CLI has ended. The ability to create Machine Learning compute environments from the CLI has also ended. If you have an existing account, the CLI and the desktop Machine Learning Workbench continue to work in this phase.|
|January 9, 2019|Support for everything else, including the remaining APIs and the desktop Machine Learning Workbench ends on this date.|

[Start migrating](how-to-migrate.md) today. All the latest capabilities are available by using the new <a href="https://aka.ms/aml-sdk" target="_blank">SDK</a>, the [CLI](reference-azure-machine-learning-cli.md), and the [portal](quickstart-get-started.md).

## What about run histories?

Run histories will remain accessible for a while. When you're ready to move to the updated version of the Azure Machine Learning service, you can export these run histories if you want to keep a copy.

Run histories are called **experiments** in the current release. You can collect your model's experiments and explore them by using the SDK, the CLI, or the Azure portal.

The portal's workspace dashboard is supported on Edge, Chrome, and Firefox browsers only:

[ ![Online portal](./media/overview-what-happened-to-workbench/image001.png)]
(./media/overview-what-happened-to-workbench/image001.png#lightbox)


## Can I still prep data?

Your preexisting data preparation files aren't portable to the latest release because we don't have Machine Learning Workbench anymore. However, you can still prepare your data for modeling.  

With smaller datasets, you can use the <a href="https://aka.ms/aml-sdk" target="_blank">Azure Machine Learning data prep SDK</a> to quickly prepare your data prior to modeling. 

You can use this same <a href="https://aka.ms/aml-sdk" target="_blank">SDK</a> for larger datasets. Or use Azure Databricks to prepare big datasets. 

## Will projects persist?

You won't lose any code or work. In the older version, projects are cloud entities with a local directory. In the latest version, you attach local directories to the Azure Machine Learning Workspace service by using a local config file. See a [diagram of the latest architecture](concept-azure-machine-learning-architecture.md).

Much of the project content was already on your local machine. So you just need to create a config file in that directory and reference it in your code to connect to your workspace. Learn how to [migrate your existing projects](how-to-migrate.md#projects).

Learn how to [get started in Python with the main SDK](quickstart-get-started.md).

## What about my registered models and images?
 
The models that you registered in your old model registry must be migrated to your new workspace if you want to continue to use them. To migrate your models, [download the models and re-register them](how-to-migrate.md) in your new workspace. 

The images that you created in your old image registry must be re-created in the new workspace to continue to use them. To re-create your images, follow the [create Docker image](how-to-deploy-to-aci.md#configure-an-image) section. 

## What about deployed web services?

The models you deployed as web services by using your Machine Learning Model Management account will continue to work for as long as Azure Container Service is supported. Those web services will work even after support has ended for Machine Learning Model Management accounts. However, when support for the old CLI ends, so does your ability to manage those web services.

In the newer version, models are deployed as web services to [Azure Container Instances](how-to-deploy-to-aci.md) or [Azure Kubernetes Service](how-to-deploy-to-aks.md) (AKS) clusters. You can also [deploy to FPGAs and to Azure IoT Edge](how-to-deploy-and-where.md). Without having to change any of your scoring files, dependencies, and schemas, you can redeploy your models by using the new SDK or CLI. 

## What about the old SDK and CLI?

Yes, they'll continue to work till January. See the preceding [timeline](#timeline). We recommend that you start creating your new experiments and models with the latest SDK or CLI.

By using the new Python SDK in the latest release, you can interact with the Azure Machine Learning service in any Python environment. Learn how to install the latest <a href="https://aka.ms/aml-sdk" target="_blank">SDK</a>. You can also use the updated [Azure Machine Learning CLI extension](reference-azure-machine-learning-cli.md) with the rich set of `az ml` commands to interact with the service in any command-line environment, including Azure Cloud Shell.

## What about Azure Machine Learning for Visual Studio Code?

In this latest release, Azure Machine Learning for Visual Studio Code has been expanded and improved to work with the preceding new features.

[ ![Azure Machine Learning for Visual Studio Code](./media/overview-what-happened-to-workbench/vscode.png)]
(./media/overview-what-happened-to-workbench/vscode-big.png#lightbox)

## What about domain packages?

The domain packages for [computer vision, text analytics, and forecasting](../desktop-workbench/reference-python-package-overview.md) can't be used with the latest version of Azure Machine Learning. However, you can still build and train computer vision, text, and forecasting models with latest Azure Machine Learning Python <a href="https://aka.ms/aml-sdk" target="_blank">SDK</a>. To learn how to migrate preexisting models built by using the computer vision, text analytics, and forecasting packages, contact [AML-Packages@microsoft.com](mailto:AML-Packages@microsoft.com).

## Next steps

Learn about the [latest architecture for the Azure Machine Learning service](concept-azure-machine-learning-architecture.md). Try one of the quickstarts or tutorials:

* [What is the Azure Machine Learning service?](overview-what-is-azure-ml.md)
* [Quickstart: Create a workspace with Python](quickstart-get-started.md)
* [Tutorial: Train a model](tutorial-train-models-with-aml.md)
