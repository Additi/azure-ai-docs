---
title: Create and use compute targets for model training
titleSuffix: Azure Machine Learning service
description: Configure the training environments (compute targets) for machine learning model training. You can easily switch between training environments. Start training locally. If you need to scale out, switch to a cloud-based compute target.
services: machine-learning
author: heatherbshapiro
ms.author: hshapiro
ms.reviewer: sgilley
ms.service: machine-learning
ms.component: core
ms.topic: article
ms.date: 01/04/2018
ms.custom: seodec18
---
# Set up compute targets for model training

With Azure Machine Learning service, you can train your model on a variety of resources or environments, collectively referred to as [__compute targets__](concept-azure-machine-learning-architecture.md#compute-target). A compute target can be a local machine or a cloud resource, such as an Azure Machine Learning Compute, Azure HDInsight or a remote virtual machine.  

You can create and manage a compute target using the Azure Machine Learning SDK, Azure portal, or Azure CLI. If you have compute targets that were created through another service (for example, an HDInsight cluster), you can use them by attaching them to your Azure Machine Learning service workspace.
 
In this article, you learn how to use various compute targets.  The steps for all compute targets follow the same workflow:
1. __Create__ a compute target if you don’t already have one.
2. __Attach__ the compute target to your workspace.
3. __Configure__ the compute target so that it contains the Python environment and package dependencies needed by your script.


## Supported compute targets

Azure Machine Learning service has varying support across different compute targets. A typical model development lifecycle starts with dev/experimentation on a small amount of data. At this stage, we recommend using a local environment. For example, your local computer or a cloud-based VM. As you scale up your training on larger data sets, or do distributed training, we recommend using Azure Machine Learning Compute to create a single- or multi-node cluster that autoscales each time you submit a run. You can also attach your own compute resource, although support for various scenarios may vary as detailed below:


|Compute target| GPU acceleration | Automated<br/> hyperparameter tuning | Automated</br> machine learning | Pipeline friendly|
|----|:----:|:----:|:----:|:----:|
|[Local computer](#local)| Maybe | &nbsp; | ✓ | &nbsp; |
|[Azure Machine Learning Compute](#amlcompute)| ✓ | ✓ | ✓ | ✓ |
|[Remote VM](#vm) | ✓ | ✓ | ✓ | ✓ |
|[Azure Databricks](how-to-create-your-first-pipeline.md#databricks)| &nbsp; | &nbsp; | ✓ | ✓[*](#pipeline-only) |
|[Azure Data Lake Analytics](how-to-create-your-first-pipeline.md#adla)| &nbsp; | &nbsp; | &nbsp; | ✓[*](#pipeline-only) |
|[Azure HDInsight](#hdinsight)| &nbsp; | &nbsp; | &nbsp; | ✓ |

<a id="pipeline-only"></a>__*__ Azure Databricks and Azure Data Lake Analytics can __only__ be used in a pipeline. The steps to use a compute target in a pipeline differ from what is shown below.  For more information about using compute targets in a pipeline see [Create and run a machine learning pipeline](how-to-create-your-first-pipeline.md).

## What's a run configuration

When training, it is common to start on your local computer, and later run that training script on a different compute target. With Azure Machine Learning service, you can run your script on various compute targets without having to change your script. 

All you need to do is define the environment for each compute target with a **run configuration**.  Then, when you want to run your training experiment on a different compute target, specify that run configuration.


Learn more about [submitting experiments](#submit) at the end of this article.

### Manage environment and dependencies

When you create a run configuration, you need to decide how to manage the environment and dependencies on the compute target. 

#### System-managed environment

Use a system-managed environment when you want [Conda](https://conda.io/docs/) to manage the Python environment and the script dependencies for you. A system-managed environment is assumed by default and the most common choice. It is useful on remote compute targets, especially when you cannot configure that target. 

All you need to do is specify each package dependency using the [CondaDependency class](https://docs.microsoft.com/python/api/azureml-core/azureml.core.conda_dependencies.condadependencies?view=azure-ml-py) Then Conda creates a file named **conda_dependencies.yml** in the **aml_config** directory in your workspace with your list of package dependencies and sets up your Python environment when you submit your training experiment. 

The initial set up of a new environment can take several minutes depending on the size of the required dependencies. As long as the list of packages remains unchanged, the set up time happens only once.
  
The following code shows an example for a system-managed environment requiring scikit-learn:
    
[!code-python[](~/aml-sdk-samples/ignore/doc-qa/how-to-set-up-training-targets/runconfig.py?name=system-managed)]

#### User-managed environment

For a user-managed environments, you're responsible for setting up your environment and installing every package your training script on the compute target. If your training environment is already configured (such as on your local machine), you can skip the set up step by setting `user_managed_dependencies` to True. Conda will not check your environment or install anything for you.

The following code shows an example of configuring training runs for a user-managed environment:

```python
from azureml.core.runconfig import RunConfiguration

run_cfg_user = RunConfiguration()
run_cfg_user.environment.python.user_managed_dependencies = True

# Point to another Python environment. For example: 
# run_config.environment.python.interpreter_path = '/home/me/miniconda3/envs/sdk2/bin/python'
```
  
## Set up compute targets with Python

Use the sections below to configure these compute targets:

* [Local computer](#local)
* [Azure Machine Learning Compute](#amlcompute)
* [Remote virtual machines](#vm)
* [Azure HDInsight](#hdinsight)


### <a id="local"></a>Local computer

1. **Create and attach**: There's no need to create or attach a compute target when you use your local computer as the training environment.  

1. **Configure**:  When you use your local computer as a compute target, the training code is run in your [development environment](how-to-configure-environment.md).  If that environment already has the Python packages you need, use the user-managed environment.

    ```python
    from azureml.core.runconfig import RunConfiguration
    
    run_local = RunConfiguration()
    run_local.user_managed_dependencies = True
    ```

Now that you’ve attached the compute and configured your run, the next step is to [submit the training run](#submit).

### <a id="amlcompute"></a>Azure Machine Learning Compute

Azure Machine Learning Compute is a managed-compute infrastructure that allows the user to easily create a single or multi-node compute. The compute is created within your workspace region as a resource that can be shared with other users in your workspace. The compute scales up automatically when a job is submitted, and can be put in an Azure Virtual Network. The compute executes in a containerized environment and packages your model dependencies in a [Docker container](https://www.docker.com/why-docker).

You can use Azure Machine Learning Compute to distribute the training process across a cluster of CPU or GPU compute nodes in the cloud. For more information on the VM sizes that include GPUs, see [GPU-optimized virtual machine sizes](https://docs.microsoft.com/azure/virtual-machines/linux/sizes-gpu).

Azure Machine Learning Compute has default limits, such as the number of cores that can be allocated. For more information, see [Manage and request quotas for Azure resources](https://docs.microsoft.com/azure/machine-learning/service/how-to-manage-quotas).


You can create an Azure Machine Learning compute environment on demand when you schedule a run, or as a persistent resource.

#### Run-based creation

You can create Azure Machine Learning Compute as a compute target at run time. The compute is automatically created for your run. The cluster scales up to the number of **max_nodes** that you specify in your run config. The compute is deleted automatically once the run completes.

> [!IMPORTANT]
> Run-based creation of Azure Machine Learning compute is currently in Preview. Don't use run-based creation if you use automated hyperparameter tuning or automated machine learning. To use hyperparameter tuning or automated machine learning, create a [persistent compute](#persistent) target instead.

1.  **Create, attach, and configure**: The run-based creation performs all the necessary steps to create, attach, and configure the compute target with the run configuration.  

    ```python
    from azureml.core.compute import ComputeTarget, AmlCompute
    
    # First, list the supported VM families for Azure Machine Learning Compute AmlCompute.supported_vmsizes()
    from azureml.core.runconfig import RunConfiguration
    
    # Create a new runconfig object 
    run_temp_compute = RunConfiguration()
    
    # Signal that you want to use AmlCompute to execute the script
    run_temp_compute.target = "amlcompute"
    
    # AmlCompute is created in the same region as your workspace 
    # Set the VM size for AmlCompute from the list of supported_vmsizes
    run_temp_compute.vm_size = 'STANDARD_D2_V2'
    ```


Now that you’ve attached the compute and configured your run, the next step is to [submit the training run](#submit).

#### <a id="persistent"></a>Persistent compute

A persistent Azure Machine Learning Compute can be reused across jobs. The compute can be shared with other users in the workspace and is kept between jobs.

1. **Create and attach**: To create a persistent Azure Machine Learning Compute resource in Python, specify the **vm_size** and **max_nodes** properties. Azure Machine Learning then uses smart defaults for the other properties. The compute autoscales down to zero nodes when it isn't used.   Dedicated VMs are created to run your jobs as needed.
    
    * **vm_size**: The VM family of the nodes created by Azure Machine Learning Compute.
    * **max_nodes**: The max number of nodes to autoscale up to when you run a job on Azure Machine Learning Compute.
    
    ```python
    from azureml.core.compute import ComputeTarget, AmlCompute
    from azureml.core.compute_target import ComputeTargetException
    
    # Choose a name for your CPU cluster.  The name should be between 2-16 characters in length.
    cpu_cluster_name = "cpucluster"
    
    # Verify that the cluster doesn't already exist
    try:
        cpu_cluster = ComputeTarget(workspace=ws, name=cpu_cluster_name)
        print('Found existing cluster, use it.')
    except ComputeTargetException:
        compute_config = AmlCompute.provisioning_configuration(vm_size='STANDARD_D2_V2',
                                                               max_nodes=4)
        cpu_cluster = ComputeTarget.create(ws, cpu_cluster_name, compute_config)
    
    cpu_cluster.wait_for_completion(show_output=True)
    ```
    
    You can also configure several advanced properties when you create Azure Machine Learning Compute. The properties allow you to create a persistent cluster of fixed size, or within an existing Azure Virtual Network in your subscription.  See the [AmlCompute class](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.amlcompute.amlcompute?view=azure-ml-py
    ) for details.
    
    Or you can create and attach a persistent Azure Machine Learning Compute resource [in the Azure portal](#portal-create).

1. **Configure**: Create a run configuration for the persistent compute target.

    ```python
    from azureml.core.runconfig import RunConfiguration
    from azureml.core.conda_dependencies import CondaDependencies
    from azureml.core.runconfig import DEFAULT_CPU_IMAGE
    
    # Create a new runconfig object
    run_amlcompute = RunConfiguration()
    
    # Use the cpu_cluster you created above. 
    run_amlcompute.target = cpu_cluster
    
    # Enable Docker
    run_amlcompute.environment.docker.enabled = True
    
    # Set Docker base image to the default CPU-based image
    run_amlcompute.environment.docker.base_image = DEFAULT_CPU_IMAGE
    
    # Use conda_dependencies.yml to create a conda environment in the Docker image for execution
    run_amlcompute.environment.python.user_managed_dependencies = False
    
    # Auto-prepare the Docker image when used for execution (if it is not already prepared)
    run_amlcompute.auto_prepare_environment = True
    
    # Specify CondaDependencies obj, add necessary packages
    run_amlcompute.environment.python.conda_dependencies = CondaDependencies.create(conda_packages=['scikit-learn'])
    ```

Now that you’ve attached the compute and configured your run, the next step is to [submit the training run](#submit).


### <a id="vm"></a>Remote virtual machines

Azure Machine Learning also supports bringing your own compute resource and attaching it to your workspace. One such resource type is an arbitrary remote VM, as long as it's accessible from Azure Machine Learning service. The resource can be an Azure VM, a remote server in your organization, or on-premises. Specifically, given the IP address and credentials (user name and password, or SSH key), you can use any accessible VM for remote runs.

You can use a system-built conda environment, an already existing Python environment, or a Docker container. To execute on a Docker container, you must have a Docker Engine running on the VM. This functionality is especially useful when you want a more flexible, cloud-based dev/experimentation environment than your local machine.

Use the Azure Data Science Virtual Machine (DSVM)  as the Azure VM of choice for this scenario. This VM is a pre-configured data science and AI development environment in Azure. The VM offers a curated choice of tools and frameworks for full-lifecycle machine learning development. For more information on how to use the DSVM with Azure Machine Learning, see [Configure a development environment](https://docs.microsoft.com/azure/machine-learning/service/how-to-configure-environment#dsvm).

1. **Create**: Create a DSVM before using it to train your model. To create this resource, see [Provision the Data Science Virtual Machine for Linux (Ubuntu)](https://docs.microsoft.com/en-us/azure/machine-learning/data-science-virtual-machine/dsvm-ubuntu-intro).

    > [!WARNING]
    > Azure Machine Learning only supports virtual machines that run Ubuntu. When you create a VM or choose an existing VM, you must select a VM that uses Ubuntu.

1. **Attach**: To attach an existing virtual machine as a compute target, you must provide the fully qualified domain name (FQDN), user name, and password for the virtual machine. In the example, replace \<fqdn> with the public FQDN of the VM, or the public IP address. Replace \<username> and \<password> with the SSH user name and password for the VM.

 ```python
 from azureml.core.compute import RemoteCompute, ComputeTarget

 # Create the compute config 
 compute_target_name = "attach-dsvm"
 attach_config = RemoteCompute.attach_configuration(address = "<fqdn>",
                                                    ssh_port=22,
                                                    username='<username>',
                                                    password="<password>")

 # If you use SSH instead of a password, use this code:
 #                                                  ssh_port=22,
 #                                                  username='<username>',
 #                                                  password=None,
 #                                                  private_key_file="<path-to-file>",
 #                                                  private_key_passphrase="<passphrase>")

 # Attach the compute
 compute = ComputeTarget.attach(ws, compute_target_name, attach_config)

 compute.wait_for_completion(show_output=True)
 ```

 Or you can attach the DSVM to your workspace [using the Azure portal](#portal-reuse).

1. **Configure**: Create a run configuration for the DSVM compute target. Docker and conda are used to create and configure the training environment on the DSVM.

    ```python
    from azureml.core.runconfig import RunConfiguration
    from azureml.core.conda_dependencies import CondaDependencies

    run_dsvm = RunConfiguration(framework = "python")

    # Set the compute target to the Linux DSVM
    run_dsvm.target = compute_target_name 

    # Use Docker in the remote VM
    run_dsvm.environment.docker.enabled = True

    # Use the CPU base image 
    # To use GPU in DSVM, you must also use the GPU base Docker image "azureml.core.runconfig.DEFAULT_GPU_IMAGE"
    run_dsvm.environment.docker.base_image = azureml.core.runconfig.DEFAULT_CPU_IMAGE
    print('Base Docker image is:', run_dsvm.environment.docker.base_image)

    # Ask the system to provision a new conda environment based on the conda_dependencies.yml file 
    run_dsvm.environment.python.user_managed_dependencies = False

    # Prepare the Docker and conda environment automatically when they're used for the first time 
    run_dsvm.prepare_environment = True

    # Specify the CondaDependencies object
    run_dsvm.environment.python.conda_dependencies = CondaDependencies.create(conda_packages=['scikit-learn'])
    ```

Now that you’ve attached the compute and configured your run, the next step is to [submit the training run](#submit).

### <a id="hdinsight"></a>Azure HDInsight 

Azure HDInsight is a popular platform for big-data analytics. The platform provides Apache Spark, which can be used to train your model.

1. **Create**:  Create the HDInsight cluster before you use it to train your model. To create a Spark on HDInsight cluster, see [Create a Spark Cluster in HDInsight](https://docs.microsoft.com/azure/hdinsight/spark/apache-spark-jupyter-spark-sql). 

    When you create the cluster, you must specify an SSH user name and password. Take note of these values, as you need them to use HDInsight as a compute target.
    
    After the cluster is created, it has the FQDN \<clustername>.azurehdinsight.net, where \<clustername> is the name that you provided for the cluster. You need the FQDN address (or the public IP address of the cluster) to use the cluster as a compute target.

1. **Attach**: To attach an HDInsight cluster as a compute target, you must provide the FQDN, user name, and password for the HDInsight cluster. The following example uses the SDK to attach a cluster to your workspace. In the example, replace \<fqdn> with the public FQDN of the cluster, or the public IP address. Replace \<username> and \<password> with the SSH user name and password for the cluster.

  To find the FQDN for your cluster, go to the Azure portal and select your HDInsight cluster. Under __Overview__, you can see the FQDN in the __URL__ entry. To get the FQDN, remove the https:\// prefix from the beginning of the entry. 
    
  ![Get the FQDN for an HDInsight cluster in the Azure portal](./media/how-to-set-up-training-targets/hdinsight-overview.png)

  ```python
  from azureml.core.compute import HDInsightCompute, ComputeTarget

  try:
    # Attach an HDInsight cluster as a compute target
    attach_config = HDInsightCompute.attach_configuration(address = "<fqdn-or-ipaddress>",
                                                            ssh_port = 22,
                                                            username = "<username>",
                                                            password = None, #if using ssh key
                                                            private_key_file = "<path-to-key-file>",
                                                            private_key_phrase = "<key-phrase>")
    compute = ComputeTarget.attach(ws, "myhdi", attach_config)
  except UserErrorException as e:
    print("Caught = {}".format(e.message))
    print("Compute config already attached.")
  ```

  Or you can attach the HDInsight cluster to your workspace [using the Azure portal](#portal-reuse).

1. **Configure**: Create a run configuration for the HDI compute target. 

    ```python
    from azureml.core.runconfig import RunConfiguration
    # Configure the HDInsight run 
    # Load the runconfig object from the myhdi.runconfig file generated in the previous attach operation
    run_hdi = RunConfiguration.load(project_object = project, run_name = 'myhdi')
    
    # Ask the system to prepare the conda environment automatically when it's used for the first time
    run_hdi.auto_prepare_environment = True
    ```

Now that you’ve attached the compute and configured your run, the next step is to [submit the training run](#submit).


## Set up compute in the Azure portal

You can access the compute targets that are associated with your workspace in the Azure portal.  You can use the portal to:

* View  compute targets attached to your workspace
* Create an Azure Machine Learning Compute target
* Reuse existing compute targets

After a target is created and attached to your workspace, you will use it in your run configuration with a `ComputeTarget` object: 

```python
myvm = ComputeTarget(workspace=ws, name='my-vm-name')
```

### View compute targets


To see the compute targets for your workspace, use the following steps:

1. Navigate to the [Azure portal](https://portal.azure.com) and open your workspace. 
1. Under __Applications__, select __Compute__.

    ![View compute tab](./media/how-to-set-up-training-targets/azure-machine-learning-service-workspace.png)

### <a id="portal-create"></a>Create a compute target

Follow the previous steps to view the list of compute targets. Then use these steps to create a compute target: 

1. Select the Plus sign (+) to add a compute target.

    ![Add a compute target](./media/how-to-set-up-training-targets/add-compute-target.png) 

1. Enter a name for the compute target. 

1. Select **Machine Learning Compute** as the type of compute to use for __Training__. 

    >[!NOTE]
    >Azure Machine Learning Compute is the only  managed-compute resource you can create in the Azure portal.  All other compute resources can be attached after they are created.

1. Fill out the form. Provide values for the required properties, especially **VM Family**, and the **maximum nodes** to use to spin up the compute.  

    ![Fill out form](./media/how-to-set-up-training-targets/add-compute-form.png) 

1. Select __Create__.


1. View the status of the create operation by selecting the compute target from the list:

    ![Select a compute target to view the create operation status](./media/how-to-set-up-training-targets/View_list.png)

1. You then see the details for the compute target: 

    ![View the computer target details](./media/how-to-set-up-training-targets/compute-target-details.png) 



### <a id="portal-reuse"></a>Reuse existing compute targets

Follow the steps described earlier to view the list of compute targets. Then use these steps to reuse a compute target: 

1. Select the Plus sign (+) to add a compute target. 
1. Enter a name for the compute target. 
1. Select the type of compute to attach for __Training__:

    > [!IMPORTANT]
    > Not all compute types can be attached from the Azure portal. 
    > The compute types that can currently be attached for training include:
    >
    > * A remote VM
    > * Azure Databricks (for use in machine learning pipelines)
    > * Azure Data Lake Analytics (for use in machine learning pipelines)
    > * Azure HDInsight

1. Fill out the form and provide values for the required properties.

    > [!NOTE]
    > Microsoft recommends that you use SSH keys, which are more secure than passwords. Passwords are vulnerable to brute force attacks. SSH keys rely on cryptographic signatures. For information on how to create SSH keys for use with Azure Virtual Machines, see the following documents:
    >
    > * [Create and use SSH keys on Linux or macOS](https://docs.microsoft.com/azure/virtual-machines/linux/mac-create-ssh-keys)
    > * [Create and use SSH keys on Windows](https://docs.microsoft.com/azure/virtual-machines/linux/ssh-from-windows)

1. Select __Attach__. 
1. View the status of the attach operation by selecting the compute target from the list.


## <a id="submit"></a>Submit training run

After you create a run configuration, you use it to run your experiment.  The code pattern to submit a training run is the same for all types of compute targets:

1. Create an experiment to run
1. Submit the run.
1. Wait for the run to complete.

### Create an experiment

First, create an experiment in your workspace.

```
from azureml.core import Experiment
experiment_name = 'my experiment'

exp = Experiment(workspace=ws, name=experiment_name)
```
<a name=submit></a>

### Submit the experiment

Submit the experiment with a `ScriptRunConfig` object.  This object includes the:

* **source_directory**: The source directory that contains your training script
* **script**: Identify the training script
* **run_config**: The run configuration, which in turn defines where the training will occur.

Or you can submit the experiment with an `Estimator` object as shown in [Train ML models with estimators](how-to-train-ml-models.md).


When you submit a training run, a snapshot of the directory that contains your training scripts is created and sent to the compute target. For more information, see [Snapshots](concept-azure-machine-learning-architecture.md#snapshot).

For example, to use [the local target](#local) configuration:

```python
from azureml.core import ScriptRunConfig
import os 

script_folder = os.getcwd()
src = ScriptRunConfig(source_directory = script_folder, script = 'train.py', run_config = run_local)
run = exp.submit(src)
run.wait_for_completion(show_output = True)
```

Switch the same experiment to run in a different compute target by using a different run configuration, such as the [amlcompute target](#amlcompute):

```python
from azureml.core import ScriptRunConfig

src = ScriptRunConfig(source_directory = script_folder, script = 'train.py', run_config = run_amlcompute)
run = exp.submit(src)
run.wait_for_completion(show_output = True)
```

## Notebook examples

See these notebooks for examples of training with various compute targets:
* [how-to-use-azureml/training](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/training)
* [tutorials/img-classification-part1-training.ipynb](https://github.com/Azure/MachineLearningNotebooks/blob/master/tutorials/img-classification-part1-training.ipynb)

[!INCLUDE [aml-clone-in-azure-notebook](../../../includes/aml-clone-for-examples.md)]

## Next steps

* [Tutorial: Train a model](tutorial-train-models-with-aml.md) uses a managed compute target to  train a model.
* Once you have a trained model, learn [how and where to deploy models](how-to-deploy-and-where.md).
