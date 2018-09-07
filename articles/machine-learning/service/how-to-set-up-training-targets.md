---
title: Set up compute targets for model training with Azure Machine Learning service | Microsoft Docs
description: This article explains how to configure compute targets on which you can train your machine learning models with Azure Machine Learning service
services: machine-learning
author: heatherbshapiro
ms.author: hshapiro
manager: danielsc
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.topic: article
ms.date: 09/24/2018
---
# Select and use a compute target to train your model

With the Azure Machine Learning service, you can train your model in several different environments. These environments, called __compute targets__, can be local or in the cloud. In this document, you will learn about the supported compute targets and how to use them.

A compute target is the resource that runs your training script or hosts your model when it's deployed as a web service. They can be created and managed using the Azure Machine Learning SDK or CLI. If you have compute targets that were created by another process (for example, the Azure portal or Azure CLI), you can use them by attaching them to your Azure Machine Learning service workspace.

You can start with local runs on your machine, and then scaling up and out to other environments such as remote Data Science virtual machines with GPU or Azure Batch AI. 

## Supported compute targets

Azure Machine Learning supports the following compute targets:
Us
|Compute target| GPU acceleration | Automated hyperparameter tuning | Automated model selection | Can be used in pipelines|
|----|:----:|:----:|:----:|:----:|
|Local computer| Maybe | &nbsp; | ✓ | &nbsp; |
|Data Science Virtual Machine (DSVM) | ✓ | ✓ | ✓ | ✓ |
|Azure Batch AI| ✓ | ✓ | ✓ | ✓ | ✓ |
|Azure Container Instance| &nbsp; | &nbsp; | &nbsp; | &nbsp; |
|Azure HDInsight| &nbsp; | &nbsp; | &nbsp; | ✓ |

The key differentiators between the computer targets are:
* __GPU acceleration__: GPUs are available with the Data Science Virtual Machine and Azure Batch AI. You may have access to a GPU on your local computer, depending on the hardware that is installed.
* __Automated hyperparameter tuning__: Azure Machine Learning automated hyperparameter optimization helps you find the best hyperparameters for your model.
* __Automated model selection__: Azure Machine Learning can intelligently recommend algorithm and hyperparameter selection when building a model. This helps you converge to a high quality model faster than manually trying different combinations. For more information, see the [Tutorial: Automatically train a classification model with Azure Automated Machine Learning](tutorial-auto-train-models.md) document.
* __Pipelines__: Azure Machine Learning enables you to combine different tasks such as training and deployment into a pipeline. Pipelines can be ran in parallel or in sequence, and provide a reliable automation mechanism. For more information, see the [Build machine learning pipelines with Azure Machine Learning service](concept-ml-pipelines.md) document.

You can use the Azure Machine Learning SDK, Azure CLI, or Azure portal to create compute targets. You can also use existing compute targets by adding (attaching) them to your workspace.

> [!IMPORTANT]
> You cannot attach an existing Azure Containers Instance to your workspace. Instead, you must create a new instance.
>
> You cannot create an Azure HDInsight cluster within a workspace. Instead, you must attach an existing cluster.

## Workflow

The workflow for developing and deploying a model with Azure Machine Learning follows these steps:

1. Develop machine learning training scripts in Python.
1. Create and configure or attach an existing compute target.
1. Submit the training scripts to the compute target.
1. Inspect the results to find the best model.
1. Register the model in the model registry.
1. Deploy the model.

> [!IMPORTANT]
> Your training script isn't tied to a specific compute target. You can train initially on your local computer, then switch compute targets without having to rewrite the training script.

Switching from one compute target to another involves creating a [run configuration](concept-azure-machine-learning-architecture.md#run-configuration). The run configuration defines how to run the script on the compute target.

## Training scripts

When you start a training run, the entire directory that contains your training scripts is submitted. A snapshot is created and sent to the compute target. For more information, see [snapshots](concept-azure-machine-learning-architecture.md#snapshot).

## Local computer

When training locally, you use the SDK to submit the training operation. You can train using a user-managed or system-managed environment.

### User-managed environment

In a user-managed environment, you are responsible for ensuring that all the necessary packages are available in the Python environment you choose to run the script in.

1. Start by creating a local run config

    ```python
    from azureml.core.runconfig import RunConfiguration

    # Editing a run configuration property on-fly.
    run_config = RunConfiguration.load(project_object = project, run_config_name = "local")

    run_config.environment.python.user_managed_dependencies = True
    run_config.prepare_environment = False

    # You can choose a specific Python environment by pointing to a Python path 
    #run_config.environment.python.interpreter_path = '/home/ninghai/miniconda3/envs/sdk2/bin/pytho
    ```

2. Submit the script to run in the user-managed environment.

    ```python
    from azureml.core.run import Run

    run = Run.submit(project_object = project,
                      run_config = run_config,
                      script_to_run = 'train.py')

    # Shows output of the run on stdout.
    run.wait_for_completion(show_output = True)
    ```
  
### System-managed environment

System-managed environments rely on conda to manage the dependencies. Conda creates a file named `conda_dependencies.yml` that contains a list of dependencies. You can then ask the system to build a new conda environment and execute your scripts in it. System-managed environments can be reused later, as long as the `conda_dependencies.yml` files remains unchanged. 

The initial setup up of a new environment can take several minutes to complete, depending on the size of the required dependencies. The following code snippet demonstrates creating a system-managed environment that depends on scikit-learn:

```python
from azureml.core.conda_dependencies import CondaDependencies

# Editing a run configuration property on-fly.
run_config = RunConfiguration.load(project_object = project, run_config_name = "local")

# Use a new conda environment that is to be created from the conda_dependencies.yml file
run_config.environment.python.user_managed_dependencies = False

# Automatically create the conda environment before the run
run_config.prepare_environment = True

# add scikit-learn to the conda_dependencies.yml file
cd = CondaDependencies()
cd.add_conda_package('scikit-learn')
cd.save_to_file(project_dir = project_folder, conda_file_path = run_config.environment.python.conda_dependencies_file)
```

You can submit the run the same way as in the user-managed example. 

```python 
from azureml.core.run import Run

run = Run.submit(project_object = project,
                  run_config = run_config,
                  script_to_run = 'train.py')

# Shows output of the run on stdout.
run.wait_for_completion(show_output = True)
```

## Data Science Virtual Machine

Your local machine may not have the compute or GPU resources required to train the model. In this situation, You can scale up or scale out the training process by adding additional compute targets such as a Ubuntu-based Data Science Virtual Machines (DSVM).

> [!WARNING]
> Azure Machine Learning does not support CentOS. When creating a virtual machine or selecting an existing one, you must select one that uses Ubuntu.

The following steps use the SDK to configure a Data Science Virtual Machine (DSVM) as a training target:

1. Create or attach a Virtual Machine
    
    * To create a new DSVM:
    
        ```python
        from azureml.core.compute import DsvmCompute
        dsvm_config = DsvmCompute.provisioning_configuration(vm_size="Standard_D2_v2")
        dsvm_compute = DsvmCompute.create(ws, name="mydsvm", provisioning_configuration=dsvm_config)
        dsvm_compute.wait_for_provisioning(show_output=True)
        ```

    * To attach an existing DSVM:

2. Create a configuration for the DSVM compute target. Docker and conda are used to create and configure the training environment on DSVM:

    ```python
    from azureml.core.runconfig import RunConfiguration
    from azureml.core.conda_dependencies import CondaDependencies

    # Load the "cpu-dsvm.runconfig" file (created by the above attach operation) in memory
    run_config = RunConfiguration(project_object = project, 
                                  run_config_name = "cpu-dsvm",
                                  target = dsvm_compute.name, 
                                  framework = "python")

    # Use Docker in the remote VM
    run_config.environment.docker.enabled = True

    # Use the MMLSpark CPU based image.
    # https://hub.docker.com/r/microsoft/mmlspark/
    run_config.environment.docker.base_image = azureml.core.runconfig.DEFAULT_CPU_IMAGE
    #run_config.environment.docker.base_image = 'microsoft/mmlspark:plus-0.9.9'
    print('Base Docker image is:', run_config.environment.docker.base_image )

    # Ask system to provision a new one based on the conda_dependencies.yml file
    run_config.environment.python.user_managed_dependencies = False

    # Prepare the Docker and conda environment automatically when executingfor the first time.
    run_config.prepare_environment = True

    # create a new CondaDependencies obj
    cd = CondaDependencies()

    # add scikit-learn as a conda dependency
    cd.add_conda_package('scikit-learn')

    # overwrite the default conda_dependencies.yml file
    cd.save_to_file(project_dir = project_folder, file_name='conda_dependencies.yml')
    ```

3. Submit the script to run in the Docker environment on the DSVM.

    ```python
    from azureml.core.run import Run

    run = Run.submit(project_object = project,
                     run_config = run_config,
                     script_to_run = 'train.py')

    # Shows output of the run on stdout.
    run.wait_for_completion(show_output = True)
    ```

## Azure Batch AI

If it takes a long time to train your model, you can use Azure Batch AI to distribute the training across a cluster of compute resources in the cloud. Batch AI can also be configured to enable a GPU resource.

The following example looks for an existing Batch AI cluster by name. If one is not found, it is created:

```python
# Create a new compute target to train on Azure Batch AI
from azureml.core.compute import ComputeTarget, BatchAiCompute
from azureml.core.compute_target import ComputeTargetException

# Name the Batch AI cluster
batchai_cluster_name = "gpucluster2"

# Try to find an existing compute target in the workspace. If none exists,
#   create a new one.
try:
    compute_target = ComputeTarget(workspace = ws, name = batchai_cluster_name)
    print('found compute target. just use it.')
except ComputeTargetException:
    print('creating a new compute target...')
    provisioning_config = BatchAiCompute.provisioning_configuration(vm_size = "STANDARD_NC6", # NC6 is GPU-enabled
                                                                #vm_priority = 'lowpriority', # optional
                                                                autoscale_enabled = True,
                                                                cluster_min_nodes = 1, 
                                                                cluster_max_nodes = 4)
    # create the cluster
    compute_target = ComputeTarget.create(ws, batchai_cluster_name, provisioning_config)

    # can poll for a minimum number of nodes and for a specific timeout. 
    # if no min node count is provided it will use the scale settings for the cluster
    compute_target.wait_for_provisioning(show_output = True, min_node_count = None, timeout_in_minutes = 20)

     # For a more detailed view of current Batch AI cluster status, use the 'status' property    
    print(compute_target.status.serialize())
```

For more information on using the BatchAiCompute object, see the reference documentation. 

## Azure Container Instance (ACI)

Azure Container Instances are isolated containers that have faster startup times and do not require the user to manage any Virtual Machines. The following example shows how to use the SDK to create an ACI compute target and use it to train a model: 

```python
# Create a new compute target to train on Azure Container Instances (ACI)
from azureml.core.runconfig import RunConfiguration
from azureml.core.conda_dependencies import CondaDependencies

# create a new runconfig object
run_config = RunConfiguration(project_object = project, run_config_name = 'my-aci-run-config')

# signal that you want to use ACI to execute script.
run_config.target = "containerinstance"

# ACI container group is only supported in certain regions, which can be different than the region the Workspace is in.
run_config.container_instance.region = 'eastus'

# set the ACI CPU and Memory 
run_config.container_instance.cpu_cores = 1
run_config.container_instance.memory_gb = 2

# enable Docker 
run_config.environment.docker.enabled = True

# set Docker base image to the default CPU-based image
run_config.environment.docker.base_image = azureml.core.runconfig.DEFAULT_CPU_IMAGE
#run_config.environment.docker.base_image = 'microsoft/mmlspark:plus-0.9.9'

# use conda_dependencies.yml to create a conda environment in the Docker image for execution
run_config.environment.python.user_managed_dependencies = False

# auto-prepare the Docker image when used for execution (if it is not already prepared)
run_config.prepare_environment = True

# create a new CondaDependencies obj
cd = CondaDependencies()

# add scikit-learn as a conda dependency
cd.add_conda_package('psutil')
cd.add_conda_package('scikit-learn')

# overwrite the default conda_dependencies.yml file
cd.save_to_file(project_dir = project_folder, conda_file_path = run_config.environment.python.conda_dependencies_file)
```

## Attach an HDInsight cluster 

HDInsight is a popular platform for big-data analytics. It provides Apache Spark, which can be used to train your model. 

> [!IMPORTANT]
> You must create the HDInsight cluster before using it to train your model. To create a Spark on HDInsight cluster, see the [Create a Spark Cluster in HDInsight](https://docs.microsoft.com/azure/hdinsight/spark/apache-spark-jupyter-spark-sql) document.
>
> When creating the cluster, you must specify an SSH user name and password. Note these values, as you need them when using HDInsight as a compute target.
>
> Once the cluster has been created, it has a fully qualified domain name (FQDN) of `<clustername>.azurehdinsight.net`, where `<clustername>` is the name you provided for the cluster. You need this address (or the public IP address of the cluster) to use it as a compute target

To configure HDInsight as a compute target, you must provide the fully qualified domain name, cluster login name, and password for the HDInsight cluster. The following example uses the SDK to attach a cluster to your workspace. In the example, replace `<fqdn>` with the public fully qualified domain name of the cluster, or the public IP address. Replace `<username>` and `<password>` with the SSH user and password for the cluster:

> [!NOTE]
> To find the FQDN for your cluster, visit the Azure portal and select your HDInsight cluster. From the __Overview__ section, the FQDN is part of the __URL__ entry. Just remove the `https://` from the beginning of the value.

```python
from azureml.core.compute_target import HDIClusterTarget

try:
    # Attaches a HDInsight cluster as a compute target.
    project.attach_legacy_compute_target(HDIClusterTarget(name = "myhdi",
                                                            address = "<fqdn>", 
                                                            username = "<username>", 
                                                            password = "<password>"))
except UserErrorException as e:
    print("Caught = {}".format(e.message))
    print("Compute config already attached.")

# Configure HDInsight run
# load the runconfig object from the "myhdi.runconfig" file generated by the attach operaton above.
run_config = RunConfiguration.load(project_object = project, run_config_name = 'myhdi')

# ask system to prepare the conda environment automatically when executed for the first time
run_config.prepare_environment = True
```

## Set up compute using the Azure portal

### Create a compute target

1. Visit the web portal and navigate to your workspace.
2. Click on the __Compute link__ under the __Applications__ section.
3. Click the __+__ sign to add a compute target.
4. Enter a name for the compute target.
5. Select the type of compute to attach for __Training__. Only Batch AI and DSVM are currently supported in the portal.
6. Select __Create New__ and fill out the required forms.
7. You can view the status of the provisioning state by selecting the compute target from the list of Computes.
8. Now you can submit a run against these targets.

### Reuse existing compute in your workspace

The Web Portal makes it easy to attach existing compute targets from your subscription.

> [!NOTE]
> The SDK and CLI are limited to only showing compute targets that are associated with the workspace.

1. Visit the web portal and navigate to your workspace.
2. Click on the **Compute** link under the Applications section.
3. Click the **+** sign to add a compute target.
4. Enter a name for the compute target.
5. Select the type of compute to attach for Training. Batch AI and DSVM are currently supported in the portal.
6. Select 'Use Existing'.
    - When attaching Batch AI clusters, select the compute target from the dropdown and click Create.
    - When attaching a DSVM, enter the IP Address, Username/Password Combination, Private/Public Keys, and the Port and click Create.
7. You can view the status of the provisioning state by selecting the compute target from the list of Computes.
8. Now you can submit a run against these targets.

## Next steps

* [What is Azure Machine Learning service](overview-what-is-azure-ml.md)
* [Quickstart: Create a workspace with Python](quickstart-get-started.md)
* [Tutorial: Train a model](tutorial-train-models-with-aml.md)
* [Build machine learning pipelines with Azure Machine Learning service](concept-ml-pipelines.md)