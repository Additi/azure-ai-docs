---
title: Train and deploy models from the CLI
titleSuffix: Azure Machine Learning service
description: Learn how to use the machine learning extension for Azure CLI to train, register, and deploy a model from the command line.
ms.author: larryfr
author: Blackmist
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.date: 09/12/2019
---

# Tutorial: Train and deploy a model from the CLI

In this tutorial, you use the machine learning extension for the Azure CLI to train, register, and deploy a model.

> [!NOTE]
> The scripts in this tutorial use [scikit-learn](https://scikit-learn.org/) to train a basic model. The focus of this tutorial is not on the scripts or the model, but the process of using the CLI to work with the Azure Machine Learning.

Learn how to take the following actions:

> [!div class="checklist"]
> * Install the machine learning extension
> * Create an Azure Machine Learning workspace
> * Create the compute resource used to train the model
> * Start a training run
> * Register and download a model
> * Deploy the model as a web service
> * Score data using the web service

## Prerequisites

* An Azure subscription.

* The Azure CLI.

## Download the example project

For this tutorial, download the [https://github.com/microsoft/MLOps](https://github.com/microsoft/MLOps) project. The files in the `model-training` and `model-deployment` directories are used by the steps in this tutorial.

To get a local copy of the files, either [download a .zip archive](https://github.com/microsoft/MLOps/archive/master.zip), or use Git command to clone the repository. The URL to use when cloning the repository is `https://github.com/microsoft/MLOps.git`.

### Training files

The `model-training` directory contains the following files, which are used when training a model:

* `.azureml\sklearn.runconfig`: A __run configuration__ file. This file defines the runtime environment needed to train the model.
* `train-sklearn.py`: The training script. This file trains the model.
* `mylib.py`: A helper module, used by `train-sklearn.py`.
* `training-env.yml`: Defines the software dependencies needed to run the training script.

The training script uses the diabetes dataset provided with scikit-learn to train a model.

### Deployment files

The `model-deployment` directory contains the following files, which are used to deploy the trained model as a web service:

* `aciDeploymentConfig.yml`: A __deployment configuration__ file. This file defines the hosting environment needed for the model.
* `inferenceConfig.yml`: A __inference configuration__ file. This file defines the software environment used by the service to score data with the model.
* `score.py`: A python script that accepts incoming data, scores it using the model, and then returns a response.
* `scoring-env.yml`: The conda dependencies needed to run the model and `score.py` script.

## Connect to your Azure subscription

There are several ways that you can authenticate to your Azure subscription from the CLI. The most basic is to interactively authenticate using a browser. To authenticate interactively, open a command line or terminal and use the following command:

```azurecli
az login
```

If the CLI can open your default browser, it will do so and load a sign-in page. Otherwise, you need to open a browser and follow the instructions on the command line. The instructions involve browsing to [https://aka.ms/devicelogin](https://aka.ms/devicelogin) and entering an authorization code.

For other methods of authenticating, see [Sign in with Azure CLI](https://docs.microsoft.com/cli/azure/authenticate-azure-cli?view=azure-cli-latest).

## Install the machine learning extension

To install the machine learning extension, use the following command:

```azurecli-interactive
az extension add -n azure-cli-ml
```

## Create a resource group

A resource group is a basic container of resources on the Azure platform. When working with the Azure Machine Learning service, the resource group will contain your Azure Machine Learning service workspace. It will also contain other Azure services used by the workspace. For example, if you train your model using a cloud-based compute resource, that resource is created in the resource group.

You can use an existing resource group or create a new one. To __create a new resource group__, use the following command. Replace `<resource-group-name>` with the name to use for this resource group. Replace `<location>` with the Azure region to use for this resource group:

> [!TIP]
> You should select a region where the Azure Machine Learning service is available. For information, see [Products available by region](https://azure.microsoft.com/global-infrastructure/services/?products=machine-learning-service).

```azurecli-interactive
az group create --name <resource-group-name> --location <location>
```

The response from this command is similar to the following JSON:

```json
{
  "id": "/subscriptions/<subscription-GUID>/resourceGroups/<resourcegroupname>",
  "location": "<location>",
  "managedBy": null,
  "name": "<resource-group-name>",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null,
  "type": null
}
```

For more information on working with resource groups, see [az group](https://docs.microsoft.com//cli/azure/group?view=azure-cli-latest).

## Create a workspace

To create a new workspace use the following command. Replace `<workspace-name>` with the name you want to use for this workspace. Replace `<resource-group-name>` with the name of the resource group:

> [!TIP]
> This command creates a workspace and the Azure services it depends on. For information on creating a workspace that uses existing Azure services, see [Create workspaces with the CLI](how-to-manage-workspace-cli.md).

```azurecli-interactive
az ml workspace create -w <workspace-name> -g <resource-group-name>
```

The output of this command is similar to the following JSON:

```json
{
  "applicationInsights": "/subscriptions/<service-GUID>/resourcegroups/<resource-group-name>/providers/microsoft.insights/components/<application-insight-name>",
  "containerRegistry": "/subscriptions/<service-GUID>/resourcegroups/<resource-group-name>/providers/microsoft.containerregistry/registries/<acr-name>",
  "creationTime": "2019-08-30T20:24:19.6984254+00:00",
  "description": "",
  "friendlyName": "<workspace-name>",
  "id": "/subscriptions/<service-GUID>/resourceGroups/<resource-group-name>/providers/Microsoft.MachineLearningServices/workspaces/<workspace-name>",
  "identityPrincipalId": "<GUID>",
  "identityTenantId": "<GUID>",
  "identityType": "SystemAssigned",
  "keyVault": "/subscriptions/<service-GUID>/resourcegroups/<resource-group-name>/providers/microsoft.keyvault/vaults/<key-vault-name>",
  "location": "<location>",
  "name": "<workspace-name>",
  "resourceGroup": "<resource-group-name>",
  "storageAccount": "/subscriptions/<service-GUID>/resourcegroups/<resource-group-name>/providers/microsoft.storage/storageaccounts/<storage-account-name>",
  "type": "Microsoft.MachineLearningServices/workspaces",
  "workspaceid": "<GUID>"
}
```

## Connect local project to workspace

> [!TIP]
> This step is optional. If you do not attach the folder, then you must provide the `-w <workspace-name>` and `-g <resource-group-name>` information to all the other commands after this section.

From a terminal or command prompt, change directories to the `MLOps` directory, then use the following command to connect to your workspace:

```azurecli-interactive
az ml folder attach -w <workspace-name> -g <resource-group-name>
```

The output of this command is similar to the following JSON:

```json
{
  "Experiment name": "model-training",
  "Project path": "/Code/MLOps/model-training",
  "Resource group": "<resource-group-name>",
  "Subscription id": "<subscription-id>",
  "Workspace name": "<workspace-name>"
}
```

This command creates a `.azureml/config.json` file, which contains information needed to connect to your workspace.

## Create the compute target for training

This example uses an Azure Machine Learning Compute instance to train the model. To create a new compute instance, use the following command:

```azurecli-interactive
az ml computetarget create amlcompute -n cpu --max-nodes 4 --vm-size Standard_D2_V2
```

The output of this command is similar to the following JSON:

```json
{
  "location": "<location>",
  "name": "cpu",
  "provisioningErrors": null,
  "provisioningState": "Succeeded"
}
```

This command creates a new compute target named `cpu`, with a maximum of 4 nodes. The VM size selected provides a VM with a GPU resource. For information on the VM size, see [VM types and sizes].

> [!TIP]
> When not in use, the VM scales down to 0 nodes to reduce costs.

> [!IMPORTANT]
> The name of the compute target (`cpu` in this case), is important; it is referenced by the `.azureml

## Submit the training run

To start a training run on the `cpu` compute target, change directories to the `model-training` directory and then use the following command:

```azurecli-interactive
az ml run submit-script -e myexperiment -c sklearn -d training-env.yml -t runoutput.json train-sklearn.py
```

This command specifies a name for the experiment (`myexperiment`). The experiment stores information about this run in the workspace.

The `-c sklearn` parameter specifies the `.azureml/sklearn.runconfig` file. As mentioned earlier, this file contains information used to configure the environment used by the training run. If you inspect this file, you'll see that it references the `cpu` compute target you created earlier. It also lists the number of nodes to use when training (`"nodeCount": "4"`), and contains a `"condaDependenciees"` section that lists the Python packages needed to run the training script.

For more information on run configuration files, see [Set up and use compute targets for model training](how-to-set-up-training-targets.md#runconfiguration).

The `-t` parameter stores a reference to this run in a JSON file, and will be used in the next steps to register and download the model.

As the training run processes, it streams information from the training session on the remote compute resource. Part of the information is similar to the following text:

```text
alpha is 0.00, and mse is 3424.32
alpha is 0.05, and mse is 3408.92
alpha is 0.10, and mse is 3372.65
alpha is 0.15, and mse is 3345.15
alpha is 0.20, and mse is 3325.29
alpha is 0.25, and mse is 3311.56
alpha is 0.30, and mse is 3302.67
alpha is 0.35, and mse is 3297.66
alpha is 0.40, and mse is 3295.74
alpha is 0.45, and mse is 3296.32
alpha is 0.50, and mse is 3298.91
alpha is 0.55, and mse is 3303.14
alpha is 0.60, and mse is 3308.70
alpha is 0.65, and mse is 3315.36
alpha is 0.70, and mse is 3322.90
alpha is 0.75, and mse is 3331.17
alpha is 0.80, and mse is 3340.02
alpha is 0.85, and mse is 3349.36
alpha is 0.90, and mse is 3359.09
alpha is 0.95, and mse is 3369.13


The experiment completed successfully. Finalizing run...
Cleaning up all outstanding Run operations, waiting 300.0 seconds
```

This text is logged from the training script (`train-sklearn.py`) and displays two of the performance metrics for this model. In this case, we want the model with the highest alpha value.

> [!IMPORTANT]
> The performance metrics are specific to the model you are training. Other models will have different performance metrics.

If you inspect the `train-sklearn.py`, you'll notice that it also uses the alpha value when it stores the trained model(s) to file. In this case, it trains several models. The one with the highest alpha should be the best one. Looking at the output above, and the code, the model with an alpha of 0.95 was saved as `./outputs/ridge_0.95.pkl`

> [!IMPORTANT]
> The model was saved to the `./outputs` directory on the compute target where it was trained. In this case, the Azure Machine Learning Compute instance in the Azure cloud. The training process automatically uploads the contents of the `./outputs` directory from the compute target where training occurs to your Azure Machine Learning workspace. It's stored as part of the experiment (`myexperiment` in this example).

## Register the model

To register the model directly from the stored version in your experiment, use the following command:

```azurecli-interactive
az ml model register -n mymodel -f runoutput.json --asset-path "outputs/ridge_0.95.pkl" -t registeredmodel.json
```

This command registers the `outputs/ridge_0.95.pkl` file created by the training run as a new model registration named `mymodel`. The `--assets-path` references a path in an experiment. In this case, the experiment and run information is loaded from the `runoutput.json` file created by the training command. The `-t registeredmodel.json` creates a JSON file that references the new registered model created by this command, and is used by other CLI commands that work with registered models.

The output of this command is similar to the following JSON:

```json
{
  "createdTime": "2019-09-19T15:25:32.411572+00:00",
  "description": "",
  "experimentName": "myexperiment",
  "framework": "Custom",
  "frameworkVersion": null,
  "id": "mymodel:1",
  "name": "mymodel",
  "properties": "",
  "runId": "myexperiment_1568906070_5874522d",
  "tags": "",
  "version": 1
}
```

Note the version number. This is incremented each time you register a new model with this name. For example, you can download the model and register it from a local file by using the following commands:

```azurecli-interactive
az ml model download -i "mymodel:1" -t .
az ml model register -n mymodel -p "ridge_0.95.pkl"
```

The first command downloads the registered model to the current directory. The file name is `ridge_0.95.pkl`, which is the file referenced when you registered the model. The second command registers the local model (`-p "ridge_0.95.pkl"`) with the same name as the previous registration (`mymodel`). This time, the JSON data returned lists the version as 2.

> [!TIP]
> An important concept is that a model registration is not limited to one model. You can specify a directory instead of a file, and the contents of the directory are stored in the model registration.

## Deploy the model

To deploy a model, change directories to the `model-deployment` directory and then use the following command.:

```azurecli-interactive
az ml model deploy -n myservice -m "mymodel:1" --ic inferenceConfig.yml --dc aciDeploymentConfig.yml
```

> [!IMPORTANT]
> You may receive the message "Failed to create Docker client". If so, this is not a problem. The CLI can deploy a web service to a local Docker container and checks for Docker. In this case, we are not using a local deployment.

This command deploys a new service named `myservice`, using version 1 of the model that you registered previously.

The `inferenceConfig.yml` file provides information on how to perform inference,such as the entry script (`score.py`) and software dependencies. For more information on the structure of this file, see the [Inference configuration schema](reference-azure-machine-learning-cli.md#inference-configuration-schema). For more information on entry scripts, see [Deploy models with the Azure Machine Learning service](how-to-deploy-and-where.md#prepare-to-deploy).

The `aciDeploymentConfig.yml` describes the deployment environment used to host the service. The deployment configuration is specific to the compute type that you use for the deployment. In this case, an Azure Container Instance is used. For more information, see the [Deployment configuration schema](reference-azure-machine-learning-cli.md#deployment-configuration-schema).

> [!IMPORTANT]
> It will take several minutes before the deployment process completes.

> [!TIP]
> In this example, Azure Container Instances is used. Deployments to ACI automatically create the needed ACI resource. If you were to instead deploy to Azure Kubernetes Service, you must create an AKS cluster ahead of time and specify it as part of the `az ml model deploy` command. For an example of deploying to AKS, see [Deploy a model to an Azure Kubernetes Service cluster](how-to-deploy-azure-kubernetes-service.md).

After several minutes, information similar to the following JSON is returned:

```json
ACI service creation operation finished, operation "Succeeded"
{
  "computeType": "ACI",
  {...ommitted for space...}
  "runtimeType": null,
  "scoringUri": "http://6c061467-4e44-4f05-9db5-9f9a22ef7a5d.eastus2.azurecontainer.io/score",
  "state": "Healthy",
  "tags": "",
  "updatedAt": "2019-09-19T18:22:32.227401+00:00"
}
```

The `scoringUri` is the REST endpoint for the deployed service. You can also get this URI by using the following command:

```azurecli-interactive
az ml service show -n myservice
```

This command returns the same JSON document, including the `scoringUri`.

## Send data to the service

While you can create a client application to call the endpoint, the machine learning CLI provides a utility that can act as a test client. Use the following command to send test data to the service:

```azurecli-interactive
az ml service run -n myservicee -d '{"data":[[1,2,3,4,5,6,7,8,9,10]]}'
```

The response from the command is similar to `[4684.920839774082]`.

For information on creating a client application in several programming languages, see [Consume an Azure Machine Learning model deployed as a web service](how-to-consume-web-service.md)

## Clean up resources

> [!IMPORTANT]
> The resources you created can be used as prerequisites to other Azure Machine Learning tutorials and how-to articles.

If you don't plan to use the resources you created, delete them so you don't incur additional charges.

To delete the resource group, and all the Azure resources created in this document, use the following command. Replace `<resource-group-name>` with the name of the resource group you created earlier:

```azurecli-interactive
az group delete -g <resource-group-name> -y
```

## Next steps

In this Azure Machine Learning tutorial, you used the machine learning CLI for the following tasks:

> [!div class="checklist"]
> * Install the machine learning extension
> * Create an Azure Machine Learning workspace
> * Create the compute resource used to train the model
> * Start a training run
> * Register and download a model
> * Deploy the model as a web service
> * Score data using the web service

For more information on using the CLI, see [Use the CLI extension for Azure Machine Learning service](reference-azure-machine-learning-cli.md).