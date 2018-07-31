---
title: How does Azure Machine Learning services work?
description: Learn about the architecture and concepts that make up Azure Machine Learning Services. 
services: machine-learning
ms.service: machine-learning
ms.component: core
ms.topic: conceptual
ms.author: haining
author: hning86
ms.date: 07/31/2018

---

# Azure Machine Learning Services architecture and concepts

The __Azure Machine Learning workspace__ is the top-level Azure resource for Azure Machine Learning. It's a logical container for all artifacts created by users when using Azure Machine Learning, including run history records, compute targets, models, Docker images, and deployments, among other things. Each workspace is supported by a list of associated Azure sources including a blob storage account, a KeyVault, an Azure Container Registry, and an AppInsights instance. It is also the security boundary for enabling secured sharing and collaboration among multiple users. 

The following diagram shows the major components of Azure Machine Learning, and illustrates the general workflow when using Azure Machine Learning: 

[![Azure Machine Learning architecture and workflow](./media/concept-azure-machine-learning-architecture/workflow.png)](./media/concept-azure-machine-learning-architecture/workflow.png#lightbox)

The workflow for developing and deploying a model with Azure Machine Learning generally follows these steps:

1. Develop machine learning training scripts in __Python__ using __Jupyter Notebooks__, __Visual Studio Code__, or any other Python development environment of your choice.
2. Provision and configure a __compute target__, which can be either local or cloud compute resources, to use when training the model.
3. Submit the scripts to the configured __compute target__ to run in that environment.
4. Query the __run history__ for logged metric from this run (and past runs) of your training job. If the metric does not indicate a desired outcome, loop back to step 1 and iterate on your scripts.
5. Once a satisfactory run is found, register the persisted model in the __model registry__.
6. Develop a scoring script.
7. Deploy the model and the scoring scripts as a __web service__ in Azure.


## Workspace

The workspace provides a list of compute targets that can be used to train your model. It also keeps a history of the training runs, including logs, metrics, output, and a snapshot of your project. This information can be used to determine which training run produces the best model.

Once you've determined the best model, you can register it with the workspace. From a registered model, coupled with scoring scripts, you can create a Docker image. The Docker image can then be deployed into Azure Container Instances or Azure Kubernetes Service as a REST-based HTTP endpoint.

You can create multiple workspaces, and each workspace can be shared by multiple people. When sharing a workspace, you can assign the following roles to users:

* Owner
* Contributor
* Reader

When you create a new workspace, it automatically creates several Azure resources that are used by the workspace:

* [Azure Container Registry](https://azure.microsoft.com/en-us/services/container-registry/) - Registers docker containers that are used during training and when deploying a model.
* [Azure Storage](https://azure.microsoft.com/en-us/services/storage/) - Used as the default datastore for the workspace.
* [Azure Application Insights](https://azure.microsoft.com/en-us/services/application-insights/) - Stores monitoring information about your models.
* [Azure Key Vault](https://azure.microsoft.com/en-us/services/key-vault/) - Stores secrets used by compute targets and other sensitive information needed by the workspace.

> [!NOTE]
> Instead of creating new versions, you can also use existing Azure services. For more information, see [TBD].

The following diagram is a taxonomy of the workspace:

[![Workspace taxonomy](./media/concept-azure-machine-learning-architecture/taxonomy.png)](./media/concept-azure-machine-learning-architecture/taxonomy.png#lightbox)

## Project

A project is a local folder that contains the files for your machine learning solution.
To use a local folder with Azure Machine Learning Services, you simply attach it to a workspace with a run history name. For an example of how to attach a project to a workspace, see one of the following documents:

* [Create a workspace with Python TBD]()
* [Create a workspace with Azure CLI TBD]()

When you submit a project for training, the entire folder is copied to the compute target. The entry script is run in the Python environment configured through the __run configuration__. A snapshot of the copy is also stored in run history.

## Model

A model is a scoring logic operation materialized in one or more files. A model can be produced by a run in Azure Machine Learning. You can also use a model trained outside of Azure Machine Learning. A model can be registered under a Workspace, and can be version-managed. It is used to create a Docker image and deployment. You can use any popular machine learning framework of your choice, including scikit-learn, xgboost, TensorFlow, CNTK etc. Azure Machine Learning takes a framework-agnostic approach when it comes to model type and model file format.

## Docker image
A Docker image is created from your project, and registered with the workspace. It encapsulates:
* A model file, or a folder of model files
* A scoring script
* Any number of supporting library files (optional)
* A Conda environment file listing Python package dependencies (optional)
* Schema files for swagger generation (optional)

## Deployment

A deployment is a deployed web service in either Azure Container Instances or Azure Kubernetes Service.
It is a live Docker container created from a Docker image that encapsulates your model, script, and associated files, with an HTTP endpoint to send scoring request to.

## Datastore
A datastore is a storage abstraction over an Azure Storage Account. The datastore can use either an Azure blob container or an Azure file share as the backend storage. Each workspace has a default datastore, and you may register additional datastores. For information on using additional datastores, see [TBD]().

You can use the Python SDK API or Azure ML CLI to store and retrieve files from the datastore. For more information, see [TBD]().

## Run
A run is an execution record stored in a run history under a workspace.
It contains the following information:

* Metadata about the run (timestamp, duration etc.)
* Metrics logged by your script
* Output files auto-collected by the run history service, or explicitly uploaded by you.
* A snapshot of the project folder prior to the run is executed

A run can have zero or more child runs.

## Run history

A run history is a logical grouping of many runs executed from a given project. It always belongs to a workspace. You can submit a run using an arbitrary run history name, and the submitted run is then listed under that run history.

## Run configuration

A run configuration is a set of instructions that defines how a script should be executed in a given compute target. It includes a wide set of behavior definitions, such as whether to use an existing Python environment or use a Conda environment built from specification.

A run configuration can be persisted into a file inside your project, or can be constructed as an in-memory object and used to submit a run.

## Compute target

A compute target is the compute resource used to execute your training script or host your web service deployment. They can be created and managed by using Azure ML Python SDK or CLI. You can also attach existing compute targeted in Azure. The supported compute targets are: 

* Your local computer
* A Linux VM in Azure (such as the Data Science Virtual Machine)
* Azure Batch AI Cluster
* Apache Spark for HDInsight
* Azure Container Instance
* Azure Kubernetes Service

Compute targets are attached to a workspace. Computer targets other than the local machine are shared by users of the workspace.

## Metrics

When developing your solution, you can use the Azure Machine Learning Python SDK in your Python script to log any arbitrary metrics information. Post execution, you can query the metrics to determine if the run produces the model you want to deploy. 

For an example of using metrics, see [TBD]()

## Snapshots

When submitting a project run, Azure Machine Learning compresses the project folder as a zip file and sends it to the compute target. The project is then expanded and executed there. Azure Machine Learning also stores the zip file as a snapshot as part of the run record. Anyone with access to the workspace can browse a run record and download the snapshot.

## Task

A task represents a long running operation. The following operations are examples of tasks:

* Creating or deleting a compute target
* Executing a script on a compute target

Tasks can provide notifications through the SDK or Web UI so you can easily monitor the progress of these operations.