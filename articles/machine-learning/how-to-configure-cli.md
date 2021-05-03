---
title: 'Install, set up, and use the new Azure Machine Learning CLI'
description: Learn how to install, set up, and use the new Azure CLI extension for Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: how-to

author: lostmygithubaccount
ms.author: copeters
ms.date: 05/25/2021
ms.reviewer: laobri
---

# Install, set up, and use the new Azure Machine Learning CLI

The new `ml` extension to the [Azure CLI](/cli/azure/) is the next-generation interface for Azure Machine Learning. It enables you to train and deploy models from the command line, with features that accelerate scaling data science up and out while tracking the model lifecycle.

## Prerequisites

- To use the CLI, you must have an Azure subscription. If you don't have an Azure subscription, create a free account before you begin. Try the [free or paid version of Azure Machine Learning](https://aka.ms/AMLFree) today.

- To use the CLI commands in this document from your **local environment**, you need the [Azure CLI](/cli/azure/install-azure-cli).

    > [!TIP]
    > If you use the [Azure Cloud Shell](https://azure.microsoft.com/features/cloud-shell/), the CLI is accessed through the browser and lives in the cloud.

## Installation

The new Machine Learning extension requires Azure CLI version `>=2.15.0`. Ensure this requirement is met:

```azurecli
az version
```

If required, upgrade the Azure CLI:

```azurecli
az upgrade
```

> [!IMPORTANT]
> The `az upgrade` command was added in version TBD. If below that version, you need to manually install a newer version.

Check the extensions installed:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/how-to-configure-cli.sh" id="az_extension_list":::

Ensure no conflicting extension using the `ml` namespace is installed, including the old `azure-cli-ml` extension:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/how-to-configure-cli.sh" id="az_extension_remove":::

Now, install the new Machine Learning extension:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/setup.sh" id="az_ml_install":::

You can upgrade the extension to the latest version:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/how-to-configure-cli.sh" id="az_ml_update":::

Run the help command to verify your installation and see available subcommands:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/how-to-configure-cli.sh" id="az_ml_verify":::

## Set up

Login:

```azurecli
az login
```

If you have access to multiple Azure subscriptions, you can set your active subscription:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/how-to-configure-cli.sh" id="az_account_set":::

If it doesn't already exist, you can create the Azure resource group:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/setup.sh" id="az_group_create":::

Similarly for the machine learning workspace:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/setup.sh" id="az_ml_workspace_create":::

Nearly all machine learning subcommands require the `--workspace/-w` and `--resource-group/-g` parameters to be specified. To avoid typing these parameters repeatedly, you may configure defaults:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/setup.sh" id="az_configure_defaults":::

Note: most code examples assume you have set a default workspace and resource group. You can override these on the command line.

## Hello world

To follow along, clone the examples repository and change into the `cli` subdirectory:

```azurecli
git clone https://github.com/Azure/azureml-examples --depth 1
cd azureml-examples/cli
```

To run hello world locally via Python, see the example in the `jobs` subdirectory:

:::code language="yaml" source="~/azureml-examples-cli-preview/cli/jobs/hello-world.yml":::

> [!IMPORTANT]
> [Docker](https://docker.io) needs to be installed and running locally.

Submit the job, streaming the logs to the console output, and opening the run in the Azure Machine Learning studio:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/how-to-configure-cli.sh" id="hello_world":::

> [!IMPORTANT]
> This may take a few minutes to run the first time, as the Docker image is pulled locally and the Azure ML job is run. Subsequent runs will have the image cached locally and complete quicker.

## Next steps

- [Train models using Machine Learning CLI extension](how-to-train-cli.md)
- [Deploy models using Machine Learning CLI extension](how-to-deploy-cli.md)
- [Command reference for the Machine Learning CLI extension](../cli/azure/ext/ml/ml)
