---
title: "Tutorial:  Set up your local computer for Azure Machine Learning (Python)"
titleSuffix: Machine Learning - Azure 
description: In this tutorial, you'll to get started with the Azure Machine Learning Python SDK running on your local computer.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
author: aminsaied
ms.author: amsaied
ms.reviewer: sgilley
ms.date: 09/09/2020
ms.custom: devx-track-python
---

# Tutorial: Get started with Azure Machine Learning on your local computer (Part 1 of 4)

In this **four-part tutorial series**, you will learn the fundamentals of Azure Machine Learning and complete simple jobs-based machine learning tasks in the Azure cloud, including:

1. Set up a workspace and your local machine learning developer environment.
2. Run code in the cloud using Azure Machine Learning's Python SDK.
3. Manage the Python environment you use for model training.
4. Upload data to Azure and consume that data in training.

In **part 1 of this tutorial series**, you will:

> [!div class="checklist"]
> * Install the Azure Machine Learning SDK
> * Set up directory structure for code
> * Create an Azure Machine Learning workspace
> * Configure your local development environment
> * Set up a compute cluster

>[!NOTE]
> This tutorial series is focused heavily on the Azure Machine Learning features and concepts that help with __jobs-based__ machine learning tasks that have the following traits:
>
> - Python-based and
> - long running and/or
> - distributed (require many machines and processes) and/or
> - repeatable (require reproducible, auditable, and portable environment)
>
> If your ML tasks do not fit this profile, use the [Jupyter or RStudio functionality on a Azure Machine Learning compute instance](tutorial-1st-experiment-sdk-setup.md) to onboard to Azure Machine Learning.

## Prerequisites

- An Azure Subscription. If you don't have an Azure subscription, create a free account before you begin. Try [Azure Machine Learning](https://aka.ms/AMLFree) today.
- Familiarity with Python and [Machine Learning concepts](concept-azure-machine-learning-architecture.md). For example, environments, training, scoring, and so on.
- A local development environment - a laptop with Python installed and your favorite IDE (for example: VSCode, Pycharm, Jupyter, and so on).

## Install the Azure Machine Learning SDK

Throughout this tutorial, we make use of the Azure ML Python SDK. The SDK allows us to use
Python to interact with Azure Machine Learning.

You can use the tools most familiar to you - for example: conda, pip, and so on - to set up an environment to use throughout this tutorial. Install into the environment the Azure Machine Learning Python SDK via pip:

```bash
pip install azureml-sdk
```

## Create directory structure for code
We recommend that you have the following simple directory set up for this tutorial:

```markdown
tutorial
└──.azureml
```

- **tutorial** (top-level directory of the project)
- **.azureml** (hidden subdirectory of tutorial):  The `.azureml` directory is used to store Azure Machine Learning configuration files.

## Create an Azure Machine Learning Workspace

A workspace is a top-level resource for Azure Machine Learning and is a centralized place to:

- Manage resources such as compute
- Store assets like Notebooks, Environments, Datasets, Pipelines, Models, Endpoints, and so on
- Collaborate with other team members

In the top-parent directory - `tutorial` - add a new Python file called `01-create-workspace.py` with the code below. Adapt the parameters (name, subscription ID, resource group, and location) with your preferences. Valid locations include:

`eastasia`;`southeastasia`;`australiaeast`;`brazilsouth`;`canadacentral`;`northeurope`;`westeurope`;`francecentral`;`japaneast`;`koreacentral`;`uksouth`;`centralus`;`eastus`;`eastus2`;`westus`;`northcentralus`;`southcentralus`;`westcentralus`;`westus2`.

You can run the code in an interactive session, or as a python file.

>[!NOTE]
> When using a local development environment (e.g. laptop) you will be asked to authenticate to your workspace using a *device code* the first time you execute the code below. Follow the on-screen instructions.

```python
# tutorial/01-create-workspace.py
from azureml.core import Workspace

ws = Workspace.create(name='<my_workspace_name>', # provide a name for your workspace
                      subscription_id='<azure-subscription-id>', # provide your subscription ID
                      resource_group='<myresourcegroup>', # provide a resource group name
                      create_resource_group=True,
                      location='<NAME_OF_REGION>') # provide an azure region

# write out the workspace details to a configuration file: .azureml/config.json
ws.write_config(path='.azureml')
```

Run this code from the `tutorial` directory:

```bash
cd <path/to/tutorial>
python ./01-create-workspace.py
```

Once the above code snippet has been run, your folder structure will look like:

```markdown
tutorial
└──.azureml
|  └──config.json
└──01-create-workspace.py
```

The file `.azureml/config.json` contains the metadata necessary to connect to your Azure Machine Learning
workspace - namely your subscription ID, resource group and workspace name. We'll make use
of this as you progress through the tutorial with the following:

```python
ws = Workspace.from_config()
```

> [!NOTE]
> The contents of `config.json` are not secrets - it is perfectly fine to share these details.
> Authentication is still required to interact with your Azure Machine Learning workspace.

## Create an Azure Machine Learning compute cluster

Create a python script in the `tutorial` top-level directory called `02-create-compute.py` and populate with the following code to create an Azure Machine Learning compute cluster that will auto-scale between zero and four nodes:

```python
# tutorial/02-create-compute.py
from azureml.core import Workspace
from azureml.core.compute import ComputeTarget, AmlCompute
from azureml.core.compute_target import ComputeTargetException

ws = Workspace.from_config()

# Choose a name for your CPU cluster
cpu_cluster_name = "cpu-cluster"

# Verify that cluster does not exist already
try:
    cpu_cluster = ComputeTarget(workspace=ws, name=cpu_cluster_name)
    print('Found existing cluster, use it.')
except ComputeTargetException:
    compute_config = AmlCompute.provisioning_configuration(vm_size='STANDARD_D2_V2',
                                                            max_nodes=4, 
                                                            idle_seconds_before_scaledown=2400)
    cpu_cluster = ComputeTarget.create(ws, cpu_cluster_name, compute_config)

cpu_cluster.wait_for_completion(show_output=True)
```

Run the python file:

```bash
python ./02-create-compute.py
```


> [!NOTE]
> When the cluster has been created it will have 0 nodes provisioned. Therefore, **do not** incur costs until you submit a job. This cluster will scale down when it has been idle for 2400 seconds (40 minutes). 

Your folder structure will now look as follows:

```bash
tutorial
└──.azureml
|  └──config.json
└──01-create-workspace.py
└──02-create-compute.py
```

## Next steps

In this setup tutorial you have:

- Created an Azure ML workspace
- Set up your local development environment
- Created an Azure Machine Learning compute cluster.

In the next tutorial, you walk through submitting an ML script to the Azure Machine Learning compute cluster.

> [!div class="nextstepaction"]
> [Tutorial: Run "Hello World" Python Script on Azure](tutorial-1st-experiment-hello-world.md)
