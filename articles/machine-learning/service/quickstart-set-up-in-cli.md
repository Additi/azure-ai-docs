﻿---
title: Installation Quickstart with Azure Machine Learning CLI | Microsoft Docs
description: In this quickstart, you will learn how to get started with Azure Machine Learning Services using the Azure Machine Learning CLI extension.
ms.service: machine-learning
ms.component: core
ms.topic: quickstart
ms.reviewer: jmartens
author: rastala
ms.author: roastala
ms.date: 7/27/2018
---

# Quickstart: Create a workspace and project with Azure Machine Learning's CLI extension

In this quickstart, you'll use a machine learning CLI extension to get started with [Azure Machine Learning Services](overview-what-is-azure-ml.md).

Using the CLI, you'll learn how to:
1. Create a workspace, which is the top-level resource for this service.
1. Attach a project containing your machine learning scripts.
1. Run a script @@TO DO WHAT and view the output.

This CLI was built on top of the [Python-based SDK for Azure Machine Learning services](reference-azure-machine-learning-sdk.md).

## Prerequisites

Make sure you have the following prerequisites before starting the quickstart steps:

+ An Azure subscription. If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.
+ Adequate permissions to create Azure assets such as resource groups
+ [Python 3.5 or higher](https://www.python.org/) installed
+ [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) installed

## Install the CLI extension

On your computer, open a command-line editor and install [the machine learning extension to Azure CLI](reference-azure-machine-learning-cli.md).  The installation can take several minutes to complete.

```azurecli
az extension add azureml-sdk
```

## Create a resource group

A resource group is a container that holds related resources for an Azure solution. Using Azure CLI, sign into Azure, specify the subscription, and create a resource group.

1. In a command-line window, sign in with the Azure CLI command, [`az login`](https://docs.microsoft.com/cli/azure/reference-index?view=azure-cli-latest#az-login). Follow the prompts for interactive login:
    
    ```azurecli
    az login
    ```

1. List the available Azure subscriptions, and specify the one you want to use: 
   ```azurecli
   az account list --output table
   az account set --subscription <your-subscription-id>
   az account show
   ```
   where \<your-subscription-id\> is ID value for the subscription you want to use that was output by az account list. Do not include the brackets.

1. Create a resource group to hold your workspace.  
   In this quickstart:
   + The name of the resource group is `myrg`.
   + The region is `eastus2`. You can use any [available region](https://azure.microsoft.com/global-infrastructure/services/) close to your data.  

    ```azurecli
    az group create --name myrg --location eastus2
    ```

## Create a workspace and attach a project

1. In the command-line window, create an Azure Machine Learning Workspace under the resource group. 

   An **Azure Machine Learning Workspace** is the top-level resource that can be used by one or more users to store their compute resources, models, deployments, and run histories. For your convenience, the following resources are added automatically to your workspace when regionally available: [Azure Container Registry](https://azure.microsoft.com/en-us/services/container-registry/), [Azure storage](https://azure.microsoft.com/en-us/services/storage/), [Azure Application Insights](https://azure.microsoft.com/en-us/services/application-insights/), and [Azure Key Vault](https://azure.microsoft.com/en-us/services/key-vault/).

   In this quickstart:
   + The workspace name is `myws`.
   + The resource group name is `myrg`

   ```azurecli
   az ml workspace create --name myws --group myrg
   ```

1. In the command-line window, create a folder on your local machine for your Azure Machine Learning project. 
   
   A **project** is a local folder that contains the scripts needed to solve your machine learning problem and the configuration files  required to attach the project to your workspace in Azure Cloud.

   ```
   mkdir myproject
   cd myproject
   ```

1. Attach the folder as a project to the workspace. The `--history` argument specifies a name for the run history file that captures the metrics for each run.  

   ```azurecli
   az ml project attach --history myhistory -w myws
   ```

## Run scripts and view output

1. In your local project directory, create a script and name it `helloworld.py`. 

1. Copy the following code into that script:
   ```python

   # Log metric values
   run.log("A single value",1.23)
   run.log_list("A list of values",[1,2,3,4,5])
 
   # Save an output artifact, such as model or data file  
   with open("myOutputFile.txt","w") as f:
   f.write("My results")
   run.upload_file(name="results",path_or_stream="myOutputFile.txt")
 
   run.complete()
   ```

1. Run the script on your local computer.

   ```azurecli
   az ml run submit -c local helloworld.py
   ```

   This command runs the code and outputs a web link to your console. Copy-paste the link into your web browser.

1. In a web browser, visit the URL. A web portal appears with the results of the run. You can inspect the results of that run or previous runs, if they exist.

## Clean up resources 

[!INCLUDE [aml-delete-resource-group](../../../includes/aml-delete-resource-group.md)]

## Next steps
You have now created the necessary resources to start experimenting and deploying models. You also created a project, ran a script, and explored the run history of the script.

For an in-depth workflow experience, follow the Azure Machine Learning tutorial on building, training, and deploying a model.

> [!div class="nextstepaction"]
> [Tutorial: Build, train, and deploy](tutorial-build-train-deploy-with-azure-machine-learning.md)