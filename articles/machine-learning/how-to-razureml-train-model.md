---
title: Train R models in Azure Machine Learning
titleSuffix: Azure Machine Learning
description: 'Learn how to train R models in Azure Machine Learning.'
ms.service: machine-learning
ms.date: 01/04/2023
ms.topic: how-to
author: wahalulu
ms.author: mavaisma
ms.reviewer: sgilley
ms.devlang: r
---

# How to train R models in Azure Machine Learning

[!INCLUDE [cli v2](../../includes/machine-learning-cli-v2.md)]

This article explains how to take the R script that you [adapted to run in production](how-to-razureml-modify-script-for-prod.md) and set it up to run as an R job using the AzureML CLI V2.

> [!NOTE]
> Although the title of this article refers to _training_ a model, you can actually run any kind of R script as long as it meets the requirements listed in the adapting article.

## Prerequisites

- An [Azure Machine Learning workspace](quickstart-create-resources.md).
- [A registered data asset](how-to-create-data-assets.md) that your training job will use.
- Azure [CLI and ml extension installed](how-to-configure-cli.md).  Or use a [compute instance in your workspace](quickstart-create-resources.md), which has the CLI pre-installed.
- [A compute cluster](how-to-create-attach-compute-cluster.md) to run your training job.
- [An R environment](how-to-razureml-modify-script-for-prod.md#create-an-environment) for the compute cluster to use to run the job.

## Create a folder with this structure

Create this folder structure for your project:

```
📁 r-job-azureml
├─ src
│  ├─ azureml_utils.R
│  ├─ r-source.R
├─ job.yml
```

> [!IMPORTANT]
> All source code goes in the `src` directory.

* The **r-source.R** file is the R script that you adapted to run in production
* The **azureml_utils.R** file is necessary. The source code is shown [here](how-to-razureml-modify-script-for-prod.md#source-the-azureml_utilsr-helper-script)



## Prepare the job YAML

AzureML CLI v2 has different [different YAML schemas](reference-yaml-overview.md) for different operations. You'll use the [job YAML schema](reference-yaml-job-command.md) to submit a job. This is the **job.yml** file that is a part of this project.

You'll need to gather specific pieces of information to put into the YAML:

- The name of the registered data asset you'll use as the data input (with version): `azureml:<REGISTERED-DATA-ASSET>:<VERSION>`
- The name of the environment you created (with version): `azureml:<R-ENVIRONMENT-NAME>:<VERSION>`
- The name of the compute cluster: `azureml:<COMPUTE-CLUSTER-NAME>`


> [!TIP]
> For AzureML artifacts that require versions (data assets, environments), you can use the shortcut URI `azureml:<AZUREML-ASSET>@latest` to get the latest version of that artifact if you don't need to set a specific version.


### Sample YAML schema to submit a job

Edit your **job.yml** file to contain the following.  Make sure to replace values shown `<IN-BRACKETS-AND-CAPS>` and remove the brackets.

```yml
$schema: https://azuremlschemas.azureedge.net/latest/commandJob.schema.json
# the Rscript command goes in the command key below. Here you also specify 
# which parameters are passed into the R script and can reference the input
# keys and values further below
# Modify any value shown below <IN-BRACKETS-AND-CAPS> (remove the brackets)
command: >
Rscript <NAME-OF-R-SCRIPT>.R
--data_file ${{inputs.datafile}}  
--other_input_parameter ${{inputs.other}}
code: src   # this is the code directory
inputs:
  datafile: # this is a registered data asset
    type: uri_file
    path: azureml:<REGISTERED-DATA-ASSET>@latest
  other: 1  # this is a sample parameter, which is the number 1 (as text)
environment: azureml:<R-ENVIRONMENT-NAME>@latest
compute: azureml:<COMPUTE-CLUSTER-NAME>
experiment_name: <NAME-OF-EXPERIMENT>
description: <DESCRIPTION>
```

## Submit the job

Gather other pieces of information about your AzureMl workspace to use in the job submission:

- The AzureML workspace name
- The resource group name where the workspace is
- The subscription ID where the workspace is

Find these values from [Azure Machine Learning studio](https://ml.azure.com):

1. Sign in and open your workspace.
1. In the upper right Azure Machine Learning studio toolbar, select your workspace name.
1. Copy the values from the section that opens.  

:::image type="content" source="media/find-values.png" alt-text="Screenshot: Find the values to use in your CLI command." lightbox="media/find-values.png":::

In a terminal window:

1. Change directories into the `r-job-azureml`.

    ```bash
    cd r-job-azureml
    ```

1. Open a terminal window and sign in to Azure.

    ```azurecli
    az login
    ```

    Follow the prompt to authenticate.

1. If you have multiple Azure subscriptions, set the active subscription to the one you're using for your workspace. (You can skip this step if you only have access to a single subscription.)  Replace `<SUBSCRIPTION-NAME>` with your subscription name.  Also remove the brackets `<>`.

    ```azurecli
    az account set --subscription "<SUBSCRIPTION-NAME>"
    ```

1. Now use CLI to submit the job, after replacing the `<VALUES-IN-BRACKETS>` with their values (also remove the brackets `<>`). If you are doing this on a compute instance in your workspace, you can use the environment variables for the workspace name and resource group.  If you are not on a compute instance, replace these values as well.

    ```azurecli
    az ml job create -f job.yml  --workspace-name $CI_WORKSPACE --resource-group $CI_RESOURCE_GROUP
    ```

## Next steps

[How to deploy an R model to an online (real time) endpoint](how-to-razureml-deploy-r-model.md)
